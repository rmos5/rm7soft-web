# rm7soft Website

Static website for `rm7soft.com`.

## Structure

- `index.html` - rm7soft home page
- `privacy.html` - rm7soft privacy policy
- `support.html` - rm7soft support page
- `terms.html` - rm7soft terms of use
- `routinerun/` - RoutineRun product pages
- `assets/` - shared images and CSS

## Deployment

The live site is hosted on Cloudflare. It uses plain HTML and CSS with no build step.

`wrangler.jsonc` configures static assets for the existing Cloudflare Worker deployment. Keep that deployment configuration when publishing updates.

Pushes to the GitHub `main` branch automatically trigger Cloudflare Workers Builds for `rm7soft-web-github-worker`. Verify the build succeeds and the updated content is live at `https://rm7soft.com/`.

## RoutineRun content and branding

Website content was checked against the sibling `RmSoftTimers2` project on October 16, 2026: background shortcut options, shared playback, notification controls, localization resources, and Android requirements. The local APK inspected on that date was `RoutineRun-20261016-231531-release.apk`, version `1.2481.2315.33` (versionCode `1792181733`), Android API 23+, ARM64 only. Its copy in `RoutineRun Releases` matches the build file by SHA-256. On October 18, the owner supplied a newer APK link: `https://drive.google.com/file/d/19geisBlnDY4MOKRV9ghPImaXnB2kcRVy/view`. The download button uses this new link; its APK version and contents have not been independently inspected.

- `assets/routinerun-icon.png` is the official Google Play icon from `RoutineRun.App.Maui/Resources/Store/GooglePlay/routine-run-play-icon-512.png`, refreshed with the open-ring/Play mark on October 23, 2026.
- `assets/routinerun-brand.png` is the official feature graphic from the same directory.
- RoutineRun's brand blue is `#000CCC`.
- RoutineRun is in development. Android preview requests go to `routinerun@rm7soft.com`; access is reviewed and granted individually.
- Background shortcuts support routines and plans, with optional automatic start; routine shortcuts also offer Repeat. Pinning requires Android 8+, launcher support, and enabled notifications for background mode. The October 23 source has separate **Pin app shortcut** and **Pin background shortcut** commands; background mode is no longer a checkbox in the pin dialog.
- Expanded notifications offer Previous, Play/Pause, Next, Repeat, and Close. Navigation and Close are disabled while running; pause first. Automatic start is off by default. Do not promise survival after force-stop or device-specific task removal without device verification.
- The old `routinerun-hero.png` illustration is not used: it is not a current app screenshot.

When updating product details, verify them against the app source and replace imagery with official assets or actual screenshots.

## Shortcut icon guide and framework section

The October 23 icon guide uses the exact Android vector paths from `RmSoftTimers2/RoutineRun.App.Maui/Platforms/Android/Resources/drawable/routine_run_brand*.xml`, converted to SVG in `assets/shortcuts/`. The app and feature PNGs are copied directly from the current Google Play assets. No new logo was invented.

- `open` uses `routine_run_brand_open`; `start` uses `routine_run_brand`.
- Repeat variants use `routine_run_brand_open_repeat` or `routine_run_brand_start_repeat`, plus the separate `routine_run_brand_repeat` layer.
- The guide shows standard routine blue `#1976D2`, plan blue `#000CCC`, and white. Background shortcuts invert foreground/background, as in `AndroidHomeScreenShortcutService.cs`. The rm7soft theme uses a different routine blue.
- The SVG viewBox crops the adaptive canvas for a rounded launcher preview. Device masks and Android app badges vary. Existing pinned icons need recreating to pick up new artwork.
- Font Awesome attribution and the source license are included alongside the SVGs.

The home-page framework section is based on `rm7soft.Maui.Framework/docs/framework-guide.md` and that repository's README, read October 23. It describes the .NET 10 App, MAUI, and optional SQLite packages, Android/Windows targets, and their use in RoutineRun. Packages are privately hosted; the app template is still being revised. Keep public availability and platform claims tied to those sources.

## Private preview access

- The website provides an email request and an "Open download" button for previously approved users. The button opens the restricted ARM64 APK in Google Drive; publishing its URL does not grant access. Separate x64 builds are linked by email when available. Requests include name, Android device or emulator/version, and a Google account email. Organization and evaluation purpose are explicitly optional; architecture (ARM64 or x64) is requested only if known. A Google account is required for the private Drive download, and visitors are told this before requesting access.
- Review each request before granting access. Reply with the private build link, requirements, and installation/update instructions for that build.
- The preview is Android-only, requiring Android 6.0+ (API 23). The currently shared APK is ARM64 (`arm64-v8a`). Android x64 (`x86_64`) users can request a compatible build; confirm availability after building and testing it. The website must not imply the existing ARM64 APK also supports x64.

Keep APKs outside this repository. In Google Drive, set both the release folder and the APK's General access to **Restricted**, and grant Viewer access only to approved recipients. Check inherited folder access as well. Removing a website link does not revoke an existing public Drive link or copies already downloaded. Verify that a signed-out visitor cannot access the file.

For each shared preview, verify the version, signature, architecture, size, and checksum, and test its installation/update behavior. Do not promise that a preview can update an existing installation without first checking compatibility. Keep signing keys and their backups separate from shared builds.

## Website visual design

The October 17 design uses the owner's supplied rm7soft banner as a visual reference: dark navy, electric blue, fine wave lines, and the headline "Thoughtful software. Simple experiences." The headline and wordmark are live text for responsive sizing. `assets/rm7soft-code-mark.svg` and `assets/brand-waves.svg` are lightweight vector artwork made for this layout. RoutineRun retains its official timer icon and feature graphic. Shared styles cover product, support, and legal pages; private-preview links and access requirements are preserved. The supplied profile silhouette is not used.
