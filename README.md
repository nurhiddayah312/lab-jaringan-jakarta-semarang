# Lab Jaringan Dua Kantor: Jakarta dan Semarang

Simulasi jaringan perusahaan dengan kantor pusat di Jakarta dan cabang di Semarang, dihubungkan lewat ISP. Lab ini dibuat di Cisco Packet Tracer 8.2.2 sebagai bagian dari portofolio NOC dan jaringan.

![Topologi jaringan](images/01-topologi.JPEG)

## Yang dikerjakan

- VLAN per departemen dan trunk 802.1Q di switch
- Inter-VLAN routing dengan router-on-a-stick
- DHCP di router untuk VLAN pengguna, IP statis untuk server dan printer
- NAT overload ke jaringan ISP
- GRE tunnel antar-kantor dengan OSPF area 0
- DNS dan HTTP di server internet dan server intranet
- ACL extended per VLAN untuk membatasi akses antar-departemen

## Perangkat

Total 20 perangkat: 3 router Cisco 2911, 3 switch Cisco 2960-24TT, 10 PC, 2 server, dan 2 printer.

## Rencana IP

| Lokasi | VLAN | Jaringan | Fungsi |
|---|---|---|---|
| Jakarta | 10 | 192.168.10.0/24 | IT |
| Jakarta | 20 | 192.168.20.0/24 | Keuangan |
| Jakarta | 30 | 192.168.30.0/24 | Tamu |
| Jakarta | 40 | 192.168.40.0/24 | Server (SERVER-INTERNAL .10) |
| Jakarta | 50 | 192.168.50.0/24 | Printer (PRN-JKT .10) |
| Semarang | 10 | 192.168.110.0/24 | Staf |
| Semarang | 20 | 192.168.120.0/24 | Tamu |
| Semarang | 30 | 192.168.130.0/24 | Printer (PRN-SMG .10) |
| ISP ke Jakarta | - | 203.0.113.0/30 | Link WAN |
| ISP ke Semarang | - | 203.0.113.4/30 | Link WAN |
| Jaringan internet | - | 198.51.100.0/24 | SRV-INTERNET .10 (DNS dan HTTP) |
| Tunnel GRE | - | 10.255.255.0/30 | Jakarta .1, Semarang .2 |

Gateway tiap VLAN memakai alamat .1. VLAN pengguna mendapat IP lewat DHCP, sedangkan server dan printer memakai IP statis.

## Kebijakan akses (ACL)

| Asal | Diizinkan | Diblokir |
|---|---|---|
| Jakarta IT | Semua akses (administrator) | Tidak ada |
| Jakarta Keuangan | Web (port 80) ke server internal, printer Jakarta, internet | Ping ke server, VLAN lain, jaringan Semarang |
| Jakarta Tamu | Internet saja | Semua jaringan internal 192.168.x.x |
| Semarang Staf | Web ke server internal di Jakarta, printer Semarang, internet | Ping ke server, printer Jakarta, VLAN lain |
| Semarang Tamu | Internet saja | Semua jaringan internal 192.168.x.x |

Pola setiap ACL: izin yang spesifik ditulis lebih dulu, lalu seluruh jaringan internal diblokir, dan terakhir sisanya (internet) diizinkan. Urutan ini penting karena router membaca ACL dari atas ke bawah.

## Hasil pengujian

### VLAN di switch Jakarta
Setiap VLAN berisi port yang sesuai rencana.

![show vlan brief di SW-JKT](images/02-vlan-jkt.JPEG)

### Trunk ke router
Port menuju router membawa semua VLAN.

![show interfaces trunk](images/03-trunk.JPEG)

### DHCP dan NAT
PC di VLAN IT mendapat IP otomatis, dan alamat privat diterjemahkan menjadi 203.0.113.2 saat keluar ke internet.

![DHCP dan NAT](images/04-dhcp-nat.JPEG)

### OSPF lewat GRE tunnel
Router Jakarta dan Semarang menjadi tetangga OSPF berstatus FULL, dan saling mempelajari jaringan kantor masing-masing.

![OSPF](images/05-ospf.JPEG)

### Jalur Semarang ke server Jakarta
Hasil tracert dari PC Semarang ke server internal menunjukkan paket melewati tunnel antar-kantor.

![Tracert](images/06-tracert.JPEG)

### DNS dan web
PC membuka alamat bernama lewat DNS di server internet.

![Web dan DNS](images/07-web-dns.JPEG)

### ACL: akses yang diizinkan
PC Keuangan bisa membuka web server internal, karena port 80 diizinkan.

![ACL berhasil](images/08-acl-berhasil.JPEG)

### ACL: akses yang diblokir
Ping dari PC Keuangan ke server internal dan akses PC Tamu ke intranet ditolak oleh ACL.

![ACL diblokir](images/09-acl-diblokir.JPEG)

## Isi repositori

| Lokasi | Isi |
|---|---|
| `lab-jakarta-semarang.pkt` | File Packet Tracer, bisa langsung dibuka |
| `images/` | Gambar topologi dan bukti pengujian |
| `configs/` | Konfigurasi tiap perangkat (R-JKT, R-SMG, R-ISP, SW-JKT, SW-SMG). SW-ISP memakai konfigurasi default |

## Catatan desain

- Trafik antar-kantor dikecualikan dari NAT lewat access list 100, supaya paket menuju kantor lain tidak ikut diterjemahkan.
- ACL bersifat stateless. Akibatnya, ping dari VLAN IT ke VLAN yang dibatasi gagal karena balasannya ditahan ACL departemen tujuan. Untuk jaringan produksi, ini bisa diganti dengan ACL yang mengenali koneksi (reflexive atau firewall).
- GRE tidak mengenkripsi trafik. Pengembangan berikutnya adalah menambahkan IPsec di atas tunnel.
- Rekaman DNS untuk nama intranet ditaruh di server internet demi menyederhanakan lab. Di jaringan nyata, nama internal dilayani oleh DNS internal.
- OSPF belum memakai passive-interface di sisi VLAN pengguna, karena perintah itu tidak bisa dipakai bersama pengaturan tunnel di Packet Tracer ini. Di jaringan produksi, antarmuka ke pengguna sebaiknya dibuat pasif.

## Cara membuka

Buka `lab-jakarta-semarang.pkt` dengan Packet Tracer 8.2.2 atau lebih baru. Konfigurasi tiap perangkat ada di folder `configs` kalau ingin dilihat tanpa membuka Packet Tracer.

## Pembuat

Nurul Hidayah, lulusan D4 Teknik Telekomunikasi, Politeknik Negeri Semarang.
[LinkedIn](https://linkedin.com/in/nurhiddayah)
