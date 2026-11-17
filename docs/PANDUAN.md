# Panduan — Papan Pergerakan Staf JTMK

App sendiri yang menggantikan papan putih: siapa **ada** di pejabat, **HBH**, atau **tiada** (kursus, bengkel, cuti…).

## Seni bina

| Bahagian | Di mana | Fungsi |
|---|---|---|
| Laman (app) | **PBUHub** — `pbuhub.com/pergerakanstaff` (fail dalam `pbuhub_fasa1/public/pergerakanstaff/`) | `index.html`, `app.js`, `app.css`, `config.js`, PWA (boleh dipasang di telefon) |
| Jambatan | **Google Apps Script** (projek berasingan) | `apps-script/Code.gs`: log masuk emel (pilihan + kod OTP), simpan rekod, tulis ke Google Sheet |
| Data staf & papan bulanan | Google Sheet **PERGERAKAN STAFF** | tab `Maklumat Staff` (senarai staf) + tab bulanan `NOV2026`, `DIS2026`… |
| Data sistem (private) | Google Sheet **PERGERAKAN STAF JTMK — DATA SISTEM** | rekod penuh, cuti umum, tetapan, sesi, log (dicipta oleh `setup()`) |

PBUHub hanya menghidangkan laman (tiada data dalam pangkalan data PBUHub). Semua penyimpanan dibuat melalui jambatan Apps Script ke Google Sheet.

## Langkah 1 — Jambatan Apps Script (sekali sahaja)

1. Buka https://script.google.com.
   - Projek kosong **"Keberadaan Staf JTMK"** sudah ada. Buka projek itu dan namakan semula kepada **Pergerakan Staf JTMK**, atau cipta projek baharu.
2. Tampal seluruh kandungan `apps-script/Code.gs` ke dalam `Code.gs`, kemudian tekan **Ctrl+S**.
3. Pilih fungsi **setup** dan tekan **Run**. Benarkan akses: Sheets, hantar emel, pencetus dan sambungan luar.
   - Log akan menunjukkan: *Siap … staf dari Maklumat Staff: 56 · tab dikemas kini: NOV2026*.
4. Deploy sebagai Web app: pergi ke **Deploy → New deployment → Web app**.
   - **Execute as:** Me
   - **Who has access:** **Anyone**
   - Salin URL yang berakhir dengan `/exec`.
5. Tampal URL itu dalam `pbuhub_fasa1/public/pergerakanstaff/config.js` (dan `pergerakan-staf/config.js`):
   `window.PERGERAKAN_API = 'https://script.google.com/macros/s/…/exec';` — atau beri URL itu kepada Claude untuk ditampal.

**Syarat:** akaun yang deploy mesti ada akses **edit** pada sheet PERGERAKAN STAFF.

**Jika pilihan "Anyone" tiada** (disekat oleh pentadbir MyGovUC), deploy dari akaun Gmail peribadi. Kongsi sheet PERGERAKAN STAFF (akses edit) dengan akaun itu terlebih dahulu.

**Mengemas kini kod kemudian:** pergi ke **Deploy → Manage deployments**, tekan ✏️, pilih **New version**. URL `/exec` kekal sama.

## Langkah 2 — PBUHub (`/pergerakanstaff`)

App sudah dipasang dalam PBUHub Fasa 1 (13/11/2026):

- Fail app: `pbuhub_fasa1/public/pergerakanstaff/` (salinan dari folder ini; `apps-script/` dan `docs/` TIDAK disalin).
- Tempatan: http://localhost/pbuhub_fasa1/pergerakanstaff (peraturan **1b** dalam `pbuhub_fasa1/.htaccess`).
- Pelayan sebenar (document root = `public/`): `https://pbuhub.com/pergerakanstaff` terbuka terus dari folder
  `public/pergerakanstaff/` (`.htaccess` folder itu: `DirectoryIndex index.html`). Tiada route Laravel atau jadual DB.
- Selagi `PERGERAKAN_API` kosong, app memaparkan "Sistem belum disambungkan ke pelayan".
- eBOARD (pilihan): isi `const PERGERAKAN_URL = 'https://pbuhub.com/pergerakanstaff/';` dalam `eboard-jtmk/index.html`.

**Kemas kini app kemudian:** ubah fail dalam folder ini, naikkan `VERSI` dalam `sw.js`, kemudian salin `index.html`, `app.js`,
`app.css`, `config.js`, `manifest.webmanifest`, `sw.js` dan `icons/` ke `pbuhub_fasa1/public/pergerakanstaff/`.
Pengguna akan nampak mesej "Versi baharu tersedia — Muat semula".

GitHub Pages / Cloudflare (`untuk-github/`, `pergerakan-staf-cloudflare.zip`) tidak diperlukan lagi.

## Pasang di telefon

Selepas log masuk, staf nampak kad **"Pasang app Pergerakan Staf"** di atas papan (juga butang kuning **Pasang App** di bar atas).
Kad disorok sendiri jika app sudah dipasang; butang **Nanti** menyoroknya 7 hari.

Dalam app (telefon atau app yang dipasang), kad staf disusun **2 lajur** dan nama panjang boleh 2 baris; paparan web komputer kekal 1 lajur.
Butang **carian** (kanta) di bar atas selepas log masuk terus membawa ke kotak "Cari nama staf…" (pintasan papan kekunci: `/`).

- **Android (Chrome) / komputer (Chrome, Edge):** tekan **Pasang app**, kemudian **Pasang**.
- **iPhone (Safari):** tekan **Kongsi**, kemudian **Tambah ke Skrin Utama**. Buka app dan log masuk sekali lagi (iPhone simpan log masuk app berasingan).
- **Syarat:** pbuhub.com mesti **HTTPS** (sijil SSL). Tanpa HTTPS pelayar tidak membenarkan pemasangan app (localhost dikecualikan).

## Peraturan sistem

- **Log masuk:** **emel sahaja** (keputusan pemilik 13/11/2026) — tiada kata laluan, tiada kod.
  - Hanya emel dalam tab **Maklumat Staff** boleh masuk; paling banyak 10 log masuk bagi satu emel dalam 15 minit.
  - Sesi diingati selama 30 hari. Setiap log masuk dicatat dalam Log audit (cara EMEL/OTP, peranti).
  - **Risiko yang diterima:** senarai emel staf terbuka dalam sheet awam, jadi sesiapa yang tahu emel staf/admin boleh masuk atas nama mereka.
  - **Suis kecemasan:** Admin → Tetapan → **Cara log masuk** → *Emel + kod 6 digit*. Semua peranti (termasuk admin) dilog keluar serta-merta
    dan perlu masuk semula dengan kod yang dihantar ke emel (kod sah 10 minit; 5 cubaan; 3 permintaan kod dalam 15 minit).
    Tukar semula ke *Emel sahaja* bila-bila masa. Tetapan disimpan sebagai `LOG_MASUK` dalam tab TETAPAN data sistem.
- **Lajur Maklumat Staff** dikenal pasti mengikut tajuk:
  - **NAMA** dan **EMEL** wajib.
  - **GRED**, **JAWATAN** dan **TREK** pilihan.
  - Susunan lajur boleh berubah tanpa menjejaskan sistem.
- **Corak lalai:** Isnin, Rabu dan Jumaat **Di Pejabat**; Selasa dan Khamis **HBH**.
  - **Tiada HBH** (arahan pemilik 13/11): KJ **Pn. Shardilla** dan **Pn. Noor Farizah** sentiasa Di Pejabat pada hari HBH.
    Senarai boleh diubah: Admin → Tetapan → *Sentiasa di pejabat (tiada HBH)*. Staf berjawatan KETUA JABATAN dalam Maklumat Staff juga automatik.
  - Hari cuti umum ditandakan **Cuti Umum**.
  - Keutamaan: rekod individu, kemudian cuti umum, kemudian corak minggu.
- **Tab bulanan:**
  - Setiap simpan atau batal menulis status ke tab bulan berkenaan.
  - Tab bulan baharu (cth. DIS2026) dicipta automatik.
  - Pencetus harian 6 pagi mengemas kini tab bulan semasa, dan mencipta tab bulan depan mulai 25 hb.
  - Jenis cuti ditulis sebagai **CUTI** sahaja, kerana sheet ini boleh dilihat orang lain.
- **Admin** (`farizahhazain@pbu.edu.my`): boleh mengemas kini semua staf, membuat rekod pukal, mengurus cuti umum, corak minggu, nama paparan dan jenis status, melihat log, membuat eksport CSV, dan menekan butang **Kemas kini Sheet sekarang**.
- **Staf:** hanya boleh mengemas kini status sendiri, untuk hari ini dan tarikh akan datang.

## Ujian tempatan

```
node apps-script/dev/test.js      # 24 ujian jambatan
node apps-script/dev/server.js    # API ujian di http://localhost:8787/exec
```

Selepas pelayan ujian berjalan, buka http://localhost/pergerakan-staf/ (folder pembangunan sahaja; salinan PBUHub guna pelayan ujian hanya jika `localStorage['pg-api']` diset) dan log masuk dengan `ujian.admin@eboard.test`. (mod emel + kod: kod dipaparkan terus pada skrin).

Pelayan ujian **tidak** mengubah sheet sebenar; tab bulanan ditiru dalam memori.

Padam fail `apps-script/dev/.dev-*.json` selepas selesai menguji.
