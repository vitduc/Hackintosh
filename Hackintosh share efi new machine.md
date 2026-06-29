# Hướng dẫn Share EFI & Cài Hackintosh trên Máy Mới

> **Áp dụng cho:** Dell Latitude 5300 · i5-8365U · Card WiFi Intel 8265  
> **Yêu cầu:** Máy nguồn đang chạy macOS Ventura, máy đích cùng model/cấu hình  
> **Lưu ý quan trọng:** Dù cùng model, mỗi máy **bắt buộc phải có SMBIOS riêng** — dùng chung serial number sẽ gây lỗi iCloud, iMessage, App Store.

---

## Tổng quan quy trình

```
Máy nguồn                          Máy đích
──────────                         ────────
1. Export EFI ra USB          →    4. Tạo USB cài đặt
2. Tạo SMBIOS mới cho máy đích     5. Copy EFI vào USB
3. Lưu EFI đã chỉnh vào USB   →    6. Cài macOS
                                   7. Copy EFI vào ổ cứng
```

---

## Phần 1 — Chuẩn bị EFI trên máy nguồn

### Bước 1: Export EFI từ máy nguồn

Trên máy đang chạy macOS (máy nguồn), mở Terminal:

```bash
# Mount EFI ổ cứng
sudo diskutil mount disk0s1

# Copy EFI ra Desktop
cp -rf /Volumes/EFI/EFI ~/Desktop/EFI_backup
```

Kiểm tra:
```bash
ls ~/Desktop/EFI_backup/
# Phải thấy: BOOT  OC
```

### Bước 2: Tạo SMBIOS mới cho máy đích

> ⚠️ **Bắt buộc phải tạo SMBIOS mới** — không được dùng chung serial với máy nguồn. Dùng chung sẽ gây:
> - Không đăng nhập được iCloud/iMessage
> - Bị Apple khóa tài khoản
> - Lỗi activation

```bash
cd ~/Desktop/GenSMBIOS
bash GenSMBIOS.command
```

Nếu chưa có GenSMBIOS:
```bash
cd ~/Desktop
curl -L https://github.com/corpnewt/GenSMBIOS/archive/refs/heads/master.zip -o GenSMBIOS.zip
unzip -q GenSMBIOS.zip
bash GenSMBIOS-master/GenSMBIOS.command
```

Trong menu:
1. Gõ **1** → Enter (tải MacSerial)
2. Gõ **3** → Enter
3. Gõ `MacBookPro15,2` → Enter

Ghi lại output — đây là SMBIOS cho **máy đích**:
```
Type:         MacBookPro15,2
Serial:       C0XXXXXXXXX       ← ghi lại
Board Serial: C0XXXXXXXXX       ← ghi lại
SmUUID:       XXXXXXXX-XXXX-... ← ghi lại
Apple ROM:    XXXXXXXXXXXX      ← ghi lại
```

**Kiểm tra serial mới:** Vào https://checkcoverage.apple.com → nhập serial vừa tạo.  
Phải thấy **"Số sê-ri không hợp lệ"** → được dùng.  
Nếu thấy thông tin máy Mac thật → chạy lại GenSMBIOS.

### Bước 3: Điền SMBIOS mới vào EFI bản copy

```bash
PLIST=~/Desktop/EFI_backup/OC/config.plist

# Thay bằng giá trị thật từ GenSMBIOS ở bước 2
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemProductName MacBookPro15,2" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemSerialNumber YOUR_SERIAL" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:MLB YOUR_BOARD_SERIAL" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemUUID YOUR_SMUUID" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:ROM YOUR_APPLE_ROM" $PLIST
```

Xác nhận đã điền đúng:
```bash
/usr/libexec/PlistBuddy -c "Print :PlatformInfo:Generic" $PLIST
# Kiểm tra SystemSerialNumber khác với máy nguồn
```

---

## Phần 2 — Tạo USB cài đặt

### Bước 4: Chuẩn bị USB

Cắm USB 16GB+ vào MacBook. Mở **Disk Utility**:

1. Menu **View → Show All Devices**
2. Chọn **thiết bị USB cấp trên cùng** (tên hãng, không phải partition)
3. Nhấn **Erase**:
   - Name: `MyVolume`
   - Format: **Mac OS Extended (Journaled)**
   - Scheme: **GUID Partition Map**
4. Erase → Done

> ⚠️ **Phải chọn đúng thiết bị cấp trên** — nếu không thấy mục "Scheme" nghĩa là đang chọn partition, không phải thiết bị.

### Bước 5: Ghi bộ cài macOS vào USB

```bash
sudo /Applications/Install\ macOS\ Ventura.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume
```

Nhập mật khẩu → gõ `y` → chờ 15–25 phút đến khi thấy `Copy complete. Done.`

> Nếu chưa có installer: Mở App Store → tìm **macOS Ventura** → Tải về.

### Bước 6: Copy EFI vào USB

```bash
# Mount EFI của USB
cd ~/Desktop/MountEFI
bash MountEFI.command
```

Nếu chưa có MountEFI:
```bash
cd ~/Desktop
curl -L https://github.com/corpnewt/MountEFI/archive/refs/heads/update.zip -o MountEFI.zip
unzip -q MountEFI.zip
bash MountEFI-update/MountEFI.command
```

Khi menu hiện → chọn số tương ứng **"Install macOS Ventura"** (USB) → Enter → Q thoát.

Copy EFI đã chỉnh SMBIOS vào USB:
```bash
cp -rf ~/Desktop/EFI_backup /Volumes/EFI/EFI
```

Kiểm tra:
```bash
ls /Volumes/EFI/EFI/
# Phải thấy: BOOT  OC
```

Kiểm tra SMBIOS trong USB đã đúng chưa:
```bash
/usr/libexec/PlistBuddy -c "Print :PlatformInfo:Generic:SystemSerialNumber" /Volumes/EFI/EFI/OC/config.plist
# Phải hiện serial của máy đích, không phải máy nguồn
```

---

## Phần 3 — Cài macOS trên máy đích

### Bước 7: Thiết lập BIOS máy đích

Cắm USB vào máy đích, bật máy, nhấn **F2** liên tục vào BIOS:

| Mục | Giá trị |
|---|---|
| General → Secure Boot | **Disabled** |
| System Configuration → SATA Operation | **AHCI** |
| Virtualization Support → VT-d | **Disabled** |
| Intel Software Guard Extensions → SGX | **Disabled** |
| General → Advanced Boot Options → Legacy Option ROMs | **Disabled** |

Lưu **F10** → Yes.

### Bước 8: Boot từ USB

Bật máy đích, nhấn **F12** → chọn USB.

Trong OpenCore menu chọn **Install macOS Ventura (external)** → chờ 2–5 phút.

### Bước 9: Format ổ cứng máy đích

Khi vào Recovery:

1. Chọn **Disk Utility** → Continue
2. **View → Show All Devices**
3. Chọn ổ NVMe cấp trên cùng
4. Erase:
   - Name: `Macintosh HD`
   - Format: **APFS**
   - Scheme: **GUID Partition Map**
5. Erase → Done → Đóng Disk Utility

### Bước 10: Cài macOS

1. Chọn **Install macOS Ventura** → Continue → Agree
2. Chọn **Macintosh HD** → Continue
3. Máy tự restart **3–4 lần** — mỗi lần nhấn F12, boot USB, chọn **macOS Installer** hoặc **Macintosh HD**
4. Chờ **30–45 phút**

### Bước 11: Thiết lập ban đầu

- Chọn **Vietnam**
- **Wi-Fi:** "Continue without Wi-Fi" (dùng LAN trước)
- **Apple ID:** "Set Up Later"
- Tạo tài khoản và mật khẩu

---

## Phần 4 — Chuyển EFI vào ổ cứng máy đích

### Bước 12: Copy EFI từ USB sang ổ cứng

Sau khi vào Desktop, mở Terminal:

```bash
# Kiểm tra disk identifier
diskutil list | grep EFI
# Thường: disk0s1 = EFI của USB, disk2s1 = EFI của ổ cứng
```

Mount cả hai:
```bash
sudo diskutil mount disk0s1   # EFI USB
sudo diskutil mount disk2s1   # EFI ổ cứng
```

Xác nhận:
```bash
ls /Volumes/
# Phải thấy: EFI  EFI 1  Macintosh HD  Install macOS Ventura
```

Copy EFI:
```bash
cp -rf /Volumes/EFI/EFI "/Volumes/EFI 1/"

ls "/Volumes/EFI 1/EFI/"
# Phải thấy: BOOT  OC
```

### Bước 13: Set boot mặc định trong BIOS

Khởi động lại, nhấn **F2** → **General → Boot Sequence**:

1. **Add Boot Option**
2. Name: `OpenCore`
3. File System: chọn ổ NVMe
4. File Name: `\EFI\OC\OpenCore.efi`
5. OK → kéo **OpenCore** lên đầu danh sách
6. **Apply** → **Exit**

Rút USB — máy tự boot vào macOS.

---

## Phần 5 — Cài WiFi & Bluetooth (nếu có card Intel 8265)

> Xem hướng dẫn chi tiết trong file `hackintosh-wifi-bluetooth-intel.md`

### WiFi — tóm tắt nhanh

```bash
# Tải AirportItlwm từ Safari (không dùng curl)
# https://github.com/OpenIntelWireless/itlwm/releases
# File: AirportItlwm_v2.3.0_stable_Ventura.kext.zip

sudo diskutil mount disk0s1
cp -rf ~/Downloads/AirportItlwm.kext /Volumes/EFI/EFI/OC/Kexts/

python3 - <<'EOF'
import plistlib
path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if "AirportItlwm" not in k.get("BundlePath", "")]
plist["Kernel"]["Add"].append({
    "Arch": "x86_64",
    "BundlePath": "AirportItlwm.kext",
    "Comment": "Intel WiFi 8265 Ventura",
    "Enabled": True,
    "ExecutablePath": "Contents/MacOS/AirportItlwm",
    "MaxKernel": "22.9.9",
    "MinKernel": "22.0.0",
    "PlistPath": "Contents/Info.plist"
})
with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Xong!")
EOF

sudo reboot
```

### Bluetooth — tóm tắt nhanh

```bash
# Tải IntelBluetooth v2.3.0 từ Safari
# https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases

# Tải BlueToolFixup v2.6.8 bằng curl
cd ~/Downloads
curl -L "https://github.com/acidanthera/BrcmPatchRAM/releases/download/2.6.8/BrcmPatchRAM-2.6.8-RELEASE.zip" -o BrcmPatchRAM268.zip
unzip -q BrcmPatchRAM268.zip -d BrcmPatchRAM268

sudo diskutil mount disk0s1

KEXTS=/Volumes/EFI/EFI/OC/Kexts
cp -rf ~/Downloads/IntelBluetooth-v2.3.0/IntelBluetoothFirmware.kext $KEXTS/
cp -rf ~/Downloads/BrcmPatchRAM268/BlueToolFixup.kext $KEXTS/

python3 - <<'EOF'
import plistlib
path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)
remove = ["IntelBluetoothFirmware", "IntelBluetoothInjector", "IntelBTPatcher", "BlueToolFixup"]
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if not any(x in k.get("BundlePath", "") for x in remove)]
plist["Kernel"]["Add"].extend([
    {
        "Arch": "x86_64",
        "BundlePath": "IntelBluetoothFirmware.kext",
        "Comment": "Intel BT Firmware 8265 v2.3.0",
        "Enabled": True,
        "ExecutablePath": "Contents/MacOS/IntelBluetoothFirmware",
        "MaxKernel": "",
        "MinKernel": "",
        "PlistPath": "Contents/Info.plist"
    },
    {
        "Arch": "x86_64",
        "BundlePath": "BlueToolFixup.kext",
        "Comment": "BT Fix Ventura - BrcmPatchRAM 2.6.8",
        "Enabled": True,
        "ExecutablePath": "Contents/MacOS/BlueToolFixup",
        "MaxKernel": "",
        "MinKernel": "21.0.0",
        "PlistPath": "Contents/Info.plist"
    }
])
with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Đã thêm 2 BT kexts!")
EOF

sudo reboot
```

---

## Lưu ý quan trọng khi share EFI

| Điều cần làm | Lý do |
|---|---|
| **Tạo SMBIOS mới cho mỗi máy** | Dùng chung serial → lỗi iCloud/iMessage |
| **Kiểm tra serial trên checkcoverage.apple.com** | Đảm bảo serial không trùng máy Mac thật |
| **Không chia sẻ file EFI có chứa SMBIOS của bạn** | Lộ serial → ảnh hưởng tài khoản Apple |
| **Xóa SMBIOS trước khi share** | Người nhận tự tạo SMBIOS của họ |

### Cách tạo EFI sạch để share (không có SMBIOS)

Nếu muốn share EFI cho nhiều người, xóa SMBIOS trước:

```bash
PLIST=~/Desktop/EFI_backup/OC/config.plist

/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemSerialNumber CHANGEME" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:MLB CHANGEME" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:SystemUUID 00000000-0000-0000-0000-000000000000" $PLIST
/usr/libexec/PlistBuddy -c "Set :PlatformInfo:Generic:ROM 000000000000" $PLIST
```

Người nhận EFI này sẽ tự chạy GenSMBIOS và điền SMBIOS của họ vào.

---

## Checklist trước khi boot máy đích

- [ ] USB format đúng: GUID Partition Map (không phải FDisk)
- [ ] EFI có trong USB: `ls /Volumes/EFI/EFI/` thấy BOOT và OC
- [ ] SMBIOS trong USB là serial mới (không phải serial máy nguồn)
- [ ] BIOS: Secure Boot = Disabled, SATA = AHCI, VT-d = Disabled
- [ ] Ổ cứng đã format APFS + GUID trước khi cài

---

## Xử lý lỗi thường gặp

**USB boot vào PXE network:**
→ USB format sai (FDisk). Format lại với GUID Partition Map.

**OpenCore menu hiện nhưng boot bị treo:**
→ Reset NVRAM: trong OpenCore chọn "Reset NVRAM" → boot lại.

**Sau cài xong, rút USB không boot được:**
→ Vào BIOS F2 → Boot Sequence → Add Boot Option → `\EFI\OC\OpenCore.efi` trên ổ NVMe → kéo lên đầu.

**iCloud/iMessage không đăng nhập được:**
→ SMBIOS bị trùng với máy nguồn. Tạo SMBIOS mới và cập nhật vào EFI ổ cứng.
