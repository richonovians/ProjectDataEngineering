# ProjectDataEngineering
# Proyek : Klasifikasi Provinsi Rawan Bencana di Indonesia Berdasarkan Data Kejadian Bencana Alam (2021–2023)

## Deskripsi Proyek  
Proyek ini bertujuan untuk membangun pipeline ETL (Extract, Transform, Load) untuk mengolah dan menganalisis data kejadian bencana alam di Indonesia selama periode 2021–2023. Data dikumpulkan secara otomatis dari situs resmi Badan Pusat Statistik (BPS), kemudian dibersihkan dan distandardisasi untuk menghilangkan inkonsistensi serta nilai yang hilang. Selanjutnya, data digabungkan antar tahun dan dianalisis lebih lanjut untuk menambah atribut baru seperti total kejadian bencana dan klasifikasi provinsi rawan bencana.

Dengan memanfaatkan AutoML dari PyCaret, proyek ini mengembangkan model klasifikasi untuk memprediksi status kerawanan suatu provinsi berdasarkan jumlah dan jenis bencana yang terjadi. Model dievaluasi, disimpan, dan digunakan untuk menghasilkan prediksi terhadap seluruh data historis. Selain itu, analisis korelasi antar jenis bencana dilakukan untuk memahami keterkaitan antar kejadian bencana, yang divisualisasikan dalam bentuk heatmap interaktif.

Hasil akhir dari proyek ini adalah dataset bersih yang dilengkapi dengan label kerawanan, model machine learning yang dapat digunakan ulang, serta visualisasi prediktif dan analitis yang mendukung pengambilan keputusan berbasis data untuk mitigasi risiko bencana di tingkat provinsi.

---

## Manfaat Data / Use Case  
- **Tujuan Proyek:** Mengembangkan pipeline otomatis untuk mengklasifikasi provinsi rawan bencana di Indonesia berdasarkan data kejadian bencana alam dari tahun 2021 hingga 2023, serta mengevaluasi hubungan antar jenis bencana melalui analisis korelasi.

- **Manfaat:**
  - Menyediakan dataset terstruktur dan bersih yang mencerminkan tren bencana alam per provinsi dan per tahun.
  - Menghasilkan model machine learning yang dapat memprediksi tingkat kerawanan suatu provinsi terhadap bencana alam.
  - Memberikan insight visual dan analitis tentang hubungan antar jenis bencana yang dapat mendukung kebijakan mitigasi.
  - Mendukung pengambilan keputusan berbasis data oleh pemerintah daerah, BNPB, dan lembaga kebencanaan lainnya.
    
---

## Serving Analisis  
Data hasil ETL disimpan dalam format CSV yang bersih dan terstruktur, sehingga siap digunakan untuk analisis eksploratif dan pemodelan lanjutan. Dataset ini dapat dengan mudah dimuat ke dalam Looker Studio untuk keperluan visualisasi interaktif, pelacakan tren, serta pemetaan tingkat kerawanan bencana secara geografis dan temporal. Visualisasi ini membantu pengambil kebijakan dan pemangku kepentingan memahami distribusi risiko bencana secara lebih intuitif dan berbasis data.

## Serving Machine Learning  
Dataset yang telah dibersihkan dan distandardisasi digunakan untuk melatih model klasifikasi menggunakan pustaka PyCaret. Berbagai algoritma klasifikasi dievaluasi secara otomatis untuk mengidentifikasi model terbaik dalam memprediksi status kerawanan bencana provinsi berdasarkan data kejadian bencana alam. Model terbaik yang terpilih digunakan untuk menghasilkan prediksi pada seluruh dataset historis dan disimpan untuk kebutuhan deployment atau analisis lanjutan.

---

# Pipeline
## Extract ( Pengambilan Data ) 
- **Sumber Data:**
  - Jumlah Bencana Alam menurut Provinsi dan Jenis Bencana Alam – Badan Pusat Statistik (BPS)
    https://www.bps.go.id/id/statistics-table/3/TUZaMGVteFVjSEJ4T1RCMlIyRjRTazVvVDJocVFUMDkjMw==/jumlah-bencana-alam-menurut-provinsi-dan-jenis-bencana-alam--kejadian---2023.html
  - Data dikumpulkan secara otomatis menggunakan web scraping dengan Selenium untuk setiap tahun dalam rentang 2021 hingga 2023.

- **Metode Pengambilan:**
  - Data diambil secara otomatis dari situs resmi BPS menggunakan Selenium WebDriver dalam mode headless (tanpa tampilan browser).
  - Halaman web dimuat untuk setiap tahun (2021–2023) dan kontennya diambil dalam bentuk HTML.
  - Tabel statistik diekstrak langsung dari HTML menggunakan fungsi pandas.read_html().
  - Setiap tabel hasil scraping disimpan sebagai file CSV terpisah untuk setiap tahun.

- **Penanganan Error:**
  - Saat melakukan scraping data dari situs BPS, terkadang halaman membutuhkan waktu lebih lama untuk dimuat atau struktur tabel tidak terdeteksi. Untuk mengatasinya, digunakan time.sleep() untuk memberi jeda pemuatan halaman sebelum parsing HTML.
  - Jika terjadi kegagalan saat membaca tabel dengan pandas.read_html(), blok try-except digunakan untuk menangkap dan mencetak pesan error tanpa menghentikan keseluruhan proses scraping, sehingga scraping tahun lain tetap bisa dilanjutkan.
  - Tabel kosong atau halaman tanpa data ditangani dengan pengecekan if tables: sebelum menyimpan file CSV.
    
---

## Transform ( Pembersihan & Transformasi )   
- **Pembersihan:**
  - Mengganti nilai tidak valid seperti "..." dan "-" dengan angka 0.
  - Mengisi nilai kosong (NaN) dengan 0 agar tidak mengganggu analisis numerik.
  - Menyederhanakan nama kolom untuk konsistensi dan kemudahan pemrosesan (contoh: Jumlah Bencana Alam - Gempa Bumi menjadi gempa_bumi).
  - Menghapus baris ringkasan nasional seperti "Indonesia" agar fokus hanya pada data tingkat provinsi.

- **Transformasi:**
  - Menambahkan kolom tahun ke setiap DataFrame tahunan dan menyusunnya agar urut secara konsisten.
  - Menggabungkan data dari tahun 2021, 2022, dan 2023 menjadi satu DataFrame gabungan.
  - Menambahkan kolom total_bencana (jumlah seluruh jenis bencana per provinsi) dan kolom target klasifikasi rawan berdasarkan threshold tertentu (≥20).
  - Menyimpan data yang telah dibersihkan dan digabungkan ke dalam file CSV sebagai output akhir.

---

## Load ( Pemindahan ke Target ) 
- **Target:**
  - File CSV gabungan (data_bencana_gabungan.csv) yang berisi data kejadian bencana alam per provinsi dari tahun 2021 hingga 2023. File ini menjadi output utama yang dapat digunakan untuk analisis lebih lanjut, pelatihan model machine learning, dan visualisasi di Power BI atau Looker Studio.
    
- **Metode:**
  - Data yang telah dibersihkan dan digabung disimpan ke dalam file CSV menggunakan fungsi to_csv() dari pandas.
  - File CSV ini kemudian dapat dimuat ke dalam sistem visualisasi atau database eksternal seperti PostgreSQL untuk analisis lanjutan.
  - Format CSV dipilih karena fleksibel dan kompatibel dengan berbagai platform analisis data.

---

## Arsitektur / Workflow ETL  
- **Alur Modular:**
  - Proses ETL dibagi dalam beberapa fungsi modular:
    - crawl_bencana_bps() untuk proses extract data dari situs BPS menggunakan Selenium.
    - bersihkan_data_bencana() untuk membersihkan dan menstandarkan data dari masing-masing tahun.
    - gabungkan_data_bencana() untuk menggabungkan data multi-tahun menjadi satu DataFrame konsisten.
    - Proses transformasi lanjutan, pelabelan klasifikasi, pelatihan model, dan visualisasi dilakukan secara berurutan di dalam notebook.

- **Tools yang Digunakan:**  
  - Python 3.x  
  - Library: `pandas`, `numpy`, `selenium`, `pycaret`, `plotlib`, `seaborn`, `plotly.express`, `scikit-learn`.
  - Environment: Google Colab digunakan sebagai platform utama untuk pemrosesan data, pelatihan model, dan eksplorasi visual
  - Visualisasi: Power BI dan Looker Studio digunakan untuk menyajikan hasil analisis secara interaktif.

---

## Kode Program  
- **Struktur Kode:**
  - Kode dibagi ke dalam dua notebook terpisah: satu untuk proses ETL (Extract, Transform, Load), dan satu lagi untuk pelatihan serta evaluasi model Machine Learning.
  - Penamaan variabel dan fungsi menggunakan konvensi yang deskriptif, seperti bersihkan_data_bencana, gabungkan_data_bencana, df_cleaned_2021, dll.
    
- **Machine Learning:**
  - Menggunakan pustaka PyCaret untuk klasifikasi provinsi rawan bencana berdasarkan jumlah kejadian bencana alam.
  - Model terbaik dipilih dari hasil compare_models(), kemudian disimpan dan dievaluasi menggunakan evaluate_model() serta predict_model().
  - Keluaran mencakup label prediksi (prediction_label) dan skor keyakinan model (prediction_score).
  - Visualisasi performa model ditampilkan dalam bentuk barplot dan heatmap korelasi. 

- **Link Projek:**  
  - ETL Pipeline dan Machine Learning:  
    https://colab.research.google.com/drive/1nIJ_yO_uHtEmGO-5dUDlvMfUzh54TYlh?usp=sharing
  - Looker
    https://lookerstudio.google.com/reporting/4d07785f-246c-404c-8db9-62ea8101afd9 
---

## Kontributor

| Nama Lengkap                        | NIM         | Peran                |
|------------------------------------|-------------|----------------------|
| Richo Novian Saputra               | 234311024   | Project Manager      |
| Arya Yudha Prasetya                | 234311007   | Data Analyst         |
| Mohammad Fakhriza Maftukhin        | 234311018   | Data Engineer        |
| Muhammad Aulia Al Farouq           | 234311020   | Data Engineer        |
