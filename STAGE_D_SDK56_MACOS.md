# Stage D — Expo SDK 56 (macOS) on react-native-macos@0.85

Goal: an Expo **SDK 56** app builds for **macOS** using our
`officialunofficial/react-native-macos@uno/desktop-0.85-wip` (RN 0.85.3),
the way upstream expo-desktop enables SDK 54 / RN 0.81.

## What we found (good news)

- expo-desktop already ships an SDK-56 catalog: `react-native-85` (expo ^56.0.9,
  `@expo/config-plugins ^56.0.8`, react-native 0.85.3). We added
  `react-native-macos: 0.85.3` to it (this commit).
- The `create-app` CLI is already **0.85-aware** (`prompt-for-version.ts` maps
  react-native@0.85 → Expo 56). Minimal CLI change needed for macOS.
- `expo-desktop-modules-core` / `expo-desktop-stubs` are **Windows-only**
  ("expo-modules-core for react-native-windows", peerDeps react-native-windows).
  **The macOS path does not need them bumped to 56.** macOS uses the standard
  Expo modules (experimental macOS support) + the macOS config-plugins.

So the macOS SDK-56 surface = catalog (done) + `expo-desktop-config-plugins` /
`expo-desktop-prebuild-config` resolving to the SDK-56 `@expo/*` (via the
`react-native-85` catalog) + consuming our react-native-macos@0.85.

## The one real complication: consuming our fork (chosen: no public publish)

Our fork's `react-native` package is named `react-native-macos` but is a **yarn
monorepo** whose deps are `workspace:*` (e.g. `@react-native-macos/virtualized-lists`).
A `file:`/git dependency to that inner package will NOT resolve those workspace
specs in a consuming app. The supported local path (no public npm publish) is the
fork repo's **verdaccio** local registry:

1. In `react-native-macos` (uno/desktop-0.85-wip): run the release pipeline to a
   local verdaccio (the repo has `.ado/verdaccio` + `nx release`) → publishes
   `react-native-macos@0.85.x` + its `@react-native(-macos)/*` satellites with
   workspace:* resolved to real versions.
2. Point npm/yarn at the verdaccio registry for the test app.

## Remaining steps to a validated SDK-56 macOS build

1. Verdaccio-publish the rn-macos@0.85 fork (+ satellites) locally.
2. Point `expo-desktop-config-plugins` / `expo-desktop-prebuild-config` package.json
   deps at `catalog:react-native-85` (currently `catalog:react-native-81`) so the
   macOS plugins build against SDK-56 `@expo/config-plugins`.
3. `create-app` an SDK-56 app (RN 0.85) against the verdaccio registry, or bump the
   existing `experiments/expo-desktop-poc` to SDK 56 + repoint react-native-macos.
4. `pod install` (RCT_NEW_ARCH_ENABLED=1) + `xcodebuild` the macOS target →
   the macOS app builds + launches on RN 0.85 / SDK 56.

This is the heavy tail (verdaccio + scaffold + first macOS app build, ~the
expo-desktop-poc effort) and is best run as a focused pass. The hard prerequisite —
a green `react-native-macos@0.85` — is done (officialunofficial/react-native-macos
`uno/desktop-0.85-wip`).
