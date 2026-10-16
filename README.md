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

Website content was checked against the sibling `RmSoftTimers2` project on October 14, 2026: the routine action and plan documentation, current run controls, localization resources, and Android notification implementation.

- `assets/routinerun-icon.png` is the official Google Play icon from `RoutineRun.App.Maui/Resources/Store/GooglePlay/routine-run-play-icon-512.png`.
- `assets/routinerun-brand.png` is the official feature graphic from the same directory.
- RoutineRun's brand blue is `#000CCC`.
- Android downloads use the approved Google Drive release link. Windows availability remains an email contact.
- The old `routinerun-hero.png` illustration is not used: it is not a current app screenshot.

When updating product details, verify them against the app source and replace imagery with official assets or actual screenshots.

## Android download

- Release: `1.2480.2113.48`, October 15, 2026.
- File: `RoutineRun-20261015-211347-release.apk`, 24,317,076 bytes (23.19 MiB).
- Requirements: Android 6.0+ (API 23), ARM64 (`arm64-v8a`) only.
- Download: https://drive.google.com/file/d/1atn0XRdBTP0DDvldHQXmGtJ2Yq4l6RCw/view
- SHA-256: `112a38b0e7cd47119d929ada496698d42be834066897508530b2d39e8731a950`.

The APK is kept in the dedicated Google Drive `RoutineRun Releases` folder, outside this repository. Share only approved public releases with viewer/download access. For each new release, verify the signature, version, architecture, size, and checksum; update the Drive links and release details together. Test the download without signing in before deploying. Google Drive can show a virus-scan confirmation for APK files.
