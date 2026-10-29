# PSMoveServiceEx (Arch Linux PKGBUILD)

Arch Linux PKGBUILD for modernized [PSMoveServiceEx](https://github.com/ichikand/PSMoveServiceEx) running on branch `linux-cxx17-modernization`.

Designed for PCs and tracking servers using Sony PS3 Eye cameras and PS Move / Navigation controllers.

---

## Features & Modernizations

- **Modern Toolchains:** C++17 support, Clang/LLVM compatibility, and OpenCV 4/5 integration.
- **Native BlueZ Pairing:** Direct Bluetooth pairing for controllers without manual PIN hacks.
- **PS3 Eye libusb Direct Access:** Automatic detachment of conflicting `gspca_ov534` kernel drivers.
- **System Integration:** Sandboxed systemd service running under system user `psmovecam` (`CAP_NET_ADMIN`, `CAP_NET_RAW`) with preconfigured udev rules.

---

## Installation

```bash
git clone [https://github.com/ichikand/psmoveserviceex-git.git](https://github.com/ichikand/psmoveserviceex-git.git)
cd psmoveserviceex-git
makepkg -srci
```

---

## Usage

**Start as a system service:**
```bash
sudo systemctl enable --now psmoveservice.service
```

**Run manually in terminal:**
```bash
sudo -u psmovecam -H PSMoveService
```

**Launch configuration tool:**
```bash
PSMoveConfigTool
```