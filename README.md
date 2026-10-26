# NobiTerminal v15 — full source snapshot

This repo is the **complete patched app source tree for NobiTerminal v15**
(the version shipped as the v15 APK on 2026-10-26). It exists so the app's
source can be browsed, understood, and edited by hand in one place.

> This snapshot is for viewing and editing. It is **not** the build route.
> The real build route is the generator repo
> (`tale92266-hash/nobiterminal`): it clones upstream, applies the Nobi
> patches, renames the package, builds the bootstrap and the app in GitHub
> Actions, and produces the APK. Edits made here do **not** flow into
> generator builds automatically — to ship an edit, it must become a patch
> in the generator repo.

## What this snapshot was made from

- **Base:** upstream `termux/termux-app`
  `@8629e632fcb95da272221be327db653fb24befe9` (2026-09-26).
- **Patches:** all 27 Nobi patches from generator branch `v15-toolbar-fix`
  `@63afe490fed1ac88980bd78b3cbed61005a93fee`
  (built on v14-theme-fix `@721c3f2f`; the first 26 app patches are identical
  to v14 — v15 adds exactly one new app patch, w17, see below), applied
  with GNU `patch -p1` in plain lexical order (the same order the
  generator uses). All 27 applied cleanly: 0 `.rej`, 0 `.orig` files.
- **Shipped v15 APK for cross-reference:** 35,502,168 bytes,
  SHA-256 `919df58ae8f9e23a446a3d27e274394b10ba4b60a77adb4cef1f9c73467ebf0e`,
  generator run `40309356622` (cache HIT build on the already-seeded
  bootstrap — no new seed was published),
  signer
  SHA-256 fingerprint
  `B6:DA:01:48:0E:EF:D5:FB:F2:CD:37:71:B8:D1:02:1E:C7:91:30:4B:DD:6C:4B:F4:1D:3F:AA:BA:D4:8E:E5:E1`.

### The 27 patches, in apply order

1. `nobiterminal-app-name.patch`
2. `nobiterminal-drawer-default.patch`
3. `nobiterminal-neon-default.patch`
4. `nobiterminal-settings-suite.patch`
5. `nobiterminal-tabs-bar.patch`
6. `nobiterminal-v5-core.patch`
7. `nobiterminal-v5-drawer.patch`
8. `nobiterminal-v5-icon-splash.patch`
9. `nobiterminal-v5-nobicode-icons-a.patch`
10. `nobiterminal-v5-nobicode-icons-b.patch`
11. `nobiterminal-v5-nobicode-res.patch`
12. `nobiterminal-v5-nobicode-src.patch`
13. `nobiterminal-v5-tabs-rename.patch`
14. `nobiterminal-v6-batch.patch`
15. `nobiterminal-v7-batch.patch`
16. `nobiterminal-v7-fix.patch`
17. `nobiterminal-v7-fix2.patch`
18. `nobiterminal-v8-extrakeys.patch`
19. `nobiterminal-v9-drawer-notify.patch`
20. `nobiterminal-w10-editor-permissions.patch`
21. `nobiterminal-w11-followup.patch`
22. `nobiterminal-w12-fix-duplicate-permissions.patch`
23. `nobiterminal-w13-fix-vectors-kotlin.patch`
24. `nobiterminal-w14-autorepo.patch`
25. `nobiterminal-w15-keys-settings-theme.patch`
26. `nobiterminal-w16-settings-follow-terminal.patch`
27. `nobiterminal-w17-settings-toolbar-identity.patch`

### What is new in v15 (one new app patch: w17)

w17 (`nobiterminal-w17-settings-toolbar-identity.patch`, 3 diffs) fixes
the "app theme not matching with my app" complaint Sam raised with
screenshots (16:04 IST, 2026-10-26): the main Settings screen's toolbar
and status bar were bright red in BOTH modes, clashing with the Nobi
navy+cyan identity, while the rest of the app is navy.

- **Settings-only identity theme:** a new style
  `Theme.NobiSettings.DayNight` in `app/.../values/themes.xml`
  (parent `Theme.TermuxApp.DayNight.NoActionBar`) sets `colorPrimary`,
  `colorPrimaryVariant`, `colorPrimaryDark` and `android:statusBarColor`
  to `@color/nobi_navy`, with `@color/white` for `colorOnPrimary`. It is
  applied to **SettingsActivity only** (app manifest) — every other
  activity keeps the shared theme untouched.
- **White title + back arrow in both modes:** SettingsActivity now
  forces the toolbar title text white and tints the navigation icon
  white, because the DayNight action-bar overlay would otherwise give
  dark text/icons in day mode, unreadable on navy.
- **Same navy in both modes on purpose:** the toolbar is brand chrome,
  not body — so it stays navy day and night, while the w16
  luminance-follow still controls the Settings *body* (light/dark from
  the terminal background). The two mechanisms do not interfere: w17
  owns the chrome, w16 owns the body.
- Everything else is v14 unchanged: w16 luminance-follow, w15 keys/theme
  fixes, w14 autorepo, v11 follow-up features, and the v13 dead-PNG
  cleanup hook (see below).

> **On-device note:** the toolbar identity change itself (navy chrome +
> white title/arrow) is visible in the shipped v15 APK's Settings
> screen; like every version, Sam's on-device verdict is the final
> word.

### The v13 cleanup, retained in v15

The generator build-time cleanup step (`remove_dead_nobi_assets()`,
called after `patch_apps`, before the Gradle build) deletes 11 dead
files from the build tree before packaging:

- `app/src/main/res/mipmap-{hdpi,mdpi,xhdpi,xxhdpi,xxxhdpi}/ic_launcher.png`
  and `ic_launcher_round.png` (10 legacy upstream launcher PNGs, the old
  `>_` icons), and
- `app/src/main/res/drawable/banner.png` (1 legacy upstream banner PNG).

These files had no reference anywhere (manifest icon is the Nobi vector
`nobi_app_icon`), but the patch channel cannot delete binary files, so
they rode along in every APK until v12. The shipped v15 APK was verified (by this snapshot agent, `unzip -l`)
to contain **0** of these 11 PNGs.

> **Important nuance — tree vs APK for these 11 files:** the 11 PNGs are
> **still present in this tree** (they are upstream files in the patched
> source). They are absent only from the **shipped v15 APK**, because
> the cleanup is a generator build-time step, not a tree change. Do NOT
> claim this tree == the v15 APK for these files.

## An honesty note about scope (read this before calling this "complete")

This snapshot follows the same pattern as v11–v14: it contains the
**plugin patches** (the 27 listed above). The generator CI, after the
plugin patches, also applies 7 **base** `f-droid-patches/app-patches`
(`patch_apps`: `auto-hide-extra-keys`, `increase-gradle-memory`,
`local-bootstraps`, `local-maven`, `local-termux-am-library`,
`local-termux-gui`, `multi-window`) to the same tree. Those 7 base
patches are **not** included in this tree — for example, the
auto-hide-extra-keys behavior from the base patch is in the shipped APK
but its code change is not in this tree (verified: the v13 tree contains
no `auto_hide` reference). So this tree is the complete *Nobi-patched*
tree, not a byte-exact copy of the full CI tree. If a future snapshot
adopts full CI fidelity, these 7 patches (plugin patches first, base
patches second, per the generator's order) need to be applied on top.

## Package name: this tree is `com.termux`, not `com.nobiterminal`

That is intentional and matches what the generator patches produce. The
`com.nobiterminal` rename happens **at generator build time, after
patching**, via `TERMUX_APP__PACKAGE_NAME=com.nobiterminal`
(`replace_termux_name` + `migrate_termux_folder_tree` in the generator
scripts): it rewrites `com.termux` → `com.nobiterminal` in text files and
moves the `com/termux/...` Java folders. So:

- If you edit this tree, edit the `com.termux` paths. Do not rename
  things to `com.nobiterminal` here by hand.
- Building this tree directly produces a `com.termux` app. To get a
  `com.nobiterminal` build, build through the generator.

## Signing: local debug builds use the same certificate as the shipped APK

`app/testkey_untrusted.jks` (the upstream debug keystore) is included in
this tree. Its certificate was verified with `keytool`: Owner
`CN=APK Signer, OU=Earth, O=Earth`, SHA-256 fingerprint
`B6:DA:01:48:0E:EF:D5:FB:F2:CD:37:71:B8:D1:02:1E:C7:91:30:4B:DD:6C:4B:F4:1D:3F:AA:BA:D4:8E:E5:E1`
— identical to the shipped v15 APK's signer. A debug build from this tree
therefore signs with the same certificate and installs over the shipped
v15 build. (This is the upstream-published debug keystore, not a private
release key.)

## What is NOT in this snapshot

- **Bootstrap files.** There is no `app/src/main/assets/` here. In CI the
  bootstrap (`bootstrap-aarch64.tar.xz` + `xz` libs) is built from
  `termux/termux-packages`
  `@b8bf79d027fd9c046804fbce17283dfc1547d476` (2026-10-26, the HEAD at
  v15 build time — the v15 build was a cache HIT on the seed v14 had
  published on this HEAD, so no new seed was published)
  and placed into the app's assets by the generator build. A local build
  from this tree needs bootstrap files supplied separately.
- **Generator CI files.** The generator's workflows, cache system and
  scripts live in the generator repo, not here. The `.github/` folder in
  this tree is upstream Termux's own CI, not NobiTerminal's build.
- **Binary patch hunks.** GNU `patch` cannot apply git binary hunks, so
  any binary file a patch tried to add is absent here — exactly as in
  the generator CI tree (CI uses the same GNU `patch -p1`). The legacy
  `mipmap` PNGs present here are upstream's, not Nobi's — and, per
  the v13/v14 nuance above, they are present here even though the shipped
  v15 APK no longer contains them.

## Building on a phone (ACS): read this first

This snapshot's `main` branch is **not directly buildable in ACS
(AndroidIDE) as-is**. Three modules wire `externalNativeBuild` ndkBuild
in this tree:

| Module | Native lib |
|---|---|
| `app` | `libtermux-bootstrap` (`app/src/main/cpp/Android.mk`) |
| `terminal-emulator` | `libtermux` (`terminal-emulator/src/main/jni/Android.mk`) |
| `termux-shared` | `liblocal-socket` (`termux-shared/src/main/cpp/Android.mk`) |

Phone IDEs cannot run Google NDK's inner `ndk-build` (on v4 this failed
with exit 126 on `termux-shared`). v4 solved it with a separate
`acs-prebuilt` branch: prebuilt `.so` files taken from the shipped APK
plus the `externalNativeBuild` blocks removed, with `main` untouched.
**No v15 `acs-prebuilt` branch was created in this snapshot** — that is a
separate next step if on-phone ACS builds are wanted. Two facts for that
step: the shipped v15 APK contains only `libtermux.so` and
`liblocal-socket.so` (× 4 ABIs each) — it contains **no**
`libtermux-bootstrap.so`, even though the `app` module wires ndkBuild in
this tree, so the `app` module's ndk step needs investigation before any
`acs-prebuilt` treatment. The Gradle wrapper jar
(`gradle/wrapper/gradle-wrapper.jar`) **is** included in this tree, so an
ACS import does not hang on a missing wrapper jar.

## Modules in this tree

`app`, `termux-shared`, `terminal-emulator`, `terminal-view`, plus
`nobicode` (the inbuilt editor, added by the v5 patches and extended by
later batches, including the v11 follow-up features, auto repo setup in
`app/src/main/java/com/termux/app/nobi/NobiRepoSetup.java`, the v12
keys/theme fixes, and the v14 luminance-follow Settings fix; v15 adds w17
on top of v14).

## License

Upstream Termux code is GPL-3.0 (see `LICENSE.md`). The Nobi patches on
top are Sam's own changes.
