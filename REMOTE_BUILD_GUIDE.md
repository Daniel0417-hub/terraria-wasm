# Terrarium remote-build kit (Chromebook-friendly)

This kit adds a GitHub Actions workflow to a **fork of `MercuryWorkshop/terraria-wasm`**.
It does not contain Terraria game binaries, decompiled Terraria source, or prebuilt WASM.
The workflow uses the upstream Terrarium code and builds it on a hosted Ubuntu runner.

## Important: supported game version

The upstream repository README does not pin a numeric Terraria version. It requires a
Windows or Linux Terraria installation and decompiles the `Terraria.exe` supplied to the
build. Therefore this workflow builds against the installation archive you provide.
Start with a clean, unmodified Steam installation that matches the version supported by
the upstream source at the time you run the workflow. Do not assume Terraria 1.4.5 is
supported: upstream issue #15 asks about 1.4.5 compatibility and is not proof of support.

Upstream build instructions currently specify:
- x86_64 Linux
- .NET SDK 9.0.4
- `mono-devel`
- Node.js and pnpm
- `ilspycmd` exactly 9.0.0.7889
- Terraria Windows or Linux installation
- recursive Git submodules

## Part A — Fork the upstream project

1. Open https://github.com/MercuryWorkshop/terraria-wasm
2. Click **Fork** and create a fork under your GitHub account.
3. The upstream default branch is `master`; keep that as the fork's default branch.
4. Upload the files from this kit into the fork:
   - `.github/workflows/terrarium-build.yml`
   - `REMOTE_BUILD_GUIDE.md`
5. The workflow is preconfigured for `master`. You can also start it manually from Actions.

## Part B — Store your own game installation privately

Do **not** upload Terraria binaries or assets to the public Terrarium fork.

1. Create a separate **Private** GitHub repository, e.g. `terraria-private-assets`.
2. On your own computer, locate your legally obtained Terraria installation folder.
   It must include `Terraria.exe` and the adjacent DLL files such as `FNA.dll` and
   `SteelSeriesEngineWrapper.dll`. Keep the complete installation folder structure.
3. Compress the complete installation folder into a ZIP file named
   `terraria-windows-install.zip`. Do not include personal saves unless you intend to.
4. In the private repository, create a GitHub Release with tag `terraria-assets`.
5. Upload `terraria-windows-install.zip` as a release asset. Do not commit the ZIP into
   the repository's Git history.
6. Create a fine-grained personal access token with **Contents: read-only** access to
   this private asset repository. Add it to the Terrarium fork as a repository secret
   named `GAME_ASSET_TOKEN`. Never put the token in a file or commit it.

The release asset is downloaded only by your Actions runner. Keep the asset repository
private. A GitHub release asset in a private repository is not publicly downloadable.

## Part C — Configure the Terrarium fork

In the Terrarium fork, open **Settings → Secrets and variables → Actions**.

Add these **Variables**:
- `GAME_ASSET_REPO` = `YOUR_GITHUB_NAME/terraria-private-assets`
- `GAME_ASSET_TAG` = `terraria-assets`
- `GAME_ASSET_FILE` = `terraria-windows-install.zip`
- `DEPLOY_TO_CLOUDFLARE` = `false` initially

Add this **Secret**:
- `GAME_ASSET_TOKEN` = the fine-grained token created above

The workflow checks out the fork and its submodules, installs the upstream-pinned toolchain,
downloads your private ZIP, decompiles the executable, applies the upstream Vanilla patches,
and runs `make publish`. The output is saved as the Actions artifact
`terrarium-web-dist`.

## Part D — Run the build

1. Open the fork's **Actions** tab.
2. If GitHub asks, enable Actions for the fork.
3. Select **Build Terrarium for WebAssembly**.
4. Click **Run workflow**.
5. Open the run and inspect each step's log.
6. When successful, download `terrarium-web-dist` from the run's Artifacts section.

The first build may take a long time. GitHub-hosted runner time limits and account
allowances apply. If the workflow fails, copy the first failing step and its error log.

## Part E — Optional Cloudflare Pages deployment

GitHub Pages is not configured here. This project requires cross-origin isolation headers
for WebAssembly threads. The build adds `public/_headers`, which Vite copies into `dist`.
Cloudflare Pages supports a `_headers` file.

1. Create a Cloudflare Pages project named `terrarium-web` (or another name) in your
   Cloudflare dashboard.
2. Create a Cloudflare API token with permission to deploy to Cloudflare Pages.
3. In the Terrarium fork's Actions secrets, add:
   - `CLOUDFLARE_API_TOKEN`
   - `CLOUDFLARE_ACCOUNT_ID`
4. Add Actions variables:
   - `CLOUDFLARE_PAGES_PROJECT` = `terrarium-web`
   - `DEPLOY_TO_CLOUDFLARE` = `true`
5. Run the workflow again. The deploy job runs only when that variable is `true`.

## After deployment

Open the Cloudflare Pages URL. The site still needs the game content files in the browser:
use the page's own **Copy local assets**, **Download assets from Steam**, or **Extract
terrarium archive** option. The build-time executable archive is not a replacement for
the runtime game assets. Use only your own legally obtained game files.

## Known limitations

- This is a build-and-deploy pipeline, not a guarantee that every Terraria release works.
- Upstream currently says mods are not feasible due to WASM runtime/performance limitations.
- The upstream README requires COOP/COEP headers. Do not remove `public/_headers`.
- This workflow intentionally keeps the game installation in a separate private repository.
- GitHub Actions minutes, artifact limits, Cloudflare limits, and storage limits may apply.

## Upstream references

- https://github.com/MercuryWorkshop/terraria-wasm
- https://github.com/MercuryWorkshop/terraria-wasm/issues/15
- https://terraria.mercurywork.shop/
