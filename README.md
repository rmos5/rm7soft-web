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
- RoutineRun is in development. Android preview requests go to `routinerun@rm7soft.com`; access is reviewed and granted individually.
- The old `routinerun-hero.png` illustration is not used: it is not a current app screenshot.

When updating product details, verify them against the app source and replace imagery with official assets or actual screenshots.

## Private preview access

- The website provides an email request, not a public APK link. Requests include name, organization if relevant, evaluation purpose, Android device/version, and a Google account email. A Google account is required for the private Drive download, and visitors are told this before requesting access.
- Review each request before granting access. Reply with the private build link, requirements, and installation/update instructions for that build.
- Current Android preview requirements: Android 6.0+ (API 23), ARM64 (`arm64-v8a`) only. The preview is currently Android-only.

Keep APKs outside this repository. In Google Drive, set both the release folder and the APK's General access to **Restricted**, and grant Viewer access only to approved recipients. Check inherited folder access as well. Removing a website link does not revoke an existing public Drive link or copies already downloaded. Verify that a signed-out visitor cannot access the file.

For each shared preview, verify the version, signature, architecture, size, and checksum, and test its installation/update behavior. Do not promise that a preview can update an existing installation without first checking compatibility. Keep signing keys and their backups separate from shared builds.
