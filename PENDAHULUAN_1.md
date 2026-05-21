WORKSHOP ADMINISTRASI JARINGAN

LAPORAN PENDAHULUAN PRAKTIKUM MINGGU KE - 1

**PENS**

**Disusun Oleh Kelompok 1/ D4 IT A/D4TIA-K01 :**

- Septia Cindi Artha Mevia (3124600003)
- Indra Rahmat Maulidi (3124600017)
- Fathu Alwi Nabiil (3124600020)

**Dosen Pengajar:** Dr Ferry Astika Saputra ST, M.Sc

**PROGRAM STUDI STR TEKNIK INFORMATIKA** **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA (PENS) TAHUN 2026**

---

## 1. Jelaskan fungsi masing-masing lapisan pada OSI layer!

**Jawab:**

1. **Physical Layer:** Mengatur transmisi fisik data dalam bentuk _bit_ melalui media seperti kabel, radio, atau cahaya.

2. **Data Link Layer:** Mengelompokkan data menjadi _frame_, mengatur _error detection_, dan mengatur pengalamatan fisik (_MAC Address_).

3. **Network Layer:** Menentukan jalur _routing_ terbaik, melakukan pengalamatan logis (_IP Address_), dan meneruskan paket data ke tujuan.

4. **Transport Layer:** Memecah data menjadi paket-paket, memberikan nomor urut, serta memastikan data sampai dengan benar.

5. **Session Layer:** Membuka, mengelola, memelihara, dan mengakhiri sesi komunikasi antar dua perangkat.

6. **Presentation Layer:** Menerjemahkan data, melakukan enkripsi/dekripsi, dan kompresi data agar dapat dipahami oleh aplikasi.

7. **Application Layer:** Antarmuka langsung antara pengguna dan aplikasi jaringan, menyediakan layanan jaringan ke aplikasi seperti _web browser_.

## 2. Jelaskan perbedaan konsep penjelasan dari OSI layer dan TCP/IP!

Mengapa Perlu penjelasan TCP/IP

**Jawab:**

- **OSI (Open Systems Interconnection) Model:** Adalah model konseptual/referensi dengan 7 _layer_. Tujuannya adalah standarisasi agar berbagai vendor komputer dapat berkomunikasi. Ini memisahkan fungsi jaringan secara teoritis dan detail.

- **TCP/IP (Transmission Control Protocol/Internet Protocol) Model:** Adalah model implementasi praktis dengan 4 _layer_ (Application, Transport, Internet, Network Interface). Ini adalah protokol standar yang digunakan untuk komunikasi data di internet saat ini.

- **Mengapa Perlu:** Perlu karena TCP/IP adalah bahasa komunikasi di internet yang sebenarnya digunakan di dunia sebagai standar. OSI hanya panduan teoritis untuk memahami alur data, sedangkan TCP/IP sebagai aturan teknis yang dijalankan oleh _hardware_ dan _software_ untuk mentransfer paket data dengan cepat dan tepat.

## 3. Apa perbedaan konfigurasi IP dengan menggunakan "/etc/network/interfaces" dan "netplan" !

**Jawab:**

1.  **/etc/network/interfaces**
    - Menggunakan sistem _backend_ `ifupdown`.

    - Konfigurasi ditulis dalam format sintaks khusus Debian yang bersifat imperatif.

    - Perubahan sering memerlukan _restart service networking_ atau perintah `ifdown`/`ifup`.

    - Umum ditemukan pada distro Debian lama atau _server_ minimalis.

2.  **netplan**
    - Menggunakan format YAML.

    - Berfungsi sebagai _frontend_ yang menghasilkan konfigurasi untuk _backend_ lain (biasanya `systemd-networkd` untuk _server_ atau `NetworkManager` untuk _desktop_).

    - Menerapkan konfigurasi dengan perintah `netplan apply`.

    - Lebih modular dan standar baru di Ubuntu (sejak 17.10).

## 4. Buat tabel perbandingan antara Debian dan Ubuntu OS:

- a. Philosophy

- b. Target Audience

- c. Release Cycle

- d. Software Age

- e. Installation

- f. Proprietary Software

- g. Commercial Support

- h. Hardware Support

**Jawab:**

| Kriteria             | Debian                                                                  | Ubuntu                                                         |
| -------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------- |
| Philosophy           | Stabilitas, Perangkat Lunak Bebas, Komunitas                            | Kemudahan penggunaan, Inovasi, Dukungan perusahaan             |
| Target Audience      | Pengguna Linux berpengalaman, sysadmin, puritan                         | Pemula, pengembang, pengguna desktop, perusahaan               |
| Release Cycle        | Sekitar 2 tahun ("saat siap")                                           | 6 bulan (jadwal tetap)                                         |
| Software Age         | Lebih konservatif / versi lebih tua (untuk stabilitas)                  | Versi lebih baru / cutting-edge                                |
| Installation         | Installer berbasis teks (ncurses), lebih teknis                         | Installer GUI (Ubiquity/Subiquity), ramah pengguna             |
| Proprietary Software | Tidak disertakan secara default; harus mengaktifkan repositori non-free | Mudah tersedia dan sering disertakan (mis. driver proprietary) |
| Commercial Support   | Dukungan berbasis komunitas (pihak ketiga)                              | Dukungan resmi dari Canonical (Ubuntu Pro/Advantage)           |
| Hardware Support     | Cocok untuk komputer lama dan server yang membutuhkan stabilitas        | Cocok untuk perangkat lebih baru; pengalaman plug-and-play     |

## 5. Buat deskripsi dengan bahasamu sendiri :

a. Apa itu Virtualisasi?
b. Apa yang anda ketahui tentang Proxmox!
c. Apa bedanya dengan Virtualbox ?

**Jawab:**

- a. Virtualisasi: Cara membagi _hardware_ yang menjalankan beberapa komputer simulasi yang berjalan sendiri-sendiri secara bersamaan.

- b. Proxmox: Aplikasi _open source_ berbasis debian yang digunakan untuk mengelola VM dan kontainer pada komputer dalam satu tempat.

- c. Perbedaan Proxmox dengan Virtualbox: _Proxmox_ berjalan diatas _hardware_ (bisa dibilang OS sendiri), sedangkan _Virtualbox_ berjalan diatas OS lain (seperti windows). Bedanya sangat terlihat karena _Proxmox_ bisa memanipulasi dan mengakses _hardware_ secara langsung sedangkan _Virtualbox_ harus melewati perantara OS.
