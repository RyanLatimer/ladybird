# Running Ladybird on your MacBook Air M2

Your fork: **https://github.com/RyanLatimer/ladybird**

This repo adds one file on top of upstream Ladybird:

- `.github/workflows/build-macos-m2-app.yml` — builds `Ladybird.app` for
  Apple Silicon in the cloud and uploads it as a `.zip` + `.dmg` artifact.

Because Apple Silicon binaries must be built on macOS, you can't build the
`.app` on Linux — you either build on your Mac, or let GitHub's Mac runners
do it for you. Both paths are covered below.

---

## Option A — Download a cloud-built .app (easiest, no build on your Mac)

1. Go to https://github.com/RyanLatimer/ladybird/actions/workflows/build-macos-m2-app.yml
2. Click **Run workflow** → **Run workflow** (on `master`).
   - Or push to the `macos-app` branch — that also triggers it.
3. Wait for the run to finish. A full Ladybird build takes a while
   (typically 60–180 min on `macos-15`, mostly vcpkg + compile).
4. On the finished run, download either artifact:
   - `Ladybird-macOS-arm64-zip`, or
   - `Ladybird-macOS-arm64-dmg`
5. Unzip / open the dmg, drag `Ladybird.app` to `/Applications`
   (or just double-click it from Downloads).
6. First launch: right-click `Ladybird.app` → **Open** → **Open**.
   This is needed because the app is ad-hoc signed, not notarized by Apple.
   If macOS says it's damaged, run once:
   ```bash
   xattr -dr com.apple.quarantine /Applications/Ladybird.app
   ```

To check status from the terminal:
```bash
gh run list --repo RyanLatimer/ladybird --workflow build-macos-m2-app.yml
gh run watch --repo RyanLatimer/ladybird <run-id>
```

## Option B — Build directly on your MacBook Air M2

Requirements: macOS with Xcode command line tools (~30 GB free disk).

```bash
# 1. Tools
xcode-select --install
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install autoconf autoconf-archive automake ccache cmake libtool nasm ninja pkg-config
brew install rustup && rustup-init -y  # or: brew install rust

# 2. Clone YOUR fork (not upstream)
git clone https://github.com/RyanLatimer/ladybird.git
cd ladybird

# 3. Build + run (Release, Apple Silicon)
./Meta/ladybird.py run

# The bundle lives at:
#   Build/release/bin/Ladybird.app
# Run it like upstream docs suggest:
open -W --stdout $(tty) --stderr $(tty) ./Build/release/bin/Ladybird.app

# Optional: install to /Applications
cp -R Build/release/bin/Ladybird.app /Applications/
```

Speed tips for the Air (fanless — thermals matter):

- Keep it plugged in, close other heavy apps.
- First build downloads + compiles ~30 vcpkg deps; leave it alone.
- Rebuilds are incremental and much faster thanks to ccache.

## Keeping your fork fresh

```bash
git remote add upstream https://github.com/LadybirdBrowser/ladybird.git
git fetch upstream
git checkout master
git merge upstream/master
git push origin master
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| "Ladybird.app is damaged" | `xattr -dr com.apple.quarantine /Applications/Ladybird.app` |
| Right-click Open still blocked | System Settings → Privacy & Security → allow Ladybird |
| Slow first launch | Normal — macOS validates the binary once; allow Terminal in Privacy & Security → Developer Tools |
| Build OOM on 8 GB Air | Close browsers, build with `./Meta/ladybird.py build` (fewer parallel jobs if needed) |
| Workflow fails on vcpkg | Re-run failed jobs; transient network flakes are common |

Upstream build docs: `Documentation/BuildInstructionsLadybird.md`.
