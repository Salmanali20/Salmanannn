# ASV-Visualizer

Notebook Jupyter untuk memvisualisasikan hasil **MitoNGS** (tabel ASV + hasil taksonomi):

| Visualisasi | Output |
|---|---|
| **Krona** (interaktif) | `result/krona.html` |
| **Sankey** (interaktif) | `result/sankey.html`, `result/figures/sankey.png` |
| **Stacked bar plot** Class, Order, Family | `result/figures/stacked_bar_{class,order,family}.png/.svg` |
| Tabel kelimpahan per rank | `result/abundance_{class,order,family}.csv` |

## Struktur folder

```
ASV-Visualizer/
├── sample/
│   └── (file hasil MitoNGS)  <- taruh file kamu di sini, nama bebas
├── result/
│   └── figures/             <- gambar hasil (PNG/SVG)
├── visualizer.ipynb
├── environment.yml
└── README.md
```

## Format input

Taruh file di folder `sample/`. Nama file bebas, notebook mencarinya sendiri. Ada **dua cara**:

**A. Satu file gabungan MitoNGS** (misalnya `ASVTable (5).tsv.gz`)
- Berisi kolom `MD5`, `Top_hits`, `Superkingdom ... Species`, `Percent_identity`, `Query_coverage`, `E_value`, `Sequence`, lalu satu kolom per sampel.
- File **`.gz` langsung dibaca** (tidak perlu diekstrak). `taxresults.csv` tidak diperlukan.
- Nama sampel dirapikan otomatis: `3.fastq` menjadi `3`.

**B. Dua file terpisah**: `asv_table.tsv` (ID ASV + read per sampel) dan `taxresults.csv` (ID ASV + taksonomi). Taksonomi boleh sebagai kolom terpisah atau satu kolom lineage (`Chordata;Aves;...`).

Catatan penanganan data:
- **Hit ganda**: MitoNGS menggabungkan taksa dengan `;` bila beberapa hit setara (contoh `Alepes melanoptera;Alepes vari`). Di rank itu ASV dianggap *tak teridentifikasi* (`Unclassified <taksa induk>`). Jumlahnya dilaporkan di bagian ringkasan notebook. Ubah `AMBIGUOUS = "first"` di sel 1 untuk mengambil nama pertama.
- Nama diawali `;` (misalnya `;Tetraodontidae`) dibersihkan otomatis.
- Rank kosong menjadi **Unclassified <taksa induk>**, dan ASV tanpa taksonomi sama sekali menjadi **Unassigned**.
- Jika hanya ada **1 sampel**, stacked bar berisi satu batang. Untuk membandingkan sampel, gunakan tabel dengan beberapa kolom sampel.

---

## Cara menjalankan

> Butuh **Anaconda** atau **Miniconda** terpasang.
> Pengguna Windows: gunakan **Anaconda Prompt**. Pengguna macOS/Linux: gunakan Terminal.

### 1. Ekstrak zip dan masuk ke foldernya

```bash
cd path/ke/ASV-Visualizer
```

### 2. Cek environment conda yang sudah ada

```bash
conda env list
```

Pastikan belum ada environment bernama `ASV-Visualizer`. Kalau sudah ada dan ingin dibuat ulang: `conda env remove -n ASV-Visualizer`.

### 3. Buat environment dari `environment.yml`

```bash
conda env create -f environment.yml
```

Proses ini mengunduh semua paket dan bisa memakan beberapa menit.

> **Windows:** paket `krona` (bioconda) tidak tersedia untuk Windows. Buka `environment.yml`, beri tanda `#` di depan baris `- bioconda::krona`, lalu jalankan perintah di atas. Notebook akan otomatis membuat **sunburst Plotly** yang tampilannya mirip Krona.
> **Mac Apple Silicon (M1/M2/M3) gagal menemukan krona?** Jalankan: `CONDA_SUBDIR=osx-64 conda env create -f environment.yml`

### 4. Inisialisasi conda (cukup sekali)

```bash
conda init
```

Setelah itu **tutup dan buka kembali terminal**. Jika `conda activate` sudah berfungsi di komputermu, langkah ini boleh dilewati. (Windows: `conda init powershell` atau `conda init cmd.exe` sesuai terminal yang dipakai.)

### 5. Aktifkan environment

```bash
conda activate ASV-Visualizer
```

Prompt akan berubah menjadi `(ASV-Visualizer) ...`. Cek Krona (opsional, tidak berlaku di Windows):

```bash
ktImportText
```

### 6. Letakkan file MitoNGS

Salin file hasil MitoNGS ke folder `sample/` (cukup `ASVTable (...).tsv.gz`, atau pasangan `asv_table.tsv` + `taxresults.csv`).

### 7. Jalankan Jupyter

Dari dalam folder `ASV-Visualizer/` (environment sudah aktif):

```bash
jupyter lab
```

Browser akan terbuka. Klik dua kali `visualizer.ipynb`.

### 8. Pilih kernel yang benar

Pojok kanan atas notebook: pastikan kernel **Python 3 (ipykernel)** dari environment aktif. Jika diminta memilih, pilih yang ada tulisan `ASV-Visualizer`.

### 9. Jalankan semua sel

Menu **Run > Run All Cells**. Atur parameter (jumlah taksa teratas, tingkat Sankey, dll.) di **sel nomor 1**.

### 10. Lihat hasil

- Gambar: `result/figures/`
- Krona / Sankey interaktif: buka file `.html` di `result/` dengan **browser** (klik dua kali dari File Explorer/Finder). Alamat file juga dicetak di output notebook.

### 11. Selesai

```bash
conda deactivate
```

Untuk memakai lagi: `conda activate ASV-Visualizer` lalu `jupyter lab`.

---

## Pengaturan penting (sel nomor 1 di notebook)

| Variabel | Fungsi | Default |
|---|---|---|
| `TOP_N` | Jumlah taksa teratas di stacked bar, sisanya jadi *Others* | `15` |
| `RELATIVE` | `True` = persen, `False` = jumlah read | `True` |
| `SANKEY_RANKS` | Tingkat taksonomi di Sankey | `phylum, class, order, family` |
| `SANKEY_MIN_PCT` | Node di bawah x% digabung jadi *Others* | `0.5` |
| `SAMPLE_ORDER` | Urutan sampel di grafik | urutan di tabel |
| `AMBIGUOUS` | Hit ganda `A;B`: `unresolved` (tak teridentifikasi) atau `first` (ambil pertama) | `unresolved` |
| `CLEAN_SAMPLE_NAMES` | Hapus `.fastq` dari nama sampel | `True` |
| `TAX_ID_COL`, `RANK_COLS`, `LINEAGE_STR_COL` | Isi manual jika deteksi kolom gagal | `None` |

## Troubleshooting

| Masalah | Solusi |
|---|---|
| `conda: command not found` | Jalankan langkah 4 (`conda init`), lalu tutup dan buka ulang terminal. |
| `PackagesNotFoundError: krona` | Kamu di Windows / Mac ARM, lihat catatan di langkah 3. |
| `File ASV table tidak ditemukan` | Pastikan file ada di `sample/` dan Jupyter dijalankan dari folder `ASV-Visualizer/`. |
| `Tidak ada kolom ... yang cocok dengan ID ASV` | ID di `taxresults.csv` berbeda dengan `asv_table.tsv`. Set `TAX_ID_COL` di sel 1. |
| `Kolom rank ... tidak terdeteksi` | Set `RANK_COLS` atau `LINEAGE_STR_COL` di sel 1. |
| PNG Sankey gagal dibuat | Tidak masalah, file HTML tetap dibuat. Pakai tombol kamera di plot, atau `pip install kaleido==0.2.1`. |
| Ingin memperbarui environment | `conda env update -f environment.yml --prune` |
| Ingin menghapus environment | `conda env remove -n ASV-Visualizer` |
