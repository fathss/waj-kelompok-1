WORKSHOP ADMINISTRASI JARINGAN

LAPORAN PRAKTIKUM MINGGU KE - 1

**PENS**

**Disusun Oleh Kelompok 1/ D4 IT A/D4TIA-K01 :**

- Septia Cindi Artha Mevia (3124600003)
- Indra Rahmat Maulidi (3124600017)
- Fathu Alwi Nabiil (3124600020)

**Dosen Pengajar:** Dr Ferry Astika Saputra ST, M.Sc

**PROGRAM STUDI STR TEKNIK INFORMATIKA** **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA (PENS) TAHUN 2026**

---

## Konfigurasi Router

### Variabel Konfigurasi

```
==========================================
# VARIABEL KONFIGURASI
# Ganti sesuai Kelompok dan Kelas
# ==========================================
# KELOMPOK_NUM = 1-10
# KELAS        = A/B/C/D/E/F
# VLAN_ID      = Sesuai tabel VLAN
# WAN_IP       = 10.252.108.(50+KELOMPOK_NUM)
# LAN_NETWORK  = 192.168.KELOMPOK_NUM.0/24
# ==========================================
# Contoh: Kelas A, Kelompok 5
# VLAN_ID      = 55
# WAN_IP       = 10.252.108.55
# LAN_NETWORK  = 192.168.5.0/24
```

---

### Reset Konfigurasi

```
/system reset-configuration no-defaults=yes skip-backup=yes
```

**Penjelasan:**

Sebelum mulai konfigurasi, saya reset router ke kondisi pabrik agar bersih dari pengaturan lama. Perintahnya:

```
/system reset-configuration no-defaults=yes skip-backup=yes
```

- `no-defaults=yes` untuk menghapus semua konfigurasi bawaan.
- `skip-backup=yes` agar proses cepat tanpa backup.

Setelah reboot, router siap dikonfigurasi dari awal.

---

### Set Identity

```
/system identity set name="RB3011-KELAS_A-KLP1"
```

**Penjelasan:**

Setelah reset, langkah berikutnya adalah memberi identitas pada router agar mudah dikenali, terutama saat kita mengelola banyak perangkat. Saya menggunakan perintah:

```
/system identity set name="RB3011-KELAS_A-KLP1"
```

Perintah ini mengubah nama router menjadi `"RB3011-KELAS_A-KLP1"` yang mencerminkan tipe perangkat (RB3011), kelas (A), dan nomor kelompok (1). Setelah itu, prompt terminal berubah dari `[admin@MikroTik]` menjadi `[admin@RB3011-KELAS_A-KLP1]`, menandakan bahwa pengaturan nama berhasil. Hal ini memudahkan saat melakukan konfigurasi atau _troubleshooting_, karena nama router langsung terlihat di setiap sesi.

---

## Interface Configuration

> Urutan: Physical → Bond → Bridge → VLAN → IP

### Step 1: Konfigurasi Interface WAN (ether1)

Port _access_ di switch, tidak perlu VLAN tagging di router.

```
/interface ethernet set ether1 name=ether1-wan
```

**Penjelasan:**

Interface `ether1` diubah namanya menjadi `ether1-wan` untuk memperjelas bahwa port ini digunakan sebagai jalur WAN (koneksi ke jaringan luar/backbone). Penamaan ini memudahkan identifikasi saat konfigurasi _firewall_, _routing_, dan _NAT_.

---

### Step 2: Buat Bridge untuk LAN (ether2–ether10)

```
/interface bridge add name=bridge-lan protocol-mode=none
```

**Penjelasan:**

Dibuat sebuah _bridge_ bernama `bridge-lan` untuk menggabungkan semua port LAN (ether2 hingga ether10) menjadi satu segmen jaringan. Penggunaan `protocol-mode=none` menonaktifkan STP (_Spanning Tree Protocol_) karena tidak diperlukan dalam topologi sederhana ini.

---

### Step 3: Tambahkan Port ke Bridge LAN

```
/interface bridge port
add bridge=bridge-lan interface=ether2
add bridge=bridge-lan interface=ether3
add bridge=bridge-lan interface=ether4
add bridge=bridge-lan interface=ether5
add bridge=bridge-lan interface=ether6
add bridge=bridge-lan interface=ether7
add bridge=bridge-lan interface=ether8
add bridge=bridge-lan interface=ether9
add bridge=bridge-lan interface=ether10
```

**Penjelasan:**

Setiap port dari ether2 hingga ether10 ditambahkan sebagai anggota dari `bridge-lan`. Dengan demikian, semua perangkat yang terhubung ke port-port tersebut akan berada dalam satu jaringan LAN yang sama dan dapat saling berkomunikasi.

---

## IP Address Configuration

### WAN IP

```
/ip address add address=10.252.108.55/24 interface=ether1-wan network=10.252.108.0 comment="WAN-Backbone"
```

### LAN IP

```
/ip address add address=192.168.5.1/24 interface=bridge-lan network=192.168.5.0 comment="LAN-Kelompok"
```

**Penjelasan:**

- **WAN IP** (`10.252.108.55/24`) dikonfigurasi pada interface `ether1-wan` untuk menghubungkan router ke jaringan _backbone_ kampus.
- **LAN IP** (`192.168.5.1/24`) dikonfigurasi pada `bridge-lan` sebagai _gateway_ untuk seluruh perangkat di jaringan LAN kelompok.

---

## Routing

```
/ip route add gateway=10.252.108.1 comment="Default Gateway ke RB1100 HO"
```

**Penjelasan:**

_Default route_ ditambahkan dengan _gateway_ mengarah ke `10.252.108.1` (router utama RB1100 HO). Ini memungkinkan semua lalu lintas yang tidak dikenali di tabel _routing_ lokal untuk diteruskan ke jaringan yang lebih luas (internet).

---

## DNS

```
/ip dns set servers=10.252.108.10 allow-remote-requests=yes
```

**Penjelasan:**

DNS server dikonfigurasi ke `10.252.108.10` (DNS server lokal kampus). Opsi `allow-remote-requests=yes` mengizinkan router untuk menerima dan menjawab permintaan DNS dari klien di jaringan LAN, sehingga router sekaligus berfungsi sebagai DNS _forwarder_.

---

## DHCP Server untuk LAN

```
/ip pool add name=dhcp-pool-lan ranges=192.168.5.100-192.168.5.200

/ip dhcp-server add name=dhcp-lan address-pool=dhcp-pool-lan interface=bridge-lan disabled=no

/ip dhcp-server network add address=192.168.5.0/24 gateway=192.168.5.1 dns-server=10.252.108.10,8.8.8.8 comment="DHCP Network LAN"
```

**Penjelasan:**

- **IP Pool** didefinisikan dari `192.168.5.100` hingga `192.168.5.200` sebagai rentang alamat yang akan dibagikan secara dinamis ke klien.
- **DHCP Server** diaktifkan pada interface `bridge-lan`.
- **DHCP Network** mengatur _gateway_ ke `192.168.5.1` dan DNS server ke `10.252.108.10` (lokal) serta `8.8.8.8` (Google DNS) sebagai cadangan.

---

## NAT / Masquerade

```
/ip firewall nat add chain=srcnat out-interface=ether1-wan action=masquerade comment="NAT untuk akses internet"
```

**Penjelasan:**

Aturan _NAT masquerade_ diterapkan pada interface `ether1-wan` agar semua lalu lintas dari jaringan LAN yang keluar ke WAN akan mengganti alamat sumbernya dengan IP WAN router. Ini memungkinkan seluruh perangkat LAN mengakses internet menggunakan satu IP publik.

---

## Firewall Dasar

```
/ip firewall filter

# Accept established, related, untracked
add chain=input action=accept connection-state=established,related,untracked comment="Accept established"
add chain=input action=drop connection-state=invalid comment="Drop invalid"
add chain=input action=accept protocol=icmp comment="Accept ICMP"
add chain=input action=accept src-address=10.252.108.0/24 comment="Accept from backbone"
add chain=input action=accept src-address=192.168.5.0/24 comment="Accept from LAN"
add chain=input action=drop in-interface=ether1-wan comment="Drop all from WAN"

# Forward chain
add chain=forward action=accept connection-state=established,related,untracked
add chain=forward action=drop connection-state=invalid
add chain=forward action=accept in-interface=bridge-lan
add chain=forward action=drop in-interface=ether1-wan connection-nat-state=!dstnat
```

**Penjelasan:**

Konfigurasi _firewall_ dasar terbagi dalam dua _chain_:

- **Input chain** mengatur lalu lintas yang masuk ke router itu sendiri:
  - Koneksi yang sudah terjalin (_established/related_) diterima.
  - Koneksi tidak valid (_invalid_) dibuang.
  - ICMP (ping) diterima untuk keperluan diagnostik.
  - Koneksi dari jaringan _backbone_ (`10.252.108.0/24`) dan LAN (`192.168.5.0/24`) diterima.
  - Semua koneksi masuk dari WAN dibuang untuk keamanan.

- **Forward chain** mengatur lalu lintas yang melewati router:
  - Koneksi yang sudah terjalin diterima.
  - Koneksi tidak valid dibuang.
  - Lalu lintas dari LAN diizinkan.
  - Lalu lintas dari WAN yang bukan hasil _dst-nat_ dibuang.
