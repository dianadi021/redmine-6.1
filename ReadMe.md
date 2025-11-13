# Redmine 6.1 Deployment Stack (Docker Compose)

Stack deployment otomatis untuk **Redmine 6.1** terintegrasi dengan **PostgreSQL 15** dan **Docker Mailserver**.

---

## 🚀 Quick Start Deployment

### 1. Persiapan Environment Variables
Salin template konfigurasi dan atur kredensial aman:
```bash
cp .env.example .env
chmod 600 .env
```
Generate kunci rahasia Rails:
```bash
openssl rand -hex 64
```
Masukkan kunci tersebut ke `REDMINE_SECRET_KEY_BASE` pada file `.env`, lalu lengkapi variabel database dan SMTP.

### 2. Validasi Konfigurasi
Pastikan syntax compose valid tanpa error:
```bash
docker compose config
```

### 3. Menjalankan Layanan
```bash
docker compose up -d
```
Periksa status container dan healthcheck database:
```bash
docker compose ps
```
Akses aplikasi Redmine di browser melalui port `http://<IP_SERVER>:10083`.

---

## 💻 Panduan Setup Host OS (Non-Dockerized)

Sebelum menjalankan container pada server produksi, lakukan persiapan sistem operasi host berikut:

### Swap RAM (Rekomendasi 4GB untuk VPS 1GB–2GB RAM)
```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Instalasi Docker Engine di Linux (Ubuntu / Debian)
```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 📧 Mail Server & SMTP Setup

### Inisialisasi Akun Email di Container Mailserver
```bash
docker exec -it redmine_mail setup email add admin@<DNS_VPS> "<PASSWORD_KUAT>"
docker exec -it redmine_mail setup email list
```

### Setup Relay Eksternal (Opsional - Jika Port 25 Ditutup)
Contoh integrasi smart-host relay ke Mailjet:
```bash
docker exec -it redmine_mail setup relay add-auth in-v3.mailjet.com <MJ_PUBLIC_API_KEY> <MJ_PRIVATE_API_KEY> 587
```

---

## 🔒 Konfigurasi SSL Reverse Proxy (Certbot)

### NginX
```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d redmine.<DNS_VPS>
```

### Apache
```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d redmine.<DNS_VPS>
```

---

## 📋 Checklist & Dokumentasi Lengkap
- Checklist perbaikan kepatuhan audit: [todo.md](file:///home/wannacry021/Public/Project/Skuad/redmine-6.1/todo.md)
- Checklist lengkap konfigurasi host & VPS: [TODO_HOST_SETUP.md](file:///home/wannacry021/Public/Project/Skuad/redmine-6.1/TODO_HOST_SETUP.md)
- Arsitektur & spesifikasi operasional: [CONTEXT.md](file:///home/wannacry021/Public/Project/Skuad/redmine-6.1/CONTEXT.md)
