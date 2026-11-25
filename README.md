# nugget-tgc: a Tiger Game.com emulator

nugget-tgc is a standalone emulator for the Tiger Game.com (1997) that runs on Windows, Linux and macOS.

- **Core:** a C99 port of MAME's Game.com driver and Sharp SM8500 CPU core (BSD-3-Clause, by Wilbert Pol and Robbbert). It doesn't depend on SDL or any other library, so it can be reused in other frontends.
- **Frontend:** SDL2. You get the same code and behaviour on every OS.

Version 1.0.0.

## Hardware emulated

| Part | Details |
|---|---|
| CPU | Sharp SM8521 (SM8500 core) at 4.9152 MHz (9.8304 MHz ÷ 2), from cycle-counted delay loops in Tiger's SDK. `--cpu-clock mame` uses MAME's 5.5296 MHz. The sound generators, timers and LCD refresh all run off the same clock |
| Memory | 1 KB internal RAM/registers, 8 KB extended RAM, 16 KB VRAM (two 8 KB pages), 4 KB internal ROM, 256 KB kernel ROM, cartridges up to 2 MB through four 8 KB MMU banks |
| Display | 200 × 160 pixels, 2 bits per pixel. The LCD control register picks one of four shade mappings. The display is column-major and double-buffered |
| Blitter | Hardware DMA: VRAM→VRAM, ROM→VRAM, RAM→VRAM and VRAM→RAM, with palette remap, transparency and mirroring |
| Sound | 2 × 4-bit wavetable channels (32 samples each), 1 noise channel, 1 × 8-bit PCM DAC |
| Timers | 2 programmable timers, a 1 s / 1 min clock timer, LCD vblank interrupt |
| Input | D-pad, A/B/C/D, Menu, Pause, Sound, Power, and a 13 × 10 resistive touch grid |
| Slots | Both cartridge slots (`--cart2`) |

## Firmware (required)

The Game.com won't boot without its own firmware. Put MAME's `gamecom.zip`, or these two files, in a folder called `bios` next to the program. You can also pass `--bios <folder or zip>`:

| File | Size | CRC32 |
|---|---|---|
| `internal.bin` | 4 096 bytes (SM8521 internal ROM) | `a0cec361` |
| `external.bin` | 262 144 bytes (system kernel ROM) | `e235a589` |

If you start without a cartridge, the console's built-in menu boots, including the PDA functions.

## Using it

Games load from `.bin`/`.tgc` files or straight from `.zip` archives, as used by MAME. The emulator picks the `.bin` inside the zip.

```
nugget-tgc "Sonic Jam.bin"
nugget-tgc sonicjam.zip
nugget-tgc --scale-mode sharp --palette lcd --ghosting 40 --fullscreen game.bin
```

On Windows you can also drag a `.bin` or `.zip` onto `nugget-tgc.exe`, or drop one onto the open window. All options can go in `nugget-tgc.ini` (copy `nugget-tgc.ini.example`).

### The menu

Press **Esc** (or right-click, or press **Back + Start** together on a controller) to open the
emulator menu. The game pauses while it's open. From there you can:

- **Load Game...**: browse your folders for `.bin` / `.zip` games (type a letter to jump to it,
  Backspace goes up a folder). **Recent Games** lists the last ten you played.
- **Save State / Load State** in the chosen **State Slot** (0–9), **Reset**, **Screenshot**.
- **Display**: palette, scaling, LCD ghosting, LCD grid, fullscreen.
- **Sound & Options**: volume, starting the cartridge automatically, pausing when the window
  is in the background.
- **Controls**: redefine the keyboard or the controller, or restore either to its defaults.
- **Quit**.

Use the arrow keys and Enter, the mouse, or a controller (d-pad or stick to move, A to choose,
B to go back, Start to close). Left/Right change a setting. Display and sound choices are saved
to the settings file when the menu closes.

**Starting games automatically:** a real Game.com always boots to its main menu, and a game
starts only when you tap CARTRIDGE. nugget-tgc does that tap for you as soon as the menu appears,
so a game you load starts on its own. It recognises the menu on screen, and doesn't patch anything.
Turn it off in **Sound & Options** (or `autostart = off`) if you want to use the console's own
applications (calculator, calendar, phone book) with a cartridge in.

### Controls

| Game.com | Keyboard | Controller (XInput / DirectInput / evdev) |
|---|---|---|
| D-pad | Arrow keys / numpad | D-pad or left stick |
| A B C D | Z X C V | A B X Y |
| Pause | Enter | Start |
| Menu | M / Backspace | Back / Guide |
| Sound | S | LB |
| Power | P | – |
| Stylus | Left mouse button | Right stick moves the pointer, RB or RT touches |
| Fast forward | Tab (hold) | LT (hold) |

**The pointer:** the mouse and a controller's right stick move the same pointer. Over the
game screen it's a crosshair and acts as the stylus (left button, or RB / RT on a controller,
touches the screen); in the menu it points at items and clicks them. In fullscreen the pointer
hides after 3 seconds without movement and comes back as soon as you move the mouse or the
stick. Set `pointer_hide = 0` to keep it visible, or another number of seconds, and
`pointer_speed` (default 1.0) to make the stick faster or slower.

Controllers hot-plug. If you put the community [`gamecontrollerdb.txt`](https://github.com/mdqinc/SDL_GameControllerDB) next to the program, more obscure DirectInput pads get a proper layout. Pads with no mapping still work through a generic layout: the hat or first stick is the d-pad, and buttons 1–4 are A–D.

**Redefining the controls:** press **F10** for keyboard setup or **Shift+F10** for controller setup.
The emulator pauses and asks for each control in turn. Press the key or button you want, or:
F1 to keep the current one, Delete for none, Backspace to go back a step, Esc to cancel. When the
last control is set, the layout is saved to `nugget-tgc.ini` and used from then on. If the key you
press was already used for another control, the screen tells you that control is now unset.
On a controller, mapping a button replaces that control's buttons but keeps its stick direction
(and the reverse), and generic DirectInput pads keep their own mapping.

You can also edit the bindings in `nugget-tgc.ini` (see `nugget-tgc.ini.example`): `key_<control>` takes
up to two key names, `pad_<control>` up to eight controller inputs such as `a`, `dpup`, `-lefty`,
`righttrigger`, or `joy3` / `joyaxis1+` / `joyhat0up` for generic pads. `none` unbinds.
The controls are `up down left right a b c d menu pause sound power stylus fastforward`.

**Hotkeys** (each can be changed in `nugget-tgc.ini`, see below):

| Key | Action | Setting |
|---|---|---|
| F2 | Cycle palette | `hotkey_palette` |
| F3 | Ghosting on/off | `hotkey_ghosting` |
| F4 | Cycle scale mode | `hotkey_scale_mode` |
| F6 | LCD grid on/off | `hotkey_grid` |
| F5 / F7 | Save / load state | `hotkey_save_state` / `hotkey_load_state` |
| 0–9 | Choose state slot | |
| F8 | Reset | `hotkey_reset` |
| F9 or Pause | Pause emulation | `hotkey_pause` |
| F10 / Shift+F10 | Keyboard / controller setup | `hotkey_keyboard_setup` / `hotkey_controller_setup` |
| F11 or Alt+Enter | Fullscreen | `hotkey_fullscreen` |
| F12 | Screenshot at native 200×160 | `hotkey_screenshot` |
| Shift+F12 | Screenshot as displayed (scaled, with palette and grid) | `hotkey_screenshot_screen` |
| Esc | Emulator menu | `hotkey_emulator_menu` |
| (none) | Quit straight away | `hotkey_quit` |

A hotkey is a key name with optional `Ctrl+`, `Alt+` or `Shift+` in front, for example
`hotkey_screenshot = Ctrl+P` or `hotkey_screenshot = Print Screen`. `none` disables it.

**Screenshots** go to a `screenshots` folder next to the program, created on first use, as
`<game> <date> <time>.bmp`. Set `screenshot_dir = <folder>` to use another folder. If the
program's folder isn't writable (say, under Program Files), they go to `screenshots` in the save
folder instead; the path is printed and shown on screen.

**Firmware location:** nugget-tgc looks for `gamecom.zip`, or `internal.bin` and `external.bin`, in a
`bios` folder next to the program, next to the program itself, and next to the game. To keep it
anywhere else, set `bios = <folder or zip>` in `nugget-tgc.ini` (a relative path is taken from the
program's folder, and `~` means your home folder), or pass `--bios`. If no firmware is found,
nugget-tgc shows a screen asking you to drag `gamecom.zip` or the firmware folder onto the window;
the location is then saved to `nugget-tgc.ini` so it is only asked once.

Saves and states go to the per-user folder: `%APPDATA%\nugget-tgc\nugget-tgc\` on Windows, `~/.local/share/nugget-tgc/nugget-tgc/` on Linux.
Settings are read from `nugget-tgc.ini` next to the program, then the one in that folder. Settings
changed from inside the emulator are written to the one next to the program if it exists, and
otherwise to the per-user one.

## Internet cartridge (online)

The Game.com Internet cartridge is a text terminal that drives an external modem with AT commands
over the serial port. nugget-tgc emulates the SM8521 serial port (UART) and a Hayes-compatible modem
with a **built-in text web gateway**. Nothing else needs to be installed or running.

1. Start nugget-tgc with the Internet cartridge and tap **Connect To Other Service → Connect**.
2. Choose **Manual Dial** with the d-pad and press A.
3. Open the keyboard (**Ky**), type `ATDT1` and press Enter. Any number reaches the gateway.

The gateway serves a menu: Wikipedia, Hacker News, NPR and CNN text editions, FrogFind search, or
any web address. Pages arrive as 31-column text with numbered links. Type a link number, N/P for
next/previous page, B for back, M for the menu, `G address` or `W words`, then Enter. Q hangs up.
Typing on the Game.com's own on-screen keyboard is part of the experience.

Pages are fetched with the operating system's own HTTPS support: WinHTTP on Windows, and the
`curl` program that ships with macOS and almost every Linux distribution. The gateway runs on its
own thread, so a slow website never slows the emulator. Nothing on the network side runs until
the cartridge dials, so other games aren't affected.

**Telnet BBSes:** dial a host name directly (`ATDT bbs.example.com:23`), or map numbers to hosts
with `--phonebook 5551234=bbs.example.com:23`. `--modem host:port` sends every call to another
server, such as the standalone `tools/gcgateway.py` running on a different machine. Set
`NUGGET_TGC_MODEM_LOG=1` to log the modem conversation.

The "9600 baud" setting programs timer 0 to exactly 9600 baud at 4.9152 MHz, and nugget-tgc uses
that rate on the emulated serial line.

The gateway menu also has **7 Game.com High Scores**, which shows your Web Link Hall of Fame (below).

## Saved data and the Web Link Hall of Fame

The Game.com keeps its phone book, calendar and every game's high score in 8 KB of extended RAM
that belongs to the console, not the cartridge. nugget-tgc saves it to `gamecom.nv` in your save
folder (the path is printed at start-up), shared by all games, just as on the real console.

With saved data, the firmware asks **DATA FOUND.. PRESS A TO KEEP / PRESS B TO CLEAR** at power-on.
nugget-tgc answers A about 0.06 seconds after power-on, and ignores B until then, so the data
can't be cleared by accident and the prompt barely shows. To start afresh, delete `gamecom.nv` (and `gamecom.nv.bak`) while nugget-tgc is closed.
`--no-nvram` starts with blank memory and saves nothing, leaving the file untouched.

`gamecom.nv` is always exactly 8 KB, the size of the console's memory. Each save replaces the
whole file, so it never grows however long you use it. nugget-tgc saves within 30 seconds of any
change, and again on exit. Each save goes to a temporary file that is then swapped in, with the
previous copy kept as `gamecom.nv.bak`, so a crash or power cut can't leave a half-written file.
If `gamecom.nv` is ever damaged, nugget-tgc moves it aside as `gamecom.nv.bad` and loads the backup.
Loading a save state doesn't roll the high-score table back.

**Web Link** was a cable and PC program for uploading those high scores to Tiger's web site,
which closed long ago. nugget-tgc plays the PC program instead:

1. Play some games. Their high scores are saved to the console's memory as usual.
2. Load the Web Link cartridge, tap it on the menu and press **START** on its title screen.
3. nugget-tgc reads the score table and records it in the **Hall of Fame**. The Game.com then shows
   "N SCORES SAVED TO THE NUGGET-TGC HALL OF FAME!" for a few seconds and returns to the menu.

The Hall of Fame lives in your save folder:

- `hall_of_fame.html`: a recreation of a late-90s game.com high-score web page. Each game shows
  its best score (or, for times and other non-numeric scores, the latest), plus an upload log.
- `hall_of_fame.txt`: the same as plain text. The gateway's High Scores page shows this file.
- `hall_of_fame.tsv`: the record of every upload, one score per line, including the raw 64-byte
  record. Edit or delete it to change the history; the other two files are rebuilt from it.

Each game keeps one record on the console, so uploading regularly builds up a history that the
console itself never kept. Re-uploading unchanged scores adds nothing. The window title shows
the link's progress; `NUGGET_TGC_WEBLINK_LOG=1` logs the conversation to the console.

The link protocol, decoded from the cartridge and firmware, is documented in
`src/frontend/weblink.h`. Records can also be read back one game at a time (command 07) and
written to the console (command 09). nugget-tgc doesn't use those yet.

## Graphics and hardware compatibility

Video goes through SDL's 2D renderer, which picks the best backend each machine supports. If one fails, it falls back to the next:

- Windows: Direct3D 11 → Direct3D 9 → OpenGL → software
- Linux: OpenGL → OpenGL ES 2 → software. This covers Raspberry Pi and other ARM boards, and works on X11, Wayland and KMS/DRM consoles
- macOS: Metal → OpenGL → software

You can force a backend with `--renderer`. That helps on machines with buggy drivers: `software` works anywhere.

**Scale modes:**

| Mode | What it does |
|---|---|
| `integer` (default) | Every Game.com pixel becomes an identical N×N square. The image is pixel-perfect, with black borders where the window isn't an exact multiple |
| `sharp` | Scales up by an integer amount, then fits the window with bilinear filtering. It fills the screen and stays crisp. It needs render-target support and falls back to `fit` without it |
| `fit` | Nearest-neighbour, aspect-correct, fills the window. You may see slightly uneven pixel widths |
| `stretch` | Fills the window and ignores the aspect ratio |

The LCD grid and ghosting effects are drawn with plain 2D operations, so they also work with the software renderer. No GPU shaders are needed.

**32-bit builds:** for very old PCs, the CI also produces a 32-bit Windows build.

## Building

The emulation core has no dependencies. The frontend needs SDL 2.0.10 or later.

**Linux:**
```
sudo apt install build-essential cmake libsdl2-dev
cmake -S . -B build && cmake --build build -j
```

**Windows (MSYS2, makes a single portable .exe):**
```
pacman -S mingw-w64-x86_64-{gcc,cmake,ninja,SDL2}
cmake -S . -B build -G Ninja -DTGC_STATIC_SDL=ON && cmake --build build
```

**Windows (Visual Studio + vcpkg):**
```
vcpkg install sdl2:x64-windows-static
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=%VCPKG_ROOT%/scripts/buildsystems/vcpkg.cmake ^
      -DVCPKG_TARGET_TRIPLET=x64-windows-static -DTGC_STATIC_SDL=ON
cmake --build build --config Release
```

The GitHub Actions workflow in `.github/workflows/build.yml` builds Linux, Win64, Win32 and macOS binaries on every push. It also runs a self-test.

**Headless runner** (for testing and CI):
```
python3 tools/make_test_bios.py testbios            # tiny hand-assembled CPU/LCD/DMA test
nugget-tgc-headless --bios testbios --frames 60 --ppm out.ppm --state-test
nugget-tgc-headless --bios bios --cart game.bin --frames 1800 --ppm shot.ppm --wav audio.wav
```

### Windows builds from Linux

Besides MSYS2, the Windows x64 version can be cross-built on Linux with clang and lld: build the
mingw-w64 headers and CRT with clang, add compiler-rt's builtins for `x86_64-w64-mingw32`, build
SDL2 statically for that target, then configure with a toolchain file like
`tools/mingw64-clang.cmake.example` and `-DTGC_STATIC_SDL=ON`. The result is a single
`nugget-tgc.exe` that needs nothing but Windows itself (Windows 10/11; Windows 7/8.1 with the
Universal C Runtime update). `-DTGC_WINDOWS_CONSOLE=ON` gives a build with a console window for
log messages.

## Accuracy notes and known issues

Tested on the real firmware with 19 games: Lights Out, Sonic Jam, Tiger Casino, Fighters Megamix, Mortal Kombat Trilogy, Resident Evil 2, Indy 500, Quiz Wiz, Jeopardy!, Wheel of Fortune, Wheel of Fortune 2, Duke Nukem 3D, The Lost World: Jurassic Park, Scrabble, Monopoly, Batman & Robin, Centipede, Henry and Williams Arcade Classics. Not yet tested: Frogger and the Castlevania prototype. All boot (tap CARTRIDGE with the stylus), render correctly and pass the save-state determinism test. Emulation runs at about 18–20× real time on one CPU core.

**Fixed compared with MAME:**
- **Blitter DMA starts the moment HALT executes**, as the datasheet describes. Without this, a pending timer interrupt could wake the CPU before the blit ran. In Fighters Megamix, that dropped letters, digits and portraits.
- **Interrupts are latched.** MAME threw away any timer interrupt that arrived while interrupts were disabled. During speech that lost 24% of Timer 1 interrupts, which made speech play about 24% slow. Requests now stay in IR0/IR1 until the CPU accepts them, as on the real SM8521. Losses are now about 0.2%, and boot speech plays at 7,856 samples per second against an intended 7,877.
- **CPU clock is 4.9152 MHz.** This matches Tiger's SDK delay loops and the datasheet: a 9.8304 MHz crystal, with CKC set to main ÷ 2 by the firmware. MAME's 11.0592 MHz crystal comes from an example in the SM8521 datasheet, not from the Game.com.
- **Game countdowns run in real time.** MAME halves every timer prescaler, and its driver notes that "countdown goes twice as fast". nugget-tgc halves only prescaler setting 0 (fCK/2, used for speech playback). Settings 1–7 (/1024 to /65536) use the full divisor. Quiz Wiz's question timer now counts one second every 1.016 s; under MAME's timing it counts one every 0.49 s. Speech rate and boot time are unaffected.

**Checked against real hardware:**
- **Power-on to main menu takes about 8.3 s on a real unit** (stopwatch). The boot is paced by timer-driven speech, not by screen frames: it takes the same time at any refresh rate from 30 to 74 Hz. nugget-tgc takes about 8.0 s. MAME's timing gives 6.85 s.
- **Game speed comes from the timers, not the LCD refresh.** Sonic Jam and Tiger Casino run at the same speed at 30 Hz and 49 Hz. The refresh rate only decides how often the screen is shown.

**Still open:**
- The two wavetable channels don't sound quite like the real thing. The SG2 noise channel is a guess; disable it with `--noise off`.
- The CPU starting at 1/32 speed after reset (until the firmware's clock-switch STOP) isn't emulated. It only affects the first few milliseconds.
- On the main menu, the A button highlights CARTRIDGE but doesn't launch the game. Tap it with the stylus: the mouse, or a controller's right stick plus RB.

## Licence

BSD-3-Clause. See `LICENSE`. Copyright of the original driver belongs to Wilbert Pol, Robbbert and the MAME team.

## Disclaimer

Development and assistance was provided by Claude.ai throughout the course of this project.
