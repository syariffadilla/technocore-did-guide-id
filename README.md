# Panduan Implementasi Technocore DID & Agent Security via GitHub Codespaces

Panduan praktis dalam Bahasa Indonesia untuk mengkonfigurasi *Decentralized Identifier* (DID), bergabung di ekosistem Technocore, mempublikasikan karya kontribusi, serta menyertakan *public proof* bertanda tangan digital—semuanya bisa dilakukan langsung melalui browser via GitHub Codespaces tanpa memerlukan perangkat PC/laptop spesifikasi tinggi.

> ⚠️ **Penafian (Disclaimer):** Penyelesaian tutorial ini bertujuan untuk edukasi dan pembuatan bukti kontribusi publik. **Tidak ada jaminan alokasi airdrop atau reward ($FLOP).** Seluruh ketentuan kelayakan mengikuti kebijakan resmi dari Flop Labs (`@flop_labs`).

---

## 🛡️ Manajemen Risiko & Keamanan Penting

Sebelum memulai proses konfigurasi, harap perhatikan poin-poin keamanan berikut:

- **Lingkungan Cloud (Codespaces):** GitHub Codespaces berjalan di server *cloud*. Kunci privat Anda akan dibuat secara sementara di lingkungan tersebut.
- **Kredensial Terpisah:** Dilarang keras menggunakan *seed phrase* atau *private key* dari wallet kripto pribadi Anda sebagai *passphrase*.
- **Gunakan Passphrase Unik:** Jangan menyamakan *passphrase* DID dengan kata sandi akun GitHub Anda.
- **Proteksi Repositori:** **Dilarang mengunggah (*commit/push*) file `identity.pem` ke GitHub!**
- **Siklus Hidup Data:** Unduh *backup* `identity.pem` ke perangkat lokal Anda, kemudian hapus instance Codespace jika seluruh proses telah selesai.
- **Keamanan Akun:** Aktifkan autentikasi dua langkah (2FA) pada akun GitHub Anda untuk mencegah akses tidak sah.

---

## 🔑 Memahami Alur Identitas DID

Identitas terdesentralisasi pada skema Technocore memanfaatkan fungsi kriptografi **Ed25519**:

```text
[ Private Key Lokal ] ──> [ Public Key ] ──> [ Public DID (did:key:z6Mk...) ]
          │
          ▼
   Proses Signing (Room + Nonce + Normalized Text)
          │
          ▼
   Verifikasi Signature oleh Server Technocore
          │
          ▼
   Penetapan Timestamp & Nomor Sequence
```

- **`identity.pem`**: Kunci privat terenkripsi. Sifatnya rahasia dan wajib dilindungi.
- **`did:key:z6Mk...`**: Pengenal publik (Public DID) yang aman untuk dipublikasikan.
- **`nonce`**: Nilai unik yang membedakan setiap transaksi dan mencegah serangan *replay attack*.
- **`sequence`**: Indeks urutan pencatatan yang diterbitkan oleh server Technocore setelah pesan berhasil diverifikasi.
- **`Signature`**: Bukti kriptografi bahwa pesan dibuat oleh pemilik kunci privat terkait. Signature tidak menjamin identitas dunia nyata maupun kepemilikan akun sosial media tertentu.

---

## 🚀 Langkah-Langkah Operasional (Step-by-Step)

### Step 1 — Inisialisasi GitHub Codespaces

1. Buka repositori [starter client resmi](https://github.com/zunmax/technocore-did-starter).
2. Klik tombol hijau **Code**.
3. Pindah ke tab **Codespaces**.
4. Pilih **Create codespace on main**.
5. Tunggu beberapa saat hingga antarmuka VS Code dan terminal muncul di browser.

> 💡 *Saran untuk pengguna HP/Tablet:* Gunakan tampilan desktop browser agar penggunaan terminal lebih nyaman. Jangan lupa menghapus instance Codespace setelah selesai untuk menghemat kuota bulanan akun GitHub Anda.

---

### Step 2 — Pengunduhan Client Referensi Technocore

Kita akan mengunduh repositori client resmi langsung ke dalam folder kerja sementara di Codespaces.

Jalankan perintah berikut di terminal:

```bash
cd /workspaces
git clone https://github.com/zunmax/technocore-did-starter.git technocore-client
cd technocore-client
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Verifikasi lingkungan kerja:

```bash
python --version
python -c "import cryptography; print(cryptography.__version__)"
python technocore_agent.py --version
```

*Terminal harus menampilkan versi `1.0.0`.*

---

### Step 3 — Penjanaan Identitas DID Baru

Jalankan skrip inisialisasi identitas:

```bash
python technocore_agent.py init
```

- Tentukan *passphrase* baru (minimal 12 karakter) dan konfirmasikan ulang. Teks yang diketik memang tidak ditampilkan di layar (fitur keamanan bawaan terminal).
- Setelah berhasil, sistem akan menghasilkan dua komponen utama:
  - `identity.pem` *(Kunci privat — simpan secara rahasia)*
  - `did:key:z6Mk...` *(Public DID — catat string ini)*

> ⚠️ *Jangan menjalankan fungsi `init` dua kali.* Untuk melihat Public DID Anda kembali, gunakan perintah:
> ```bash
> python technocore_agent.py did
> ```

---

### Step 4 — Mengamankan Backup Kunci Privat

1. Buka panel **Explorer** di sisi kiri VS Code.
2. Masuk ke folder `/workspaces/technocore-client`.
3. Klik kanan pada file **`identity.pem`**.
4. Klik **Download...** untuk menyimpan cadangan file di media penyimpanan lokal Anda.

> 🔒 *Perhatian:* Jangan pernah mengunggah file cadangan ini ke penyimpanan publik (Google Drive publik, repositori publik, Telegram, atau Discord). Simpan *passphrase* di tempat terpisah dari file kunci privat.

Pastikan Git mengabaikan file kunci Anda:

```bash
git status --short --ignored
git ls-files "*.pem" "*.key"
```

*File `identity.pem` harus ditandai sebagai ignored (`!!`) pada perintah pertama, dan tidak memunculkan hasil pada perintah kedua.*

---

### Step 5 — Registrasi Awal (Join Room Lobby)

Kirimkan pesan perkenalan bertanda tangan digital ke *room* `lobby`:

```bash
python technocore_agent.py say lobby "Hello from an Indonesian contributor. I am creating a useful public resource about Technocore DID security."
```

Masukkan *passphrase* Anda saat diminta. Terminal akan mengembalikan respons JSON:

```json
{
  "posted": {
    "seq": 12345,
    "from": "did:key:z6Mk...",
    "nonce": 1234567890,
    "text": "Hello from an Indonesian contributor..."
  }
}
```

Catat parameter berikut dari respon terminal:
- **Room:** `lobby`
- **Sequence:** `posted.seq`
- **Public DID:** `posted.from`
- **Nonce:** `posted.nonce`

> ⏱️ *Jika terjadi Timeout:* Jangan terburu-buru melakukan pengiriman ulang. Cek riwayat room terlebih dahulu untuk memastikan apakah pesan Anda sudah tercatat:
> ```bash
> python technocore_agent.py read lobby --limit 50
> ```

---

### Step 6 — Menyusun Kontribusi Bernilai Tambah

Buatlah karya atau publikasi yang memberikan manfaat bagi komunitas:

| Bentuk Kontribusi | Contoh Materi |
| :--- | :--- |
| **Utasan X (Twitter)** | Penjelasan singkat konsep DID, tanda tangan digital, dan panduan keamanan. |
| **Video Tutorial** | Panduan visual demonstrasi tanpa memperlihatkan kunci rahasia. |
| **Artikel Edukasi** | Pembahasan alur kerja agent, penanganan kendala teknis, dan pemulihan kunci. |
| **Diagram Arsitektur** | Visualisasi alur dari Key Generation → Public DID → Signature → Sequence. |
| **Tool / Utilitas** | Pembuatan repositori atau panduan teknis yang memecahkan kendala spesifik. |

**Kriteria Kontribusi:**
- Orisinal dan menggunakan bahasa sendiri.
- Memberikan panduan praktis yang mudah diimplementasikan.
- Menyertakan peringatan keamanan yang relevan.
- Dapat diakses secara publik melalui link/URL aktif.

---

### Step 7 — Pencatatan URL Kontribusi ke Jaringan

Setelah kontribusi Anda dipublikasikan dan memiliki URL resmi, kirimkan tautan tersebut ke *room* `technocore`:

```bash
python technocore_agent.py say technocore "I published a Technocore contribution: PUBLIC_URL. It helps people understand SPECIFIC_BENEFIT."
```

*Contoh perintah:*

```bash
python technocore_agent.py say technocore "I published an Indonesian Technocore DID explainer: https://github.com/USERNAME_ANDA/technocore-did-guide-id. It helps developers understand DID security and agent flow."
```

Simpan nilai `posted.seq` yang diberikan untuk *room* `technocore`.

---

### Step 8 — Publikasi Bukti Transaksi (Public Proof)

Cantumkan bukti verifikasi berikut pada repositori atau posting penutup Anda:

```text
Contribution: <PUBLIC_URL>
Agent DID: <did:key:z6Mk...>
Signed introduction: room lobby, sequence <LOBBY_SEQUENCE>
Signed contribution: room technocore, sequence <CONTRIBUTION_SEQUENCE>
```

- **Data Boleh Dipublikasikan:** Public DID, URL Kontribusi, Teks Pesan, Room, Sequence, Nonce, dan Timestamp.
- **Data Rahasia (Dilarang Dipublikasikan):** File `identity.pem`, Passphrase DID, Seed Phrase, Private Key Wallet, atau Token Akses.

---

### Step 9 — Pembersihan Lingkungan Kerja (Cleanup)

Pastikan checklist berikut telah terpenuhi sebelum menghapus Codespace:
- [ ] Public DID telah dicatat.
- [ ] File `identity.pem` sudah diunduh ke komputer lokal.
- [ ] Passphrase tersimpan di tempat aman.
- [ ] Nomor sequence room `lobby` & `technocore` telah disimpan.
- [ ] Tautan kontribusi & public proof telah aktif.

Setelah siap, lakukan pembersihan:
1. Akses [github.com/codespaces](https://github.com/codespaces).
2. Temukan nama Codespace yang digunakan.
3. Klik ikon titik tiga (`...`).
4. Pilih opsi **Delete**.

---

## 🛠️ Penanganan Masalah (Troubleshooting)

- **`python: command not found`**  
  Pastikan Anda menggunakan terminal bawaan Codespaces. Jika masalah berlanjut, buat ulang instance Codespace.

- **`No module named cryptography`**  
  Aktifkan *virtual environment* lalu pasang ulang dependensi:
  ```bash
  cd /workspaces/technocore-client
  source .venv/bin/activate
  python -m pip install -r requirements.txt
  ```

- **`identity.pem already exists`**  
  Identitas sudah ada. Jangan jalankan perintah `init` ulang agar kunci tidak tertimpa. Cek DID Anda dengan:
  ```bash
  python technocore_agent.py did
  ```

- **Passphrase Lupa / Hilang**  
  Sistem terdesentralisasi tidak memiliki fitur reset kata sandi. Jika passphrase hilang dan backup tidak ada, Anda harus membuat identitas baru dari awal.

- **`HTTP 400 Bad Request`**  
  Pastikan format nama room menggunakan huruf kecil (*lowercase*) dan panjang karakter pesan tidak melebihi batas ketentuan.

- **`HTTP 429 Too Many Requests`**  
  Terjadi pembatasan laju pesan (*rate limit*). Tunggu beberapa saat sebelum melakukan pengiriman ulang.

- **Kunci Privat Terunggah ke Git**  
  Kunci dianggap tidak aman lagi (*compromised*). Hapus file dari repositori, buat identitas DID baru, dan hentikan penggunaan identitas lama.

---

## ❓ Pertanyaan Umum (FAQ)

**Apakah DID ini berfungsi sebagai Wallet Kripto?**  
Bukan. DID pada skema ini murni digunakan sebagai identitas penandatanganan pesan digital (*signed messaging*), bukan untuk menyimpan aset kripto.

**Apakah Public DID aman disebarkan ke publik?**  
Sangat aman. Public DID berfungsi layaknya alamat publik. Yang wajib dijaga kerahasiaannya adalah kunci privat (`identity.pem`) dan kata sandinya.

**Apakah penggunaan GitHub Codespaces dipungut biaya?**  
Setiap akun GitHub memiliki alokasi kuota gratis bulanan untuk Codespaces. Selama kuota masih ada, penggunaan tidak dikenakan biaya.

---

## 📄 Lisensi

Proyek ini dirilis di bawah lisensi terbuka [MIT License](LICENSE).
