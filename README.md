# UTS Graf: Analisis Graf dan Sistem Rekomendasi Retail Rocket

## Deskripsi Proyek

Proyek ini mengimplementasikan analisis graf dan sistem rekomendasi menggunakan Retail Rocket E-commerce Dataset, Python, Neo4j, dan Jupyter Notebook. Proyek mencakup pengolahan data interaksi, pembentukan graf, Vector Space Modelling (VSM), proyeksi graf, perhitungan cosine similarity, serta evaluasi rekomendasi menggunakan K-Nearest Neighbors (KNN).

## Tujuan

1. Membagi data interaksi ke dalam periode training dan testing berdasarkan timestamp.
2. Membentuk graf interaksi pengguna dan item di Neo4j.
3. Menganalisis representasi matriks User-Item dan Item-User.
4. Membandingkan proyeksi graf dan menghitung kemiripan antaritem.
5. Menyimpan relasi kemiripan pada Neo4j menggunakan `SIMILAR_TO`.
6. Menerapkan dan mengevaluasi sistem rekomendasi KNN menggunakan Hit@5 dan NDCG@5.

## Dataset

Dataset yang digunakan adalah Retail Rocket E-commerce Dataset.

Sumber: https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset

Kolom utama meliputi `timestamp`, `visitorid`, `event`, `itemid`, dan `transactionid`.

Periode training dan testing serta proses pembentukan file CSV dijelaskan pada notebook.

## Teknologi

- Python
- Neo4j
- Neo4j Python Driver
- Pandas dan NumPy
- SciPy
- Scikit-learn
- Matplotlib
- Jupyter Notebook

## Struktur Repository

- `UTS_Graf.ipynb`: dokumentasi dan analisis utama.
- `scripts/`: skrip pengolahan data, pemodelan graf, similarity, dan rekomendasi.
- `data/`: data yang disertakan atau petunjuk unduhan dataset.
- `figures/`: visualisasi dan dokumentasi graf.
- `requirements.txt`: dependensi Python.
- `.gitignore`: pengecualian file lokal dan sensitif.

## Cara Menjalankan

1. Siapkan Python dan Neo4j.
2. Buat dan aktifkan virtual environment.
3. Instal dependensi dari `requirements.txt`.
4. Unduh dataset dari sumber yang dicantumkan dan tempatkan sesuai konfigurasi skrip.
5. Konfigurasikan koneksi Neo4j secara lokal.
6. Jalankan notebook sesuai urutan sel.

## Evaluasi

Evaluasi sistem rekomendasi menggunakan Hit@5 dan NDCG@5. Hasil KNN dibandingkan dengan baseline popularitas dan dibahas bersama keterbatasan metode pada notebook.

## Keamanan

Password, token, dan informasi autentikasi tidak disertakan dalam repository.
