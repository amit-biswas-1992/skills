---
name: npm-publish
description: Publish (or re-publish) an npm package from this machine. Handles the common failure modes — wrong working directory, EOTP when the user has only a security key for 2FA, first-time auth, version bumps, and scoped packages.
trigger: /npm-publish
---

# /npm-publish

Publish a package to the npm registry. Most real-world friction isn't the publish itself — it's:

1. Running from the wrong directory (the monorepo root instead of the package folder).
2. Hitting `EOTP` when the user's npm 2FA is a WebAuthn security key and they have no TOTP code.
3. Forgetting `--access public` on a first-time scoped package.

This skill walks through all of those.

## Preflight

Before publishing, verify:

1. **You are in the package directory**, not the monorepo root.
   - The `package.json` you're about to publish must NOT have `"private": true`.
   - Root-level `package.json` files usually have `"private": true` — good, that protects against accidental monorepo publishes.
2. **The package builds clean.** Run the package's `build` script and `typecheck` (if it has one).
3. **The version is new.** npm refuses to republish the same version. Bump `package.json#version` first if republishing.
4. **`files` or `.npmignore`** is correct so you don't ship source or node_modules.
5. **`bin`**, if present, points to a built entry that has `#!/usr/bin/env node` at the top.

Sanity-check what will ship:

```bash
npm publish --dry-run
```

The tarball listing should be small and contain only `dist/`, `README.md`, `package.json`, and maybe `LICENSE`.

## Auth: the three paths

Pick ONE based on how the user's npm account is set up.

### Path A — Already logged in, TOTP 2FA (authenticator app)

This is the easy path. Just run:

```bash
npm publish --access public --otp=XXXXXX
```

Replace `XXXXXX` with the current 6-digit code from Google Authenticator / 1Password / Authy. OTPs rotate every 30s — run it fast.

### Path B — Already logged in, security-key-only 2FA (WebAuthn / passkey)

**This is the trap.** If `npm whoami` returns a username but `npm publish` fails with:

```
npm error code EOTP
npm error This operation requires a one-time password from your authenticator.
```

…and the user has no TOTP (only a passkey / Touch ID / YubiKey on `npmjs.com/settings/<user>/tfa`), there is no 6-digit code to provide. The npm CLI cannot perform a WebAuthn challenge — it only understands TOTP.

**Fix: create a granular access token with "Bypass 2FA" enabled.**

1. Open `https://www.npmjs.com/settings/<username>/tokens/granular-access-tokens/new`.
2. Fill:
   - Token name: `<package>-publish-<yyyymmdd>` (must be unique).
   - **Check "Bypass two-factor authentication (2FA) for this token"** — critical.
   - Packages and scopes → Permissions: **Read and write**.
   - Packages → **All packages** (or pick the specific package if it already exists on npm).
   - Expiration: 7 days is enough for a one-off publish.
3. Click Generate. **Copy the token immediately** — npm shows it once.
4. Publish with an isolated npmrc so the token is the only credential in play:

```bash
cd path/to/package-dir

TMPNPMRC=$(mktemp)
cat > "$TMPNPMRC" <<EOF
//registry.npmjs.org/:_authToken=npm_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
registry=https://registry.npmjs.org/
EOF

HOME=/tmp/npm-iso-$$ NPM_CONFIG_USERCONFIG="$TMPNPMRC" \
  npm publish --access public

rm -f "$TMPNPMRC"
```

Why the isolated `HOME`: npm's normal config precedence means a logged-in user session in `~/.npmrc` can still trigger EOTP even when you pass a token-bearing `.npmrc`. Using a throwaway `HOME` guarantees only your token-bearing config is read.

**After publish: revoke the token.** If the token appeared in the terminal (it always does, at least on the npmjs.com confirmation page), treat it as burnt. Delete it at `https://www.npmjs.com/settings/<username>/tokens`.

### Path C — Fresh machine, never logged in

```bash
npm adduser
```

This opens a browser for login. If the user has only a security key, completing the browser flow lets the CLI session authenticate — but publishing from that session still hits EOTP without a TOTP. Fall back to Path B.

## Version bumps

```bash
npm version patch   # 0.1.0 → 0.1.1
npm version minor   # 0.1.0 → 0.2.0
npm version major   # 0.1.0 → 1.0.0
```

This edits `package.json` AND creates a git tag + commit. Push the tag after publishing:

```bash
git push --follow-tags
```

## First-time public publish of a scoped package

Scoped packages default to private. Must pass `--access public`:

```bash
npm publish --access public
```

Unscoped packages ignore this flag, but it's harmless — include it by default.

## Verification

After a successful publish, confirm the registry has it:

```bash
curl -s "https://registry.npmjs.org/<package-name>/latest" | head -c 300
```

You should see the JSON for the new version. The web page at `https://www.npmjs.com/package/<package-name>` can take a minute to refresh CDN caches — don't panic if it looks stale.

Then test the install:

```bash
npx <package-name> --version   # or whatever the CLI entrypoint is
```

## Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| `403 EPRIVATE` | Published from monorepo root with `"private": true` | `cd` into the actual package directory |
| `403 EOTP` | 2FA required; you passed the wrong/expired OTP or none | Use `--otp=XXXXXX` with a fresh code, OR switch to Path B (bypass-2FA token) |
| `403 You cannot publish over the previously published versions` | Version in package.json already exists on npm | `npm version patch` then retry |
| `402 Payment Required` | First publish of a scoped package without `--access public` | Add `--access public` |
| `E404 package name not found` | Typo, or never published before (fine for first publish) | Ignore on first publish |
| `Duplicate token names are not allowed` | Reusing a token name from a previous attempt | Pick a new unique name |

## Cleanup checklist

After a successful publish:

- [ ] Revoke any token that appeared in chat transcripts or shared terminals.
- [ ] Commit the version bump + any build output policy changes.
- [ ] Push the git tag: `git push --follow-tags`.
- [ ] If this was a monorepo CLI subpackage, remind the user to exclude it from the root `tsconfig.json` `include` if TS errors leak upstream.
