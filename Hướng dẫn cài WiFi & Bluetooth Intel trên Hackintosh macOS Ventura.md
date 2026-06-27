# Hướng dẫn cài WiFi & Bluetooth Intel trên Hackintosh macOS Ventura

> **Áp dụng cho:** Dell Latitude 5300 · macOS Ventura 13.7.x · Card Intel 8265NGW  
> **Yêu cầu:** Đã cài macOS thành công, đang boot từ ổ cứng, có kết nối LAN  
> **Ghi chú:** Tài liệu dựa trên quá trình thực tế, bao gồm tất cả lỗi đã gặp và cách debug.

---

## Tổng quan

| Thành phần | Kext cần dùng | Kết quả |
|---|---|---|
| WiFi Intel 8265 | AirportItlwm v2.3.0 (Ventura) | ✅ Native, không cần app |
| Bluetooth Intel 8265 | IntelBluetoothFirmware v2.3.0 + BlueToolFixup v2.6.8 | ✅ Hoạt động đầy đủ |

---

## Phần 1 — Cài WiFi

### Bước 1: Tải kext WiFi

> ⚠️ **Không dùng `curl` để tải từ GitHub** — sẽ chỉ nhận được file 9 bytes do bị redirect. Phải tải thủ công trên Safari.

1. Mở Safari → vào: **https://github.com/OpenIntelWireless/itlwm/releases**
2. Tìm phiên bản **v2.3.0** → mở rộng mục **Assets**
3. Tải đúng file: **`AirportItlwm_v2.3.0_stable_Ventura.kext.zip`**

> ⚠️ **Chọn đúng file theo phiên bản macOS:**
> - Ventura (13.x) → `AirportItlwm_v2.3.0_stable_Ventura.kext.zip`
> - Sonoma (14.x) → `AirportItlwm_v2.3.0_stable_Sonoma14.x.kext.zip`
> - Monterey (12.x) → `AirportItlwm_v2.3.0_stable_Monterey.kext.zip`
>
> Dùng sai file → **kernel panic** ngay khi boot.

> ⚠️ **Không dùng `itlwm.kext`** — cần thêm app Heliport mới kết nối được. Dùng `AirportItlwm.kext` để WiFi hiện native trên menu bar như Mac thật.

Giải nén → sẽ thấy `AirportItlwm.kext` trong Downloads.

### Bước 2: Kiểm tra file tải đúng chưa

```bash
ls ~/Downloads/ | grep -i airport
# Phải thấy: AirportItlwm.kext
```

Kiểm tra kext hợp lệ:
```bash
ls ~/Downloads/AirportItlwm.kext/Contents/MacOS/
# Phải thấy: AirportItlwm
```

Nếu thư mục `Contents/MacOS/` trống hoặc không tồn tại → file tải bị lỗi, tải lại.

### Bước 3: Copy kext vào EFI

```bash
sudo diskutil mount disk0s1
```

Kiểm tra mount thành công:
```bash
ls /Volumes/EFI/EFI/
# Phải thấy: BOOT  OC
```

Copy kext:
```bash
cp -rf ~/Downloads/AirportItlwm.kext /Volumes/EFI/EFI/OC/Kexts/
```

Xác nhận:
```bash
ls /Volumes/EFI/EFI/OC/Kexts/ | grep -i airport
# Phải thấy: AirportItlwm.kext
```

### Bước 4: Thêm vào config.plist

```bash
python3 - <<'EOF'
import plistlib

path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)

# Xóa entry cũ nếu có
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if "AirportItlwm" not in k.get("BundlePath", "")]

# Thêm mới
new_kext = {
    "Arch": "x86_64",
    "BundlePath": "AirportItlwm.kext",
    "Comment": "Intel WiFi 8265 - Ventura",
    "Enabled": True,
    "ExecutablePath": "Contents/MacOS/AirportItlwm",
    "MaxKernel": "22.9.9",
    "MinKernel": "22.0.0",
    "PlistPath": "Contents/Info.plist"
}
plist["Kernel"]["Add"].append(new_kext)

with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Đã thêm AirportItlwm.kext!")
EOF
```

Kiểm tra đã thêm thành công:
```bash
/usr/libexec/PlistBuddy -c "Print :Kernel:Add" /Volumes/EFI/EFI/OC/config.plist | grep -i airport
# Phải thấy: BundlePath = AirportItlwm.kext
```

### Bước 5: Restart và kiểm tra

```bash
sudo reboot
```

Sau khi boot xong, kiểm tra WiFi đã hoạt động chưa:

```bash
# Kiểm tra kext đã load chưa
kextstat | grep -i itlwm
# Phải thấy: com.apple.airport.itlwm hoặc tương tự

# Kiểm tra interface WiFi
ifconfig | grep -A5 "en1\|en2\|en3"
# Phải thấy interface với địa chỉ IP nếu đã kết nối
```

---

## Phần 2 — Debug WiFi nếu không hoạt động

### Kiểm tra kext có load không

```bash
kextstat | grep -i itlwm
```

**Nếu không có output:**
- Kext chưa load → kiểm tra lại config.plist có entry AirportItlwm không
- Hoặc MinKernel/MaxKernel sai → kiểm tra kernel version:

```bash
uname -r
# Ventura 13.7.8 → 22.6.0
```

Ventura dùng kernel 22.x → MinKernel phải là `22.0.0`, MaxKernel là `22.9.9`.

### Kiểm tra kext file hợp lệ

```bash
sudo diskutil mount disk0s1
ls /Volumes/EFI/EFI/OC/Kexts/AirportItlwm.kext/Contents/MacOS/
```

Nếu trống → kext bị lỗi, copy lại từ file zip.

### Kiểm tra xung đột với kext khác

```bash
kextstat | grep -i airport
```

Nếu thấy nhiều airport kext đang load cùng lúc → có thể xung đột. Đảm bảo chỉ có một AirportItlwm trong EFI.

### Kernel panic khi boot sau khi thêm AirportItlwm

Nguyên nhân thường gặp:
1. Dùng sai file (Sonoma/Monterey thay vì Ventura)
2. MinKernel/MaxKernel không đúng

**Xử lý:** Cắm USB boot vào macOS, xóa kext:
```bash
sudo diskutil mount disk0s1
rm -rf /Volumes/EFI/EFI/OC/Kexts/AirportItlwm.kext

python3 - <<'EOF'
import plistlib
path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if "AirportItlwm" not in k.get("BundlePath", "")]
with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Đã xóa!")
EOF

sudo reboot
```

Sau khi vào được macOS → tải lại đúng file Ventura và làm lại từ Bước 1.

---

## Phần 3 — Cài Bluetooth

### Tổng quan kexts cần dùng

| Kext | Version | Nguồn | Ghi chú |
|---|---|---|---|
| IntelBluetoothFirmware | v2.3.0 | OpenIntelWireless | Upload firmware vào card |
| BlueToolFixup | v2.6.8 | BrcmPatchRAM 2.6.8 | Redirect BT stack sang Intel |

> ⚠️ **Version BlueToolFixup rất quan trọng:**
> - v2.6.8 → ✅ hoạt động
> - v2.7.2 → ❌ BT không bật được trên Ventura 13.7.x

### Bước 1: Tải IntelBluetoothFirmware v2.3.0

> ⚠️ **Tải thủ công trên Safari** — `curl` từ GitHub bị redirect, chỉ nhận được file rỗng.

1. Mở Safari → vào: **https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases**
2. Tìm **v2.3.0** → tải **`IntelBluetooth-v2.3.0.zip`**
3. Giải nén → có thư mục `IntelBluetooth-v2.3.0/`

> ⚠️ **Không dùng v2.4.0** — gây kernel panic trên một số cấu hình.

Kiểm tra file đã tải đúng:
```bash
ls ~/Downloads/IntelBluetooth-v2.3.0/
# Phải thấy nhiều kext: IntelBluetoothFirmware.kext, IntelBTPatcher.kext, v.v.

ls ~/Downloads/IntelBluetooth-v2.3.0/IntelBluetoothFirmware.kext/Contents/MacOS/
# Phải thấy: IntelBluetoothFirmware
```

### Bước 2: Tải BlueToolFixup v2.6.8

Cái này tải được bằng `curl`:

```bash
cd ~/Downloads
curl -L "https://github.com/acidanthera/BrcmPatchRAM/releases/download/2.6.8/BrcmPatchRAM-2.6.8-RELEASE.zip" -o BrcmPatchRAM268.zip
```

Kiểm tra file hợp lệ:
```bash
file BrcmPatchRAM268.zip
# Phải thấy: Zip archive data — nếu thấy "ASCII text" là bị lỗi redirect, tải lại
```

Giải nén:
```bash
unzip -q BrcmPatchRAM268.zip -d BrcmPatchRAM268
ls BrcmPatchRAM268/
# Phải thấy BlueToolFixup.kext trong danh sách
```

Kiểm tra kext hợp lệ:
```bash
ls ~/Downloads/BrcmPatchRAM268/BlueToolFixup.kext/Contents/MacOS/
# Phải thấy: BlueToolFixup
```

### Bước 3: Copy kexts vào EFI

```bash
sudo diskutil mount disk0s1

KEXTS=/Volumes/EFI/EFI/OC/Kexts

cp -rf ~/Downloads/IntelBluetooth-v2.3.0/IntelBluetoothFirmware.kext $KEXTS/
cp -rf ~/Downloads/BrcmPatchRAM268/BlueToolFixup.kext $KEXTS/
```

Xác nhận:
```bash
ls /Volumes/EFI/EFI/OC/Kexts/ | grep -iE "intel|blue"
# Phải thấy: IntelBluetoothFirmware.kext  BlueToolFixup.kext
```

### Bước 4: Thêm vào config.plist

Thứ tự kext trong config **rất quan trọng** — IntelBluetoothFirmware phải đứng trước BlueToolFixup:

```bash
python3 - <<'EOF'
import plistlib

path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)

# Xóa tất cả BT entry cũ nếu có
remove = ["IntelBluetoothFirmware", "IntelBluetoothInjector", "IntelBTPatcher", "BlueToolFixup"]
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if not any(x in k.get("BundlePath", "") for x in remove)]

# Thêm đúng thứ tự
new_kexts = [
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
]
plist["Kernel"]["Add"].extend(new_kexts)

with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Đã thêm 2 BT kexts!")
EOF
```

Kiểm tra config đã đúng:
```bash
/usr/libexec/PlistBuddy -c "Print :Kernel:Add" /Volumes/EFI/EFI/OC/config.plist | grep -iE "intel|blue"
# Phải thấy: BundlePath = IntelBluetoothFirmware.kext
#            BundlePath = BlueToolFixup.kext
```

### Bước 5: Restart và kiểm tra

```bash
sudo reboot
```

Sau khi boot, kiểm tra từng bước:

**Kiểm tra 1: Kext đã load chưa**
```bash
kextstat | grep -iE "bluetooth|bluetool"
```
Phải thấy:
- `com.zxystd.IntelBluetoothFirmware`
- `as.acidanthera.BlueToolFixup`

**Kiểm tra 2: Card được nhận đúng chưa**
```bash
system_profiler SPBluetoothDataType | head -15
```
Phải thấy:
- Address: **không phải NULL** (có địa chỉ MAC thật)
- Chipset: **không phải BCM** (nếu vẫn thấy BCM → BlueToolFixup chưa load)

**Kiểm tra 3: USB device**
```bash
system_profiler SPUSBDataType | grep -A5 -i bluetooth
```
Phải thấy:
- Vendor ID: `0x8087` (Intel Corporation)
- Product ID: `0x0a2b` (Intel 8265)

---

## Phần 4 — Debug Bluetooth nếu không hoạt động

### BT không bật được trong System Settings

**Kiểm tra BlueToolFixup có load không:**
```bash
kextstat | grep -i bluetool
```

Nếu không có output → BlueToolFixup chưa load. Nguyên nhân có thể:
1. Kext file lỗi → kiểm tra lại `ls BlueToolFixup.kext/Contents/MacOS/`
2. Entry trong config sai → kiểm tra lại bằng PlistBuddy
3. Dùng sai version (2.7.2 thay vì 2.6.8) → tải lại đúng version

**Kiểm tra version BlueToolFixup:**
```bash
kextstat | grep -i bluetool
# Phải thấy version 2.6.8 trong output
```

Nếu thấy version khác → xóa kext cũ, tải lại BrcmPatchRAM 2.6.8.

### BT bật được nhưng không tìm thấy thiết bị

**Kiểm tra firmware có upload vào card không:**
```bash
log show --last 5m | grep -i "bluetooth" | grep -i "firmware"
```

**Kiểm tra địa chỉ MAC:**
```bash
system_profiler SPBluetoothDataType | grep "Address"
```
- Nếu `Address: NULL` → firmware chưa load vào card
- Nếu có địa chỉ MAC thật → firmware đã load, vấn đề khác

**Kiểm tra dây anten đã cắm chưa:**

Card 8265 có 2 đầu anten (MAIN và AUX). Nếu không cắm hoặc cắm ngược → BT/WiFi yếu hoặc không tìm thấy thiết bị. Lật máy, kiểm tra 2 dây anten đã cắm vào đúng cổng MAIN(2) và AUX(1) trên card chưa.

### Kernel panic sau khi thêm BT kexts

Xác định kext nào gây lỗi bằng cách thêm từng cái một:

**Thử 1: Chỉ IntelBluetoothFirmware**
```bash
sudo diskutil mount disk0s1
rm -rf /Volumes/EFI/EFI/OC/Kexts/BlueToolFixup.kext

python3 - <<'EOF'
import plistlib
path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if "BlueToolFixup" not in k.get("BundlePath", "")]
with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Xong!")
EOF
sudo reboot
```

- Nếu boot OK → IntelBluetoothFirmware không gây lỗi, vấn đề ở BlueToolFixup
- Nếu vẫn panic → IntelBluetoothFirmware gây lỗi, thử tải lại v2.3.0

**Thử 2: Chỉ BlueToolFixup**
```bash
sudo diskutil mount disk0s1
rm -rf /Volumes/EFI/EFI/OC/Kexts/IntelBluetoothFirmware.kext
# Thêm lại BlueToolFixup, xóa IntelBluetooth khỏi config
sudo reboot
```

- Nếu boot OK → BlueToolFixup không gây lỗi, vấn đề ở IntelBluetoothFirmware
- Nếu panic → BlueToolFixup gây lỗi, kiểm tra version

### Xóa toàn bộ BT kexts để reset

```bash
sudo diskutil mount disk0s1

rm -rf /Volumes/EFI/EFI/OC/Kexts/IntelBluetoothFirmware.kext
rm -rf /Volumes/EFI/EFI/OC/Kexts/IntelBluetoothInjector.kext
rm -rf /Volumes/EFI/EFI/OC/Kexts/IntelBTPatcher.kext
rm -rf /Volumes/EFI/EFI/OC/Kexts/BlueToolFixup.kext

python3 - <<'EOF'
import plistlib
path = "/Volumes/EFI/EFI/OC/config.plist"
with open(path, "rb") as f:
    plist = plistlib.load(f)
remove = ["IntelBluetoothFirmware", "IntelBluetoothInjector", "IntelBTPatcher", "BlueToolFixup"]
plist["Kernel"]["Add"] = [k for k in plist["Kernel"]["Add"] if not any(x in k.get("BundlePath", "") for x in remove)]
with open(path, "wb") as f:
    plistlib.dump(plist, f)
print("Đã xóa hết BT kexts!")
EOF

sudo reboot
```

---

## Phần 5 — Bảng so sánh kext combinations đã thử

> Bảng này ghi lại kết quả thực tế trên Dell Latitude 5300, macOS Ventura 13.7.8, card Intel 8265NGW.

| Combination | Kết quả | Ghi chú |
|---|---|---|
| IntelBluetoothFirmware v2.4.0 | ❌ Kernel panic | v2.4.0 không tương thích |
| IntelBluetoothFirmware v2.3.0 + IntelBTPatcher + IntelBluetoothInjector | ❌ Kernel panic | Injector không dùng cho Ventura |
| IntelBluetoothFirmware v2.3.0 + IntelBTPatcher + BlueToolFixup v2.7.2 | ❌ BT không bật | v2.7.2 xung đột |
| IntelBluetoothFirmware v2.3.0 + BlueToolFixup v2.7.2 | ❌ BT không bật | v2.7.2 xung đột |
| IntelBluetoothFirmware v2.3.0 (một mình) | ⚠️ BT bật, không thấy thiết bị | Thiếu BlueToolFixup |
| BlueToolFixup v2.7.2 (một mình) | ❌ BT không bật | Thiếu firmware |
| **IntelBluetoothFirmware v2.3.0 + BlueToolFixup v2.6.8** | ✅ **Hoạt động hoàn toàn** | Combination chính xác |

---

## Phần 6 — Lệnh debug tổng hợp

Chạy tất cả kiểm tra cùng lúc:

```bash
echo "=== KEXT STATUS ==="
kextstat | grep -iE "itlwm|bluetooth|bluetool|intel"

echo ""
echo "=== WIFI STATUS ==="
system_profiler SPAirPortDataType 2>/dev/null | grep -A5 "Card Type\|Firmware"

echo ""
echo "=== BLUETOOTH STATUS ==="
system_profiler SPBluetoothDataType | grep -A8 "Bluetooth Controller"

echo ""
echo "=== USB DEVICES ==="
system_profiler SPUSBDataType | grep -B2 -A5 "0x8087"

echo ""
echo "=== KEXT FILES IN EFI ==="
sudo diskutil mount disk0s1 2>/dev/null
ls /Volumes/EFI/EFI/OC/Kexts/ | grep -iE "airport|intel|blue"
```

---

## Tóm tắt nhanh

**Cài WiFi:**
1. Tải `AirportItlwm_v2.3.0_stable_Ventura.kext.zip` từ Safari
2. Copy `AirportItlwm.kext` vào `/Volumes/EFI/EFI/OC/Kexts/`
3. Thêm entry vào config.plist với MinKernel=22.0.0, MaxKernel=22.9.9
4. Reboot

**Cài Bluetooth:**
1. Tải `IntelBluetooth-v2.3.0.zip` từ Safari
2. Tải `BrcmPatchRAM-2.6.8-RELEASE.zip` bằng curl
3. Copy `IntelBluetoothFirmware.kext` và `BlueToolFixup.kext` vào EFI
4. Thêm cả 2 vào config.plist (IntelBluetoothFirmware trước)
5. Reboot

---

## Tài liệu tham khảo

- Intel WiFi kext: https://github.com/OpenIntelWireless/itlwm/releases
- Intel Bluetooth kext: https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases
- BrcmPatchRAM (BlueToolFixup): https://github.com/acidanthera/BrcmPatchRAM/releases
- OpenCore guide: https://dortania.github.io/OpenCore-Install-Guide
