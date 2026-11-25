# Panduan Lengkap Implementasi dan Operasi NSE 1–8, Studi Kasus Firewall, dan Contoh Konfigurasi FortiOS v6 dan v7

## Daftar Isi

- Bagian 1 — Panduan Implementasi dan Operasi NSE 1–8 [Bagian 1 — Panduan Implementasi dan Operasi NSE 1–8](https://github.com/anggrdwjy/Library-Fortinet-NSE/blob/main/README.md#bagian-1--panduan-implementasi-dan-operasi-nse-18)
- Bagian 2 — Studi Kasus Implementasi dan Operasi Firewall
- Bagian 3 — Contoh Konfigurasi FortiOS v6 dan v7

# Bagian 1 — Panduan Implementasi dan Operasi NSE 1–8

## Modul 1 — NSE 1: Cybersecurity Fundamentals

### Tujuan

Peserta mampu menjelaskan threat landscape, mengklasifikasikan jenis attack, membedakan kerahasiaan, integritas, dan ketersediaan, serta menjelaskan peran Security Fabric dalam pertahanan berlapis.

### Konsep Kunci

| Konsep | Penjelasan |
| --- | --- |
| CIA Triad | Confidentiality, Integrity, Availability sebagai dasar tujuan security |
| Threat Actor | Pelaku threat, mulai dari peretas oportunistik hingga kelompok terorganisasi |
| Attack Surface | Seluruh titik yang dapat dieksploitasi, termasuk endpoint, aplikasi, dan identitas |
| Zero Trust | Prinsip tidak ada kepercayaan implisit, verifikasi dilakukan secara berkelanjutan |
| Security Fabric | Integrasi produk security untuk berbagi threat intelligence dan respons otomatis |

### Implementasi

1. Lakukan inventarisasi aset. Catat seluruh device, aplikasi, akun istimewa, dan data sensitif.
2. Klasifikasikan data berdasarkan dampak kebocoran. Contoh: data publik, data internal, data rahasia.
3. Petakan threat terhadap setiap aset. Contoh: phishing untuk akun user, ransomware untuk file server, DDoS untuk layanan publik.
4. Tentukan kontrol yang sesuai. Contoh: MFA untuk akun istimewa, isolated backup untuk ransomware, rate limiting untuk DDoS.
5. Susun program security awareness. Materi mencakup phishing, password, penggunaan USB, dan pelaporan insiden.
6. Tentukan jalur eskalasi insiden. Setiap user mengetahui kepada siapa insiden dilaporkan dan dalam jangka waktu yang ditetapkan.

### Contoh Implementasi

Klasifikasi insiden yang digunakan dalam operasi harian:

```
Kritis    : Ransomware aktif, eksfiltrasi data, kendali domain dikuasai
Tinggi    : Phishing berhasil pada akun istimewa, malware pada server
Sedang    : Percobaan login brute force masif, malware pada satu endpoint
Rendah    : Spam, scanning port tunggal, upaya phishing yang gagal
```

### Operasi dan Verifikasi

- Periksa laporan phishing yang masuk setiap hari dan ukur rasio pelaporan user.
- Periksa status backup dan lakukan uji recovery berkala.
- Periksa akun yang tidak aktif lebih dari 90 hari dan nonaktifkan akun tersebut.
- Periksa hasil simulasi phishing dan tindak lanjuti unit dengan rasio klik tertinggi melalui pelatihan tambahan.

### Studi Kasus

**Situasi:** Sebuah organisasi menerima email yang menyerupai faktur dari pemasok. Satu staf keuangan membuka lampiran dan mengaktifkan makro. Dalam waktu 20 menit, file di workstation mulai terenkripsi.

**Analisis:** Insiden diklasifikasikan sebagai Kritis karena ransomware aktif. Jalur eskalasi diaktifkan. Workstation diisolasi dari network. Akun user dinonaktifkan sementara. Backup yang terisolasi digunakan untuk recovery.

**Tindakan Perbaikan:** Makro diblokir melalui policy endpoint, lampiran berbahaya diblokir melalui FortiMail, pelatihan phishing diulang untuk unit keuangan, dan aturan FortiGate diperketat untuk memblokir komunikasi ke domain command and control.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| User tidak melaporkan phishing | Jalur pelaporan tidak jelas | Sosialisasikan tombol pelaporan dan prosedur eskalasi |
| Insiden berulang pada unit yang sama | Pelatihan tidak menyasar perilaku | Lakukan pelatihan berbasis peran, bukan pelatihan umum |
| Backup ikut terenkripsi | Backup terhubung terus menerus | Terapkan isolated backup dan uji recovery |


## Modul 2 — NSE 2: NGFW Basics

### Tujuan

Peserta mampu menjelaskan perbedaan firewall tradisional dan Next Generation Firewall, fungsi deep inspection, serta peran FortiGate dalam Security Fabric.

### Konsep Kunci

| Konsep | Penjelasan |
| --- | --- |
| Stateful Firewall | Memeriksa status koneksi dan hanya mengizinkan traffic yang sesuai dengan session yang sah |
| NGFW | Menambahkan identifikasi aplikasi, identifikasi user, IPS, antivirus, dan filtering web |
| Profile-Based Mode | Security profile diterapkan pada firewall policy secara terpisah |
| Policy-Based Mode | Security profile diterapkan langsung di dalam policy |
| Security Fabric | Pertukaran threat intelligence antar FortiGate, FortiAnalyzer, FortiManager, dan FortiClient |

### Implementasi

1. Tentukan mode inspection yang digunakan. Untuk operasi yang konsisten, gunakan Profile-Based Mode pada awal implementasi.
2. Aktifkan SSL inspection untuk encrypted traffic sesuai organizational policy.
3. Terapkan security profile secara bertahap. Mulai dari mode monitor pada IPS dan Application Control, kemudian ubah ke mode block setelah periode observasi.
4. Hubungkan FortiGate ke FortiAnalyzer untuk visibilitas log terpusat.
5. Hubungkan FortiGate ke Security Fabric melalui FortiManager atau FortiGate root.

### Contoh Konfigurasi

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping ssh
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
end

config system settings
    set inspection-mode flow
end

config firewall ssl-ssh-profile
    edit "deep-inspection"
        set comment "Digunakan sesuai privacy policy organisasi"
        config ssl
            set inspect-all enable
        end
    next
end
```

### Operasi dan Verifikasi

```
get system status
get system interface physical
diagnose sys session list
diagnose debug flow filter addr 10.10.10.50
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug disable
```

Periksa FortiView untuk melihat aplikasi teratas, sumber teratas, dan threat teratas. Pastikan tidak ada aplikasi berisiko tinggi yang lolos tanpa profile.

### Studi Kasus

**Situasi:** Traffic menuju internet meningkat, tetapi log hanya menampilkan port 443 tanpa informasi aplikasi. Tim tidak dapat membedakan traffic aplikasi bisnis dan traffic streaming video.

**Analisis:** FortiGate masih bekerja seperti firewall tradisional karena Application Control belum diterapkan. SSL inspection belum diaktifkan sehingga aplikasi dalam encrypted traffic tidak terlihat.

**Penyelesaian:** Application Control diaktifkan dalam mode monitor selama tujuh hari. Setelah profile aplikasi bisnis dan non bisnis terpetakan, policy dibuat untuk memprioritaskan aplikasi bisnis dan membatasi streaming. SSL inspection diterapkan pada kelompok user yang telah menyetujui policy.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| Aplikasi tidak terdeteksi | Application Control belum diterapkan | Terapkan profile pada policy yang sesuai |
| Traffic HTTPS tidak terlihat | SSL inspection belum aktif | Terapkan SSL-SSH profile pada policy |
| Kinerja menurun setelah inspection | Mode proxy digunakan pada device kecil | Gunakan flow-based inspection dan sesuaikan profile |


## Modul 3 — NSE 3: FortiGate Operator

### Tujuan

Peserta mampu melakukan operasi harian FortiGate, membaca log dan FortiView, melakukan backup dan recovery, mengelola firmware, serta melakukan triase awal gangguan.

### Konsep Kunci

| Konsep | Penjelasan |
| --- | --- |
| FortiView | Visualisasi traffic, aplikasi, threat, dan user secara waktu nyata |
| Log dan Event | Rekaman aktivitas policy, security, sistem, dan administrasi |
| Admin Profile | Pembatasan hak akses administrator berdasarkan peran |
| Firmware Management | Pengelolaan versi FortiOS, termasuk peningkatan dan rollback |
| Configuration Backup | Salinan konfigurasi untuk recovery dan audit |

### Implementasi

1. Buat akun administrator berbasis peran. Administrator NOC hanya diberikan hak baca dan diagnostik. Administrator senior diberikan hak tulis.
2. Aktifkan scheduled backup ke FortiManager atau server TFTP.
3. Aktifkan pengiriman log ke FortiAnalyzer. Pastikan zone waktu dan NTP konsisten.
4. Tentukan sindikasirdd penamaan policy. Contoh: `LAN-to-WAN-Web-Allow`.
5. Tentukan prosedur perubahan. Setiap perubahan dicatat, diuji di jendela pemeliharaan, dan dapat dikembalikan.

### Contoh Konfigurasi

```
config system admin
    edit "noc-operator"
        set accprofile "read-only"
        set vdom "root"
        set password <kata-sandi-kuat>
    next
end

config system ntp
    set ntpsync enable
    config ntpserver
        edit 1
            set server "pool.ntp.org"
        next
    end
end

config log fortianalyzer setting
    set status enable
    set server 192.168.99.11
    set upload-option realtime
    set reliable enable
end

execute backup config tftp HQ-FGT-2026-11-25.conf 192.168.99.20
```

### Operasi dan Verifikasi

```
get system performance status
get system session status
execute ping 8.8.8.8
execute traceroute 8.8.8.8
diagnose sys top 1 20
diagnose hardware deviceinfo nic port1
diagnose log test
get log fortianalyzer setting
```

Periksa setiap hari: status HA, status lisensi, status disk log, penggunaan CPU dan memori, serta antrian log ke FortiAnalyzer.

### Studi Kasus

**Situasi:** User melaporkan internet lambat pada pukul 09.00. Tidak ada perubahan konfigurasi yang tercatat pada malam sebelumnya.

**Analisis:** Operator memeriksa FortiView dan menemukan satu workstation menghasilkan ribuan session ke address eksternal. Pemeriksaan log menunjukkan aktivitas yang konsisten dengan malware. Penggunaan CPU FortiGate meningkat karena session yang berlebihan.

**Penyelesaian:** Workstation diisolasi. Session dari workstation tersebut dibersihkan dengan filter source address. Policy diperketat untuk membatasi jumlah session per source address. Insiden dieskalasikan ke tim security untuk analisis endpoint.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| Log tidak sampai ke FortiAnalyzer | Route atau port terblokir | Periksa konektivitas UDP 514 dan TCP 514 |
| CPU tinggi | Session berlebihan atau inspection berat | Periksa session teratas dan sesuaikan profile |
| Backup gagal | Ruang penyimpanan penuh | Periksa server TFTP dan rotasi file |
| Firmware gagal diperbarui | Jalur peningkatan tidak berurutan | Ikuti jalur peningkatan resmi FortiOS |


## Modul 4 — NSE 4: FortiOS Administrator

Modul ini merupakan inti sertifikasi. Seluruh implementasi di modul lain bergantung pada kemampuan administrasi FortiOS yang dikuasai pada modul ini.

### Tujuan

Peserta mampu mengonfigurasi interface, zone, firewall policy, NAT, VPN, security profile, user authentication, dan high availability secara mandiri dan terdokumentasi.

### Konsep Kunci

| Konsep | Penjelasan |
| --- | --- |
| Zone | Pengelompokan interface dengan tingkat kepercayaan yang sama |
| Firewall Policy | Aturan yang menentukan traffic yang diizinkan atau ditolak antar zone |
| NAT | Penerjemahan address, termasuk SNAT dan DNAT atau Virtual IP |
| UTM Profile | Antivirus, IPS, Web Filter, Application Control, DNS Filter, File Filter |
| VPN | IPsec dan SSL VPN untuk konektivitas antar situs dan akses jarak jauh |
| HA | High Availability Active-Passive atau Active-Active |

### Implementasi Tahap 1 — Interface dan Zone

```
config system zone
    edit "LAN-Zone"
        set interface "port2"
    next
    edit "DMZ-Zone"
        set interface "port3"
    next
    edit "WAN-Zone"
        set interface "port1"
    next
end

config system interface
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
end
```

### Implementasi Tahap 2 — Firewall Policy dan NAT

Policy ditulis dari yang paling spesifik ke yang paling umum. Policy deny implisit berada pada akhir.

```
config firewall address
    edit "LAN-Subnet"
        set subnet 10.10.10.0 255.255.255.0
    next
    edit "DMZ-Server"
        set subnet 172.16.10.10 255.255.255.255
    next
end

config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set application-list "default"
        set ssl-ssh-profile "certificate-inspection"
        set nat enable
        set logtraffic all
    next
    edit 20
        set name "LAN-to-DMZ-Server-Allow"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "LAN-Subnet"
        set dstaddr "DMZ-Server"
        set action accept
        set schedule "always"
        set service "HTTPS" "SSH"
        set logtraffic all
    next
end
```

### Implementasi Tahap 3 — Virtual IP dan Akses Server

```
config firewall vip
    edit "VIP-Web-DMZ"
        set extip 203.0.113.2
        set extintf "port1"
        set mappedip 172.16.10.10
        set portforward enable
        set extport 443
        set mappedport 443
    next
end

config firewall policy
    edit 30
        set name "WAN-to-DMZ-Web-Allow"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "VIP-Web-DMZ"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set utm-status enable
        set ips-sensor "default"
        set webfilter-profile "default"
        set logtraffic all
        set nat enable
    next
end
```

### Implementasi Tahap 4 — IPsec VPN Antar Situs

```
config vpn ipsec phase1-interface
    edit "VPN-HQ-BR"
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal aes256-sha256
        set remote-gw 203.0.113.6
        set psksecret <kunci-bersama-yang-kuat>
    next
end

config vpn ipsec phase2-interface
    edit "VPN-HQ-BR-P2"
        set phase1name "VPN-HQ-BR"
        set proposal aes256-sha256
        set src-addr-type name
        set src-name "LAN-Subnet"
        set dst-addr-type name
        set dst-name "BR-LAN-Subnet"
    next
end

config router static
    edit 10
        set dst 10.20.20.0 255.255.255.0
        set device "VPN-HQ-BR"
        set gateway 10.255.0.2
    next
end

config firewall policy
    edit 40
        set name "LAN-to-BR-VPN-Allow"
        set srcintf "port2"
        set dstintf "VPN-HQ-BR"
        set srcaddr "LAN-Subnet"
        set dstaddr "BR-LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Implementasi Tahap 5 — SSL VPN dan Authentication

```
config user local
    edit "vpn-user-01"
        set type password
        set passwd <kata-sandi-kuat>
    next
end

config user group
    edit "SSL-VPN-Users"
        set member "vpn-user-01"
    next
end

config vpn ssl settings
    set servercert "Fortinet_SSL"
    set tunnel-ip-pools "SSLVPN-TUNNEL-ADDR1"
    set port 10443
    set source-interface "port1"
    set source-address "all"
    set default-portal "full-access"
end

config firewall policy
    edit 50
        set name "SSLVPN-to-LAN-Allow"
        set srcintf "ssl.root"
        set dstintf "port2"
        set srcaddr "SSLVPN-TUNNEL-ADDR1"
        set dstaddr "LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set groups "SSL-VPN-Users"
        set logtraffic all
    next
end
```

### Implementasi Tahap 6 — Profile Security yang Benar

Contoh berikut menggunakan sintaks yang valid pada FortiOS v7. Pada FortiOS v6, struktur `config entries` pada IPS menggunakan pendekatan yang sama, tetapi opsi `block-malicious-url` dan `scan-botnet-connections` diatur pada level sensor.

```
config ips sensor
    edit "default"
        set block-malicious-url enable
        set scan-botnet-connections block
        config entries
            edit 1
                set location server
                set severity high
                set action block
                set status enable
            next
            edit 2
                set location client
                set severity high
                set action block
                set status enable
            next
        end
    next
end

config webfilter profile
    edit "default"
        config ftgd-wf
            config filters
                edit 1
                    set category 2
                    set action block
                next
            end
        end
    next
end

config antivirus profile
    edit "default"
        set feature-set proxy
        config http
            set options scan
        end
        config ftp
            set options scan
        end
    next
end
```

### Implementasi Tahap 7 — High Availability

```
config system ha
    set group-name "HQ-HA"
    set mode a-p
    set password <kata-sandi-ha>
    set hbdev "port5" 50
    set session-pickup enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "port4"
            set gateway 192.168.99.1
        next
    end
    set override disable
    set priority 200
end
```

### Operasi dan Verifikasi

```
get firewall policy
diagnose firewall iprope list 100015
diagnose vpn tunnel list
diagnose vpn ipsec status
diagnose vpn ssl list
get system ha status
diagnose sys ha status
diagnose firewall proute list
```

### Studi Kasus

**Situasi:** Sebuah kantor cabang tidak dapat mengakses server di pusat setelah perubahan firewall policy. User di pusat tetap dapat mengakses internet.

**Analisis:** Pemeriksaan `diagnose debug flow` menunjukkan traffic dari cabang ditolak oleh policy implisit. Address cabang belum dimasukkan ke address object pada policy VPN. Selain itu, static route menuju cabang mengarah ke interface yang salah.

**Penyelesaian:** Address object cabang diperbaiki. Static route diperbaiki ke interface VPN. Policy VPN diperiksa ulang dan ditempatkan sebelum policy deny. Pengujian dilakukan dari cabang ke server pusat dan sebaliknya. Perubahan didokumentasikan.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| Traffic ditolak padahal policy ada | Urutan policy salah | Tempatkan policy spesifik sebelum policy umum |
| NAT tidak bekerja | VIP bertabrakan dengan policy | Periksa VIP, policy, dan route |
| VPN Phase 1 gagal | PSK atau proposal tidak cocok | Samakan proposal dan kunci pada kedua sisi |
| VPN Phase 2 gagal | Selector traffic tidak cocok | Samakan network sumber dan tujuan |
| HA tidak sinkron | Versi firmware berbeda | Samakan versi dan periksa heartbeat |
| SSL VPN login gagal | Grup user salah | Periksa grup pada policy dan portal |

## Modul 5 — NSE 5: Secure Networking atau SASE Administrator

### Tujuan

Peserta mampu mengelola FortiGate secara terpusat melalui FortiManager, mengelola log melalui FortiAnalyzer, mengelola endpoint melalui FortiClient EMS, serta mengimplementasikan akses aman berbasis identitas dan lokasi.

### Konsep Kunci

| Konsep | Penjelasan |
| --- | --- |
| ADOM | Administrative Domain untuk pemisahan policy antar unit atau pelanggan |
| Policy Package | Kumpulan policy dan objek yang diinstal ke FortiGate |
| FortiClient EMS | Management terpusat untuk FortiClient pada endpoint |
| ZTNA | Zero Trust Network Access berbasis identitas dan postur device |
| SASE | Penggabungan SD-WAN dan cloud security dalam satu layanan |

### Implementasi Tahap 1 — FortiManager

1. Buat ADOM sesuai struktur organisasi. Contoh: ADOM HQ, ADOM Branch.
2. Tambahkan FortiGate ke FortiManager. Pastikan FortiGate berada dalam mode managed.
3. Buat address object dan layanan bersama di ADOM.
4. Buat Policy Package dan instal ke FortiGate target.
5. Aktifkan revisi dan lock sebelum instalasi untuk audit perubahan.

```
config system central-management
    set type fortimanager
    set fmg "192.168.99.10"
end
```

### Implementasi Tahap 2 — FortiAnalyzer

1. Tambahkan FortiGate sebagai device terkelola di FortiAnalyzer.
2. Aktifkan pengiriman log realtime.
3. Buat laporan harian untuk policy yang ditolak dan threat teratas.
4. Buat alert untuk peristiwa kritis seperti kegagalan login administrator dan deteksi IPS kritis.

```
config log fortianalyzer setting
    set status enable
    set server 192.168.99.11
    set upload-option realtime
    set reliable enable
end
```

### Implementasi Tahap 3 — FortiClient EMS dan ZTNA

1. Buat endpoint profile di FortiClient EMS. Tentukan status antivirus, enkripsi disk, dan versi minimum.
2. Buat tag endpoint berdasarkan kepatuhan. Contoh: tag `Compliant` untuk endpoint yang memenuhi sindikasird.
3. Buat policy ZTNA di FortiGate yang hanya mengizinkan tag `Compliant`.
4. Uji akses dari endpoint yang patuh dan endpoint yang tidak patuh.

```
config firewall access-proxy
    edit "ZTNA-Server"
        set vip "VIP-ZTNA"
        set server-host "172.16.10.10"
    next
end

config firewall proxy-policy
    edit 10
        set name "ZTNA-to-DMZ-Allow"
        set proxy access-proxy
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "ZTNA-Server"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set groups "ZTNA-Users"
        set logtraffic all
    next
end
```

### Operasi dan Verifikasi

```
diagnose fmg-cwm show
diagnose log test
diagnose endpoint record list
diagnose wad user list
get system central-management
```

Periksa status instalasi Policy Package di FortiManager. Pastikan tidak ada FortiGate yang tertinggal pada versi policy lama.

### Studi Kasus

**Situasi:** Tiga puluh FortiGate cabang perlu menerima perubahan policy web filter dalam satu jendela pemeliharaan. Perubahan manual ke setiap device memakan waktu dan rawan kesalahan.

**Analisis:** Setiap FortiGate dikelola secara mandiri sehingga perubahan tidak konsisten. Beberapa cabang menggunakan profile lama.

**Penyelesaian:** Seluruh FortiGate dipindahkan ke FortiManager. Policy Package dibuat dan diinstal secara bertahap, dimulai dari dua cabang percontohan. Setelah verifikasi, instalasi dilanjutkan ke seluruh cabang. FortiAnalyzer digunakan untuk memverifikasi bahwa log web filter konsisten di seluruh cabang.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| FortiGate offline di FortiManager | Konektivitas atau sertifikat | Periksa route, port 541, dan authorization |
| Instalasi Policy Package gagal | Objek tidak ditemukan | Periksa objek yang digunakan pada policy |
| Log tidak muncul di FortiAnalyzer | ADOM tidak cocok | Samakan ADOM dan device terkelola |
| ZTNA ditolak | Tag endpoint tidak sesuai | Periksa profile EMS dan tag endpoint |

## Modul 6 — NSE 6: Advanced Endpoint Protection dan Data Security

### Tujuan

Peserta mampu mengimplementasikan perlindungan endpoint, email security, web application security, kontrol akses network, dan perlindungan data sesuai dengan produk spesifik yang digunakan.

### Konsep Kunci

| Produk | Fungsi Utama |
| --- | --- |
| FortiEDR | Deteksi dan respons endpoint berbasis perilaku |
| FortiMail | Security email, anti spam, anti phishing, dan enkripsi |
| FortiWeb | Web Application Firewall untuk aplikasi web |
| FortiNAC | Kontrol akses network berbasis identitas dan postur device |
| FortiDLP | Pencegahan kehilangan data dan klasifikasi data sensitif |

### Implementasi FortiEDR

1. Pasang agen FortiEDR pada endpoint percontohan.
2. Aktifkan mode simulasi selama tujuh hari untuk memetakan perilaku normal.
3. Buat policy isolasi untuk endpoint yang terinfeksi.
4. Hubungkan FortiEDR ke Security Fabric untuk berbagi intelijen.

Verifikasi:

```
diagnose endpoint edr status
```

### Implementasi FortiMail

1. Konfigurasikan FortiMail sebagai gateway email.
2. Aktifkan pemeriksaan SPF, DKIM, dan DMARC.
3. Buat policy anti spam dan anti phishing.
4. Buat policy enkripsi untuk email yang memuat data sensitif.

Konfigurasi pada FortiGate yang terintegrasi dengan FortiMail dilakukan melalui policy dan profile, bukan melalui perintah `mailsetting` pada FortiGate. Perintah tersebut hanya berlaku pada FortiMail.

### Implementasi FortiWeb

1. Buat profile perlindungan untuk aplikasi web.
2. Aktifkan perlindungan SQL injection dan cross-site scripting.
3. Aktifkan pembatasan laju permintaan untuk mencegah brute force.
4. Uji dengan attack simulasi pada lingkungan staging sebelum produksi.

Konfigurasi FortiWeb yang diterapkan:

```
Server Pool: WEB-POOL -> 172.16.10.10:443
Virtual Server: WEB-VS -> 203.0.113.2:443
WAF Profile: OWASP-Top10-Block
Signature: High Severity Block, Medium Severity Alert
```

### Implementasi FortiNAC

1. Hubungkan FortiNAC ke switch dan FortiGate.
2. Buat profile device untuk workstation, printer, kamera, dan IoT device.
3. Buat policy akses berdasarkan peran. Device IoT hanya dapat mengakses server yang ditentukan.
4. Aktifkan penilaian postur untuk workstation.

### Implementasi FortiDLP

1. Klasifikasikan data sensitif. Contoh: nomor identitas, data keuangan, data kesehatan.
2. Buat policy yang memblokir pengiriman data sensitif melalui email dan unggahan web.
3. Aktifkan mode monitor sebelum mode block.

### Operasi dan Verifikasi

- Periksa dashboard FortiEDR setiap hari untuk endpoint yang terisolasi.
- Periksa karantina FortiMail dan lepaskan pesan yang sah setelah verifikasi.
- Periksa log FortiWeb untuk attack yang diblokir dan false positive.
- Periksa port yang tidak terotorisasi pada FortiNAC.

### Studi Kasus

**Situasi:** Email phishing lolos ke kotak masuk keuangan. Email tersebut menyerupai email direksi dan meminta transfer dana. Di sisi lain, aplikasi web menerima upaya SQL injection.

**Analisis:** FortiMail belum mengaktifkan DMARC. FortiWeb masih dalam mode monitor sehingga attack tidak diblokir.

**Penyelesaian:** DMARC diaktifkan dalam mode reject setelah periode monitor. Policy FortiMail diperketat untuk email yang menyerupai domain internal. FortiWeb diubah ke mode block untuk aturan SQL injection. Pelatihan phishing dilakukan untuk unit keuangan.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| FortiEDR memblokir aplikasi sah | Policy terlalu ketat | Buat pengecualian berbasis hash aplikasi |
| Email sah masuk karantina | Skor spam terlalu agresif | Sesuaikan ambang dan daftar pengirim tepercaya |
| FortiWeb memblokir legitimate user | False positive | Periksa log dan sesuaikan aturan |
| Device IoT tidak mendapat akses | Profile device salah | Periksa klasifikasi FortiNAC |


## Modul 7 — NSE 7: Specialized Expert Tracks

### Tujuan

Peserta mampu merancang SD-WAN, cloud security, OT security, dan firewall enterprise, serta melakukan troubleshooting pada arsitektur yang kompleks.

### Konsep Kunci

| Jalur | Fokus |
| --- | --- |
| Enterprise Firewall | Desain policy skala besar, segmentasi, dan troubleshooting mendalam |
| SD-WAN | Pemilihan jalur berbasis kinerja, BGP, dan overlay |
| Cloud Security | FortiGate VM, FortiWeb Cloud, security multi cloud |
| OT Security | Segmentasi network operasional, FortiGate Rugged, dan protokol industri |
| LAN Edge | FortiSwitch, FortiAP, dan FortiNAC dalam satu fabric |

### Implementasi SD-WAN

1. Tentukan anggota SD-WAN. Contoh: port1 sebagai ISP utama, port5 sebagai ISP cadangan.
2. Tentukan Performance SLA. Contoh: latensi maksimum 100 ms dan packet loss maksimum 2 persen.
3. Buat aturan SD-WAN. Aplikasi bisnis diprioritaskan ke jalur dengan latensi terendah. Traffic backup diarahkan ke jalur dengan biaya terendah.
4. Buat firewall policy dari SD-WAN zone ke LAN zone.

```
config system sdwan
    set status enable
    config zone
        edit "SDWAN-Zone"
        next
    end
    config members
        edit 1
            set interface "port1"
            set gateway 203.0.113.1
        next
        edit 2
            set interface "port5"
            set gateway 203.0.113.9
        next
    end
    config health-check
        edit "SLA-Internet"
            set server "8.8.8.8"
            set members 1 2
            config sla
                edit 1
                    set latency-threshold 100
                    set jitter-threshold 30
                    set packetloss-threshold 2
                next
            end
        next
    end
    config service
        edit 10
            set name "Business-App-Best-Quality"
            set mode sla
            set dst "all"
            set src "LAN-Subnet"
            config sla
                edit "SLA-Internet"
                    set id 1
                next
            end
            set priority-members 1 2
        next
        edit 20
            set name "Backup-Low-Cost"
            set mode manual
            set dst "all"
            set src "LAN-Subnet"
            set priority-members 2 1
        next
    end
end

config router bgp
    set as 65001
    set router-id 10.10.10.1
    config neighbor
        edit "10.255.0.2"
            set remote-as 65002
        next
    end
    config network
        edit 1
            set prefix 10.10.10.0 255.255.255.0
        next
    end
end
```

### Implementasi Security Cloud

1. Sebar FortiGate VM di cloud dengan dua interface: public interface dan private interface.
2. Terapkan policy yang sama dengan pusat melalui FortiManager.
3. Aktifkan log ke FortiAnalyzer Cloud.
4. Terapkan segmentasi antara VPC produksi dan VPC pengembangan.

### Implementasi Security OT

1. Segmentasikan network IT dan OT melalui zone dan policy deny by default.
2. Izinkan hanya protokol industri yang diperlukan. Contoh: Modbus TCP hanya dari workstation SCADA ke PLC.
3. Terapkan IPS dengan signature OT.
4. Batasi akses management OT hanya dari jump host.

### Operasi dan Verifikasi

```
diagnose sys sdwan member
diagnose sys sdwan service
diagnose sys sdwan health-check
get router info bgp summary
get router info routing-table all
diagnose firewall iprope list
diagnose debug flow filter daddr 172.16.10.10
```

### Studi Kasus SD-WAN

**Situasi:** Kantor cabang menggunakan dua ISP. Aplikasi video conference sering terputus ketika ISP utama mengalami latensi tinggi. Failover manual memakan waktu hingga 10 menit.

**Analisis:** Pemilihan jalur masih berbasis static route. Tidak ada Performance SLA sehingga FortiGate tidak mengetahui penurunan kualitas jalur.

**Penyelesaian:** SD-WAN diaktifkan dengan Performance SLA. Aplikasi video conference diarahkan ke jalur dengan latensi terendah secara otomatis. Failover terjadi dalam hitungan detik. BGP digunakan untuk pertukaran route dengan pusat. Setelah implementasi, gangguan video conference menurun dan waktu failover menjadi kurang dari lima detik.

### Penyelesaian Masalah

| Gejala | Penyebab yang Mungkin | Tindakan |
| --- | --- | --- |
| SD-WAN tidak berpindah jalur | SLA tidak terpenuhi | Periksa health-check dan ambang SLA |
| BGP tidak terbentuk | AS atau address tetangga salah | Periksa remote-as dan konektivitas |
| Traffic OT terblokir | Protokol belum diizinkan | Periksa policy dan signature IPS |
| Biaya WAN meningkat | Aturan SD-WAN salah | Periksa prioritas anggota dan mode |


## Modul 8 — NSE 8: Expert Design dan Troubleshooting

### Tujuan

Peserta mampu merancang arsitektur security end to end, mempertahankannya dalam session desain, serta memecahkan masalah kompleks yang melibatkan banyak produk dan banyak situs.

### Konsep Kunci

| Tahap | Pertanyaan Kunci |
| --- | --- |
| Definisikan Masalah | Apa yang gagal, kapan mulai, siapa yang terdampak, dan apa yang berubah |
| Kumpulkan Bukti | Log, FortiView, debug flow, status HA, status BGP, status VPN |
| Bentuk Hipotesis | Susun tiga hipotesis yang paling mungkin berdasarkan bukti |
| Uji Hipotesis | Uji satu per satu, mulai dari yang paling mudah dibuktikan |
| Perbaiki dan Verifikasi | Perbaiki akar masalah, bukan gejala, dan uji dari sisi user |
| Dokumentasikan | Catat akar masalah, perubahan, dan pencegahan |

### Implementasi Metodologi Perancangan

1. Pahami kebutuhan bisnis. Tentukan aplikasi kritis, toleransi gangguan, dan kewajiban kepatuhan.
2. Petakan arus traffic. Gambarkan arus dari user ke aplikasi, dari cabang ke pusat, dari cloud ke pusat data.
3. Tentukan model kepercayaan. Tentukan zone, segmentasi, dan policy identitas.
4. Pilih arsitektur. Tentukan penempatan FortiGate, FortiManager, FortiAnalyzer, FortiWeb, FortiMail, dan FortiNAC.
5. Tentukan ketahanan. Tentukan HA, redundansi WAN, redundansi daya, dan rencana recovery.
6. Tentukan operasi. Tentukan monitoring, backup, change management, dan pelatihan.

### Contoh Konfigurasi Desain

Desain tidak ditulis sebagai satu blok konfigurasi, melainkan sebagai kumpulan keputusan yang terdokumentasi. Contoh keputusan yang harus dicatat:

```
Keputusan 1: HA Active-Passive dipilih karena aplikasi ERP tidak mendukung
             load balancing stateful pada level firewall.
Keputusan 2: Flow-based inspection dipilih karena device harus menangani
             5.000 concurrent session dengan latensi rendah.
Keputusan 3: ZTNA dipilih untuk akses ERP jarak jauh, bukan SSL VPN penuh,
             karena akses harus dibatasi per aplikasi.
```

### Operasi dan Verifikasi

```
get system status
get system ha status
diagnose sys sdwan health-check
diagnose vpn tunnel list
diagnose log test
```

### Studi Kasus Komprehensif

**Situasi:** Sebuah organisasi dengan satu pusat dan sepuluh cabang mengalami tiga gejala sekaligus: video conference terputus, email tertunda, dan aplikasi ERP lambat. Gejala muncul setelah migrasi ke SD-WAN.

**Analisis Bertahap:**

- Video conference terputus menunjukkan masalah Performance SLA atau pemilihan jalur yang tidak tepat.
- Email tertunda menunjukkan kemungkinan FortiMail mengantre karena pemeriksaan DMARC yang baru diaktifkan.
- Aplikasi ERP lambat menunjukkan kemungkinan SSL inspection yang membebani FortiGate atau BGP route yang tidak optimal.

**Pengujian:**

- `diagnose sys sdwan health-check` menunjukkan latensi ISP utama naik hingga 250 ms, tetapi aturan SD-WAN masih memaksa video conference ke jalur tersebut.
- Antrean FortiMail menunjukkan penundaan karena server DNS eksternal tidak merespons.
- `diagnose debug flow` untuk ERP menunjukkan session melewati jalur yang lebih panjang karena preferensi BGP.

**Penyelesaian:**

- Aturan SD-WAN diperbaiki ke mode SLA sehingga video conference berpindah otomatis.
- Server DNS FortiMail diperbaiki dan ditambahkan DNS cadangan.
- Preferensi BGP disesuaikan dan route ERP diperpendek.
- Seluruh perubahan diuji dari cabang, pusat, dan cloud. Dokumentasi diperbarui dan prosedur migrasi diperbaiki.

### Penyelesaian Masalah Tingkat Expert

| Gejala | Pendekatan |
| --- | --- |
| Masalah intermiten | Korelasikan waktu kejadian dengan perubahan, log, dan metrik kinerja |
| Masalah hanya pada satu cabang | Bandingkan konfigurasi cabang tersebut dengan cabang yang sehat |
| Masalah hanya pada satu aplikasi | Isolasi dengan debug flow dan periksa security profile aplikasi tersebut |
| Masalah setelah perubahan | Kembalikan perubahan terakhir dan uji secara bertahap |


# Bagian 2 — Studi Kasus Implementasi dan Operasi Firewall

## Studi Kasus 1 — Implementasi FortiGate Baru untuk Kantor Pusat

### Situasi

Sebuah kantor pusat dengan 120 user belum memiliki firewall terpusat. Setiap divisi menggunakan router kecil tanpa inspection security. Traffic internet tidak tercatat, tidak ada pemisahan antara user network dan server, dan tidak ada configuration backup. Management meminta implementasi firewall yang mampu mengontrol akses internet, melindungi server, dan menyediakan log terpusat.

Kondisi awal yang dicatat:
- User: 120 workstation pada satu flat network 10.10.10.0/24
- Server: satu server file dan satu server aplikasi pada network yang sama dengan user
- Internet: satu ISP dengan address 203.0.113.2/30
- Device: FortiGate 100F dengan FortiOS 7.2, FortiManager dan FortiAnalyzer tersedia pada management network 192.168.99.0/24

### Tujuan

1. Memisahkan user network, server, dan management ke dalam zone yang berbeda.
2. Mengontrol akses internet berbasis policy dan security profile.
3. Mengaktifkan pencatatan log ke FortiAnalyzer secara waktu nyata.
4. Menyediakan prosedur backup dan recovery konfigurasi.

### Implementasi

**Tahap 1 — Interface dan Zone**

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
    edit "port4"
        set alias "MGMT"
        set mode static
        set ip 192.168.99.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
end

config system zone
    edit "LAN-Zone"
        set interface "port2"
    next
    edit "DMZ-Zone"
        set interface "port3"
    next
end
```

**Tahap 2 — Objek Address dan Policy Dasar**

```
config firewall address
    edit "LAN-Subnet"
        set subnet 10.10.10.0 255.255.255.0
    next
    edit "DMZ-Server"
        set subnet 172.16.10.10 255.255.255.255
    next
end

config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set application-list "default"
        set ssl-ssh-profile "certificate-inspection"
        set nat enable
        set logtraffic all
    next
    edit 20
        set name "LAN-to-DMZ-App-Allow"
        set srcintf "port2"
        set dstintf "port3"
        set srcaddr "LAN-Subnet"
        set dstaddr "DMZ-Server"
        set action accept
        set schedule "always"
        set service "HTTPS" "SMB"
        set logtraffic all
    next
    edit 30
        set name "DMZ-to-WAN-Deny"
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

**Tahap 3 — DNS, NTP, dan Logging**

```
config system dns
    set primary 8.8.8.8
    set secondary 1.1.1.1
end

config system ntp
    set ntpsync enable
    config ntpserver
        edit 1
            set server "pool.ntp.org"
        next
    end
end

config log fortianalyzer setting
    set status enable
    set server 192.168.99.11
    set upload-option realtime
    set reliable enable
end
```

**Tahap 4 — Backup**

```
execute backup config tftp HQ-FGT-Initial-2026-11-25.conf 192.168.99.20
```

Backup terjadwal diatur melalui FortiManager dengan retensi 30 hari.

### Operasi dan Verifikasi

```
get system status
get system interface physical
get firewall policy
diagnose firewall iprope list 100015
diagnose sys session list
diagnose log test
get log fortianalyzer setting
```

Verifikasi yang dilakukan:
- User dapat mengakses internet dan aplikasi internal.
- Server DMZ tidak dapat menginisiasi koneksi ke internet.
- Log traffic muncul di FortiAnalyzer dalam waktu kurang dari satu menit.
- File backup dapat diunduh dan dibuka.

### Hasil dan Evaluasi

Setelah implementasi, seluruh internet traffic tercatat dengan identitas policy dan security profile. Pemisahan zone mencegah akses langsung dari user network ke server management. Waktu penanganan gangguan menurun karena log tersedia secara terpusat. Backup konfigurasi memungkinkan recovery dalam waktu kurang dari 30 menit pada pengujian recovery.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| User tidak dapat mengakses internet | Policy belum mengizinkan DNS | Tambahkan layanan DNS pada policy dan periksa NAT |
| Log tidak muncul di FortiAnalyzer | Route ke management terblokir | Periksa route dan port 514 |
| Server tidak dapat diakses dari LAN | Address object salah | Periksa subnet DMZ dan policy |


## Studi Kasus 2 — Migrasi Firewall Lama ke FortiGate Tanpa Gangguan Layanan

### Situasi

Sebuah organisasi menggunakan firewall lama yang sudah mendekati akhir dukungan. Firewall tersebut memiliki 180 policy, 90 address object, dan 40 aturan NAT. Dokumentasi tidak lengkap. Management meminta migrasi ke FortiGate tanpa menghentikan layanan selama jam kerja.

Kondisi yang dicatat:
- Jam kerja: 08.00 sampai 17.00
- Jendela pemeliharaan yang tersedia: Sabtu 22.00 sampai Minggu 02.00
- Layanan kritis: aplikasi ERP, email, dan VPN cabang

### Tujuan

1. Memigrasikan seluruh policy, objek, dan NAT dengan hasil yang setara.
2. Meminimalkan gangguan layanan selama migrasi.
3. Menyediakan rencana rollback yang dapat dieksekusi dalam waktu kurang dari 15 menit.

### Implementasi

**Tahap 1 — Audit dan Pemetaan**

1. Ekspor konfigurasi firewall lama.
2. Identifikasi policy yang tidak pernah digunakan dalam 90 hari terakhir melalui log.
3. Kelompokkan policy berdasarkan fungsi: internet, server publik, VPN, management.
4. Buat matriks pemetaan: policy lama ke policy baru, objek lama ke objek baru, NAT lama ke VIP atau policy NAT baru.

Hasil audit: dari 180 policy, 42 policy tidak pernah digunakan dan tidak dimigrasikan setelah mendapat persetujuan tertulis.

**Tahap 2 — Pembangunan Paralel**

FortiGate baru dipasang secara paralel, tidak langsung menggantikan firewall lama. Konfigurasi dibangun di laboratorium, kemudian diuji dengan traffic tiruan.

```
config firewall vip
    edit "VIP-ERP"
        set extip 203.0.113.2
        set extintf "port1"
        set mappedip 172.16.10.20
        set portforward enable
        set extport 443
        set mappedport 443
    next
    edit "VIP-Mail"
        set extip 203.0.113.3
        set extintf "port1"
        set mappedip 172.16.10.21
        set portforward enable
        set extport 25
        set mappedport 25
    next
end
```

**Tahap 3 — Pengujian Pra Migrasi**

1. Uji setiap policy dengan sumber pengujian yang menyerupai user, server, dan cabang.
2. Uji VPN dengan satu cabang percontohan.
3. Uji akses server publik dari internet melalui address pengujian.
4. Catat hasil pengujian dan perbaiki policy yang gagal.

**Tahap 4 — Cutover**

1. Pada jendela pemeliharaan, nonaktifkan firewall lama dan aktifkan FortiGate sebagai gateway utama.
2. Pindahkan kabel WAN dan LAN sesuai rencana.
3. Periksa status interface, route, VPN, dan policy.
4. Lakukan pengujian layanan kritis: ERP, email, VPN, dan internet.

**Tahap 5 — Rollback**

Rencana rollback disiapkan sebelum cutover: kabel dikembalikan ke firewall lama, FortiGate dinonaktifkan, dan layanan diverifikasi. Rollback hanya dilakukan apabila dua atau lebih layanan kritis gagal setelah 30 menit perbaikan.

### Operasi dan Verifikasi

```
diagnose debug flow filter addr 10.10.10.50
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug disable
diagnose vpn tunnel list
diagnose sys session list
get firewall policy
```

Verifikasi pasca migrasi dilakukan setiap 30 menit selama 4 jam pertama, kemudian setiap 2 jam selama 24 jam berikutnya.

### Hasil dan Evaluasi

Migrasi selesai dalam 2 jam 10 menit. Seluruh layanan kritis berfungsi. Dua policy perlu diperbaiki karena address object yang tidak lengkap. Waktu penanganan gangguan membaik karena FortiView menyediakan visibilitas yang tidak tersedia pada firewall lama. Policy yang tidak digunakan tidak dimigrasikan sehingga jumlah policy menurun dari 180 menjadi 138 dan audit menjadi lebih mudah.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| Aplikasi ERP lambat setelah migrasi | Profile IPS memblokir ERP traffic | Buat pengecualian IPS untuk server ERP setelah verifikasi |
| VPN cabang tidak terbentuk | Proposal tidak cocok | Samakan proposal Phase 1 dan Phase 2 |
| Email tertunda | Policy SMTP belum mengizinkan relay | Periksa VIP dan policy WAN ke DMZ |


## Studi Kasus 3 — Implementasi High Availability dan SD-WAN Dual ISP

### Situasi

Sebuah kantor cabang dengan 60 user hanya menggunakan satu ISP. Ketika ISP mengalami gangguan, seluruh layanan internet dan VPN terhenti. Kantor tersebut juga hanya menggunakan satu FortiGate sehingga kegagalan device menyebabkan seluruh network terhenti. Management meminta ketersediaan tinggi pada device dan jalur internet.

Kondisi awal:
- FortiGate tunggal
- Satu ISP: 203.0.113.6/30
- VPN IPsec ke pusat sering terputus saat ISP mengalami latensi tinggi

### Tujuan

1. Mengimplementasikan HA Active-Passive untuk FortiGate.
2. Menambahkan ISP kedua sebagai jalur cadangan dan jalur berbasis kinerja.
3. Mengaktifkan SD-WAN dengan Performance SLA untuk aplikasi bisnis.

### Implementasi

**Tahap 1 — High Availability**

```
config system ha
    set group-name "BR-HA"
    set mode a-p
    set password <kata-sandi-ha>
    set hbdev "port5" 50
    set session-pickup enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "port4"
            set gateway 192.168.99.1
        next
    end
    set override disable
    set priority 200
end
```

Pada FortiGate sekunder, konfigurasi sama dengan `priority 100`.

**Tahap 2 — SD-WAN Dual ISP**

```
config system sdwan
    set status enable
    config zone
        edit "SDWAN-Zone"
        next
    end
    config members
        edit 1
            set interface "port1"
            set gateway 203.0.113.5
        next
        edit 2
            set interface "port6"
            set gateway 203.0.113.9
        next
    end
    config health-check
        edit "SLA-Internet"
            set server "8.8.8.8"
            set members 1 2
            config sla
                edit 1
                    set latency-threshold 100
                    set jitter-threshold 30
                    set packetloss-threshold 2
                next
            end
        next
    end
    config service
        edit 10
            set name "Video-Conference-Best-Quality"
            set mode sla
            set dst "all"
            set src "LAN-Subnet"
            config sla
                edit "SLA-Internet"
                    set id 1
                next
            end
            set priority-members 1 2
        next
        edit 20
            set name "Backup-Low-Cost"
            set mode manual
            set dst "all"
            set src "LAN-Subnet"
            set priority-members 2 1
        next
    end
end
```

**Tahap 3 — Policy dari Zone SD-WAN**

```
config firewall policy
    edit 100
        set name "LAN-to-SDWAN-Internet-Allow"
        set srcintf "port2"
        set dstintf "SDWAN-Zone"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set nat enable
        set logtraffic all
    next
end
```

### Operasi dan Verifikasi

```
get system ha status
diagnose sys ha status
diagnose sys sdwan member
diagnose sys sdwan health-check
diagnose sys sdwan service
diagnose vpn tunnel list
```

Pengujian failover dilakukan dengan dua cara:
- Failover device: matikan FortiGate primer, pastikan FortiGate sekunder mengambil alih dalam waktu kurang dari 10 detik dan session yang ada tetap berjalan.
- Failover jalur: simulasikan latensi tinggi pada ISP utama, pastikan aplikasi video conference berpindah ke ISP kedua secara otomatis.

### Hasil dan Evaluasi

Setelah implementasi, gangguan ISP tunggal tidak lagi menghentikan layanan. Video conference berpindah jalur dalam waktu kurang dari 5 detik ketika latensi ISP utama melebihi ambang. Failover device berhasil tanpa kehilangan session yang signifikan. Waktu henti yang sebelumnya dapat mencapai 4 jam per insiden menurun menjadi kurang dari 1 menit untuk gangguan device dan jalur.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| HA tidak sinkron | Versi firmware berbeda | Samakan versi FortiOS pada kedua device |
| SD-WAN tidak berpindah | Health-check tidak berjalan | Periksa gateway dan server health-check |
| Session terputus saat failover | Session pickup tidak aktif | Aktifkan session-pickup pada konfigurasi HA |


## Studi Kasus 4 — Operasi Harian: Internet Lambat dan Tabel Session Penuh

### Situasi

Pada pukul 09.15, user melaporkan internet lambat dan beberapa aplikasi tidak dapat dibuka. Tim NOC memeriksa FortiGate dan menemukan penggunaan CPU 92 persen dan session table mendekati batas. Tidak ada perubahan konfigurasi pada malam sebelumnya.

### Tujuan

1. Mengidentifikasi sumber session berlebihan.
2. Memulihkan layanan tanpa menghentikan seluruh network.
3. Mencegah kejadian berulang melalui pembatasan session dan monitoring.

### Implementasi Operasi

**Tahap 1 — Identifikasi**

```
get system performance status
get system session status
diagnose sys session list
diagnose sys top 1 20
```

Hasil pemeriksaan:
- Satu workstation 10.10.10.77 menghasilkan 45.000 session ke banyak address eksternal.
- FortiView menunjukkan aplikasi tidak dikenal dan traffic ke port yang tidak umum.
- Log IPS menunjukkan percobaan koneksi berulang.

**Tahap 2 — Isolasi yang Benar**

Workstation diisolasi dari network melalui FortiNAC atau dengan memindahkan port switch ke VLAN karantina. Session dari workstation tersebut dibersihkan dengan filter, bukan dengan pembersihan seluruh session table.

```
diagnose sys session filter src 10.10.10.77
diagnose sys session clear
diagnose sys session filter clear
```

Perintah `diagnose sys session clear` tanpa filter tidak digunakan karena akan memutus seluruh user session yang sah.

**Tahap 3 — Pembatasan Session yang Benar**

Pembatasan dilakukan melalui DoS Policy dan Traffic Shaper yang diterapkan pada policy, bukan melalui perintah yang tidak valid.

```
config firewall DoS-policy
    edit 1
        set interface "port2"
        set srcaddr "all"
        set dstaddr "all"
        set service "ALL"
        config anomaly
            edit "tcp_syn_flood"
                set status enable
                set log enable
                set threshold 1000
            next
            edit "udp_flood"
                set status enable
                set log enable
                set threshold 1000
            next
        end
    next
end

config firewall traffic-shaper
    edit "Limit-Per-Policy"
        set maximum-bandwidth 50000
    next
end

config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set logtraffic all
        set nat enable
        set traffic-shaper "Limit-Per-Policy"
        set traffic-shaper-reverse "Limit-Per-Policy"
    next
end
```

### Operasi dan Verifikasi

```
diagnose sys session stat
diagnose firewall iprope list 100015
diagnose log test
get system performance status
```

Setelah isolasi, CPU turun ke 28 persen dalam 5 menit. User dapat mengakses internet kembali. Workstation diserahkan ke tim security untuk analisis endpoint.

### Hasil dan Evaluasi

Gangguan berlangsung 35 menit. Tanpa isolasi yang cepat, gangguan dapat berlangsung lebih lama dan memengaruhi seluruh user. Setelah insiden, session monitoring per source address ditambahkan ke dashboard harian. Ambang alert diatur pada 70 persen penggunaan CPU dan 80 persen penggunaan session table.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| CPU tinggi berulang | Satu endpoint terinfeksi | Isolasi endpoint dan periksa FortiEDR |
| Session tidak berkurang | Session lama tidak kedaluwarsa | Sesuaikan waktu kedaluwarsa session |
| User sah ikut terblokir | Pembatasan terlalu ketat | Sesuaikan ambang DoS dan traffic shaper |


## Studi Kasus 5 — Operasi Security: Deteksi Malware dan Respons Insiden

### Situasi

FortiAnalyzer mengirim alert pada pukul 14.20: deteksi IPS kritis dari workstation 10.10.10.88. User tersebut baru saja membuka lampiran email. Dalam waktu 10 menit, tiga workstation lain menunjukkan alert yang serupa.

### Tujuan

1. Menghentikan penyebaran malware.
2. Mengidentifikasi sumber dan cakupan insiden.
3. Memulihkan workstation yang terdampak dan memperketat kontrol.

### Implementasi Operasi

**Tahap 1 — Triase**

```
diagnose log test
get log fortianalyzer setting
```

Pemeriksaan log menunjukkan:
- Pengirim email menyerupai domain pemasok.
- Lampiran mengandung makro.
- Workstation yang terinfeksi mencoba menghubungi domain command and control.

**Tahap 2 — Blokir dan Isolasi**

Domain command and control diblokir melalui Web Filter dan DNS Filter.

```
config firewall address
    edit "C2-Blocked"
        set type fqdn
        set fqdn "malicious-example.com"
    next
end

config dnsfilter profile
    edit "default"
        config domain-filter
            edit 1
                set name "Block-C2"
                set domain "malicious-example.com"
                set action block
            next
        end
    next
end

config webfilter profile
    edit "default"
        config ftgd-wf
            config filters
                edit 1
                    set category 2
                    set action block
                next
            end
        end
    next
end
```

Workstation yang terinfeksi diisolasi melalui FortiNAC dan FortiEDR.

**Tahap 3 — Recovery**

1. Workstation diisolasi dan dipindai.
2. Akun user dinonaktifkan sementara.
3. Password user diatur ulang.
4. Workstation dipulihkan dari citra bersih, bukan dari backup yang terkontaminasi.

### Operasi dan Verifikasi

```
diagnose endpoint record list
diagnose wad user list
diagnose sys session list
```

Verifikasi dilakukan dengan memeriksa:
- Tidak ada lagi koneksi ke domain command and control pada log.
- Tidak ada alert IPS baru dari workstation yang sama.
- Endpoint dinyatakan bersih oleh FortiEDR.

### Hasil dan Evaluasi

Penyebaran berhasil dihentikan pada empat workstation. Tidak ada server yang terinfeksi. Waktu respons dari alert pertama hingga isolasi adalah 18 menit. Setelah insiden, policy FortiMail diperketat untuk memblokir makro, dan pelatihan phishing diulang untuk unit yang terdampak.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| Alert berulang setelah isolasi | Domain C2 belum diblokir | Blokir domain pada DNS Filter dan Web Filter |
| Workstation bersih tetapi alert tetap muncul | Session lama masih aktif | Bersihkan session dengan filter dan periksa ulang log |
| Email phishing lolos | DMARC belum reject | Aktifkan DMARC reject setelah periode monitor |


## Studi Kasus 6 — Implementasi VPN Antar Cabang dan Akses Jarak Jauh

### Situasi

Sebuah organisasi memiliki kantor pusat dan tiga kantor cabang. Setiap cabang menggunakan ISP berbeda. Karyawan yang bekerja dari luar kantor membutuhkan akses ke aplikasi ERP di pusat. Sebelumnya, cabang menggunakan VPN manual yang tidak terpusat dan sulit dipantau.

### Tujuan

1. Membangun VPN IPsec antar situs yang terpusat dan terpantau.
2. Menyediakan SSL VPN untuk akses jarak jauh berbasis grup user.
3. Membatasi akses jarak jauh hanya ke aplikasi yang diperlukan.

### Implementasi

**Tahap 1 — IPsec VPN Pusat ke Cabang**

```
config vpn ipsec phase1-interface
    edit "VPN-HQ-BR1"
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal aes256-sha256
        set remote-gw 203.0.113.6
        set psksecret <kunci-bersama-yang-kuat>
    next
end

config vpn ipsec phase2-interface
    edit "VPN-HQ-BR1-P2"
        set phase1name "VPN-HQ-BR1"
        set proposal aes256-sha256
        set src-addr-type name
        set src-name "LAN-Subnet"
        set dst-addr-type name
        set dst-name "BR1-LAN-Subnet"
    next
end

config router static
    edit 10
        set dst 10.20.20.0 255.255.255.0
        set device "VPN-HQ-BR1"
        set gateway 10.255.0.2
    next
end

config firewall policy
    edit 40
        set name "LAN-to-BR1-VPN-Allow"
        set srcintf "port2"
        set dstintf "VPN-HQ-BR1"
        set srcaddr "LAN-Subnet"
        set dstaddr "BR1-LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
    edit 41
        set name "BR1-to-LAN-VPN-Allow"
        set srcintf "VPN-HQ-BR1"
        set dstintf "port2"
        set srcaddr "BR1-LAN-Subnet"
        set dstaddr "LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

**Tahap 2 — SSL VPN untuk Karyawan Jarak Jauh**

```
config user local
    edit "remote-user-01"
        set type password
        set passwd <kata-sandi-kuat>
    next
end

config user group
    edit "SSL-VPN-ERP-Users"
        set member "remote-user-01"
    next
end

config vpn ssl settings
    set servercert "Fortinet_SSL"
    set tunnel-ip-pools "SSLVPN-TUNNEL-ADDR1"
    set port 10443
    set source-interface "port1"
    set source-address "all"
    set default-portal "ERP-Only"
end

config firewall policy
    edit 50
        set name "SSLVPN-to-ERP-Allow"
        set srcintf "ssl.root"
        set dstintf "port3"
        set srcaddr "SSLVPN-TUNNEL-ADDR1"
        set dstaddr "ERP-Server"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set groups "SSL-VPN-ERP-Users"
        set logtraffic all
    next
end
```

**Tahap 3 — Authentication Dua Faktor**

MFA diaktifkan untuk seluruh SSL VPN user melalui FortiAuthenticator atau penyedia MFA yang digunakan organisasi. User yang belum mengaktifkan MFA tidak dapat menyelesaikan login.

### Operasi dan Verifikasi

```
diagnose vpn tunnel list
diagnose vpn ipsec status
diagnose vpn ssl list
diagnose debug application sslvpn -1
diagnose debug enable
diagnose debug disable
```

Verifikasi:
- Terowongan IPsec aktif dan stabil selama 72 jam.
- User SSL VPN hanya dapat mengakses server ERP, tidak dapat mengakses seluruh LAN.
- Log VPN tercatat di FortiAnalyzer dengan identitas user.

### Hasil dan Evaluasi

Seluruh cabang terhubung melalui VPN yang terpantau. Waktu penanganan gangguan VPN menurun karena status terowongan dapat dilihat dari FortiManager. Akses jarak jauh menjadi lebih aman karena dibatasi berdasarkan grup dan aplikasi. User yang mencoba mengakses di luar jam kerja akan ditolak sesuai jadwal policy.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| Phase 1 gagal | PSK atau proposal tidak cocok | Samakan proposal dan kunci pada kedua sisi |
| Phase 2 gagal | Selector traffic tidak cocok | Samakan network sumber dan tujuan |
| SSL VPN login berhasil tetapi tidak dapat mengakses ERP | Policy salah | Periksa grup dan destination address pada policy |
| VPN sering terputus | DPD atau NAT timeout | Sesuaikan DPD dan waktu kedaluwarsa |

---

## Studi Kasus 7 — Implementasi DMZ, Server Publik, dan Proteksi Aplikasi Web

### Situasi

Sebuah organisasi menyediakan portal pelanggan melalui internet. Portal tersebut sebelumnya berada pada network yang sama dengan user internal. Scanning security menemukan kerentanan SQL injection dan percobaan brute force pada halaman login. Management meminta pemisahan DMZ dan proteksi aplikasi web.

### Tujuan

1. Memindahkan server web ke DMZ yang terisolasi.
2. Mengontrol akses dari internet ke DMZ melalui VIP dan policy yang ketat.
3. Mengaktifkan proteksi aplikasi web dan pembatasan laju permintaan.
4. Mencegah akses dari DMZ ke internal network kecuali yang diizinkan.

### Implementasi

**Tahap 1 — Pemindahan Server ke DMZ**

Server web dipindahkan ke 172.16.10.10 pada DMZ. Akses management ke server hanya diizinkan dari management network melalui jump host.

**Tahap 2 — VIP dan Policy WAN ke DMZ**

```
config firewall vip
    edit "VIP-Portal"
        set extip 203.0.113.2
        set extintf "port1"
        set mappedip 172.16.10.10
        set portforward enable
        set extport 443
        set mappedport 443
    next
end

config firewall policy
    edit 60
        set name "WAN-to-DMZ-Portal-Allow"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "VIP-Portal"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set utm-status enable
        set ips-sensor "default"
        set webfilter-profile "default"
        set ssl-ssh-profile "certificate-inspection"
        set logtraffic all
        set nat enable
    next
    edit 61
        set name "DMZ-to-LAN-DB-Limited"
        set srcintf "port3"
        set dstintf "port2"
        set srcaddr "DMZ-Server"
        set dstaddr "DB-Server"
        set action accept
        set schedule "always"
        set service "MySQL"
        set logtraffic all
    next
    edit 62
        set name "DMZ-to-WAN-Deny"
        set srcintf "port3"
        set dstintf "port1"
        set srcaddr "all"
        set dstaddr "all"
        set action deny
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

**Tahap 3 — FortiWeb**

FortiWeb ditempatkan di depan server web sebagai Web Application Firewall.

Konfigurasi FortiWeb yang diterapkan:
- Virtual Server: 203.0.113.2:443 ke Server Pool 172.16.10.10:443
- WAF Profile: OWASP Top 10 Block
- Signature: High Severity Block, Medium Severity Alert
- Rate Limit: maksimum 100 permintaan per menit per source address untuk halaman login
- Brute Force Protection: blokir source address setelah 5 upaya login gagal dalam 5 menit

**Tahap 4 — Pembatasan Akses Management**

```
config firewall policy
    edit 70
        set name "MGMT-to-DMZ-Admin-Allow"
        set srcintf "port4"
        set dstintf "port3"
        set srcaddr "MGMT-Subnet"
        set dstaddr "DMZ-Server"
        set action accept
        set schedule "always"
        set service "SSH" "HTTPS"
        set logtraffic all
    next
end
```

### Operasi dan Verifikasi

```
diagnose firewall iprope list 100015
diagnose sys session list
get firewall policy
```

Verifikasi yang dilakukan:
- Portal dapat diakses dari internet.
- Percobaan SQL injection diblokir oleh FortiWeb dan tercatat pada log.
- Server DMZ tidak dapat mengakses internet secara langsung.
- Server DMZ hanya dapat mengakses database pada port yang diizinkan.

### Hasil dan Evaluasi

Setelah implementasi, upaya SQL injection menurun dan tidak ada lagi upaya yang berhasil mencapai server. Brute force pada halaman login berhasil dibatasi. Audit security menyatakan pemisahan DMZ telah memenuhi sindikasird. Waktu respons terhadap web application attack meningkat karena log FortiWeb tersedia di FortiAnalyzer.

### Penyelesaian Masalah

| Gejala | Penyebab | Tindakan |
| --- | --- | --- |
| Portal tidak dapat diakses | VIP atau policy salah | Periksa VIP, policy, dan route |
| User sah diblokir FortiWeb | False positive | Periksa log FortiWeb dan sesuaikan aturan |
| Server DMZ dapat mengakses internet | Policy deny tidak berada di akhir | Tempatkan policy deny setelah policy yang diizinkan |
| Database tidak dapat diakses dari DMZ | Port tidak diizinkan | Periksa policy DMZ ke LAN |

# Bagian 3 — Contoh Konfigurasi FortiOS v6 dan v7

## Perbedaan Utama FortiOS v6 dan v7

| Aspek | FortiOS v6.4 | FortiOS v7.0, v7.2, v7.4 |
| --- | --- | --- |
| Mode Policy | Profile-Based dan Policy-Based tersedia, Profile-Based masih umum | Policy-Based semakin dianjurkan, management profile lebih terintegrasi |
| SSL VPN | Konfigurasi melalui `vpn ssl settings` dan `firewall policy` dengan `ssl.root` | Sintaks dasar sama, tetapi ZTNA dan Access Proxy semakin diprioritaskan sebagai pengganti SSL VPN untuk akses aplikasi |
| SD-WAN | `system sdwan` dengan `health-check` dan `service` | Struktur sama, penambahan `sla` yang lebih rinci, `performance SLA` dan integrasi dengan BGP lebih baik |
| ZTNA | Belum tersedia secara penuh pada v6.2, mulai diperkenalkan pada v6.4 | Tersedia penuh melalui `firewall access-proxy`, `proxy-policy`, dan tag endpoint |
| SSL inspection | `ssl-ssh-profile` dengan `certificate-inspection` dan `deep-inspection` | Konsep sama, penambahan dukungan TLS 1.3 dan profile yang lebih granular |
| FortiManager | ADOM berbasis versi, Policy Package | ADOM dan Policy Package tetap digunakan, dengan validasi policy yang lebih ketat |
| Log dan Diagnostik | `diagnose debug flow` dan `diagnose vpn` | Perintah diagnostik sama, penambahan `diagnose wad` dan `diagnose endpoint` untuk ZTNA |

Catatan penting: perbedaan sintaks pada dokumen ini difokuskan pada perubahan yang berdampak pada implementasi dan operasi. Perintah yang tidak berubah ditulis identik pada kedua versi agar mudah dibandingkan.

## Konvensi Penulisan

- `HQ-FGT` menunjukkan FortiGate pusat.
- Address yang digunakan bersifat contoh:
  - WAN: 203.0.113.2/30, gateway 203.0.113.1
  - LAN: 10.10.10.1/24
  - DMZ: 172.16.10.1/24
  - Management: 192.168.99.1/24, FortiManager 192.168.99.10, FortiAnalyzer 192.168.99.11
  - Cabang: LAN 10.20.20.0/24, WAN 203.0.113.6/30
- Setiap bagian menampilkan konfigurasi v6 terlebih dahulu, kemudian konfigurasi v7, kemudian catatan perbedaan.
- Nilai password dan pre-shared key ditulis sebagai `<kata-sandi-kuat>` dan harus diganti sesuai sindikasird organisasi.

## 1. Interface dan Zone

### FortiOS v6.4

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
end

config system zone
    edit "LAN-Zone"
        set interface "port2"
    next
    edit "DMZ-Zone"
        set interface "port3"
    next
end
```

### FortiOS v7.2 / v7.4

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
end

config system zone
    edit "LAN-Zone"
        set interface "port2"
    next
    edit "DMZ-Zone"
        set interface "port3"
    next
end
```

### Catatan Perbedaan

Konfigurasi interface dan zone identik pada v6 dan v7. Perbedaan terletak pada validasi GUI. Pada v7, FortiGate memberikan peringatan apabila interface yang sudah menjadi anggota zone digunakan secara langsung pada policy.

### Verifikasi

```
get system interface physical
get system zone
diagnose hardware deviceinfo nic port1
```

## 2. Objek Address dan Layanan

### FortiOS v6.4 dan v7.2 / v7.4 — Sintaks Sama

```
config firewall address
    edit "LAN-Subnet"
        set subnet 10.10.10.0 255.255.255.0
    next
    edit "DMZ-Server"
        set subnet 172.16.10.10 255.255.255.255
    next
    edit "BR-LAN-Subnet"
        set subnet 10.20.20.0 255.255.255.0
    next
    edit "ERP-Server"
        set subnet 172.16.10.20 255.255.255.255
    next
end

config firewall addrgrp
    edit "Internal-Servers"
        set member "DMZ-Server" "ERP-Server"
    next
end

config firewall service custom
    edit "ERP-HTTPS"
        set tcp-portrange 8443
    next
end
```

### Catatan Perbedaan

Sintaks address object tidak berubah. Pada v7, FortiManager melakukan validasi objek yang tidak digunakan dan memberikan rekomendasi pembersihan. Objek FQDN pada v7 mendukung pembaruan DNS yang lebih cepat.

### Verifikasi

```
get firewall address
get firewall addrgrp
diagnose firewall iprope list 100015
```

## 3. Firewall Policy dan NAT

### FortiOS v6.4 — Profile-Based

```
config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set application-list "default"
        set ssl-ssh-profile "certificate-inspection"
        set nat enable
        set logtraffic all
    next
end
```

### FortiOS v7.2 / v7.4 — Policy-Based yang Dianjurkan

```
config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set application-list "default"
        set ssl-ssh-profile "certificate-inspection"
        set nat enable
        set logtraffic all
    next
end
```

### Catatan Perbedaan

Perintah dasar policy sama. Perbedaan utama terletak pada cara FortiOS v7 menangani profile. Pada v7, FortiGate memberikan peringatan apabila policy menggunakan profile yang tidak konsisten dengan mode inspection. Pada v6, kesalahan tersebut sering baru terlihat pada log.

### Verifikasi

```
get firewall policy
diagnose firewall iprope list 100015
diagnose sys session list
diagnose debug flow filter addr 10.10.10.50
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug disable
```

## 4. Virtual IP dan IP Pool

### FortiOS v6.4 dan v7.2 / v7.4 — Sintaks Sama

```
config firewall vip
    edit "VIP-Web-DMZ"
        set extip 203.0.113.2
        set extintf "port1"
        set mappedip 172.16.10.10
        set portforward enable
        set extport 443
        set mappedport 443
    next
end

config firewall ippool
    edit "SNAT-Pool"
        set startip 203.0.113.10
        set endip 203.0.113.15
    next
end

config firewall policy
    edit 30
        set name "WAN-to-DMZ-Web-Allow"
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "VIP-Web-DMZ"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set utm-status enable
        set ips-sensor "default"
        set logtraffic all
        set nat enable
    next
end
```

### Catatan Perbedaan

Sintaks VIP dan IP Pool identik. Pada v7, FortiGate memberikan visualisasi yang lebih jelas pada GUI untuk VIP yang tidak memiliki policy yang merujuk. Hal ini membantu audit policy DNAT.

### Verifikasi

```
get firewall vip
get firewall ippool
diagnose firewall iprope list 100015
```

## 5. Route Statis dan BGP

### FortiOS v6.4 dan v7.2 / v7.4 — Sintaks Sama

```
config router static
    edit 1
        set dst 0.0.0.0 0.0.0.0
        set gateway 203.0.113.1
        set device "port1"
    next
    edit 10
        set dst 10.20.20.0 255.255.255.0
        set device "VPN-HQ-BR"
        set gateway 10.255.0.2
    next
end

config router bgp
    set as 65001
    set router-id 10.10.10.1
    config neighbor
        edit "10.255.0.2"
            set remote-as 65002
        next
    end
    config network
        edit 1
            set prefix 10.10.10.0 255.255.255.0
        next
    end
end
```

### Catatan Perbedaan

Konfigurasi static route dan BGP tidak berubah. Pada v7, integrasi BGP dengan SD-WAN lebih baik. Route BGP dapat digunakan sebagai dasar pemilihan jalur SD-WAN.

### Verifikasi

```
get router info routing-table all
get router info bgp summary
get router info bgp neighbors
diagnose firewall proute list
```

## 6. IPsec VPN Antar Situs

### FortiOS v6.4 dan v7.2 / v7.4 — Sintaks Sama

```
config vpn ipsec phase1-interface
    edit "VPN-HQ-BR"
        set interface "port1"
        set ike-version 2
        set peertype any
        set net-device disable
        set proposal aes256-sha256
        set remote-gw 203.0.113.6
        set psksecret <kunci-bersama-yang-kuat>
    next
end

config vpn ipsec phase2-interface
    edit "VPN-HQ-BR-P2"
        set phase1name "VPN-HQ-BR"
        set proposal aes256-sha256
        set src-addr-type name
        set src-name "LAN-Subnet"
        set dst-addr-type name
        set dst-name "BR-LAN-Subnet"
    next
end

config firewall policy
    edit 40
        set name "LAN-to-BR-VPN-Allow"
        set srcintf "port2"
        set dstintf "VPN-HQ-BR"
        set srcaddr "LAN-Subnet"
        set dstaddr "BR-LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set logtraffic all
    next
end
```

### Catatan Perbedaan

Sintaks IPsec identik. Pada v7, FortiGate mendukung IKEv2 dengan DPD yang lebih stabil dan integrasi dengan SD-WAN overlay. Pada v6, penggunaan `net-device disable` sering diperlukan untuk route berbasis policy. Pada v7, pendekatan berbasis route lebih dianjurkan.

### Verifikasi

```
diagnose vpn tunnel list
diagnose vpn ipsec status
diagnose vpn ike gateway list
diagnose debug application ike -1
diagnose debug enable
diagnose debug disable
```

## 7. SSL VPN dan Authentication

### FortiOS v6.4 — SSL VPN

```
config user local
    edit "vpn-user-01"
        set type password
        set passwd <kata-sandi-kuat>
    next
end

config user group
    edit "SSL-VPN-Users"
        set member "vpn-user-01"
    next
end

config vpn ssl settings
    set servercert "Fortinet_SSL"
    set tunnel-ip-pools "SSLVPN-TUNNEL-ADDR1"
    set port 10443
    set source-interface "port1"
    set source-address "all"
    set default-portal "full-access"
end

config firewall policy
    edit 50
        set name "SSLVPN-to-LAN-Allow"
        set srcintf "ssl.root"
        set dstintf "port2"
        set srcaddr "SSLVPN-TUNNEL-ADDR1"
        set dstaddr "LAN-Subnet"
        set action accept
        set schedule "always"
        set service "ALL"
        set groups "SSL-VPN-Users"
        set logtraffic all
    next
end
```

### FortiOS v7.2 / v7.4 — SSL VPN dan ZTNA

SSL VPN tetap didukung pada v7, tetapi untuk akses aplikasi, ZTNA lebih dianjurkan.

```
config firewall access-proxy
    edit "ZTNA-Portal"
        set vip "VIP-ZTNA"
        set server-host "172.16.10.10"
    next
end

config firewall proxy-policy
    edit 10
        set name "ZTNA-to-DMZ-Allow"
        set proxy access-proxy
        set srcintf "port1"
        set dstintf "port3"
        set srcaddr "all"
        set dstaddr "ZTNA-Portal"
        set action accept
        set schedule "always"
        set service "HTTPS"
        set groups "ZTNA-Users"
        set logtraffic all
    next
end
```

### Catatan Perbedaan

Perbedaan terbesar terletak pada pendekatan akses jarak jauh. Pada v6, SSL VPN merupakan pendekatan utama. Pada v7, ZTNA diperkenalkan sebagai pendekatan berbasis identitas dan postur device. ZTNA tidak memberikan akses network penuh, melainkan akses per aplikasi. Hal ini mengurangi attack surface.

### Verifikasi

```
diagnose vpn ssl list
diagnose wad user list
diagnose endpoint record list
diagnose debug application sslvpn -1
```

## 8. Profile Security: Antivirus, IPS, Web Filter, Application Control, DNS Filter

### FortiOS v6.4

```
config ips sensor
    edit "default"
        config entries
            edit 1
                set location server
                set severity high
                set action block
                set status enable
            next
        end
    next
end

config webfilter profile
    edit "default"
        config ftgd-wf
            config filters
                edit 1
                    set category 2
                    set action block
                next
            end
        end
    next
end

config firewall ssl-ssh-profile
    edit "certificate-inspection"
        config ssl
            set inspect-all disable
        end
    next
    edit "deep-inspection"
        config ssl
            set inspect-all enable
        end
    next
end
```

### FortiOS v7.2 / v7.4

```
config ips sensor
    edit "default"
        set block-malicious-url enable
        set scan-botnet-connections block
        config entries
            edit 1
                set location server
                set severity high
                set action block
                set status enable
            next
            edit 2
                set location client
                set severity high
                set action block
                set status enable
            next
        end
    next
end

config webfilter profile
    edit "default"
        config ftgd-wf
            config filters
                edit 1
                    set category 2
                    set action block
                next
            end
        end
    next
end

config firewall ssl-ssh-profile
    edit "certificate-inspection"
        config ssl
            set inspect-all disable
        end
    next
    edit "deep-inspection"
        config ssl
            set inspect-all enable
        end
    next
end
```

### Catatan Perbedaan

Sintaks security profile pada dasarnya sama. Pada v7, database IPS, antivirus, dan Application Control diperbarui lebih cepat dan mendukung TLS 1.3. Pada v7, opsi `block-malicious-url` dan `scan-botnet-connections` tersedia langsung pada sensor dan harus diaktifkan secara eksplisit.

### Verifikasi

```
diagnose autoupdate versions
diagnose ips anomaly list
diagnose test application ipsmonitor 3
diagnose debug application urlfilter -1
```

## 9. SD-WAN

### FortiOS v6.4 dan v7.2 / v7.4 — Struktur Sama, Perilaku v7 Lebih Lengkap

```
config system sdwan
    set status enable
    config zone
        edit "SDWAN-Zone"
        next
    end
    config members
        edit 1
            set interface "port1"
            set gateway 203.0.113.1
        next
        edit 2
            set interface "port5"
            set gateway 203.0.113.9
        next
    end
    config health-check
        edit "SLA-Internet"
            set server "8.8.8.8"
            set members 1 2
            config sla
                edit 1
                    set latency-threshold 100
                    set jitter-threshold 30
                    set packetloss-threshold 2
                next
            end
        next
    end
    config service
        edit 10
            set name "Business-App-Best-Quality"
            set mode sla
            set dst "all"
            set src "LAN-Subnet"
            config sla
                edit "SLA-Internet"
                    set id 1
                next
            end
            set priority-members 1 2
        next
        edit 20
            set name "Backup-Low-Cost"
            set mode manual
            set dst "all"
            set src "LAN-Subnet"
            set priority-members 2 1
        next
    end
end
```

### Catatan Perbedaan

Struktur SD-WAN sama. Pada v7, FortiGate mendukung lebih banyak health-check dan integrasi dengan overlay VPN. Pada v7, aturan SD-WAN dapat menggunakan Application Control sebagai kriteria pemilihan jalur, bukan hanya destination address.

### Verifikasi

```
diagnose sys sdwan member
diagnose sys sdwan health-check
diagnose sys sdwan service
get router info routing-table all
```

## 10. High Availability

### FortiOS v6.4 dan v7.2 / v7.4 — Sintaks Sama

```
config system ha
    set group-name "HQ-HA"
    set mode a-p
    set password <kata-sandi-ha>
    set hbdev "port5" 50
    set session-pickup enable
    set ha-mgmt-status enable
    config ha-mgmt-interfaces
        edit 1
            set interface "port4"
            set gateway 192.168.99.1
        next
    end
    set override disable
    set priority 200
end
```

### Catatan Perbedaan

Konfigurasi HA identik. Pada v7, sinkronisasi konfigurasi lebih cepat dan FortiGate memberikan peringatan apabila versi firmware kedua device tidak sama. Pada v7, mode Active-Active mendukung load balancing yang lebih baik untuk traffic yang tidak memerlukan inspection stateful yang ketat.

### Verifikasi

```
get system ha status
diagnose sys ha status
diagnose sys ha checksum show
```

## 11. Administrasi, Logging, dan Backup

### FortiOS v6.4 dan v7.2 / v7.4

```
config system admin
    edit "noc-operator"
        set accprofile "read-only"
        set vdom "root"
        set password <kata-sandi-kuat>
    next
    edit "net-admin"
        set accprofile "super_admin"
        set vdom "root"
        set password <kata-sandi-kuat>
    next
end

config system ntp
    set ntpsync enable
    config ntpserver
        edit 1
            set server "pool.ntp.org"
        next
    end
end

config log fortianalyzer setting
    set status enable
    set server 192.168.99.11
    set upload-option realtime
    set reliable enable
end

config system central-management
    set type fortimanager
    set fmg "192.168.99.10"
end
```

### Catatan Perbedaan

Perintah administrasi sama. Pada v7, opsi `reliable enable` pada FortiAnalyzer lebih dianjurkan untuk memastikan log tidak hilang saat koneksi terputus. Pada v7, FortiManager melakukan validasi policy yang lebih ketat sebelum instalasi.

### Verifikasi dan Backup

```
get system central-management
get log fortianalyzer setting
diagnose log test
execute backup config tftp HQ-FGT-Backup.conf 192.168.99.20
```

## 12. Konfigurasi Lengkap: FortiGate Kantor Pusat v6 dan v7

### FortiOS v6.4 — Konfigurasi Lengkap

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
end

config firewall address
    edit "LAN-Subnet"
        set subnet 10.10.10.0 255.255.255.0
    next
end

config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set nat enable
        set logtraffic all
    next
end

config router static
    edit 1
        set dst 0.0.0.0 0.0.0.0
        set gateway 203.0.113.1
        set device "port1"
    next
end
```

### FortiOS v7.2 / v7.4 — Konfigurasi Lengkap

```
config system interface
    edit "port1"
        set alias "WAN"
        set mode static
        set ip 203.0.113.2 255.255.255.252
        set allowaccess ping
        set role wan
    next
    edit "port2"
        set alias "LAN"
        set mode static
        set ip 10.10.10.1 255.255.255.0
        set allowaccess ping ssh https
        set role lan
    next
    edit "port3"
        set alias "DMZ"
        set mode static
        set ip 172.16.10.1 255.255.255.0
        set allowaccess ping
        set role dmz
    next
end

config firewall address
    edit "LAN-Subnet"
        set subnet 10.10.10.0 255.255.255.0
    next
end

config firewall policy
    edit 10
        set name "LAN-to-WAN-Web-Allow"
        set srcintf "port2"
        set dstintf "port1"
        set srcaddr "LAN-Subnet"
        set dstaddr "all"
        set action accept
        set schedule "always"
        set service "HTTP" "HTTPS" "DNS"
        set utm-status enable
        set av-profile "default"
        set ips-sensor "default"
        set webfilter-profile "default"
        set application-list "default"
        set ssl-ssh-profile "certificate-inspection"
        set nat enable
        set logtraffic all
    next
end

config router static
    edit 1
        set dst 0.0.0.0 0.0.0.0
        set gateway 203.0.113.1
        set device "port1"
    next
end

config log fortianalyzer setting
    set status enable
    set server 192.168.99.11
    set upload-option realtime
    set reliable enable
end
```

### Catatan Perbedaan pada Konfigurasi Lengkap

Konfigurasi v7 menambahkan `application-list` dan `ssl-ssh-profile` secara eksplisit pada policy. Pada v6, kedua profile tersebut sering ditambahkan pada tahap lanjutan. Pada v7, penerapan profile secara eksplisit lebih dianjurkan agar policy mudah diaudit.

### Jalur Peningkatan Versi

| Dari | Ke | Catatan |
| --- | --- | --- |
| 6.2.x | 6.4.x | Periksa kompatibilitas FortiManager dan FortiAnalyzer |
| 6.4.x | 7.0.x | Cadangkan konfigurasi dan periksa policy yang menggunakan SSL VPN lama |
| 7.0.x | 7.2.x | Periksa profile ZTNA dan SD-WAN |
| 7.2.x | 7.4.x | Periksa kompatibilitas hardware dan lisensi |

Peningkatan harus dilakukan secara bertahap sesuai jalur resmi. Peningkatan langsung dari 6.2 ke 7.4 tidak dianjurkan.

### Verifikasi Setelah Peningkatan

```
get system status
get system performance status
get firewall policy
diagnose vpn tunnel list
diagnose sys sdwan member
get system ha status
diagnose log test
diagnose autoupdate versions
```

### Tindakan Apabila Terjadi Masalah Setelah Peningkatan

| Gejala | Tindakan |
| --- | --- |
| Policy tidak bekerja | Periksa mode inspection dan profile yang digunakan |
| VPN tidak terbentuk | Periksa proposal dan pre-shared key |
| Log tidak sampai ke FortiAnalyzer | Periksa server FortiAnalyzer dan opsi reliable |
| HA tidak sinkron | Samakan versi firmware dan periksa heartbeat |

# Lampiran — Perintah Diagnostik dan Sindikasird Penamaan

## Perintah Diagnostik Umum

```
get system status
get system performance status
get system session status
diagnose sys session list
diagnose sys session stat
get firewall policy
diagnose firewall iprope list 100015
diagnose firewall proute list
get router info routing-table all
diagnose vpn tunnel list
diagnose vpn ipsec status
diagnose vpn ssl list
get system ha status
diagnose sys ha status
diagnose sys sdwan member
diagnose sys sdwan health-check
diagnose log test
diagnose debug flow filter addr 10.10.10.50
diagnose debug flow show function-name enable
diagnose debug enable
diagnose debug disable
execute backup config tftp FGT-Backup.conf 192.168.99.20
```
