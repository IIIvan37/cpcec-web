# CPCEC-Web

**Amstrad CPC emulator running in your browser via WebAssembly**

This is a fork of [CPCEC](https://github.com/cpcitor/cpcec) by [CNGSOFT](http://cngsoft.no-ip.org/cpcec.htm) with added WebAssembly/browser support.

## Try it online

> **[Launch CPCEC-Web](https://cpcec-web.netlify.app/)** *(coming soon)*

## What's new in this fork

- **Browser support**: Run the emulator directly in modern web browsers (Chrome, Firefox, Safari, Edge)
- **WebAssembly**: Compiled with Emscripten for near-native performance
- **Drag & Drop**: Load `.dsk`, `.sna`, `.cdt` files by dropping them on the emulator
- **Touch controls**: Basic support for mobile devices
- **No installation**: Just open the webpage and play

## Building for Web

### Prerequisites
- [Emscripten SDK](https://emscripten.org/docs/getting_started/downloads.html)
- ROM files (cpc464.rom, cpc664.rom, cpc6128.rom, cpcplus.rom, cpcados.rom)

### Build
```bash
# Activate Emscripten
source /path/to/emsdk/emsdk_env.sh

# Build
./build-web.sh

# Test locally
./build-web.sh serve
# Open http://localhost:8080
```

## Files added/modified

| File | Description |
|------|-------------|
| `cpcec-web.c` | Emscripten wrapper with async main loop |
| `cpcec.c` | Added `CPCEC_NO_MAIN` guard |
| `cpcec-rt.h` | Auto-detect Emscripten |
| `web/index.html` | Browser UI |
| `build-web.sh` | Build script |
| `Makefile.emscripten` | Makefile for web builds |

## JavaScript API

The WebAssembly build exposes several functions that can be called from JavaScript to control the emulator. These functions are available on the `Module` object after the WASM module has loaded.

### Emulator Control Functions

| Function | Description |
|----------|-------------|
| `Module._em_load_file(path)` | Loads a file (`.dsk`, `.sna`, `.cdt`) into the emulator and runs it automatically |
| `Module._em_inject_file(path)` | Injects a file into the emulator without executing it (for manual selection) |
| `Module._em_reset()` | Resets the emulator to its initial state |
| `Module._em_pause()` | Toggles pause/unpause (has no effect in debug mode) |
| `Module._em_set_speed(fast)` | Sets emulation speed (0 = normal, 1 = fast) |
| `Module._em_is_running()` | Returns 1 if emulator is running, 0 otherwise |
| `Module._em_get_status()` | Returns current status as string ("initializing", "stopped", "paused", "running", "running (fast)") |

### Keyboard Input Functions

| Function | Description |
|----------|-------------|
| `Module._em_key_press(cpc_code)` | Presses a CPC key by its matrix code (0-127) |
| `Module._em_key_release(cpc_code)` | Releases a CPC key by its matrix code (0-127) |
| `Module._em_press_fn(fn_num)` | Presses a function key by number (0-9, where 0=F0, 1=F1, etc.) |
| `Module._em_release_fn(fn_num)` | Releases a function key by number (0-9) |

### CPC Keyboard Matrix Codes

The CPC uses a matrix-based keyboard system. Common key codes include:
- Function keys: F0=0x0F, F1=0x0D, F2=0x0E, F3=0x05, F4=0x14, F5=0x0C, F6=0x04, F7=0x0A, F8=0x0B, F9=0x03
- Arrow keys: Up=0x00, Down=0x02, Left=0x08, Right=0x01
- Space: 0x40
- Return: 0x06

### Usage Example

```javascript
// Load and run a game automatically
Module._em_load_file("game.dsk");

// Inject a disk without running it (for manual selection)
Module._em_inject_file("game.dsk");

// Reset the emulator
Module._em_reset();

// Press F1 (function key 1)
Module._em_press_fn(1);
setTimeout(() => Module._em_release_fn(1), 100);

// Check if running
if (Module._em_is_running()) {
    console.log("Status:", Module._em_get_status());
}
```

## License

GPLv3 - Same as the original project.

## Credits

- **Original CPCEC**: [CNGSOFT](http://cngsoft.no-ip.org/cpcec.htm) (Cesar Nicolas-Gonzalez)
- **Git archive**: [cpcitor](https://github.com/cpcitor/cpcec)


---

# Original README

An emulator by CNGSOFT under GPLv3 license.

This repository was created by a script turning the zip files available on [http://cngsoft.no-ip.org/cpcec.htm](http://cngsoft.no-ip.org/cpcec.htm) into a git history.

The emulator can compile on Windows and SDL2-supported platforms including Linux desktop.

See [cpcec.txt](cpcec.txt) for instructions in English, including build instructions.

Below is a snapshot of [http://cngsoft.no-ip.org/cpcec.htm](http://cngsoft.no-ip.org/cpcec.htm) turned into markdown format.

------------------------------------------------------------------------

---
---

![](http://cngsoft.no-ip.org/cpcec192.png) ![](http://cngsoft.no-ip.org/zxsec192.png) ![](http://cngsoft.no-ip.org/csfec192.png) ![](http://cngsoft.no-ip.org/msxec192.png)  
**CPCEC** Amstrad CPC emulator  
and its siblings **ZXSEC**, **CSFEC** and **MSXEC**  
82692 hits  
  

## Foreword

**CPCEC** is an emulator of the family of home microcomputers **Amstrad CPC** (models 464, 664, 6128 and Plus) whose goal is to be loyal to the original hardware and efficient in standard modern systems. Thus it brings a faithful emulation of the Z80 microprocessor and it replicates the behavior of the CRTC 6845 and Gate Array video chips, the PSG AY-3-8912 sound chip, the remaining circuits found in the original hardware, and the tape deck and floppy disc drive that made possible loading and running software.

CPCEC includes several related projects. **ZXSEC** is an emulator of the **Sinclair Spectrum family (48K, 128K, +2/Plus2 and +3/Plus3)** based on the components it shared with the Amstrad CPC family: the Z80 microprocessor, the PSG AY-3-8912 sound chip, the tape system and the NEC765 disc drive controller; **CSFEC** is an emulator of the **Commodore 64** platform, similarly based on shared code; and **MSXEC** is an emulator of the **MSX family (1983 MSX, 1985 MSX2, 1988 MSX2+)**, also based on shared code.

The default build of CPCEC requires a Microsoft Windows 2000 operating system or later, while the SDL2 build requires any operating system supported by the SDL2 library. The minimal hardware requirements are those fitting the operating system, and it's advised that the main microprocessor runs at 400 MHz at least. Screen resolution in pixels must be 800x600 at least. A sound card is optional. Using a joystick is optional, too.

Software and documentation are provided "as is" with no warranty. The source code of CPCEC and its binaries follow the **GNU General Public License** v3, described in the file GPL.TXT within the package.

## Gallery

See [original CPCEC page](http://cngsoft.no-ip.org/cpcec.htm) for an image gallery.

## Acknowledgements

This emulator owes its existence to a series of people and societies that are listed as follows:

-   The firmware files included in the package are **Amstrad**'s properties, who allows the emulation of their old computer systems and supports the distribution of their firmwares as long as their authorship and contents are respected, and whom I wholeheartedly thank the creation of those magnificent computers and the good will towards their emulation.
-   This emulator was my final project for the Computer Engineering postdegree at the **Distance-Learning National University (Universidad Nacional de Enseñanza a Distancia, UNED)**, a project directed by professor **José Manuel Díaz Martínez** and ultimately awarded a 100% and the right to a honorable mention.
-   The documentation about the system comes from **cpcwiki.eu, cpc-power.com, cpcrulez.fr** and **quasar.cpcscene.net**.
-   The alpha tests were handled by **Denis Lechevalier**.

## Version log

See [git history](https://github.com/cpcitor/cpcec/commits)

------------------------------------------------------------------------

\[ [amstrad.es](http://www.amstrad.es/forum/viewtopic.php?t=5332) \| [cpcrulez.fr](http://cpcrulez.fr/forum/viewtopic.php?t=6195) \| [cpcwiki.eu](http://www.cpcwiki.eu/forum/emulators/cpcec-a-new-emulator-from-cngsoft/new#new) \| [CPCEC Git archive (cpcitor)](https://github.com/cpcitor/cpcec) \| Norecess464: [CPCEC-GTK](https://gitlab.com/norecess464/cpcec-gtk) + [CPCEC-PLUS](https://gitlab.com/norecess464/cpcec-plus) \]

------------------------------------------------------------------------

  
[Spanish CPC firmware](http://cngsoft.no-ip.org/cpc-rom-esp.zip) (1052 hits) [French CPC firmware](http://cngsoft.no-ip.org/cpc-rom-fra.zip) (970 hits)

------------------------------------------------------------------------

[Send a comment to the original author of cpcec](/comments.htm)
