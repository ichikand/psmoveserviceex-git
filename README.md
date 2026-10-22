# PSMoveServiceEx (Arch Linux PKGBUILD)

Arch Linux PKGBUILD for [PSMoveServiceEx](https://github.com/Timocop/PSMoveServiceEx), patched for modern Linux environments (C++17, OpenCV 4/5, LLVM/Clang, BlueZ 5, and systemd).

Designed for headless Linux servers or standard PC running PS3 Eye cameras.

---

## Check latest branch with test working bluez controller pairing!

---

## Patches & Fixes

* **Linux PS3 Eye Driver:** Re-enabled `HAVE_PS3EYE` and `PS3EYEDriver` build paths previously restricted to Windows/macOS.
* **Kernel Driver Detach:** Automated `gspca_ov534` kernel module detachment via `libusb` to prevent `EBUSY` conflicts.
* **OpenCV 4/5 Compatibility:** Resolved missing enumerators (`COLOR_BayerGB2BGR`, custom `cv::CAP_PROP_*` definitions) and updated `getFrame()` signatures.
* **Modern Boost:** Updated deprecated `boost::asio::io_service` references to `io_context` and standardized placeholder usage.
* **Asset Paths:** Bundled `assets/` into `/usr/share/psmoveconfigtool` with a launcher wrapper to fix missing fonts and textures in `PSMoveConfigTool`.

---

## Build & Install

```bash
git clone <repo-url>
cd <repo-dir>
makepkg -srci
```