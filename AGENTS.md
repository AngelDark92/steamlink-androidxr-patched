# steamlink-androidxr-patched — agent guidance

Distribution repository for **pre-patched Steam Link (PCVR) APKs** that run on the Samsung Galaxy XR and other Android XR headsets. No source code, no build system, no CI: patch sources live in [xr-porting-hub](https://github.com/AngelDark92/xr-porting-hub), the PC-side driver lives in [GalaxyXR-Driver](https://github.com/AngelDark92/GalaxyXR-Driver). This repository only holds the finished APKs as GitHub release assets plus the README that explains them.

## Layout

| Path | Contents |
|---|---|
| `README.md` | Public face: what the builds are, download table, install steps, patched feature list. |
| `AGENTS.md` | This file: rules and the release contract. |
| `.opencode/skills/release-steamlink-apk/SKILL.md` | Exact release playbook, commands and pitfalls. Read it before any release. |
| `assets/` | README images only. Nothing executable. |
| `*.apk` | Local, **untracked** release candidates. Never committed. |

## Hard rules

1. **APKs never enter git.** `*.apk` is gitignored on purpose. Binaries ship as release assets; git carries documentation only. If a commit ever stages an APK, stop and unstage it.
2. **One release per (Steam Link version, patch set) pair.** Tag `v<app-version>-<patch-set>`, e.g. `v2.0.24-2.4.0`, one APK asset each. Two app versions patched with the same patch set means two releases.
3. **Every release states the compatible driver version.** The owner usually names it ("compatible with driver v1.13.0"). If the owner does not say, **ask before publishing**. Never guess, and never widen the claim to "or newer" without evidence.
4. **The file name is the contract, the APK is the truth.** Expected name `Steam_Link-v<app-version>-XR_Porting_Hub-v<patch-set>.apk`; older builds used `Steam_Link-v<app-version>-patches-v<patch-set>.apk`. Verify `versionName`, `versionCode` and signer inside the APK against the name before publishing.
5. **This repository holds no code.** Do not add scripts, workflows, or patch sources here. Fixes belong in `xr-porting-hub`.
6. **No rebuilds.** Publish the APK the owner drops in this folder. If it is missing, misnamed, or its build code is not a known `xr-porting-hub` base, stop and report instead of patching it yourself.
7. **Publish only what was asked.** No extra tags, no draft releases, no retroactive edits to published builds.

## Pinned facts

- Package: `com.valvesoftware.steamlinkvr`
- Supported bases (`versionName` / `versionCode`): `2.0.20` / `5001712`, `2.0.23` / `5002363` (no longer distributed), `2.0.24` / `5002395`
- Signer of patched builds: `CN=Morphe` test key, so an existing store install must be uninstalled before installing one of these
- Patch set version and feature list: `xr-porting-hub` `CHANGELOG.md` (bump it, do not invent it)
- Driver releases: https://github.com/AngelDark92/GalaxyXR-Driver/releases

## Release flow

1. Confirm with the owner: patch set version, driver version, and which Steam Link versions are in scope. Ask if any is missing.
2. Verify each APK (`aapt2 dump badging`, `apksigner verify --print-certs`, `sha256sum`).
3. Add or update the row in the README **Downloads** table.
4. Commit and push the documentation change.
5. Create one release per pair, tag `v<app>-<patch-set>`, target the pushed commit.
6. Upload the APK plus a `.sha256` sidecar; paste the verification data and install steps into the notes.
7. Read the release back through the API: asset names, sizes, state `uploaded`. Report, and leave the APK untracked.

Exact commands and failure handling: `.opencode/skills/release-steamlink-apk/SKILL.md`.
