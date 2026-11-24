# Aset

Semua aset vektor yang dipakai `index.html` versi terakhir, diekspor sebagai file SVG mandiri. Bentuk yang di halaman dipakai bersama sudah dimasukkan ke tiap file, jadi file bisa langsung dibuka di browser, Figma, atau Illustrator.

Koordinat memakai satuan setengah piksel dari artwork 2000×1116 yang sudah di-flip. Area jendela+tirai adalah `viewBox="1800 0 2200 1920"`, yaitu 1100×960 px.

| Folder / file | Isi |
|---|---|
| `scenes/pemandangan-{malam,pagi,siang,sore}.svg` | Pemandangan di luar jendela per suasana (sudah dipotong di area kaca, latar transparan). Inilah sumber tekstur WebGL. Versi malam sudah termasuk bintang statis. |
| `room/tirai-kusen-overlay.svg` | Tirai, kusen, dan ambang jendela: putih/hitam transparan, mengikuti warna tembok di belakangnya |
| `composites/jendela-{…}.svg` | Gabungan siap pakai per suasana: warna tembok (HSL) + pemandangan + overlay tirai/kusen (+ bulan untuk malam) |
| `png/jendela-{…}@2x.png` | Render PNG 2200×1920 dari file komposit |
| `moon/bulan.svg`, `moon/bulan-tanpa-glow.svg` | Bulan sabit dengan/tanpa cahaya |
| `birds/elang-sprite-14-frame.svg` | Sprite kepakan sayap elang (14 frame berjajar horizontal, tiap frame 174×146) |
| `birds/elang-frame-01…14.svg` | Tiap frame sebagai file sendiri (sprite yang sama, `viewBox` digeser per frame) |
| `png/elang-sprite.png` | Render PNG sprite burung |
| `stars/bintang-sparkle.svg` | Bintang 4 sudut (berkelip) |
| `masks/mask-langit-bulan.svg` | Mask area langit untuk bulan (tertutup awan, gunung, kusen) |
| `masks/mask-langit-awan-burung.svg` | Mask langit + awan untuk burung (burung terbang di depan awan) |
| `masks/area-kaca-jendela.svg` | Bentuk area kaca (clip pemandangan) |
| `tokens-warna-tembok.json` | Warna tembok per suasana: nilai HSL + RGB |
