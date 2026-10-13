# AI GENERATED SO I DONT CARE

# ofox_device_xiaomi_Mi8953

Device tree **OrangeFox Recovery** (branch `fox_12.1`) untuk device Xiaomi
berplatform **MSM8953** (unified tree):

| Codename | Device |
|----------|--------|
| `mido`   | Mi Max |
| `vince`  | Redmi 5 Plus / Note 5 (varian india) |
| `ysl`    | Redmi 5 Plus / Note 5 (varian china) |

`TARGET_OTA_ASSERT_DEVICE := mi8953,mido,vince,ysl,Mi8953` - jadi satu tree
ini dipakai untuk seluruh keluarga mi8953, **bukan device tertentu saja**.

---

## Identitas build

- Lunch target: **`ofox_Mi8953-eng`**
- Produk: **`ofox_Mi8953`** (`PRODUCT_NAME := ofox_$(PRODUCT_RELEASE_NAME)`)
- Output:
  - `out/target/product/Mi8953/recovery.img`
  - `out/target/product/Mi8953/OrangeFox-<release>-Unofficial-CI-Mi8953.img`
  - `out/target/product/Mi8953/OrangeFox-<release>-Unofficial-CI-Mi8953.zip` (installer)

> Catatan penamaan: semua berkas/identitas di repo ini sudah memakai prefix
> `ofox_`. Variabel yang namanya masih `TW_*` / path seperti
> `vendor/twrp/config/common.mk` dan `recovery/root/system/etc/twrp.flags`
> **tidak diganti** karena itu nama API/lokasi file yang dibaca source
> recovery (TWRP & OrangeFox) - mengubahnya justru membuat build gagal.
> Var build OrangeFox (`FOX_*` / `OF_*`) ada di [`vendorsetup.sh`](vendorsetup.sh).

## Struktur

```
ofox_Mi8953.mk        # produk/lunch target (ofox_Mi8953)
AndroidProducts.mk    # daftarkan produk + COMMON_LUNCH_CHOICES
BoardConfig.mk        # arsitektur, kernel prebuilt, partisi dinamis (super retrofit)
device.mk             # inherit produk + PRODUCT_USE_DYNAMIC_PARTITIONS
vendorsetup.sh        # VAR BUILD OrangeFox (FOX_*/OF_*) - wajib utk OrangeFox
prebuilt/             # Image.gz-dtb (kernel 4.9 prebuilt)
recovery/root/        # fstab/flags dinamis, init.rc, binary keymaster/gatekeeper, dll
vintf/                # manifest VINTF
.github/workflows/    # workflow CI build OrangeFox dari repo ini
```

## Build di GitHub Actions (disarankan)

Workflow: [`.github/workflows/build-ofox-mi8953.yml`](.github/workflows/build-ofox-mi8953.yml)

Alur (sekali jalan, tanpa clone device tree dari repo lain):

1. Bebas-kan ruang disk + swap (source `fox_12.1` butuh ~45 GB)
2. `actions/checkout` -> **repo ini** jadi device tree
3. Sinkronisasi source OrangeFox pakai script resmi
   [`OrangeFox/sync`](https://gitlab.com/OrangeFox/sync)
   (`orangefox_sync.sh --branch 12.1`): manifest TWRP minimal + patch build
   system + clone [`OrangeFox/bootable/Recovery`](https://gitlab.com/OrangeFox/bootable/Recovery)
   & `OrangeFox/vendor/recovery`
4. Salin isi repo ini ke `device/xiaomi/Mi8953`
5. (opsional) Download Magisk terbaru
6. `. build/envsetup.sh` -> `lunch ofox_Mi8953-eng` -> `mka recoveryimage`
7. Upload artifact `recovery.img` + `OrangeFox-*.zip` + log

Jalankan manual: **Actions -> ofox-mi8953-12.1 -> Run workflow**.

Optional inputs:

| Input | Default | Fungsi |
|-------|---------|--------|
| `maintainer` | `RenzAlt` | `OF_MAINTAINER` yang tampil di recovery |
| `target` | `recoveryimage` | Make target |
| `use_latest_magisk` | `true` | Bundle Magisk terbaru ke zip installer |

Secrets yang dipakai (opsional - kalau kosong, notifikasi dilewati, build
tetap jalan):

- `TG_BOT_TOKEN`
- `TG_CHAT_ID`

## Build lokal

Prasyarat: Linux x64, ~100 GB SSD bebas, 16 GB RAM (recommended), `bash`.

```bash
# 1. tool sync OrangeFox
git clone https://gitlab.com/OrangeFox/sync.git ~/OrangeFox_sync
cd ~/OrangeFox_sync

# 2. sync source (sekali, lama)
./orangefox_sync.sh --branch 12.1 --path ~/fox_12.1

# 3. taruh device tree ini (clone repo ini, branch android-12.1)
git clone https://github.com/<kamu>/ofox_device_xiaomi_Mi8953 \
  -b android-12.1 ~/fox_12.1/device/xiaomi/Mi8953

# 4. build
cd ~/fox_12.1
export FOX_BUILD_DEVICE=Mi8953
. build/envsetup.sh
lunch ofox_Mi8953-eng
mka recoveryimage
```

Var OrangeFox (`OF_*` / `FOX_*`) sudah didefinisikan di `vendorsetup.sh`
device tree dan otomatis ter-export saat `source build/envsetup.sh`.

## Var OrangeFox yang dipakai (lihat `vendorsetup.sh`)

| Var | Nilai | Alasan |
|-----|-------|--------|
| `OF_MAINTAINER` | `RenzAlt` | Nama maintainer di layar recovery |
| `FOX_BUILD_TYPE` | `Unofficial-CI` | Jenis build |
| `OF_PATCH_AVB20` | `1` | Device Xiaomi A-only MIUI - cegah recovery stok MIUI menimpa OrangeFox |
| `FOX_RECOVERY_SYSTEM_PARTITION` | `/dev/block/mapper/system` | Super retrofit (logical partition) |
| `FOX_RECOVERY_VENDOR_PARTITION` | `/dev/block/mapper/vendor` | Super retrofit (logical partition) |
| `OF_ENABLE_ALL_PARTITION_TOOLS` | `1` | Butuh `PRODUCT_USE_DYNAMIC_PARTITIONS` (sudah ada di `device.mk`) |
| `OF_FORCE_PREBUILT_KERNEL` | `1` | Kernel prebuilt - cegah error "NO KERNEL CONFIG" |
| `OF_DEFAULT_KEYMASTER_VERSION` | *(dikosongkan)* | **TODO**: isi setelah versi keymaster ROM target dipastikan (device tree membawa binary `keymaster@3.0`) |

## Status / hal yang perlu diperiksa

- **Decrypt data**: kalau mentok di logo, itu masalah keymaster - isi
  `OF_DEFAULT_KEYMASTER_VERSION` sesuai ROM target (lihat TODO di atas).
- **Kernel**: memakai prebuilt `4.9` dari repo ini. Kalau ada masalah boot /
  touchscreen, file inilah titik pertama yang dicek.
- **Partisi dinamis**: tree ini memakai *retrofit* super (super menumpang di
  partisi fisik `cust` + `system`), terdeteksi runtime oleh
  `recovery/root/system/bin/detect-dynamic-partitions.sh` (cek signature
  `mi8953_dynpart`), sehingga fstab otomatis memilih mode dinamis vs statis.

## Kredit

- [OrangeFox Recovery Project](https://gitlab.com/OrangeFox) -
  `bootable/Recovery`, `vendor/recovery`, `sync`
- Device tree asal: [imren0x/twrp_device_xiaomi_Mi8953](https://github.com/imren0x/twrp_device_xiaomi_Mi8953)
  (basis TWRP `android-12.1`)
- Workflow referensi: `ofox-vince-12.1-1.yml`

Lisensi: GPL v3 (device tree mengikuti lisensi OrangeFox/TWRP).
