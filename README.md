# Promkes Video Explainer Engine (plugin `promkes`)

Mengimplementasikan spesifikasi **Video Explainer Promkes V3.0 — Master Orchestrator M00** sebagai plugin: mengubah poster, infografis, caption, naskah, atau audio promosi kesehatan menjadi **paket produksi yang terlacak** (fact lock kesehatan → screenplay → panduan visual & kontinuitas → storyboard → motion → prompt referensi → manifest renderer-neutral → QA), dengan **state di disk** sehingga proyek bisa dilanjutkan setelah konteks/token habis.

## Pasang
Unggah berkas `promkes.plugin` lewat antarmuka plugin Cowork (atau klik *Save plugin* pada kartu berkasnya). Plugin baru dimuat pada **tugas berikutnya**, bukan tugas yang sedang berjalan. (Keterangan pemasangan ini berasal dari tutorial pihak ketiga yang mengutip dokumentasi Anthropic dan belum diverifikasi ke dokumentasi resmi; ikuti petunjuk antarmuka bila berbeda.) Skill dan perintah bekerja di chat, Cowork, dan Claude Code; sub-agent QA hanya di Cowork dan Claude Code.

## Mulai cepat
| Perintah | Fungsi |
|---|---|
| `/promkes:start [judul] [LITE\|STANDARD\|ADVANCED]` | intake, probe kemampuan, buat folder proyek + state |
| `/promkes:resume [folder] [status\|resume-prompt\|phase N]` | lanjutkan dari state di disk; cetak blok resume |
| `/promkes:qa [folder]` | validator + QA independen + laporan defect |
Skill lain (`promkes-orchestrator`, `promkes-source-health`, dst.) terpicu otomatis oleh permintaan terkait video explainer kesehatan, atau bisa dipanggil langsung.

## Isi plugin
```
.claude-plugin/plugin.json
agents/promkes-qa-auditor.md              # QA berkonteks bersih (Cowork/Claude Code)
skills/
  start · resume · qa                      # entry point eksplisit (/promkes:…)
  promkes-orchestrator/                    # M00: alur, gerbang, state, toolbox, referensi, skema
    scripts/ promkes_state.py promkes_validate.py selftest.py
    references/ phases-and-gates · variables-enums-defaults · guardrails · recovery-and-states · ids-traceability-records · manifest.schema.json · manifest.example.json
  promkes-source-health/                   # S01+S02  Fase 01–02
  promkes-campaign-screenplay/             # S03+S04  Fase 03–04
  promkes-visual-continuity/               # S05+S13  Fase 05
  promkes-storyboard-motion/               # S06+S07  Fase 06–07
  promkes-prompt-reference/                # S08      Fase 08
  promkes-audio-subtitle/                  # S09+S11  Fase 09, 11
  promkes-renderer-manifest/               # S10      Fase 10
  promkes-qa-handoff/                      # S12      Fase 12
docs/DESIGN_REVIEW.md                      # analisis kritis, register risiko, mitigasi, bukti uji
```

## Folder proyek (dibuat `init`)
`promkes_projects/<PROJECT_ID>/` berisi `project_state.json` (sumber kebenaran), `RESUME.md` (otomatis), `tools/` (salinan skrip + skema), dan folder fase `00_intake … 13_handoff`. Setiap perubahan state ditulis atomik (dengan `.bak`), artefak di-hash, dan fase turunan otomatis STALE bila hulu berubah.

## Melanjutkan setelah kehabisan token/konteks
Buka sesi baru → `/promkes:resume <folder proyek>`. Tanpa plugin: tempel isi `RESUME.md` dan lampirkan folder proyek. Di chat claude.ai sistem berkas sandbox direset antar-tugas: unduh folder proyek setelah tiap sesi dan unggah kembali saat melanjutkan (di Cowork/Claude Code folder kerja sudah persisten). Fase berat (06, 08, 10, 12) dikerjakan per batch dan dicatat dengan `progress`, sehingga yang hilang paling banyak satu batch.

## Yang dijamin dan yang tidak
- **Dijamin oleh skrip (diuji lewat `selftest.py`)**: gerbang prasyarat fase, deteksi artefak berubah, propagasi STALE, aritmetika frame (half-up, half-open, tanpa celah), referensi silang manifest, aturan lifecycle (APPROVED butuh berkas), presisi subtitle ≤ presisi audio, klaim tak boleh dipakai bila CONTRADICTED/UNKNOWN/ditahan, ceiling status pengiriman berbasis bukti.
- **Tidak dijamin**: kebenaran klaim kesehatan, mutu visual/naratif, konsistensi identitas karakter pada model generatif, dan keberhasilan render. Validator hanya memeriksa struktur dan aritmetika. **Tidak ada persetujuan klinis, institusional, legal, atau publikasi yang tersirat**; itu selalu keputusan manusia.
- Kode renderer tidak dikirim sebagai templat siap pakai karena tidak dapat diuji secara offline; skill menyusunnya per proyek setelah platform dan versi diperiksa, dan menandainya UNVERIFIED sampai dikompilasi.

## Keamanan
Skrip bersifat lokal: hanya pustaka standar Python, tanpa akses jaringan, tanpa `shell=True`, subprocess hanya untuk probe versi biner tetap. Skrip berjalan dengan izin Anda; baca sebelum memasang (ukurannya kecil). Pencarian web untuk verifikasi klaim mengirim *kueri* ke layanan pencarian, bukan berkas pribadi.

## Pemeliharaan
Naikkan `version` di `plugin.json` setiap rilis. Jalankan `python tools/selftest.py` setelah mengubah skrip. Keputusan plugin di luar spesifikasi (FPS 30, pembulatan half-up, heuristik 2,2 kata/detik, skema ID, pemilik tujuh Guide) tercantum di `docs/DESIGN_REVIEW.md` dan semuanya reversibel.
