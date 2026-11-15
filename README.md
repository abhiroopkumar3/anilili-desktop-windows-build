# Anilili Desktop: Windows build

An unofficial fork that builds the Windows installer (.exe) of
[Anilili Desktop](https://github.com/sanilili/desktoprepo1) with GitHub Actions, so nothing has to
be installed on a PC to get it. It keeps itself in step with the original repo and publishes a
release for each new app version.

## Download the installer

Open the [Releases](../../releases) page and download `Anilili_<version>_x64-setup.exe` from the
latest release.

The installer is **not code-signed**, so Windows SmartScreen may warn you the first time. Choose
"More info", then "Run anyway", if you trust the source.

## How it works

Three workflows in `.github/workflows` run in a chain:

| Workflow | What it does | When it runs |
| --- | --- | --- |
| **Sync from upstream** (`sync-upstream.yml`) | Copies the latest code from the original repo into `main`. It never changes `.github/workflows` or this README. Then it reads the `version` in `src-tauri/tauri.conf.json` and starts a build only if that version has no release here yet. | Every Monday at 00:30 Austin time (America/Chicago, daylight saving handled), or manually |
| **Build Windows installer** (`build-windows.yml`) | Builds the NSIS installer on a Windows runner and uploads it as the artifact `Anilili-Windows-Installer` (kept 90 days). On success it starts the release workflow. | Started by the sync, or manually |
| **Publish release** (`release-from-run.yml`) | Downloads the installer from a build run, then creates the tag `v<version>` and a GitHub release with the .exe attached. If the release already exists, it replaces the file. | Started by the build, or manually with a run ID |

Sync, then build, then release. If upstream has new code but the same version number, the code is
synced and no new build or release is made. Tick **Force build** when running the sync manually
to build anyway.

## Running things by hand

Open the **Actions** tab, pick a workflow, and use **Run workflow**.

- **Sync from upstream**: options "Start a build if the sync changed anything" and "Force build".
- **Build Windows installer**: builds whatever is on `main` now.
- **Publish release**: takes the ID of a successful build run.

To get an email for each new release, use **Watch**, then **Custom**, then **Releases** on this
repo.

## Good to know

- **Edits outside `.github/workflows` and this README are overwritten.** Each sync makes the code
  match the original repo, so change workflows here and app code upstream.
- **Releases are public** because this repo is public.
- **Libmpv is fetched at build time.** The build tries the version pinned upstream, then a known
  good build, then the newest one from the mpv-winbuild project, because older builds get deleted.
- **Scheduled runs can be late.** GitHub may delay them, and it pauses schedules after about 60
  days without repo activity. Sync commits count as activity.
- **Optional secret `RELEASE_TOKEN`.** If GitHub's default token ever cannot create a release, add
  a personal access token with contents write access under this name. The release workflow uses it
  when present.

## About the app

Anilili Desktop is a Tauri 3, Rust and React app. mpv paints the video under the interface. It
browses and plays anime, with a library, downloads, subtitles and extra providers. The feature
list is in the [original README](https://github.com/sanilili/desktoprepo1#readme), and the
project layout and rules are in [AGENTS.md](AGENTS.md).

## Credits and license

All app code belongs to the [original author](https://github.com/sanilili/desktoprepo1) and is
under the license in [LICENSE](LICENSE) (MIT No Attribution). This fork only adds the build and
sync workflows and this README. It is not affiliated with the original project.
