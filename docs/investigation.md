# Investigation

## First Steps
```bash
# Check SHA-256 and file types (see docs/checksums.txt)
shasum -a 256 AirXonix.exe program.exe setup.bin gameinf.bin
file AirXonix.exe program.exe setup.bin gameinf.bin music/*

```

### Findings
- From those filenames, this looks like an Inno Setup-style installation: unins000.exe and unins000.dat are the uninstaller.
- `setup.bin` may be the game’s packed graphics/models/levels, or simply installer data. gameinf.bin is probably configuration, registration, or game metadata
- `AirXonix.exe` is the launcher/compatibility checker; `program.exe` is the game binary to investigate. 
  - Evidence: `AirXonix.exe` explicitly checks for `ddraw.dll`, requires DirectX 7+, and starts `program.exe`. So you can set the launcher aside for now—apart from documenting its DirectX requirements.
- `program.exe` has useful clues:
- It is a 32-bit Windows GUI program.
- It contains DXT1, DXT2, DXT3, and DXT5 markers, which strongly suggests the game handles DirectX texture-compression formats.
- It includes strings resembling label, xonix, speed, font, and enemy when viewed in little-endian/reversed fragments. This is consistent with embedded game/UI data or a custom resource format.
- It has no readable filenames for levels, models, images, or audio. That typically means those assets are in a packed custom archive—most likely setup.bin—or that asset names are generated/obfuscated at runtime.
- setup.bin and gameinf.bin do not identify themselves as ZIP, 7-Zip, RAR, PNG, WAV, etc. from the available strings. They are likely proprietary binary data.

## Analysis of `program.exe`

```bash
# Install Ghidra
brew install ghidra
bash /opt/homebrew/Cellar/ghidra/12.1.4/bin/pyghidraRun
```
in Ghidra:
1. Import program.exe.
2. Run the default auto-analysis.
3. Open Symbol Tree → Imports

### Findings
#### Imports
- `ADVAPI32.DLL`
- `DDRAW.DLL`
- `DSOUND.DLL`
- `GDI32.DLL`
- `KERNEL32.DLL`
- `SHELL32.DLL`
- `USER32.DLL`
- `WINMM.DLL`
