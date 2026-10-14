# Supervisi Akademik untuk Netlify

Aplikasi supervisi akademik dengan login **NPSN + PIN pribadi tiap supervisor**.
PIN diperiksa di **server** (Netlify Functions), data tersimpan di **Netlify Blobs**,
dan supervisor hanya menerima data guru dampingannya dari server.

## Isi folder

- `public/index.html` : tampilan aplikasi
- `netlify/functions/api.mjs` : server (login, data, admin PIN)
- `netlify.toml`, `package.json` : pengaturan Netlify

## Cara deploy (disarankan lewat GitHub)

Unggah folder lewat tarik-lepas (Netlify Drop) biasanya tidak menjalankan fungsi server,
jadi pakai salah satu cara di bawah.

### Cara A: GitHub + Netlify
1. Buat repositori baru di GitHub, lalu unggah seluruh isi folder ini (termasuk folder `netlify` dan `public`).
2. Di Netlify: **Add new site, Import an existing project**, pilih repositori tadi.
3. Pengaturan build terbaca otomatis dari `netlify.toml`. Klik **Deploy**.
4. Buka **Site configuration, Environment variables**, lalu tambahkan dua variabel:
   - `ADMIN_KEY` : kunci admin untuk mengunggah PIN (minimal 8 karakter, rahasia)
   - `SESSION_SECRET` : teks acak panjang (minimal 16 karakter, rahasia). Contoh membuatnya: `openssl rand -hex 32`
5. **Deploy ulang** (Deploys, Trigger deploy, Clear cache and deploy site) supaya variabel terbaca.

### Cara B: Netlify CLI
```
npm install
npm install -g netlify-cli
netlify login
netlify init            # hubungkan ke situs Anda
netlify env:set ADMIN_KEY "kunci-admin-anda"
netlify env:set SESSION_SECRET "teks-acak-panjang-anda"
netlify deploy --build --prod
```

Setelah deploy, buka `https://situs-anda.netlify.app/api/health`. Jika tampil
`{"ok":true,"adminKey":true,"sessionSecret":true}`, pengaturan sudah benar.

## Cara pakai

### 1. Admin mendaftarkan supervisor (Excel PIN)
Di halaman login klik **Masuk admin: kelola PIN supervisor**, masukkan `ADMIN_KEY`,
lalu unggah Excel (atau tempel dari Excel) dengan satu baris per supervisor:

| NPSN | Nama supervisor | PIN | Peran | Nama sekolah |
| --- | --- | --- | --- | --- |
| 12345678 | Siti Aminah, S.Pd. | 482913 | Koordinator | SDN Contoh 1 |
| 12345678 | Budi Santoso, S.Pd. | 739104 | Supervisor | SDN Contoh 1 |

- NPSN 8 digit. PIN 4 sampai 12 huruf atau angka, harus berbeda untuk tiap supervisor dalam satu sekolah.
- **Koordinator** melihat dan mengelola semua guru sekolah (unggah jadwal, tambah atau hapus guru, rekap seluruh sekolah).
  **Supervisor** (atau peran kosong) hanya melihat dan menilai guru dampingannya.
- Jika PIN diawali angka 0, format kolom PIN di Excel sebagai Teks.
- PIN disimpan sebagai hash. Simpan file Excel PIN di tempat aman karena PIN tidak bisa dilihat lagi.
- Mengunggah ulang nama supervisor yang sama akan mengganti PIN-nya.

### 2. Koordinator mengisi jadwal guru
Masuk dengan NPSN + PIN koordinator, lalu di menu **Jadwal** unggah Excel jadwal guru
(kolom: Nama, NIP, Mapel, Kelas, Pra-observasi, Observasi, Jam ke-, Pertemuan balikan, Supervisor, Status).

**Penting:** isi kolom **Supervisor** dengan nama yang sama dengan *Nama supervisor* di Excel PIN
(tanda baca dan huruf besar-kecil diabaikan). Dari kolom inilah guru dampingan tiap supervisor ditentukan.

### 3. Supervisor menilai
Supervisor masuk dengan NPSN + PIN pribadi, lalu mengisi Pra-observasi, Observasi, Penilaian, dan Balikan
untuk guru dampingannya, dan mencetak laporan per guru.

## Keamanan
- PIN dan hak akses diperiksa di server. Browser supervisor tidak pernah menerima data guru yang bukan dampingannya.
- Percobaan PIN salah dibatasi (5 kali per perangkat, lalu dikunci 5 menit). Sesi login berlaku 12 jam.
- Semua koneksi memakai HTTPS dari Netlify.
- NPSN adalah nomor publik, jadi keamanan bertumpu pada PIN. Gunakan PIN yang tidak mudah ditebak
  (jangan 123456 atau tanggal lahir) dan hapus PIN supervisor yang sudah berhenti lewat panel admin.
- Jangan bagikan `ADMIN_KEY` dan `SESSION_SECRET`. Jika `SESSION_SECRET` diganti, semua sesi login otomatis berakhir.

## Batasan
- Dua supervisor yang mengubah guru yang sama pada saat bersamaan: perubahan terakhir yang menang.
- Data tersinkron tiap 15 detik dan saat halaman dibuka kembali.
- Tidak ada fitur lupa PIN: admin cukup mengunggah ulang PIN baru.
- Pengujian dilakukan pada simulasi lokal (server dan browser Chromium). Uji juga di situs Netlify Anda sebelum dipakai sungguhan.

## Uji di komputer sendiri (opsional)
```
npm install
npm install -g netlify-cli
ADMIN_KEY=kunciadmin123 SESSION_SECRET=teksacakpanjangminimal16 netlify dev
```
