# Business Flow

Business Flow adalah aplikasi web full-stack untuk pengelolaan usaha: Next.js 16 App Router, React 19, TypeScript, PostgreSQL, dan Drizzle ORM. Source mencakup API multi-tenant, tabel dengan Row-Level Security (RLS), modul penjualan/POS, persediaan, pembelian, keuangan, CRM, HR, operasional, marketing, proyek, aset, booking, serta 16 template usaha.

> **Status rilis:** paket ini belum boleh dipakai untuk transaksi/data pelanggan produksi. Lihat [`BLOCKERS.md`](./BLOCKERS.md) dan [`docs/PRODUCTION_READINESS.md`](./docs/PRODUCTION_READINESS.md). Email nyata, Google OAuth, Neon provisioning/Auth, migrasi SQL berversi, pemisahan credential database, backup/restore drill, dan security/performance audit eksternal belum lengkap.

## Persyaratan

- Node.js 22.x (direkomendasikan) dan npm 10+.
- PostgreSQL 15+ yang dapat dijangkau dari mesin aplikasi. Untuk Vercel, jangan gunakan host `localhost`; gunakan database remote dengan TLS/SSL dan pool/URL yang direkomendasikan penyedia database.
- Satu environment variable aplikasi: `DATABASE_URL` (server-only). Jangan pernah menambahkan prefix `NEXT_PUBLIC_`.
- Untuk production scheduler di Vercel: `CRON_SECRET` disimpan sebagai secret environment variable oleh script resmi di bawah; script tidak menuliskan nilai secret ke file atau log.

## Instalasi lokal

Arsip ini tidak menyertakan `package-lock.json`. Oleh sebab itu gunakan `npm install` (bukan `npm ci`) untuk bootstrap pertama, tinjau hasilnya, lalu simpan `package-lock.json` yang dihasilkan pada repository sebelum CI/deployment yang reproducible. Versi dependency langsung di `package.json` telah dipatok.

```bash
npm install
cp .env.example .env.local
# Isi DATABASE_URL dengan database development yang kosong/khusus pengujian.
npm run db:push
npm run dev
```

Untuk production build lokal setelah konfigurasi database tersedia:

```bash
npm run build
npm start
```

`db:push` adalah bootstrap/dev convenience dan **bukan** pengganti migration SQL yang ditinjau/berversi untuk production. Jangan jalankan terhadap database produksi sebelum migration workflow dan backup/restore disetujui.

## Deployment Vercel

1. Import repository ke Vercel dan pilih Next.js preset. Gunakan Node.js 22.x.
2. Tambahkan `DATABASE_URL` sebagai Environment Variable **server-only** untuk Preview dan Production, masing-masing diarahkan ke database terpisah.
3. Jalankan `vercel link` dari direktori proyek (login melalui browser/CLI).
4. Jalankan `npm run vercel:cron-secret` untuk membuat `CRON_SECRET` acak dan memasukkannya langsung ke Environment Variables Production sebagai secret.
5. Jalankan deployment Preview, lakukan pemeriksaan koneksi database dan manual testing dengan data uji. Jangan arahkan Preview ke database produksi.
6. Cron scheduler berjalan setiap 5 menit pada Vercel plan yang mendukung interval tersebut. Endpoint menolak request tanpa Bearer `CRON_SECRET`.

Untuk aplikasi komersial, jangan gunakan Vercel Hobby; plan Hobby dibatasi untuk penggunaan personal/non-komersial. Lihat aturan paket dan cron terbaru pada dokumentasi Vercel.

## Tes

Tes integrasi mengharuskan aplikasi berjalan dalam **local development (`npm run dev`)** dan PostgreSQL khusus pengujian sudah berjalan, karena hanya local development yang boleh mengembalikan OTP development ke test runner:

```bash
BASE_URL=http://localhost:3000 npm run test:e2e
```

Jalankan juga pemeriksaan statis:

```bash
npm run typecheck
npm run lint
npm run build
```

Hasil `107/107` pada arsip asal adalah hasil historis Claude Code tertanggal 9 November 2026, bukan hasil build/test dari revisi paket ini. Tes tambahan untuk kebijakan password dan manajemen sesi ditambahkan ke skrip E2E, tetapi belum bisa dijalankan di sandbox ini.

## Dokumentasi penting

- [`MANUAL-TESTING-GUIDE.md`](./MANUAL-TESTING-GUIDE.md) — checklist uji manual per modul dan template laporan bug.
- [`BLOCKERS.md`](./BLOCKERS.md) — daftar pekerjaan dan prasyarat yang masih tertahan.
- [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) — setup deployment.
- [`docs/ENVIRONMENT.md`](./docs/ENVIRONMENT.md) — matriks environment variable yang benar-benar dibaca source.
- [`docs/AUTHENTICATION.md`](./docs/AUTHENTICATION.md) — rancangan dan status autentikasi.
- [`docs/PRODUCTION_READINESS.md`](./docs/PRODUCTION_READINESS.md) — keputusan kesiapan produksi.
- [`docs/SYSTEM_ARCHITECTURE.md`](./docs/SYSTEM_ARCHITECTURE.md) — arsitektur sistem.

## Pengamanan data

Gunakan database non-produksi selama testing. Jangan mengunggah `.env`, token, API key, session cookie, dump produksi, atau data pelanggan/karyawan ke repository/ZIP. Logout tidak menghapus data database.
