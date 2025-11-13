# 📋 Panduan Komprehensif Setup Host OS, VPS Provider, & Mail Server (Non-Dockerized)

Dokumen ini berisi panduan checklist teknis, rekomendasi vendor, serta prosedur konfigurasi pada level **Host OS (Linux)**, **Dashboard Provider VPS**, dan **Jaringan Eksternal** yang tidak di-cover secara otomatis oleh container Docker.

> [!IMPORTANT]
> Seluruh referensi domain pada panduan ini menggunakan placeholder `<DNS_VPS>`. Ganti `<DNS_VPS>` dengan nama domain publik Anda (contoh: jika domain Anda `perusahaan.my.id`, maka FQDN mail adalah `mail.perusahaan.my.id`).

---

## 🏢 1. Rekomendasi Vendor & Strategi Pemilihan

### Kategori A: Provider VPS (Dukungan Port 25 Outbound & PTR / Reverse DNS)

Menjalankan mail server mandiri yang mengirim email langsung ke inbox penyedia publik (Gmail, Outlook, Yahoo) **wajib memenuhi dua syarat mutlak**:
1. **Port 25 Outbound Terbuka** (agar server bisa menghubungi server MX tujuan).
2. **Custom Reverse DNS (PTR Record)** (agar IP publik server diakui valid oleh Gmail/Yahoo).

| Provider | Status Port 25 Outbound | Kemudahan Buka Port 25 | Custom PTR (rDNS) | Biaya Bulanan (Est.) | Rekomendasi & Catatan |
|---|---|---|---|---|---|
| **Hetzner Cloud** | Tertutup *default* | ⭐⭐⭐⭐⭐ **Sangat Mudah** (Request via tiket setelah bayar tagihan pertama / KYC) | **Tersedia Gratis** (Edit langsung di dashboard) | ~€4 – €5 (~Rp 70.000) | **Pilihan Utama (Best Value).** Reputasi IP bersih, performa CPU tinggi, tiket unblock biasanya disetujui dalam hitungan jam. |
| **OVHcloud / Kimsufi** | **Terbuka Default** | ⭐⭐⭐⭐⭐ **Langsung Aktif** (Tanpa tiket) | **Tersedia Gratis** | ~$4 – ~$6 (~Rp 70.000) | **Paling Praktis.** Port 25 terbuka otomatis dengan proteksi antispam internal. |
| **Linode (Akamai)** | Tertutup *default* | ⭐⭐⭐⭐ **Mudah** (Buka tiket support dengan use-case) | **Tersedia Gratis** | ~$5 (~Rp 85.000) | **Sangat Stabil.** Tim support responsif; wajib melampirkan nama domain dan tujuan (notifikasi Redmine). |
| **Vultr** | Tertutup *default* | ⭐⭐⭐ **Moderat** (Tiket support + verifikasi identitas akun) | **Tersedia Gratis** | ~$5 – ~$6 (~Rp 85.000) | Alternatif yang baik jika Hetzner/Linode tidak tersedia. |
| **AWS EC2** | Tertutup *default* | ⭐⭐ **Ketat** (Harus isi formulir *Request to Remove Email Sending Limitations*) | **Tersedia** (via EIP setting) | Pay-as-you-go | Proses review AWS cukup ketat untuk akun baru. |
| **DigitalOcean** | **Blokir Total** | ❌ **Mustahil** (Tidak membuka port 25 untuk droplet baru) | Tersedia | ~$4 – ~$6 | **Tidak Disarankan untuk Direct Mail.** Pengguna DO wajib menggunakan relay eksternal (Kategori B). |

#### 📝 Template Tiket Pembukaan Port 25 (Hetzner / Linode / Vultr)
Jika provider Anda menutup port 25 secara default, kirimkan tiket permohonan berikut:
```text
Subject: Request to Unblock Outbound Port 25 for Transactional Project Management Notifications

Hello Support Team,

I would like to request the unblocking of outbound port 25 on my server (IP: <IP_PUBLIK_VPS>).

Purpose / Use Case:
This server hosts an internal Redmine project management system for our team. The mail server will be used solely for sending automated transactional notifications (ticket creation, status updates, assignment alerts, and password resets) to registered team members.

Domain Name: <DNS_VPS> (Mail Hostname: mail.<DNS_VPS>)
Security Measures:
- Reverse DNS (PTR) will be configured to match mail.<DNS_VPS>.
- SPF, DKIM, and DMARC records are configured.
- We strictly enforce SMTP authentication; open relay is disabled.
- No bulk/marketing emails will ever be sent from this server.

Thank you for your assistance.
```

---

### Kategori B: Provider SMTP Relay Pihak Ketiga (Solusi Praktis & Bebas Blokir Port)

Jika VPS Anda memblokir port 25 atau Anda tidak ingin repot mengelola reputasi IP email, Anda dapat menggunakan layanan relay melalui **Port 587 (TLS)**:

| Provider | Paket Gratis (Free Tier) | Kemudahan Integrasi | Tingkat Masuk Inbox | Cocok Untuk |
|---|---|---|---|---|
| **Mailjet** | **200 email/hari (6.000 email/bln)** | Sangat Mudah (Simple SMTP API key) | Sangat Tinggi (>99%) | Tim kecil hingga menengah, integrasi cepat via docker-mailserver relay. |
| **Brevo (Sendinblue)** | **300 email/hari** | Sangat Mudah (SMTP Relay instan) | Sangat Tinggi | Proyek tim dengan volume tiket harian sedang. |
| **Resend** | **100 email/hari (3.000 email/bln)** | Sangat Modern & Developer-Friendly | Sangat Tinggi | Developer yang menyukai dashboard modern dan log pengiriman realtime. |
| **AWS SES** | 62.000 email/bln gratis (jika di EC2) / $0.10 per 1.000 email | Perlu verifikasi domain & sandbox removal | Sangat Tinggi | Skala enterprise dengan biaya termurah di pasaran. |
| **Postmark** | Trial 100 email (berbayar $15/bln) | Sangat Mudah | Terbaik di Industri (Gold Standard) | Sistem mission-critical yang butuh latensi pengiriman < 2 detik. |

---

## 🌐 2. Checklist Dashboard VPS & Jaringan Eksternal

- [ ] **Verifikasi / Buka Port 25 Outbound:**
  - [ ] Jika menggunakan direct mail, ajukan tiket unblock ke provider.
  - [ ] Uji keterbukaan port 25 keluar dari terminal VPS:
    ```bash
    nc -zv -w5 smtp.gmail.com 25
    # Respon sukses: Connection to smtp.gmail.com 25 port [tcp/smtp] succeeded!
    ```
- [ ] **Konfigurasi Reverse DNS (PTR Record) di Dashboard VPS:**
  - [ ] Buka dashboard provider VPS $\rightarrow$ Networking / IP Management $\rightarrow$ Reverse DNS.
  - [ ] Set nilai PTR untuk `<IP_PUBLIK_VPS>` menjadi: `mail.<DNS_VPS>`.
  - [ ] Verifikasi dari terminal:
    ```bash
    dig -x <IP_PUBLIK_VPS> +short
    # Output harus menghasilkan: mail.<DNS_VPS>.
    ```
- [ ] **Setup Kredensial Relay Eksternal (Jika Pakai Jalur Relay / Port 25 Ditutup):**
  - [ ] Daftarkan akun di Mailjet atau Brevo.
  - [ ] Catat kredensial SMTP:
    - **SMTP Host:** `in-v3.mailjet.com` (atau `smtp-relay.brevo.com`)
    - **Port:** `587`
    - **Username / API Key:** `<MJ_PUBLIC_API_KEY>`
    - **Password / Secret Key:** `<MJ_PRIVATE_API_KEY>`

---

## 💻 3. Hardening & Konfigurasi Host OS Linux (Ubuntu / Debian)

- [ ] **Alokasi Swap Memory 4GB (Wajib untuk Server RAM 1GB – 2GB):**
  - Redmine (Rails) + PostgreSQL + Docker Mailserver membutuhkan stabilitas memori. Tanpa swap, server berisiko crash mendadak akibat *Linux OOM Killer*.
  ```bash
  sudo fallocate -l 4G /swapfile
  sudo chmod 600 /swapfile
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
  ```
  - Atur kernel swappiness agar swap hanya digunakan saat RAM fisik hampir habis:
  ```bash
  sudo sysctl vm.swappiness=10
  sudo sysctl vm.vfs_cache_pressure=50
  echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
  echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
  ```

- [ ] **Pengaturan Hak Akses User & Docker Socket:**
  - Hindari menjalankan perintah harian sebagai user `root`. Tambahkan user standar ke grup `docker`:
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker
  ```

- [ ] **Konfigurasi Firewall Host (UFW):**
  - Buka hanya port yang sah (*least privilege*):
  ```bash
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 22/tcp       # SSH Port (atau port custom SSH Anda)
  sudo ufw allow 80/tcp       # HTTP (Wajib untuk Certbot Let's Encrypt Standalone)
  sudo ufw allow 443/tcp      # HTTPS Web
  sudo ufw allow 10083/tcp    # Akses Web Redmine (jika tanpa reverse proxy lokal)
  sudo ufw allow 25/tcp       # Inbound SMTP (menerima email)
  sudo ufw allow 587/tcp      # Submission SMTP (STARTTLS)
  sudo ufw allow 993/tcp      # IMAPS Secure
  ```
  - **Audit Keamanan Kritis:** Pastikan port `5432` (PostgreSQL) **TIDAK** ada dalam daftar `ufw allow`.
  ```bash
  sudo ufw enable
  sudo ufw status verbose
  ```

---

## 🔒 4. Sertifikat SSL / TLS Host (Let's Encrypt Certbot & Auto-Renewal)

Container mailserver me-mount sertifikat dari direktori host: `/etc/letsencrypt/live/mail.<DNS_VPS>/`. Sertifikat ini harus diterbitkan sebelum container dijalankan.

- [ ] **Instal Certbot di Host:**
  ```bash
  sudo apt update && sudo apt install -y certbot
  ```

- [ ] **Terbitkan Sertifikat SSL Standalone untuk Hostname Mail:**
  - *Catatan: Pastikan sementara port 80 tidak terpakai oleh Nginx/Apache.*
  ```bash
  sudo certbot certonly --standalone -d mail.<DNS_VPS> --agree-tos --email admin@<DNS_VPS> --non-interactive
  ```

- [ ] **Atur Izin Akses Baca untuk Docker Daemon:**
  - Direktori arsip kunci Let's Encrypt memerlukan izin baca agar dapat di-bind mount ke container:
  ```bash
  sudo chmod 755 /etc/letsencrypt/live
  sudo chmod 755 /etc/letsencrypt/archive
  ```

- [ ] **Otomasi Pembaruan Sertifikat (Certbot Renewal Deploy Hook):**
  - Ketika sertifikat diperbarui secara otomatis setiap 60–90 hari, container mailserver harus me-reload sertifikat baru. Buat deploy hook di host:
  ```bash
  sudo tee /etc/letsencrypt/renewal-hooks/deploy/reload-mailserver.sh << 'EOF'
  #!/bin/bash
  # Reload konfigurasi TLS pada container mailserver jika container sedang berjalan
  if docker ps -q -f name=redmine_mail | grep -q .; then
      docker exec redmine_mail supervisorctl restart postfix
      docker exec redmine_mail supervisorctl restart dovecot
  fi
  EOF
  sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-mailserver.sh
  ```
  - Uji mekanisme simulasi perpanjangan sertifikat:
  ```bash
  sudo certbot renew --dry-run
  ```

---

## 🔑 5. Persiapan File Environment (`.env`) di Host

- [ ] **Generate Token Kriptografis Rails:**
  ```bash
  openssl rand -hex 64
  ```
- [ ] **Buat File `.env` Produksi:**
  ```bash
  cp .env.example .env
  chmod 600 .env
  ```
- [ ] **Isi Nilai Variabel:**
  - `DOMAIN_NAME`: Isi dengan `<DNS_VPS>`.
  - `REDMINE_SECRET_KEY_BASE`: Isi dengan token acak 64-karakter di atas.
  - `POSTGRES_PASSWORD`: Tentukan password database yang kompleks.
  - `REDMINE_SMTP_PASSWORD`: Tentukan password akun SMTP pengirim.

---

## 📬 6. Inisialisasi Akun & Konfigurasi Mail Server Lokal

Setelah stack dijalankan pertama kali dengan `docker compose up -d`:

- [ ] **Buat Akun Mailbox Pengirim di Container:**
  ```bash
  docker exec -it redmine_mail setup email add admin@<DNS_VPS> "<REDMINE_SMTP_PASSWORD>"
  ```
- [ ] **Verifikasi Akun Mail Terdaftar:**
  ```bash
  docker exec -it redmine_mail setup email list
  ```
- [ ] **Generate Kunci DKIM Bawaan Container (Khusus Direct Mail):**
  - Jika Anda mengirim langsung via container tanpa relay pihak ketiga, generate DKIM key:
  ```bash
  docker exec -it redmine_mail setup config dkim
  ```
  - Ambil nilai public key DKIM yang dihasilkan untuk dipasang di DNS TXT domain Anda:
  ```bash
  cat ./smtp/config/opendkim/keys/<DNS_VPS>/mail.txt
  ```
- [ ] **Konfigurasi Smart-Host Relay (Khusus Jika Port 25 Ditutup):**
  - Jika menggunakan Mailjet sebagai relay keluar:
  ```bash
  docker exec -it redmine_mail setup relay add-auth in-v3.mailjet.com <MJ_PUBLIC_API_KEY> <MJ_PRIVATE_API_KEY> 587
  ```

---

## 🧪 7. Uji Coba, Validasi, & Runbook Debugging

- [ ] **Uji Pengiriman Email dari UI Redmine:**
  1. Login ke Redmine (`http://<IP_SERVER>:10083`) sebagai Administrator.
  2. Masuk ke: **Administration $\rightarrow$ Settings $\rightarrow$ Email notifications**.
  3. Pastikan kolom **Emission email address** sesuai dengan `admin@<DNS_VPS>`.
  4. Klik **Send a test email** di pojok kanan bawah.
  5. Periksa apakah email masuk ke inbox penerima.

- [ ] **Troubleshooting & Perintah Debugging Mail Server:**
  - **Lihat antrean email yang tertahan (Queue):**
    ```bash
    docker exec -it redmine_mail mailq
    ```
  - **Paksa pengiriman ulang antrean (Flush Queue):**
    ```bash
    docker exec -it redmine_mail postfix flush
    ```
  - **Pantau log transaksi email secara realtime:**
    ```bash
    docker logs --tail 100 -f redmine_mail
    ```
  - **Pantau log aplikasi Rails jika terjadi kesalahan koneksi SMTP:**
    ```bash
    docker logs --tail 100 -f redmine_app
    ```
