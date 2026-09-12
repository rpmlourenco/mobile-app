# Music Assistant fork nightlies

Fork: https://github.com/rpmlourenco/mobile-app

`origin` is the fork; `upstream` is https://github.com/music-assistant/mobile-app.
Local `main` tracks `origin/main`. The Android Nightly workflow runs on pushes to
`main`, manual dispatch, and daily at 03:00 UTC. It builds the checked-out fork
commit; it does not automatically merge upstream. Schedules run from the default
branch and GitHub may delay them or disable them after 60 days of inactivity in
a public repository.

## Signing setup (once)

Create your own long-lived signing key with JDK 21 `keytool`. Store it outside
the repository and back it up securely, together with its passwords. Losing it
means existing installations cannot receive updates signed with a new key.
Never use upstream's Play upload key or a runner-generated debug key.

```powershell
keytool -genkeypair -v -keystore "$env:USERPROFILE\music-assistant-nightly.jks" -storetype JKS -alias nightly -keyalg RSA -keysize 4096 -validity 10000
[Convert]::ToBase64String([IO.File]::ReadAllBytes("$env:USERPROFILE\music-assistant-nightly.jks")) | gh secret set NIGHTLY_KEYSTORE_BASE64 --repo rpmlourenco/mobile-app
gh secret set NIGHTLY_KEYSTORE_PASSWORD --repo rpmlourenco/mobile-app
gh secret set NIGHTLY_KEY_ALIAS --repo rpmlourenco/mobile-app
gh secret set NIGHTLY_KEY_PASSWORD --repo rpmlourenco/mobile-app
```

Enter the store password, alias `nightly`, and key password in the respective
interactive `gh secret set` prompts. Alternatively configure the four secrets
under Settings → Secrets and variables → Actions. No Play/Google Cloud secrets
are needed. The key is decoded in the runner temporary directory, passed to
Gradle through environment variables, and removed even on failure. Secrets
are never interpolated into shell source or written to Java properties files.

Enable Actions in the fork if GitHub displays its fork workflow opt-in banner.
Repository policy must permit the workflow's `contents: write` token for releases.
Then run:

```powershell
gh workflow run android-nightly.yml --repo rpmlourenco/mobile-app
gh run list --repo rpmlourenco/mobile-app --workflow android-nightly.yml
```

Missing secrets produce an explicit failure; there is no debug signing fallback.
Only trusted fork `main` pushes/manual/scheduled runs use the signing secrets.
Existing upstream PR checks remain available without signing secrets.

## Installation and releases

APK: `music-assistant-nightly.apk` in a commit-specific GitHub prerelease, plus
`mapping.txt` and `SHA256SUMS`; Actions also retains assets for 30 days.
Application ID: `io.music_assistant.client.rpmlourenco.nightly`.
Launcher name: **Music Assistant Nightly**. This has separate app data and can
coexist with the official app. The Kotlin namespace stays unchanged. The nightly
shortcut resource points voice intents at the fork package. OAuth/deep links
retain the upstream `musicassistant` scheme and may display an app chooser when
both apps are installed; official-domain verified links are not verified for
the fork signing certificate. Android Auto/Assistant integration may require
additional sideload/developer settings and is not established by building an APK.

Nightly inherits the optimized release build and uses an unsplit APK (all native
ABIs present in its dependencies), rather than upstream's arm64 release split.
The version code is UTC minutes since 2026-01-01, independent of upstream's
version code. Keep that epoch/formula unchanged to preserve update ordering.
The workflow serializes publishers. A non-draft release containing an APK is
the success marker for each full commit SHA. All triggers skip published commits;
failed builds and draft/partial releases can be retried. No moving nightly tag
or automatic release retention/deletion is used.

The upstream Android Play release job is restricted to the official repository.
Other inherited iOS and translation workflows are retained; configure their
separate credentials only if you intend to use them.

## Update upstream and build locally

```powershell
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

Resolve any upstream conflicts with the fork build type/workflow before pushing.
Upstream is not automatically merged into your fork.

Install JDK 21 and Android SDK, set `JAVA_HOME` and `ANDROID_HOME` (or
`local.properties` for the SDK), then run:

```powershell
$env:JAVA_HOME = 'C:\Program Files\Android\Android Studio\jbr'
$env:ANDROID_HOME = "$env:LOCALAPPDATA\Android\Sdk"
# Install SDK platform android-37 and build-tools 36.0.0 if missing.
$env:NIGHTLY_KEYSTORE_PATH = "$env:USERPROFILE\music-assistant-nightly.jks"
# Supply NIGHTLY_KEYSTORE_PASSWORD, NIGHTLY_KEY_ALIAS, NIGHTLY_KEY_PASSWORD
# in this shell securely; do not commit them.
$nightlyCode = [int][Math]::Floor(([DateTimeOffset]::UtcNow.ToUnixTimeSeconds() - 1767225600) / 60)
.\gradlew.bat :androidApp:assembleNightly :androidApp:testDebug testAndroidHostTest :androidApp:lintNightly "-PnightlyVersionCode=$nightlyCode"
```

The local nightly variant defaults to version code 1 if no property is supplied;
use the timestamp formula for APKs intended to update CI installations.

## Initial validation

Actionlint passed for the nightly and modified Play release workflows. YAML and
nightly XML parse checks passed. Gradle 9.7.1 with Android Studio's JDK 21
successfully configured the project and listed the new nightly tasks. A full
APK build/test run has not been completed: the local SDK only has platforms
33/34 and build tools through 34.0.0, and signing secrets still need provisioning.
GitHub Projects access requires an additional `project` token scope to attach
the setup issue to the user's Music Assistant Mobile App project.
