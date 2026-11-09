# Blockers — Business Flow

**Status keseluruhan: BELUM SIAP PRODUKSI / BELUM CLIENT-READY.** Tabel ini membedakan source implementation dari integrasi yang benar-benar dapat dipakai.

| Item | Status saat ini | Syarat penutupan |
|---|---|---|
| Neon database/provisioning | Belum dihubungkan; source hanya menerima `DATABASE_URL` untuk PostgreSQL | Pemilik mengotorisasi akun Neon, membuat database/branch, memilih region/plan, memverifikasi connection string dan biaya |
| Neon Auth / app-managed auth | Source menggunakan scrypt + session sendiri; bukan Neon Auth | Putuskan provider resmi; integrasikan migrasi akun, linking, verification/reset, session, dan QA |
| Email verifikasi/reset | Tidak ada provider; `EMAIL_PROVIDER_CONFIGURED=false`. OTP hanya keluar di local development non-Vercel | Domain dan sender email transaksi terautentikasi; SPF/DKIM/DMARC; bounce/retry/monitoring; uji end-to-end |
| Google login | Belum aktif | Google Identity Services, OAuth client, callback/origin, server-side ID token validation, secure linking, QA cancellation/failure |
| MFA/passkey untuk privileged | Belum diimplementasikan | TOTP/passkey/step-up policy untuk owner/superadmin/high-risk operation |
| Password blocklist | Policy min 15/max 128 sudah diterapkan; daftar kata sandi umum masih kecil dan statis | Integrasikan blocklist kompromi cukup luas dan audit KDF cost/rehash |
| DB role separation | **Blocker keamanan tinggi:** bootstrap DDL/CREATE ROLE dan query `withSystem` memakai pool yang sama dari `DATABASE_URL`; runtime credential terpisah belum ada | Jalankan schema provisioning via migrator/operator; pisahkan URL/role migrasi (DDL) dan runtime (least privilege); audit setiap `withSystem` privileged path |
| Versioned SQL migrations | Tidak ada direktori `drizzle/` atau migration SQL dalam ZIP; package memakai `drizzle-kit push` | Generate, commit, review, dan test migration SQL; uji upgrade serta rollback/restore dari salinan DB. Jangan `push` langsung ke production |
| Worker scheduler | Endpoint publik lama sudah diamankan; `/api/cron` sekarang menuntut Bearer `CRON_SECRET`, secret dapat dibuat via Vercel CLI script | Link project, jalankan `npm run vercel:cron-secret`, redeploy, konfirmasi cron berjalan dan observability tersedia |
| Neon Object Storage | Fallback file ada di PostgreSQL (batas ukuran/kuota sesuai kode); belum memakai object storage privat | Provisioning bucket private, signed URL, lifecycle/retention, isolation, malware scan |
| Upload security | Allowlist/signature validation ada; image re-encode dan antivirus belum tersedia | Re-encode raster, malware scanning, test polyglot/archive bombs, quotas dan retention |
| XLSX / PDF server-side | Import/export CSV dan print CSS; XLSX import serta PDF server-generated belum ada | Implementasi parser/exporter yang aman, resource limits, formula injection tests |
| Pembelian — retur/kredit pemasok | Belum tersedia | Model supplier return/credit note, inventory and GL posting, approval, audit and tests |
| Platform admin / break-glass | Belum tersedia | Desain role platform terpisah, MFA, audited support access, approval, emergency revocation |
| Backup & restore | Belum ada backup terjadwal atau restore drill yang terbukti | Backup terenkripsi, off-site, retention, monitoring, restore drill, bukti RPO/RTO |
| CI / SAST / DAST | Belum ada pipeline eksternal; repo belum ditautkan ke GitHub di sesi ini | Connect repo, protected branch, dependency/security scanning, static checks, automated tests |
| Android / Capacitor | Belum dibuat | Android build/signing, SDK/Studio, privacy permission review, Play Console account owner |
| Legal/privacy | Dokumentasi dasar ada; bukan legal opinion atau compliance attestation | Review UU PDP/PSE, terms, privacy notice, retention, processor agreements, data residency/transfer |
| Commercial hosting | Belum dipilih/diotorisasi | Pilih Vercel Pro/Enterprise atau host komersial yang sesuai. Vercel Hobby ditujukan untuk personal/non-commercial use |
| Production QA | Belum dilakukan pada build final | Build + typecheck + lint + integration E2E + browser manual + accessibility/performance + security test |

## Catatan penting

- `DATABASE_URL` adalah satu-satunya env wajib yang digunakan aplikasi saat ini untuk koneksi database. `CRON_SECRET` diperlukan untuk cron production dan dihasilkan otomatis oleh `scripts/configure-vercel-cron-secret.mjs`, bukan diketik ke `.env`. Integrasi email/Google akan memerlukan konfigurasi resmi bila dipilih.
- File `.env.example` tidak memiliki secret asli. Jangan commit `.env`, token, database dump, atau data pengguna.
- Jangan deploy ke production sampai blocker keamanan tinggi, email verification, migrations, backup, dan QA ditutup.
