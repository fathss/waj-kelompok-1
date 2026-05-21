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

## Instalasi Ubuntu Server di Proxmox

### Proxmox VE (Virtual Environment)

Proxmox VE adalah platform manajemen server _open-source_ tingkat _enterprise_ yang digunakan untuk virtualisasi komputasi. Platform yang dibangun di atas distribusi Debian Linux ini mengintegrasikan _hypervisor_ KVM (untuk Virtual Machine) dan LXC (untuk Container), penyimpanan yang ditentukan perangkat lunak (_software-defined storage_), serta fungsionalitas jaringan ke dalam satu antarmuka web terpusat.

---

### Perbandingan Virtual Machine (VM) vs Container

| Fitur | Virtual Machine (KVM) | Container (LXC) |
|---|---|---|
| Arsitektur | Virtualisasi tingkat perangkat keras (_hardware-level_). | Virtualisasi tingkat sistem operasi (_OS-level_). |
| Sistem Operasi | Menjalankan _Guest OS_ secara penuh beserta kernelnya sendiri. | Berbagi kernel sistem operasi yang sama dengan mesin _Host_ (Induk). |
| Penggunaan Resource | Berat. Membutuhkan alokasi CPU, RAM, dan penyimpanan yang besar dan terdedikasi. | Ringan dan efisien. Hanya menggunakan _resource_ sebesar yang dibutuhkan oleh aplikasi di dalamnya. |
| Waktu Booting | Lambat (hitungan menit) karena harus memuat kernel dan keseluruhan sistem operasi dari awal. | Sangat cepat (hitungan detik bahkan milidetik) karena kernel sudah berjalan di _Host_. |
| Isolasi & Keamanan | Isolasi penuh dan sangat aman. VM yang bermasalah tidak akan memengaruhi VM lain atau _Host_. | Isolasi dibatasi oleh _namespace_ OS. Jika kernel _Host_ mengalami _crash_ atau diretas, seluruh container bisa terdampak. |
| Fleksibilitas OS | Dapat menjalankan sistem operasi yang berbeda dari _Host_ (misal: _Host_ Linux menjalankan VM Windows, FreeBSD, atau Mikrotik). | Hanya dapat menjalankan distribusi yang memiliki arsitektur OS dan kernel yang sama dengan _Host_ (Linux based). |
| Skenario Penggunaan | Menjalankan _legacy apps_, simulasi jaringan, atau aplikasi yang membutuhkan OS spesifik di luar Linux. | Menjalankan _web server_, _database_, _microservices_, atau _environment_ Docker. |
| Performa | _Overhead_ lebih besar karena menjalankan OS tamu lengkap dan emulasi _hardware_. | _Overhead_ sangat rendah karena langsung menggunakan kernel _host_. Aplikasi berjalan hampir _native_. |
| Ukuran Image | Besar (GB) karena mencakup OS lengkap. | Kecil (MB) karena hanya berisi aplikasi dan pustaka. |
| Contoh Teknologi | Proxmox VE, VMware vSphere, VirtualBox, KVM, Hyper-V. | Docker, LXC/LXD, Podman, containerd, rkt. |

---

## Membuat Virtual Machine di Proxmox

### 1. Membuat VM Baru

- Buka antarmuka web Proxmox.
- Klik **Create VM**.
- Tab **General**:
  - VM ID: `103` (otomatis atau ubah sesuai keinginan)
  - Name: `D4ITA-K01-B`
  - (Biarkan opsi lain _default_)
- Klik **Next**.

### 2. Memilih OS

- Tab **OS**:
  - Guest OS type: `Linux`
  - Version: `6.x - 2.6 Kernel`
  - Pilih **Use CD/DVD disc image file (iso)**
  - Storage: `local`
  - ISO image: `ubuntu-24.04.3-live-server-amd64.iso` (atau nama file yang sesuai)
- Klik **Next**.

### 3. Konfigurasi System

- Tab **System**:
  - SCSI Controller: `VirtIO SCSI single`
  - Qemu Agent: (centang jika ingin, tidak wajib)
  - Add TPM: (biarkan kosong)
- Klik **Next**.

### 4. Menambahkan Disk

- Tab **Disks**:
  - Bus/Device: `SCSI`
  - Storage: `local-lvm`
  - Disk size (GiB): `20`
  - Format: `Raw disk image (raw)`
  - Cache: `Default (No cache)`
- Klik **Next**.

### 5. Mengatur CPU

- Tab **CPU**:
  - Cores: `2`
  - Type: `x86-64-v2-AES`
- Klik **Next**.

### 6. Mengatur Memori

- Tab **Memory**:
  - Memory (MiB): `4096`
- Klik **Next**.

### 7. Konfigurasi Jaringan

- Tab **Network**:
  - Model: `VirtIO (paravirtualized)`
  - Bridge: `vmbr0`
  - Firewall: (centang jika perlu)
  - VLAN Tag: (biarkan kosong)
- Klik **Next**.

### 8. Konfirmasi dan Selesai

- Periksa ringkasan konfigurasi.
- Klik **Finish**.
- VM akan muncul di daftar. Setelah itu, _start_ VM tersebut dan buka konsol melalui **Console**.

---

## Instalasi Ubuntu Server di Dalam VM

### 9. Memilih Bahasa

- Di konsol VM, akan muncul layar selamat datang.
- Pilih bahasa: **English** (gunakan tombol panah, lalu Enter).

### 10. Konfigurasi Keyboard

- Pilih Layout: `English (US)`
- Pilih Variant: `English (US)`
- Tekan **Done**.

### 11. Memilih Tipe Instalasi

- Pilih **Ubuntu Server** (_default_, bukan _minimized_).
- Opsi _Search for third-party drivers_ biarkan tidak dicentang.
- Tekan **Done**.

### 12. Konfigurasi Jaringan

- Antarmuka `ens18` akan terdeteksi tetapi gagal autokonfigurasi (karena belum diset).
- Pilih antarmuka `ens18`, lalu tekan Enter untuk mengedit.

### 13. Setting IP Manual

- Pilih **Edit ens18 IPv4 configuration**.
- IPv4 Method: `Manual`
- Subnet: `10.252.108.0/24`
- Address: `10.252.108.221`
- Gateway: `10.252.108.1`
- Name servers: `202.9.85.3`
- Search domains: `pens.corp.lab`
- Tekan **Save**, lalu **Done**.

### 14. Proxy

- Biarkan Proxy address kosong (tidak menggunakan _proxy_).
- Tekan **Done**.

### 15. Memilih Mirror

- Mirror address: `http://kebo.pens.ac.id/ubuntu`
- Sistem akan menguji koneksi, jika berhasil tekan **Done**.

### 16. Pembaruan Installer

- Muncul notifikasi _"Installer update available"_.
- Pilih **Continue without updating** (agar tidak mengunduh pembaruan).

### 17. Konfigurasi Storage – Guided

- Pilih **Use an entire disk**.
- Pilih disk: `QEMU_HARDDISK_drive-scsi0 20.000G`
- Centang **Set up this disk as an LVM group**.
- Jangan centang _Encrypt the LVM group with LUKS_.
- Tekan **Done**.

### 18. Ringkasan Storage

- Tampilan partisi yang akan dibuat:
  - `/` (root): 10G (LVM logical volume)
  - `/boot`: 1.771G (partisi biasa)
  - Sisanya 8.222G masih tersedia di VG `ubuntu-vg`.
- Tekan **Done** jika sudah sesuai.

### 19. Menambah Logical Volume untuk /home

- Karena masih ada _free space_ 8.222G di VG, kita akan membuat _logical volume_ baru.
- Klik pada `ubuntu-vg` atau pilih opsi untuk membuat LV.
- Isi:
  - Name: `home`
  - Size: `8.222G`
  - Format: `ext4`
  - Mount: `/home`
- Tekan **Create**.

### 20. Storage Final

- Sekarang terlihat:
  - `/` (root): 10G
  - `/boot`: 1.771G
  - `/home`: 8.222G
- Tekan **Done**.

### 21. Konfirmasi Tindakan Destruktif

- Peringatan bahwa data akan dihapus.
- Pilih **Continue** untuk memulai instalasi.

### 22. Profil Pengguna

- Your name: `Student`
- Your server's name: `d4tia-k01-b`
- Pick a username: `student`
- Choose a password: `******` (isi sesuai keinginan)
- Confirm password: `******`
- Tekan **Done**.

### 23. Ubuntu Pro

- Pilih **Skip for now** (tidak mengaktifkan Ubuntu Pro).
- Tekan **Continue**.

### 24. Konfigurasi SSH

- Centang **Install OpenSSH server**.
- Centang **Allow password authentication over SSH**.
- (Jika ada SSH key, bisa diimpor, tapi di sini tidak ada)
- Tekan **Done**.

### 25. Featured Server Snaps

- Muncul daftar snap yang populer. Biarkan tidak ada yang dipilih (lewatkan).
- Tekan **Done**.

### 26. Proses Instalasi

- Instalasi berjalan: partisi, menyalin file, konfigurasi sistem.
- Tunggu hingga selesai.

### 27. Instalasi Selesai

- Setelah _reboot_, muncul _prompt login_.
- Login dengan user `student` dan _password_ yang telah dibuat.
- Sistem siap digunakan.

---

## Topologi Jaringan Lab C 307

### Analisa

Topologi ini menggunakan pendekatan _Hierarchical Network Design_ (Desain Jaringan Hierarkis) yang umum digunakan pada skala kantor pusat dengan pembagian segmen _backbone_ dan kelompok.

### 1. Perimeter dan Edge (Router Utama)

- **Perangkat:** MikroTik RB1100 (HO).
- **Fungsi:** Sebagai _Gateway_ utama yang menghubungkan jaringan internal ke internet.
- **Konfigurasi IP:**
  - WAN: `192.168.89.2/24` (Menerima internet dari modem/ISP)
  - LAN (_Gateway_ Internal): `10.252.108.1/24`
- **Analisis:** RB1100 adalah router kelas industri yang sangat mumpuni untuk menangani beban trafik tinggi dan manajemen _bandwidth_.

### 2. Core/Backbone Network

- **Switching (CSS326):** Bertindak sebagai _Distribution Switch_. Menggunakan MikroTik CSS326 yang berjalan dengan SwOS, fokus pada manajemen Layer 2 (VLAN/_switching_). Port 3-12 dialokasikan khusus untuk Router Kelompok.
- **Services / Core Hosts:**
  - DNS HO (`10.252.108.10`): Resolusi nama domain lokal maupun internet.
  - RSYSLOG HO (`10.252.108.20`): Server sentralisasi log untuk monitoring keamanan dan performa perangkat.
  - Proxmox VE (`10.252.108.21`): Server virtualisasi yang menyediakan layanan _cloud_ untuk kelompok 1-10.

### 3. Jaringan Kelompok (Access Layer)

- **Router Kelompok (RB3011):** Bertindak sebagai _Downstream Router_. RB3011 memiliki performa tinggi yang cocok untuk memisahkan trafik kelompok dari _backbone_.
- **Segmentasi IP:**
  - Sisi _Uplink_ (Backbone): `10.252.108.51`
  - Sisi Lokal (LAN Kelompok): `192.168.1.0/24`
