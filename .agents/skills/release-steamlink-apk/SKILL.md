---
name: release-steamlink-apk
description: Publish a patched Steam Link APK release in this repository — verify the APK, update the README table, create the tagged GitHub release with the APK asset.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows]
metadata:
  hermes:
    tags: [release, github-releases, steamlink, galaxyxr, apk]
---

# release-steamlink-apk

Use when the owner drops a new `Steam_Link-v<app>-patches-v<patchset>.apk` in this folder and asks for a release.

## Before touching anything

Ask the owner for anything not already stated. Do not guess:

1. **Driver version** these APKs are compatible with (e.g. `v1.13.0`). Required in the release notes.
2. **Patch set version** — confirm from the file name and against `xr-porting-hub` `CHANGELOG.md`.
3. **Scope** — which Steam Link versions to publish (2.0.20, 2.0.23, both).

Then verify each APK matches its name.

## 1. Verify the APK

```bash
cd "D:/Angelo/Desktop/SteamLink-GalaxyXR-Windows-Toolkit-FULL/steamlink-androidxr-patched"
AAPT="F:/Runtimes/AndroidSdk/build-tools/36.0.0/aapt2.exe"
AS="F:/Runtimes/AndroidSdk/build-tools/36.0.0/apksigner.bat"
APK="Steam_Link-v2.0.23-patches-v1.32.1.apk"

"$AAPT" dump badging "D:/Angelo/Desktop/SteamLink-GalaxyXR-Windows-Toolkit-FULL/steamlink-androidxr-patched/$APK" | head -1
"$AS" verify --print-certs "D:/Angelo/Desktop/SteamLink-GalaxyXR-Windows-Toolkit-FULL/steamlink-androidxr-patched/$APK" | head -3
sha256sum "$APK" | tee "$APK.sha256"
```

Checks, all must pass:

| Check | Expected |
|---|---|
| `package` | `com.valvesoftware.steamlinkvr` |
| `versionName` / `versionCode` | `2.0.20`/`5001712` or `2.0.23`/`5002363`, matching the file name |
| Signature | verifies, signer `CN=Morphe` |
| Size | ~40–45 MB; near-zero or >200 MB means a broken build, stop |

The `.sha256` sidecar is a second release asset.

## 2. Update the README table

Newest Steam Link version first. One row per published pair:

```markdown
| **2.0.23** (latest) | 5002363 | 1.32.1 | v1.13.0 | [Download](https://github.com/AngelDark92/steamlink-androidxr-patched/releases/tag/v2.0.23-1.32.1) |
```

When a new patch set supersedes an old one, update the existing rows instead of adding duplicate versions, but never delete a release that still works — only the README row changes. Keep the badges at the top in sync (`Steam Link-<newest>`, `patch set-<patch set>`, `driver-<driver>`).

## 3. Commit the docs

```bash
git add README.md assets AGENTS.md .agents
git status --short          # no .apk must appear
git commit -m "docs: Steam Link <app versions> patch set <patch set> (driver <driver>)"
git push origin main
git rev-parse HEAD          # target commitish for the releases
```

`*.apk` is gitignored. If an APK shows in `git status`, stop: the ignore rule is broken.

## 4. Create the releases

Token comes from the stored git credential. Never echo it, never write it to a tracked file.

```bash
TOK=$(printf "protocol=https\nhost=github.com\n\n" \
  | "C:/Users/Angelo/AppData/Local/hermes/tools/git-2.53.0+3-win32-x64/mingw64/bin/git-credential-manager.exe" get \
  | sed -n 's/^password=//p')
test -n "$TOK" || echo "NO TOKEN"
```

Build the payload with Python so quoting and newlines cannot break the JSON:

```bash
python - <<'PY'
import json
notes = """Patched Steam Link **2.0.23** (build 5002363) for Samsung Galaxy XR / Android XR.

**Requires Galaxy XR Companion driver v1.13.0** — https://github.com/AngelDark92/GalaxyXR-Driver/releases/tag/v1.13.0

## Patch set 1.32.1
- spacewarp: warp the real video projections, half-rate mode

Full patch list: https://github.com/AngelDark92/xr-porting-hub

## Install
1. Uninstall the existing Steam Link app (different signing key).
2. `adb install -r Steam_Link-v2.0.23-patches-v1.32.1.apk`
3. Launch, allow permissions, set Battery usage to Unrestricted.
4. Start SteamVR on the PC and connect.

## Verify
SHA-256: `<hash>`
Signer: CN=Morphe (test key)
"""
json.dump({"tag_name":"v2.0.23-1.32.1",
           "name":"Steam Link 2.0.23 — patch set 1.32.1",
           "target_commitish":"<sha from step 3>",
           "body":notes,"draft":False,"prerelease":False,
           "make_latest":"true"}, open("release.json","w"), indent=1)
PY

curl -s -H "Authorization: Bearer $TOK" -H "Accept: application/vnd.github+json" \
  -X POST "https://api.github.com/repos/AngelDark92/steamlink-androidxr-patched/releases" \
  --data-binary @release.json | tee rel.json
python -c "import json;d=json.load(open('rel.json'));print(d.get('id'),d.get('tag_name'),d.get('message',''))"
```

Release rules:

- Tag `v<app-version>-<patch-set>`, e.g. `v2.0.20-1.32.1`, `v2.0.23-1.32.1`. One release per pair.
- Title `Steam Link <app-version> — patch set <patch-set>`.
- `make_latest` is `"true"` only for the newest Steam Link version; the older one gets `"false"` so the repo's Latest badge stays on the current build.
- Notes always carry the driver line, the patch-set highlights from `xr-porting-hub` `CHANGELOG.md`, install steps, and SHA-256.

## 5. Upload assets

```bash
RID=$(python -c "import json;print(json.load(open('rel.json'))['id'])")
REPO="AngelDark92/steamlink-androidxr-patched"
curl -s -X POST -H "Authorization: Bearer $TOK" \
  -H "Content-Type: application/vnd.android.package-archive" \
  --data-binary @"Steam_Link-v2.0.23-patches-v1.32.1.apk" \
  "https://uploads.github.com/repos/$REPO/releases/$RID/assets?name=Steam_Link-v2.0.23-patches-v1.32.1.apk"

curl -s -X POST -H "Authorization: Bearer $TOK" \
  -H "Content-Type: text/plain" \
  --data-binary @"Steam_Link-v2.0.23-patches-v1.32.1.apk.sha256" \
  "https://uploads.github.com/repos/$REPO/releases/$RID/assets?name=Steam_Link-v2.0.23-patches-v1.32.1.apk.sha256"
```

Upload host is `uploads.github.com`, not `api.github.com`. Asset limit 2 GB; these are ~42 MB.

## 6. Read it back

```bash
curl -s -H "Authorization: Bearer $TOK" \
  "https://api.github.com/repos/AngelDark92/steamlink-androidxr-patched/releases/tags/v2.0.23-1.32.1" \
  | python -c "
import sys,json
d=json.load(sys.stdin)
print(d['tag_name'], d['target_commitish'][:7], d['make_latest'] if 'make_latest' in d else '')
for a in d['assets']: print(a['name'], a['size'], a['state'], a['browser_download_url'])
"
```

Asset `state` must be `uploaded` and `size` must equal the local file size. Then run `git status --short`: only documentation files may be modified, APKs stay untracked. Delete `release.json`, `rel.json` and the token shell variable when done.

## Pitfalls

- **Empty repository rejects releases** with 422: push the documentation commit first; tags need a commit to point at.
- **Duplicate tag** returns 422. Fix: `DELETE /repos/<repo>/releases/<id>` then `DELETE /repos/<repo>/git/refs/tags/<tag>`, then retry.
- **Missing `make_latest`** on the older build flips the repo's Latest badge to a superseded pair.
- Do not `git add -f` an APK. If a release must be replaced, delete the release and tag, not git history.
- Version name alone is not compatibility: 2.0.20 and 2.0.23 differ by build code, and the patched payload is compiled per exact base. Refuse to publish an APK whose build code is not in the supported list.
