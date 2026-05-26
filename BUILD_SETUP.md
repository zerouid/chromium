# Custom Chromium Build Setup Guide

## Prerequisites

This workflow builds Chromium for macOS ARM64 with all codecs enabled and your custom Google API keys.

## Setup Instructions

### 1. Add GitHub Secrets

You need to add your Google API keys as GitHub Secrets. **Never commit API keys to the repository.**

**Steps:**
1. Go to your repository: `https://github.com/zerouid/chromium`
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add these three secrets:

| Secret Name | Value |
|---|---|
| `GOOGLE_API_KEY` | Your Google API key |
| `GOOGLE_DEFAULT_CLIENT_ID` | Your OAuth 2.0 Client ID |
| `GOOGLE_DEFAULT_CLIENT_SECRET` | Your OAuth 2.0 Client Secret |

**⚠️ Security Warning:**
- Never paste these values in code comments or code blocks
- Store them securely in GitHub Secrets only
- Rotate keys periodically
- Limit key permissions in Google Cloud Console

### 2. Verify Build Configuration

The `args.gn` file contains your build configuration:

```gn
proprietary_codecs = true          # All codecs enabled
ffmpeg_branding = "Chrome"         # Chrome branding
enable_widevine = true             # DRM support
enable_webrtc = true               # WebRTC support
enable_platform_ac3_eac3_audio = true
enable_mse_mpeg2ts_stream_parser = true
target_cpu = "arm64"               # Apple Silicon
```

You can modify `args.gn` directly in the repository to adjust build options.

### 3. Trigger the Build

**Option A: Manual Trigger (Recommended for testing)**
1. Go to **Actions** tab
2. Select **Build Custom Chromium for macOS ARM64**
3. Click **Run workflow**
4. Optionally select build type: `debug` or `release`

**Option B: Automatic Trigger**
- Push changes to `main` branch that modify `args.gn` or the workflow file
- The workflow will automatically start

### 4. Monitor the Build

The build takes approximately **2-4 hours** depending on caching and system load.

1. Go to **Actions** tab
2. Click on the running workflow
3. Watch real-time logs in the **build** job

**Key milestones:**
- ~5 min: Depot tools setup
- ~30-60 min: Chromium source fetch/sync
- ~20 min: Build dependencies installation
- ~60-120 min: Compilation
- ~5 min: Artifact creation

### 5. Download Build Artifacts

After successful build:

1. Go to the completed workflow run
2. Scroll to **Artifacts** section
3. Download `chromium-macos-arm64-build`
4. Extract: `tar -xzf chromium-macos-arm64-*.tar.gz`
5. The app is in `Chromium.app`

**Alternative:** Download from Releases
1. Go to **Releases** page
2. Download the `chromium-macos-arm64-*.tar.gz` file

### 6. Use Your Custom Build

After extraction:
```bash
# Option 1: Run directly
./Chromium.app/Contents/MacOS/Chromium

# Option 2: Open as app
open Chromium.app

# Option 3: Install system-wide
sudo cp -r Chromium.app /Applications/
```

## Build Configuration Details

### Codecs Enabled
- H.264 (AVC)
- VP8, VP9
- AV1
- AC3, EAC3 (Dolby Digital)
- All other proprietary codecs

### DRM & Media
- Widevine (Google's DRM)
- WebRTC (Video/Audio streaming)
- MSE MPEG2TS (Streaming protocol support)

### Performance Optimizations
- `symbol_level = 0`: No debug symbols (smaller binary, faster)
- `is_component_build = false`: Monolithic build
- `enable_iterator_debugging = false`: Performance
- `is_clang = true`: Better optimization

### Features
- Rust support (`enable_rust = true`)
- SwiftShader GPU fallback
- **No auto-updates** (for custom builds)

## Troubleshooting

### Build Times Out
- Workflow timeout is set to 8 hours (480 minutes)
- First build takes longest due to source fetching
- Subsequent builds use cache and are faster

### Out of Disk Space
- The workflow cleans up before building
- If still failing, reduce `ninja -j$CORES` parallelism by editing the workflow

### API Keys Not Working
1. Verify secrets are set correctly in GitHub Settings
2. Check secret values don't have extra spaces/newlines
3. Ensure keys have appropriate Google Cloud permissions

### Build Fails During Compilation
- Check the build logs for specific error
- Ensure `args.gn` syntax is correct (GN is Python-based)
- Try a clean rebuild by re-running the workflow

## Customization

### Change Build Options

Edit `args.gn`:
```gn
# Example: Enable debug symbols
symbol_level = 2

# Example: Enable component build (faster, larger)
is_component_build = true

# Example: Change target
target_cpu = "x86_64"  # For Intel Macs
```

Then push changes or manually trigger workflow.

### Custom Branding

Add to `args.gn`:
```gn
chrome_product_full_name = "My Chromium"
enable_widevine = true
```

## Security Considerations

1. **API Keys**: Stored securely in GitHub Secrets, not in repository
2. **Build Artifacts**: Downloaded directly from GitHub Actions
3. **Source Code**: Synchronized from official Chromium repository
4. **No Auto-Update**: Your build won't auto-update (use `enable_updater = true` if desired)

## Resources

- [Chromium Build Documentation](https://chromium.googlesource.com/chromium/src/+/main/docs/mac_build_instructions.md)
- [GN Build System](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)
- [Chromium Supported Build Flags](https://chromium.googlesource.com/chromium/src/+/main/tools/gn/docs/reference.md)
- [Google APIs Documentation](https://developers.google.com/apis-explorer)

## Support

For issues:
1. Check the workflow logs in GitHub Actions
2. Review the troubleshooting section above
3. Check official Chromium build documentation
4. Open an issue in your repository with logs
