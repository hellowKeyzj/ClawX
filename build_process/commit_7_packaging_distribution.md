# Commit 7: Packaging and Distribution

## Summary
Set up comprehensive packaging and distribution infrastructure including CI/CD workflows, multi-platform build configuration, icon generation, and macOS code signing support.

## Changes

### GitHub Actions Workflows

#### `.github/workflows/ci.yml` (New)
Continuous Integration workflow for PRs and main branch:
- **lint**: ESLint validation
- **typecheck**: TypeScript type checking
- **test**: Unit test execution
- **build**: Multi-platform build verification (macOS, Windows, Linux)

#### `.github/workflows/release.yml` (New)
Release workflow triggered by version tags:
- Matrix builds for all platforms
- Artifact collection and upload
- GitHub Release creation with auto-generated notes
- Pre-release detection for alpha/beta versions

### Build Configuration

#### `electron-builder.yml`
Enhanced configuration:
- Artifact naming with version, OS, and arch
- ASAR packaging with native module unpacking
- macOS: Universal binary, notarization ready, extended info
- Windows: NSIS installer with customization
- Linux: AppImage, DEB, RPM targets with dependencies

New features:
- `artifactName` template for consistent naming
- `asarUnpack` for native modules
- macOS `extendInfo` for privacy permissions
- DMG background and icon configuration
- Linux desktop entry with keywords

#### `package.json`
New scripts:
- `icons`: Generate icons from SVG source
- `clean`: Remove build artifacts
- `package:mac:universal`: Build universal macOS binary
- `publish`: Build and publish to GitHub
- `publish:mac/win/linux`: Platform-specific publishing
- `release`: Full release workflow

### Icon Generation

#### `resources/icons/icon.svg` (New)
Vector source icon:
- Gradient background (#6366f1 to #8b5cf6)
- White claw symbol with "X" accent
- 200px corner radius on 1024px canvas

#### `scripts/generate-icons.sh` (New)
Icon generation script:
- Generates PNG at multiple sizes (16-1024px)
- Creates macOS `.icns` via iconutil
- Creates Windows `.ico` via ImageMagick
- Creates Linux PNG set

#### `resources/icons/README.md` (New)
Documentation for icon requirements and generation.

### macOS Signing

#### `entitlements.mac.plist` (New)
macOS entitlements for:
- Unsigned executable memory (V8)
- JIT compilation
- Library validation disable
- Network client access
- Child process spawning (Gateway)
- File access permissions

### Linux Packaging

#### `scripts/linux/after-install.sh` (New)
Post-installation script:
- Update desktop database
- Update icon cache
- Create CLI symlink

#### `scripts/linux/after-remove.sh` (New)
Post-removal script:
- Remove CLI symlink
- Update databases

## Technical Details

### Build Matrix

| Platform | Target | Architecture | Format |
|----------|--------|--------------|--------|
| macOS | dmg, zip | universal | Intel + Apple Silicon |
| Windows | nsis | x64, arm64 | .exe installer |
| Linux | AppImage | x64, arm64 | Portable |
| Linux | deb | x64, arm64 | Debian package |
| Linux | rpm | x64 | Red Hat package |

### Artifact Naming Convention
```
${productName}-${version}-${os}-${arch}.${ext}
Example: ClawX-1.0.0-mac-universal.dmg
```

### Code Signing (Optional)

**macOS:**
```yaml
env:
  CSC_LINK: ${{ secrets.MAC_CERTS }}
  CSC_KEY_PASSWORD: ${{ secrets.MAC_CERTS_PASSWORD }}
  APPLE_ID: ${{ secrets.APPLE_ID }}
  APPLE_APP_SPECIFIC_PASSWORD: ${{ secrets.APPLE_APP_SPECIFIC_PASSWORD }}
  APPLE_TEAM_ID: ${{ secrets.APPLE_TEAM_ID }}
```

**Windows:**
```yaml
env:
  CSC_LINK: ${{ secrets.WIN_CERTS }}
  CSC_KEY_PASSWORD: ${{ secrets.WIN_CERTS_PASSWORD }}
```

### Release Process

1. Update version in `package.json`
2. Commit and push changes
3. Create and push version tag: `git tag v1.0.0 && git push --tags`
4. GitHub Actions builds all platforms
5. Artifacts uploaded to GitHub Release
6. Users receive update notification via electron-updater

### CI Pipeline

```
Push/PR to main
       |
       v
  ┌────────────────┐
  │     lint       │
  │   typecheck    │──> Parallel
  │     test       │
  │     build      │
  └────────────────┘
       |
       v
   All Pass?
       |
   ┌───┴───┐
   No      Yes
   |       |
   v       v
 Fail   Merge OK
```

### Release Pipeline

```
Push tag v*
       |
       v
  ┌─────────────────────────────┐
  │     Build (Matrix)          │
  │  ┌─────┬──────┬──────────┐  │
  │  │ mac │ win  │  linux   │  │
  │  └─────┴──────┴──────────┘  │
  └─────────────────────────────┘
       |
       v
  Upload Artifacts
       |
       v
  Create GitHub Release
       |
       v
  Auto-update available
```

## Version
v0.1.0-alpha (incremental)
