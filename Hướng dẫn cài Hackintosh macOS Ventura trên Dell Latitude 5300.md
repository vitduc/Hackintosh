# Hướng dẫn cài Hackintosh macOS Ventura trên Dell Latitude 5300

> **Dành cho:** Dell Latitude 5300 (clamshell, không cảm ứng) · macOS Ventura 13 · OpenCore 1.0.7  
> **Yêu cầu:** MacBook (để tạo USB) · USB 16GB+ · Cáp LAN (WiFi Qualcomm không hỗ trợ)

---

## Phần cứng hỗ trợ

| Thành phần | Trạng thái |
|---|---|
| CPU Intel i5-8365U (Whiskey Lake) | ✅ Hoạt động |
| GPU Intel UHD 620 | ✅ Hoạt động |
| Audio Realtek ALC256 | ✅ Hoạt động (alcid=13) |
| Ethernet Intel I219-LM | ✅ Hoạt động |
| Trackpad / Bàn phím | ✅ Hoạt động |
| Pin / Battery | ✅ Hoạt động |
| WiFi Qualcomm | ❌ Không hỗ trợ — cần thay card |
| WiFi Intel 9260 / AX200 | ✅ Hỗ trợ qua itlwm + Heliport |

---

## Chuẩn bị — Tải công cụ trên MacBook

Mở Terminal (Cmd+Space → gõ Terminal):

```bash
cd ~/Desktop

# Tải công cụ
git clone https://github.com/corpnewt/MountEFI
git clone https://github.com/corpnewt/GenSMBIOS
git clone https://github.com/corpnewt/ProperTree

# Tải EFI gốc cho Dell 5300
git clone https://github.com/ahianf/dell-latitude-5300-hackintosh

# Tải OpenCore mới nhất
curl -L https://github.com/acidanthera/OpenCorePkg/releases/download/1.0.7/OpenCore-1.0.7-RELEASE.zip -o OC.zip
unzip -q OC.zip -d OpenCore-1.0.7-RELEASE
```

Kiểm tra:
```bash
ls ~/Desktop/
# Phải thấy: MountEFI, GenSMBIOS, ProperTree, dell-latitude-5300-hackintosh, OpenCore-1.0.7-RELEASE
```

---

## Phần 1 — Tạo USB cài đặt trên MacBook

### Bước 1: Tải macOS Ventura

Mở **App Store** → tìm **macOS Ventura** → Tải về (~12GB).

### Bước 2: Format USB đúng cách

Cắm USB vào MacBook. Mở **Disk Utility**:

1. Menu **View → Show All Devices** (bắt buộc)
2. Chọn **thiết bị USB cấp trên cùng** (tên hãng USB, VD: "Kingston DataTraveler") — **không phải partition bên dưới**
3. Nhấn **Erase**:
   - Name: `MyVolume`
   - Format: **Mac OS Extended (Journaled)**
   - Scheme: **GUID Partition Map** ← quan trọng, nếu chọn sai sẽ không có EFI partition
4. Erase → Done

> ⚠️ **Lỗi thường gặp:** Nếu không thấy tùy chọn "Scheme" → bạn đang chọn partition, không phải thiết bị. Quay lại bước 1.

### Bước 3: Ghi bộ cài vào USB

```bash
sudo /Applications/Install\ macOS\ Ventura.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume
```

Nhập mật khẩu → gõ `y` → chờ 15–25 phút đến khi thấy:
```
Copy complete. Done.
```

---

## Phần 2 — Chuẩn bị EFI

### Bước 4: Update OpenCore core files

```bash
# Thay file OpenCore mới vào EFI gốc
cp ~/Desktop/OpenCore-1.0.7-RELEASE/X64/EFI/BOOT/BOOTx64.efi \
   ~/Desktop/dell-latitude-5300-hackintosh/EFI/BOOT/BOOTx64.efi

cp ~/Desktop/OpenCore-1.0.7-RELEASE/X64/EFI/OC/OpenCore.efi \
   ~/Desktop/dell-latitude-5300-hackintosh/EFI/OC/OpenCore.efi

cp ~/Desktop/OpenCore-1.0.7-RELEASE/X64/EFI/OC/Drivers/OpenRuntime.efi \
   ~/Desktop/dell-latitude-5300-hackintosh/EFI/OC/Drivers/OpenRuntime.efi
```

### Bước 5: Tải và cập nhật Kexts

```bash
cd ~/Desktop

curl -sL https://github.com/acidanthera/Lilu/releases/download/1.6.8/Lilu-1.6.8-RELEASE.zip -o Lilu.zip && unzip -q Lilu.zip -d Lilu_new
curl -sL https://github.com/acidanthera/WhateverGreen/releases/download/1.6.7/WhateverGreen-1.6.7-RELEASE.zip -o WEG.zip && unzip -q WEG.zip -d WEG_new
curl -sL https://github.com/acidanthera/AppleALC/releases/download/1.9.1/AppleALC-1.9.1-RELEASE.zip -o ALC.zip && unzip -q ALC.zip -d ALC_new
curl -sL https://github.com/acidanthera/VirtualSMC/releases/download/1.3.3/VirtualSMC-1.3.3-RELEASE.zip -o VSMC.zip && unzip -q VSMC.zip -d VSMC_new
curl -sL https://github.com/acidanthera/IntelMausi/releases/download/1.0.7/IntelMausi-1.0.7-RELEASE.zip -o Mausi.zip && unzip -q Mausi.zip -d Mausi_new
curl -sL https://github.com/acidanthera/VoodooPS2/releases/download/2.3.5/VoodooPS2Controller-2.3.5-RELEASE.zip -o VPS2.zip && unzip -q VPS2.zip -d VPS2_new
```

Copy kexts mới vào EFI và xóa kexts WiFi Broadcom không cần thiết:

```bash
KEXTS=~/Desktop/dell-latitude-5300-hackintosh/EFI/OC/Kexts

# Xóa Brcm kexts (WiFi Broadcom — không có trên máy này)
rm -rf $KEXTS/AirportBrcmFixup.kext
rm -rf $KEXTS/BrcmBluetoothInjector.kext
rm -rf $KEXTS/BrcmFirmwareData.kext
rm -rf $KEXTS/BrcmPatchRAM3.kext

# Copy kexts mới
cp -rf ~/Desktop/Lilu_new/Lilu.kext $KEXTS/
cp -rf ~/Desktop/WEG_new/WhateverGreen.kext $KEXTS/
cp -rf ~/Desktop/ALC_new/AppleALC.kext $KEXTS/
cp -rf ~/Desktop/VSMC_new/Kexts/VirtualSMC.kext $KEXTS/
cp -rf ~/Desktop/VSMC_new/Kexts/SMCBatteryManager.kext $KEXTS/
cp -rf ~/Desktop/VSMC_new/Kexts/SMCProcessor.kext $KEXTS/
cp -rf ~/Desktop/VSMC_new/Kexts/SMCSuperIO.kext $KEXTS/
cp -rf ~/Desktop/VSMC_new/Kexts/SMCDellSensors.kext $KEXTS/
cp -rf ~/Desktop/Mausi_new/IntelMausi.kext $KEXTS/
cp -rf ~/Desktop/VPS2_new/VoodooPS2Controller.kext $KEXTS/
```

### Bước 6: Tạo SMBIOS

```bash
cd ~/Desktop/GenSMBIOS
bash GenSMBIOS.command
```

Khi menu hiện:
1. Gõ **1** → Enter (tải MacSerial)
2. Gõ **3** → Enter
3. Gõ `MacBookPro15,2` → Enter

Ghi lại output:
```
Type:         MacBookPro15,2
Serial:       C0XXXXXXXXX
Board Serial: C0XXXXXXXXX
SmUUID:       XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
Apple ROM:    XXXXXXXXXXXX
```

> ⚠️ **Kiểm tra serial:** Vào https://checkcoverage.apple.com nhập serial vừa tạo. Phải thấy **"Số sê-ri không hợp lệ"**. Nếu thấy thông tin máy Mac thật → chạy lại GenSMBIOS.

### Bước 7: Cấu hình config.plist

Thay `YOUR_*` bằng giá trị thật từ GenSMBIOS:

```bash
PLIST=~/Desktop/dell-latitude-5300-hackintosh/EFI/OC/config.plist

# SMBIOS
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemProductName MacBookPro15,2" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemSerialNumber YOUR_SERIAL" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:MLB YOUR_BOARD_SERIAL" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemUUID YOUR_SMUUID" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:ROM YOUR_APPLE_ROM" $PLIST

# Kernel Quirks
/usr/libexec/PlistBuddy -c "Set :Kernel:Quirks:DisableIoMapper true" $PLIST
/usr/libexec/PlistBuddy -c "Set :Kernel:Quirks:PanicNoKextDump true" $PLIST
/usr/libexec/PlistBuddy -c "Set :Kernel:Quirks:PowerTimeoutKernelPanic true" $PLIST
/usr/libexec/PlistBuddy -c "Set :Kernel:Quirks:XhciPortLimit false" $PLIST

# Security
/usr/libexec/PlistBuddy -c "Set :Misc:Security:SecureBootModel Disabled" $PLIST
/usr/libexec/PlistBuddy -c "Set :Misc:Security:Vault Optional" $PLIST

# Boot args
/usr/libexec/PlistBuddy -c "Set :NVRAM:Add:7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args -v keepsyms=1 debug=0x100 alcid=13" $PLIST
```

Xóa Brcm khỏi config:

```bash
python3 - <<'EOF'
import plistlib
import os

# Thay đường dẫn nếu username khác
path = os.path.expanduser("~/Desktop/dell-latitude-5300-hackintosh/EFI/OC/config.plist")

with open(path, "rb") as f:
    plist = plistlib.load(f)

brcm = ["AirportBrcmFixup", "BrcmBluetoothInjector", "BrcmFirmwareData", "BrcmPatchRAM3"]
kexts = plist["Kernel"]["Add"]
plist["Kernel"]["Add"] = [k for k in kexts if not any(b in k.get("BundlePath", "") for b in brcm)]
removed = len(kexts) - len(plist["Kernel"]["Add"])
print(f"Đã xóa {removed} entries Brcm")

with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Lưu xong!")
EOF
```

### Bước 8: Copy EFI vào USB

```bash
# Mount EFI partition của USB
cd ~/Desktop/MountEFI
bash MountEFI.command
# Chọn số tương ứng với "Install macOS Ventura" → Enter → Q để thoát
```

```bash
# Copy EFI vào USB
cp -rf ~/Desktop/dell-latitude-5300-hackintosh/EFI /Volumes/EFI/

# Kiểm tra
ls /Volumes/EFI/EFI/
# Phải thấy: BOOT  OC
```

---

## Phần 3 — Thiết lập BIOS Dell

Khởi động Dell, nhấn **F2** liên tục để vào BIOS:

| Mục | Giá trị |
|---|---|
| General → Secure Boot | **Disabled** |
| System Configuration → SATA Operation | **AHCI** |
| Virtualization Support → VT-d | **Disabled** |
| Intel Software Guard Extensions → SGX | **Disabled** |
| General → Advanced Boot Options → Legacy Option ROMs | **Disabled** |

Lưu **F10** → Yes.

---

## Phần 4 — Cài đặt macOS

### Bước 9: Boot từ USB

Cắm USB vào Dell, bật máy, nhấn **F12** → chọn USB từ danh sách boot.

Trong menu OpenCore chọn **Install macOS Ventura (external)**.

Chờ 2–5 phút qua màn hình verbose (chữ trắng nền đen).

### Bước 10: Format ổ cứng

Khi vào màn hình Recovery:

1. Chọn **Disk Utility** → Continue
2. Menu **View → Show All Devices**
3. Chọn ổ cứng NVMe cấp trên cùng (VD: "KBG40ZNS256G NVMe Toshiba")
4. Nhấn **Erase**:
   - Name: `Macintosh HD`
   - Format: **APFS**
   - Scheme: **GUID Partition Map**
5. Erase → Done → Đóng Disk Utility

### Bước 11: Cài macOS

1. Chọn **Install macOS Ventura** → Continue → Agree
2. Chọn ổ **Macintosh HD** → Continue
3. Máy sẽ **tự restart 3–4 lần** — mỗi lần restart nhấn F12, boot từ USB, chọn **macOS Installer** hoặc **Macintosh HD** trong OpenCore
4. Tổng thời gian: **30–45 phút**

### Bước 12: Thiết lập ban đầu

- Chọn **Vietnam** làm quốc gia
- **Wi-Fi:** chọn "Continue without Wi-Fi"
- **Apple ID:** chọn "Set Up Later" (thiết lập sau)
- Tạo tài khoản user và mật khẩu

---

## Phần 5 — Sau cài đặt

### Bước 13: Copy EFI vào ổ cứng (boot không cần USB)

Sau khi vào Desktop, mở Terminal:

```bash
# Tải MountEFI
cd ~/Desktop
curl -L https://github.com/corpnewt/MountEFI/archive/refs/heads/update.zip -o MountEFI.zip
unzip -q MountEFI.zip

# Tải EFI đã chuẩn bị
curl -L https://github.com/ahianf/dell-latitude-5300-hackintosh/archive/refs/heads/main.zip -o efi.zip
unzip -q efi.zip
```

Lặp lại **Bước 5, 6, 7** (update kexts + SMBIOS + config) cho EFI mới tải.

Sau đó:

```bash
# Mount EFI ổ cứng
sudo diskutil mount disk0s1

# Copy EFI vào ổ cứng
cp -rf ~/Desktop/dell-latitude-5300-hackintosh-main/EFI /Volumes/EFI/

# Kiểm tra
ls /Volumes/EFI/EFI/
# Phải thấy: BOOT  OC
```

### Bước 14: Set boot mặc định trong BIOS

Khởi động lại, nhấn **F2** vào BIOS → **General → Boot Sequence**:

1. Nhấn **Add Boot Option**
2. Boot Option Name: `OpenCore`
3. File System List: chọn ổ NVMe Toshiba
4. File Name: `\EFI\OC\OpenCore.efi`
5. OK
6. Kéo **OpenCore** lên đầu danh sách bằng mũi tên
7. **Apply** → **Exit**

Rút USB — máy sẽ tự boot vào macOS.

---

## Phần 6 — Tối ưu sau cài đặt

### Fix Sleep

```bash
sudo pmset -a hibernatemode 0
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0
sudo pmset -a powernap 0
```

### Fix màn hình mờ

```bash
defaults write -g CGFontRenderingFontSmoothingDisabled -bool NO
```

Restart để có hiệu lực.

### Thêm WiFi (tuỳ chọn)

Thay card M.2 2230 bên trong máy:

| Card | Giá | Hỗ trợ macOS |
|---|---|---|
| Intel 9260 | ~150k | itlwm + Heliport |
| Intel AX200 | ~200k | itlwm + Heliport |
| Broadcom BCM94360NG | ~500k | Native, AirDrop/Handoff đầy đủ |

Với Intel: tải `itlwm.kext` từ https://github.com/OpenIntelWireless/itlwm/releases và copy vào `EFI/OC/Kexts/`, sau đó tải app **Heliport** để kết nối WiFi.

---

## Xử lý lỗi thường gặp

**Không thấy menu OpenCore khi boot từ USB:**
→ USB format sai scheme (FDisk thay vì GUID). Format lại từ đầu với GUID Partition Map.

**Máy treo ở verbose log, Caps Lock không sáng:**
→ Kernel panic. Thử thêm `npci=0x2000` vào boot-args:
```bash
/usr/libexec/PlistBuddy -c "Set :NVRAM:Add:7C436110-AB2A-4BBB-A880-FE41995C9F82:boot-args -v keepsyms=1 debug=0x100 alcid=13 npci=0x2000" /Volumes/EFI/EFI/OC/config.plist
```

**Không có âm thanh:**
→ Thử lần lượt các giá trị alcid: `11 → 13 → 21 → 28 → 32 → 57` trong boot-args.

**Boot vào PXE network (không tìm thấy OS):**
→ Vào BIOS F2 → Boot Sequence → Add Boot Option → chọn `\EFI\OC\OpenCore.efi` trên ổ NVMe.

---

## Tài liệu tham khảo

- OpenCore Install Guide: https://dortania.github.io/OpenCore-Install-Guide
- EFI gốc Dell 5300: https://github.com/ahianf/dell-latitude-5300-hackintosh
- Cộng đồng Việt: https://vnohackintosh.com
