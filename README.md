# Day Cycle Background

Background hero animasi **malam → pagi → siang → sore → malam**, dari 4 gambar ilustrasi jendela (sudah di-flip horizontal).

| File | Isi |
|---|---|
| `index.html` | Demo siap pakai, self-contained (SVG + CSS + JS inline, tanpa request tambahan) |
| `flipped/*.jpg` | 4 gambar asli yang sudah di-flip horizontal. Hanya referensi, **tidak dipakai** oleh halaman |

## Kenapa tidak pakai gambar

Keempat ilustrasi punya bentuk yang identik, hanya warnanya yang berbeda. Jadi gambar di-trace ulang menjadi vektor:

- **Bentuk disimpan sekali** (`<defs>`), lalu dipakai ulang oleh 4 layer suasana lewat `<use>`, masing-masing dengan palet warna/gradiennya sendiri.
- Hasilnya tajam di resolusi apa pun (termasuk 4K/retina) dan jauh lebih ringan:

| | Ukuran |
|---|---|
| 4 JPG asli | 420 KB |
| `index.html` (semua suasana + bulan + CSS + JS + demo) | 71 KB, **≈22 KB gzip** |

## Cara kerja animasi

- **Transisi warna** = crossfade `opacity` antar layer suasana via Web Animations API. Animasi ini berjalan di compositor (GPU), bukan main thread, sehingga tidak ada repaint per frame.
- **Hemat memori:** saat diam hanya 1 layer yang dirender; saat transisi maksimal 2 layer. Layer lain `display:none`.
- **Bulan** adalah layer terpisah yang di-*mask* hanya ke area langit, sehingga bisa tertutup gunung, awan, dan kusen jendela. Lintasannya **setengah lingkaran dari kiri ke kanan**:
  - sore → malam: muncul dari balik gunung di kiri, lalu naik ke posisinya di gambar malam;
  - selama malam: terus bergerak pelan melintasi puncak busur;
  - malam → pagi: turun di kanan dan tenggelam di balik gunung.

  Kecepatannya (hampir) konstan. Sabit bulan tetap tegak sepanjang lintasan.
- **Bintang:** yang statis ada di layer malam, 5 bintang berkelip ada di layer kecil terpisah.
- **Burung:** 3 siluet burung terbang dari kanan ke kiri sambil mengepakkan sayap, setiap kali halaman dimuat dan sebelum tiap pergantian suasana. Mereka juga di-mask ke langit (lewat di belakang kusen, tirai, dan gunung). Warnanya menyesuaikan suasana, dan kepakan sayapnya memakai `transform`, jadi tetap dijalankan GPU.
- **Otomatis pause** saat hero di-scroll keluar layar atau tab tidak aktif.
- **`prefers-reduced-motion`:** tampil statis, suasananya sesuai jam lokal pengunjung.
- **Tanpa JS:** suasana malam tampil statis.

## Memasang di website

Salin tiga bagian dari `index.html`:

1. **CSS** di dalam `<style>`, bagian `DAY-CYCLE BACKGROUND`. Bagian `DEMO KONTEN` opsional.
2. **Markup** `<section class="daycycle" data-daycycle …>…</section>`. Ganti isi `.dc-content` dengan konten Anda; elemen `.dc-bg` adalah background-nya.
3. **`<script>`** di akhir body.

## Konfigurasi

| Opsi | Default | Keterangan |
|---|---|---|
| `data-hold` | `4000` | Lama tiap suasana diam (ms) |
| `data-fade` | `6000` | Lama transisi (ms). Satu siklus = 4 × (hold + fade) = 40 detik |
| `data-fly` | `6000` | Lama burung melintasi jendela (ms), maksimal hold + fade |
| `data-start="clock"` | – | Mulai dari suasana sesuai jam lokal (pagi 05–11, siang 11–15, sore 15–18, malam 18–05) |
| `--dc-focus-x` | `.82` | Bagian gambar yang dipertahankan saat ter-crop horizontal (layar portrait). `.82` menjaga jendela di tengah |
| `--dc-focus-y` | `.3` | Bagian yang dipertahankan saat ter-crop vertikal (layar ultrawide), supaya bulan tetap terlihat |

Event saat suasana berganti:

```js
document.querySelector('[data-daycycle]').addEventListener('daycycle:change', e => {
  console.log(e.detail.name); // 'malam' | 'pagi' | 'siang' | 'sore'
});
```

Atribut `data-scene` di `.daycycle` juga ikut berubah, jadi bisa dipakai di CSS, mis. `.daycycle[data-scene="siang"] { … }`.

## Dukungan browser

Memakai container query units, `svh`, `aspect-ratio`, `mask-image`, dan WAAPI `commitStyles`: Chrome/Edge 105+, Safari 16+, Firefox 110+. Untuk browser lama ada fallback ke `vw`/`vh`.

Sudah diuji di Chromium pada ukuran desktop, laptop, tablet, ponsel portrait/landscape, dan ultrawide. Belum diuji langsung di Safari dan Firefox.
