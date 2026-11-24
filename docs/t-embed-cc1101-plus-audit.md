# Bruce Firmware Audit for the LILYGO T-Embed CC1101 Plus

The audit covered commit `2b6ba4d4` and the `lilygo-t-embed-cc1101` profile. The main priorities are the USB Mass Storage lifecycle and exclusive ownership of the microSD card. Separate code defects explain why the drive can disappear on re-entry, why ejection can fail, and why writes can be rejected when the card is full. On-board testing is still needed to identify which defect causes the observed hang.

On the Plus, the ST7789 display and microSD card share an SPI bus on GPIO 9/10/11, with separate chip-select pins on GPIO 41 and 13, respectively. Bruce passes the display's SPI object to the SD driver. The microSD clock is set to 4 MHz, giving a theoretical ceiling of 500 kB/s before protocol overhead and card wait times. [Board profile](../boards/lilygo-t-embed-cc1101/lilygo-t-embed-cc1101.ini#L36) · [SD initialization](../src/core/sd_functions.cpp#L68) · [LILYGO pinout](https://github.com/Xinyuan-LilyGO/LilyGoLib/blob/master/docs/hardware/lilygo-t-embed-plus.md#3-pin-map)

## USB Mass Storage

### 1. P0 · Re-entering Mass Storage breaks the USB lifecycle

Bruce creates a new `USBMSC` instance each time Mass Storage starts. In Arduino-ESP32 3.3.9, the constructor allocates a logical unit number that remains allocated after `end()`. On exit, Bruce explicitly calls the destructor of the global `USB` object and then calls `enableDFU()` on the same object. This path runs even when the card is absent. The destructor deletes the USB event loop, and TinyUSB rejects attempts to add interfaces after it starts. Bruce also ignores the return values of `msc.begin()` and `USB.begin()`.

Keep one `USBMSC` instance for the lifetime of the firmware, register the interface before the first `USB.begin()`, check the return values, and toggle media availability between sessions. Update the comment for `ARDUINO_USB_MODE=1` to identify the hardware USB Serial/JTAG controller. [Mass Storage entry and exit](../src/core/massStorage.cpp#L8) · [USBMSC construction and teardown](https://github.com/espressif/arduino-esp32/blob/3.3.9/cores/esp32/USBMSC.cpp) · [TinyUSB lifecycle](https://github.com/espressif/arduino-esp32/blob/3.3.9/cores/esp32/esp32-hal-tinyusb.c#L755) · [Serial modes](https://github.com/espressif/arduino-esp32/blob/3.3.9/cores/esp32/HardwareSerial.h#L403)

### 2. P0 · The firmware and PC can write to the microSD card concurrently

Mass Storage leaves the FAT filesystem mounted in Bruce. The background WebUI can continue performing file operations, and saving settings copies the configuration to the SD card. The SPI lock serializes bus transfers. Keeping FAT state consistent between the firmware and the PC requires a separate mechanism. On eject, the handler sets the stop flag and returns `false` to the host. Exiting with `Esc` clears the handlers while active I/O may still be running.

Grant MSC exclusive ownership of the card while preserving access to the block driver. Return ownership to the firmware after operations complete. Synchronize access to the exit request and USB status across tasks. Return success to the host on successful ejection. [MSC loop](../src/core/massStorage.cpp#L35) · [Eject handling](../src/core/massStorage.cpp#L123) · [Background WebUI](../src/core/wifi/webInterface.cpp#L781) · [Configuration save](../src/core/config.cpp#L446)

### 3. P1 · The write handler does extra work and rejects valid writes

For each write request, Bruce calls `f_getfree()` twice through the SD methods. This causes even overwrites of existing sectors to be rejected when the card is full. The read and write handlers ignore `offset` and report that the entire buffer was processed, even though their loops handle only whole sectors.

Remove the free-space check, validate request bounds and alignment, and either handle partial blocks correctly or explicitly reject them. After those fixes, measure block operation timing before considering a higher SPI clock rate. [MSC reads and writes](../src/core/massStorage.cpp#L87) · [Bruce SD driver](../lib/HAL/sd_card/SD.cpp#L80)

## User Interface and File Menu

| Priority | Problem and Fix |
| --- | --- |
| P0 · `Paste` can truncate the source file | When pasting into the same folder, Bruce opens the source path for reading and then opens the same path with `FILE_WRITE`. Compare the normalized source and destination paths before opening the destination for writing. [File operations](../src/core/sd_functions.cpp#L303) |
| P1 · The encoder and buttons become unresponsive | Multiple tasks modify `RotaryNetSteps` with non-atomic operations. When a menu opens, selection input is ignored for 600 ms. The file browser always waits 500 ms to detect a long press. At 100% brightness, dimming adds about 850 ms of software delays to the button polling task. Use an atomic read and reset for encoder steps, press and release events, and dimming that keeps polling responsive. [Encoder steps](../include/globals.h#L224) · [Menu selection](../src/core/display.cpp#L877) · [Long press](../src/core/sd_functions.cpp#L679) · [Dimming](../src/core/powerSave.cpp#L8) |
| P1 · Check the backlight and GPIO40 against the Plus board revision | Bruce uses PWM on GPIO21, while LILYGO documents a 16-step AW9364 pulse-controlled driver. Bruce also assigns GPIO40 to both `TFT_RST` and `I2S LRCLK`. The Plus documentation assigns GPIO40 to audio and lists no separate display reset pin. Check the schematic for the board revision before changing the profile. [Backlight](../boards/lilygo-t-embed-cc1101/interface.cpp#L196) · [TFT reset](../boards/lilygo-t-embed-cc1101/lilygo-t-embed-cc1101.ini#L51) · [I²S](../boards/lilygo-t-embed-cc1101/interface.cpp#L91) · [LILYGO documentation](https://github.com/Xinyuan-LilyGO/LilyGoLib/blob/master/docs/hardware/lilygo-t-embed-plus.md#2-display-specifications) |
| P2 · The submenu creates unnecessary SPI and I²C traffic | On every encoder step, `drawSubmenu()` reads the battery level over I²C, clears the content area, and clears individual rows again. The main menu grid already redraws only the cells that changed. For lists, update the visible rows and scrollbar, and poll the battery level on a timer. [Submenu rendering](../src/core/display.cpp#L1152) · [Grid rendering](../src/core/main_menu.cpp#L188) |
| P2 · The file browser is slow in large directories | Before showing the first screen, Bruce reads the whole directory, creates temporary strings while sorting, and builds a second `Option` list. Opening `Files` attempts to mount the SD card synchronously even when the user needs LittleFS. Move mounting into the `SD Card` action, remove temporary strings from sorting, and prepare data for the visible rows. [Opening Files](../src/core/menu_items/FileMenu.cpp#L10) · [Directory sorting](../src/core/sd_functions.cpp#L503) · [List construction](../src/core/sd_functions.cpp#L621) |

## Work Order and Verification

1. Fix the USB lifecycle and the microSD ownership handoff.
2. Protect the source file during `Paste`.
3. Fix input handling and check the backlight settings against the board revision.
4. Optimize submenu rendering and directory opening.

On the board, check repeated entry into Mass Storage without rebooting, safe eject on Windows, cable disconnection during copying, and reopening the card in `Files`.

A host C++ test with a simulated SD card confirmed that overwrites fail on a full card and that ejection returns an incorrect result. Build and on-board verification remained pending. Local PlatformIO failed at startup because the `urllib3` package was missing. The audit left the firmware sources unchanged.
