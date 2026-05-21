WORKSHOP ADMINISTRASI JARINGAN

LAPORAN PRAKTIKUM MINGGU KE - 4

**PENS**

**Disusun Oleh Kelompok 1/ D4 IT A/ D4TIA-K01 :**

- Septia Cindi Artha Mevia (3124600003)
- Indra Rahmat Maulidi (3124600017)
- Fathu Alwi Nabiil (3124600020)

**Dosen Pengajar:** Dr Ferry Astika Saputra ST, M.Sc

**PROGRAM STUDI STR TEKNIK INFORMATIKA** **POLITEKNIK ELEKTRONIKA NEGERI SURABAYA (PENS) TAHUN 2026**

---

## DHCP Server (ISC DHCP) & NTP (Chrony)

---

## PRAKTIKUM

### Step 5: Konfigurasi Chrony NTP Master

```bash
sudo nano /etc/chrony/chrony.conf
```

Edit konfigurasi, tambahkan/modify baris berikut:

```conf
# Upstream NTP sources (pool.ntp.org Stratum 1)
pool id.pool.ntp.org iburst
pool pool.ntp.org iburst

# Allow clients dari backbone + LAN kelompok via relay
allow 10.252.108.0/24
allow 192.168.0.0/16

# Serve time even if not synced (local stratum 10)
local stratum 10

# Log tracking
log tracking measurements statistics
logdir /var/log/chrony
```

Simpan: `Ctrl+O`, `Enter`, `Ctrl+X`.

---

### Restart Chrony

```bash
# Restart service
sudo systemctl restart chrony
sudo systemctl enable chrony

# Verify status
sudo systemctl status chrony
```

#### Check NTP sync:

```bash
# Tracking info
sudo chronyc tracking
```

Expected output:

```
# Reference ID : POOL_IP (pool.ntp.org)
# Stratum      : 3
# System time  : 0.000XXX seconds fast/slow of NTP time
# Last offset  : +0.000XXX seconds
# RMS offset   : 0.000XXX seconds
```

> **System time offset harus < 1 second untuk accurate sync.**

---

#### Check upstream sources:

```bash
sudo chronyc sources
# Expected: ^* (selected) atau ^+ (candidate) untuk pool.ntp.org servers
```

---

### Step 9: Testing NTP Sync (Client)

Dari VM _client_ atau Proxmox _host_, test sync ke Chrony master (`10.252.108.20`).

**Install Chrony di client (jika belum):**

```bash
sudo apt install chrony -y
```

**Edit `/etc/chrony/chrony.conf` di client:**

```bash
sudo nano /etc/chrony/chrony.conf
```

**Comment `pool.ntp.org`, tambah server HO:**

```conf
# pool pool.ntp.org iburst  # Comment ini
server 10.252.108.20 iburst prefer

# Makestep untuk force sync cepat (lab purpose)
makestep 1 3
```

---

### Restart Chrony Client

```bash
sudo systemctl restart chrony

# Force immediate sync (optional)
sudo chronyc makestep

# Check tracking
sudo chronyc tracking
```

**Expected output:**

```
Reference ID    : 0A FC 6C 14 (10.252.108.20)  # HO NTP server
Stratum         : 4                              # Master stratum 3, client 4
System time     : 0.000XXX seconds fast/slow
Last offset     : +0.000XXX seconds
RMS offset      : 0.000XXX seconds
```

> **System time offset < 1s = good sync.**

**Check sources:**

```bash
sudo chronyc sources -v
# Expected: ^* untuk 10.252.108.20 (selected and synced)
```

---

## PERTANYAAN PRE-LAB (Teori)

### 1. Jelaskan detail 4 tahap DORA DHCP dan message type masing-masing (broadcast/unicast)?

**Jawab:**

| Tahap | Nama Pesan | Pengirim | Metode | Deskripsi |
|---|---|---|---|---|
| D | Discovery | Client | Broadcast | Client mencari server DHCP yang tersedia di jaringan lokal. |
| O | Offer | Server | Unicast* | Server menawarkan konfigurasi (IP, Mask, Gateway) kepada client. |
| R | Request | Client | Broadcast | Client mengonfirmasi tawaran dari satu server dan memberitahu server lain bahwa tawarannya ditolak. |
| A | Acknowledgement | Server | Unicast* | Server mengonfirmasi alokasi akhir dan mengirimkan parameter opsional lainnya. |

---

### 2. Apa peran DHCP relay agent dan bagaimana forward request cross-subnet?

**Jawab:**

Secara _default_, paket DHCP (_Broadcast_) tidak bisa melewati _router_. DHCP Relay Agent (biasanya dikonfigurasi di _router_ atau _switch_ Layer 3) bertugas menjembatani ini.

- **Cara Kerja:** Saat Relay Agent menerima paket DHCP Discovery (_Broadcast_) dari _client_ di Subnet A, ia akan mengubahnya menjadi paket _Unicast_.
- **Forwarding:** Paket _Unicast_ dikirim langsung ke IP DHCP Server di Subnet B. Relay Agent menyisipkan alamat IP-nya sendiri di kolom `giaddr` (_Gateway IP Address_) sehingga DHCP Server tahu dari _pool_ mana alamat IP harus diberikan.

---

### 3. Apa perbedaan default-lease-time vs max-lease-time dan kapan renewal terjadi?

**Jawab:**

Pengaturan waktu sewa (_lease_) menentukan berapa lama _client_ boleh memegang sebuah alamat IP.

- **`default-lease-time`:** Durasi standar yang diberikan _server_ jika _client_ tidak meminta durasi spesifik.
- **`max-lease-time`:** Batas waktu maksimum yang diizinkan _server_, meskipun _client_ meminta lebih lama.

**Kapan Renewal Terjadi?**

Sesuai standar RFC 2131, ada dua _timer_ utama:

- **T1 (Renewal):** Terjadi pada 50% dari total waktu sewa. _Client_ mencoba menghubungi _server_ asal secara _Unicast_ untuk memperpanjang sewa.
- **T2 (Rebinding):** Jika T1 gagal, pada 87.5% waktu sewa, _client_ melakukan _Broadcast_ untuk mencari setiap DHCP _server_ yang tersedia guna mempertahankan konektivitas.

---

### 4. Bagaimana Chrony Stratum hierarchy bekerja (0→1→2→3) dan kenapa time sync penting untuk cybersecurity lab?

**Jawab:**

Hierarki _Stratum_ menentukan tingkat akurasi dan jarak dari sumber waktu atom primer.

- **Stratum 0:** Sumber waktu fisik presisi tinggi (Jam Atom, GPS, jam Cesium). Tidak terhubung langsung ke jaringan.
- **Stratum 1:** Server yang terhubung langsung ke perangkat Stratum 0 via kabel (Serial/USB). Disebut _Primary Time Servers_.
- **Stratum 2:** Server yang menyinkronkan waktu dari Stratum 1 melalui jaringan (Internet/LAN). Kebanyakan server publik berada di sini.
- **Stratum 3:** Komputer atau perangkat yang mengambil waktu dari Stratum 2.

**Mengapa Penting untuk Cybersecurity Lab?**

Dalam lab keamanan siber, sinkronisasi waktu bukan sekadar soal kenyamanan, melainkan integritas data:

- **Log Correlation:** Saat menganalisis serangan, _timestamp_ pada log _Firewall_, IDS, dan SIEM harus sinkron agar kronologi serangan (_timeline_) dapat disusun secara akurat.
- **Authentication Protocols:** Protokol seperti Kerberos akan menolak login jika perbedaan waktu antara _client_ dan _server_ lebih dari 5 menit (mencegah _replay attacks_).
- **Certificate Validation:** Pengecekan masa berlaku sertifikat SSL/TLS bergantung sepenuhnya pada waktu sistem yang valid.

---

## PERTANYAAN HTTP

### 1. The Evolution of HTTP

HTTP (_HyperText Transfer Protocol_) adalah protokol komunikasi yang menjadi fondasi pertukaran data di _World Wide Web_. Perkembangannya melewati beberapa versi utama:

- **HTTP/0.9 (1991):** Versi awal yang sangat sederhana. Hanya mendukung satu metode (`GET`) dan hanya bisa mengirimkan dokumen HTML mentah tanpa _header_.
- **HTTP/1.0 (1996):** Memperkenalkan _header_ pada _request_ dan _response_, status kode, dan dukungan untuk berbagai tipe konten (gambar, video, dll.) melalui MIME. Setiap _request_ membuka koneksi TCP baru dan langsung ditutup setelahnya (_non-persistent_).
- **HTTP/1.1 (1997):** Menjadi standar dominan selama lebih dari satu dekade. Memperkenalkan _persistent connection_ (koneksi TCP tetap terbuka untuk beberapa _request_), _pipelining_, dan _chunked transfer encoding_.
- **HTTP/2 (2015):** Dirancang untuk performa lebih tinggi. Menggunakan format _binary_ (bukan teks), mendukung _multiplexing_ (banyak _request_ dalam satu koneksi secara paralel), _header compression_ (HPACK), dan _server push_.
- **HTTP/3 (2022):** Menggantikan TCP dengan QUIC (berbasis UDP) sebagai protokol transport. Dirancang untuk mengatasi masalah _head-of-line blocking_ pada TCP dan mempercepat koneksi terutama di jaringan yang tidak stabil.

---

### 2. How HTTP Works (TCP and UDP)

**HTTP berbasis TCP (HTTP/1.x dan HTTP/2):**

HTTP secara tradisional berjalan di atas TCP (_Transmission Control Protocol_). TCP menyediakan mekanisme _handshake_ (3-way handshake: SYN → SYN-ACK → ACK) sebelum data dikirim, memastikan data terkirim secara berurutan, lengkap, dan tanpa error. Proses alurnya:

1. _Client_ membangun koneksi TCP ke _server_ (port 80 untuk HTTP, port 443 untuk HTTPS).
2. _Client_ mengirimkan HTTP _request_ (metode, URL, _header_, _body_).
3. _Server_ memproses dan mengirimkan HTTP _response_ (status kode, _header_, _body_).
4. Koneksi ditutup (HTTP/1.0) atau dipertahankan (HTTP/1.1 _keep-alive_).

**HTTP berbasis UDP (HTTP/3 via QUIC):**

HTTP/3 menggunakan QUIC (_Quick UDP Internet Connections_) yang berjalan di atas UDP. QUIC mengimplementasikan sendiri mekanisme _reliability_, _flow control_, dan _multiplexing_ di level aplikasi, sehingga lebih cepat (mengurangi latensi _handshake_) dan tahan terhadap _packet loss_ tanpa memblokir seluruh _stream_.

---

### 3. How HTTP Server Works?

HTTP _server_ (seperti Apache, Nginx) bertugas menerima, memproses, dan merespons _request_ dari _client_. Cara kerjanya:

1. **Listen:** _Server_ mendengarkan koneksi masuk pada port tertentu (biasanya port 80 atau 443).
2. **Accept Connection:** Saat _client_ terhubung, _server_ menerima koneksi TCP dan membuat _socket_ baru untuk komunikasi tersebut.
3. **Parse Request:** _Server_ membaca dan mengurai _request_ HTTP (_method_, URL, _header_, _body_).
4. **Process Request:** _Server_ menentukan respons yang tepat — mengambil file statis, menjalankan skrip (_PHP_, _Python_), atau meneruskan ke aplikasi _backend_.
5. **Send Response:** _Server_ mengirimkan _response_ HTTP beserta status kode (200, 404, 500, dll.), _header_, dan konten (_body_).
6. **Close/Keep Connection:** Koneksi ditutup atau dipertahankan sesuai konfigurasi _keep-alive_.

---

### 4. How HTTP Client Works?

HTTP _client_ (seperti _web browser_, `curl`, `wget`) bertugas membangun _request_ dan mengolah _response_. Cara kerjanya:

1. **Resolve DNS:** _Client_ menerjemahkan nama domain (misal `pens.ac.id`) menjadi alamat IP melalui DNS _resolver_.
2. **Establish Connection:** _Client_ membangun koneksi TCP (atau QUIC untuk HTTP/3) ke IP dan port _server_ tujuan.
3. **Send Request:** _Client_ menyusun dan mengirimkan HTTP _request_ yang berisi _method_ (GET, POST, dll.), URL, _header_ (User-Agent, Accept, Cookie, dll.), dan _body_ (untuk POST/PUT).
4. **Receive Response:** _Client_ menerima HTTP _response_ dari _server_, membaca status kode dan _header_.
5. **Process Response:** _Client_ mengolah konten _response_ — _browser_ merender HTML, CSS, JS; aplikasi mengurai data JSON/XML.
6. **Close Connection:** Koneksi ditutup atau dipertahankan untuk _request_ berikutnya.
