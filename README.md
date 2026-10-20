# UTS Graf Analytics: Analisis Graf dan Sistem Rekomendasi dengan Neo4j

## Deskripsi Proyek

Repository ini berisi implementasi tugas Ujian Tengah Semester (UTS) Graf Analytics yang menggunakan **Neo4j dan Python** untuk mengolah, menganalisis, dan membangun sistem rekomendasi berbasis interaksi pengguna (*visitor*) dengan item.

Dataset yang digunakan adalah Retail Rocket, yaitu dataset interaksi pengguna dengan item yang mencakup aktivitas melihat item (*view*), menambahkan item ke keranjang (*addtocart*), dan melakukan transaksi (*transaction*).

Proyek ini mencakup proses pengolahan data, pembentukan representasi matriks User–Item, proyeksi graf Item–Item, perhitungan kemiripan antaritem, serta implementasi dan evaluasi rekomendasi.

## Tujuan

Tujuan pengerjaan proyek ini adalah:

1. Memahami penggunaan Neo4j dan Python Driver untuk mengakses serta mengelola basis data graf.
2. Mengimpor dataset Retail Rocket ke Neo4j dan memverifikasi struktur graf.
3. Melakukan pemilihan periode data (*time fold*) untuk training dan testing.
4. Membentuk matriks User–Item dan Item–User menggunakan representasi matriks sparse.
5. Menganalisis proyeksi graf Item–Item berdasarkan interaksi pengunjung.
6. Menghitung kemiripan antaritem menggunakan cosine similarity.
7. Menerapkan pendekatan K-Nearest Neighbors (KNN) untuk menghasilkan rekomendasi item.
8. Mengevaluasi hasil rekomendasi menggunakan Hit@5 dan Normalized Discounted Cumulative Gain@5 (NDCG@5).

## Teknologi yang Digunakan

- **Python** untuk pengolahan data dan implementasi analisis.
- **Neo4j** sebagai basis data graf.
- **Cypher Query Language** untuk manipulasi dan analisis graf.
- **Neo4j Python Driver** untuk menghubungkan Python dengan Neo4j.
- **pandas** untuk pengolahan data tabular.
- **NumPy** untuk komputasi numerik.
- **SciPy** untuk representasi dan operasi matriks sparse.
- **scikit-learn** untuk implementasi KNN dan perhitungan cosine distance.
- **NetworkX** untuk pembentukan dan visualisasi graf.
- **Matplotlib** untuk visualisasi data dan graf.

Versi library dapat disesuaikan dengan environment yang digunakan. Daftar dependensi aktual tersedia pada `requirements.txt` apabila file tersebut telah disertakan dalam repository.

## Dataset

Dataset yang digunakan adalah Retail Rocket, yang memuat interaksi pengguna terhadap item. 
Windows PowerShell:
curl.exe -L -o ecommerce-dataset.zip `ω→ https://www.kaggle.com/api/v1/datasets/download/retailrocket/ecommerce-dataset

Secara umum, data interaksi mencakup atribut berikut:

| Atribut | Keterangan |
|---|---|
| `visitorid` | Identitas pengunjung |
| `itemid` | Identitas item |
| `event` | Jenis interaksi pengguna |
| `timestamp` | Waktu terjadinya interaksi |

Jenis interaksi yang dianalisis:

- `view`: pengguna melihat item.
- `addtocart`: pengguna menambahkan item ke keranjang.
- `transaction`: pengguna melakukan transaksi.

Dataset mentah tidak disertakan secara langsung dalam repository apabila ukuran file terlalu besar atau terdapat batasan distribusi. Pengguna dapat memperoleh dataset dari sumber resmi atau sumber dataset Retail Rocket yang digunakan pada perkuliahan.

**Sumber dataset:** Retail Rocket recommender system dataset. Tambahkan tautan sumber dataset yang benar-benar digunakan sebelum repository dipublikasikan.

## Tahapan Pengerjaan

### 1. Koneksi dan Pengelolaan Graf Neo4j

Python Driver digunakan untuk menghubungkan Python dengan Neo4j melalui protokol Bolt.

Pada tahap ini dilakukan:

- Pengujian koneksi ke database.
- Pembuatan atau pemeriksaan node dan relationship.
- Penggunaan Cypher untuk membaca data graf.
- Pemeriksaan jumlah node dan relationship.
- Verifikasi data setelah proses impor.

Struktur graf menggunakan label node `Visitor` dan `Item`, serta jenis relationship interaksi berikut:

| Relationship | Makna |
|---|---|
| `VIEWED` | Pengunjung melihat item |
| `ADDED_TO_CART` | Pengunjung menambahkan item ke keranjang |
| `BOUGHT` | Pengunjung melakukan transaksi |
| `SIMILAR_TO` | Relasi kemiripan antaritem |

Relationship `SIMILAR_TO` digunakan untuk menyimpan hasil kemiripan item yang dihitung pada tahap analisis.

### 2. Pemilihan Time Fold

Data dibagi menjadi periode training dan testing berdasarkan timestamp.

| Fold | Periode |
|---|---|
| Training | 19 Mei–19 Juli 2015 |
| Testing | 28 Juli–12 Agustus 2015 |

Periode training digunakan untuk membangun representasi interaksi dan menghasilkan rekomendasi. Periode testing digunakan untuk mengidentifikasi item relevan yang muncul pada interaksi pengguna setelah periode training.

Terdapat jeda antara kedua periode tersebut. Oleh karena itu, pembagian ini mengikuti rentang waktu yang ditetapkan pada eksperimen dan tidak mencakup seluruh tanggal di antara kedua fold.

Batas waktu diterapkan dengan rentang awal inklusif dan batas akhir eksklusif untuk memastikan pemfilteran timestamp konsisten.

### 3. Import dan Verifikasi Data

Data interaksi dimuat ke Neo4j dengan label node dan relationship yang sesuai.

Hasil pemrosesan yang tercatat dalam eksperimen:

| Komponen | Jumlah |
|---|---:|
| Node `Visitor` | 824.772 |
| Node `Item` | 187.153 |
| Relationship interaksi | 1.587.578 |
| Relationship `SIMILAR_TO` tambahan | 5 |

Jumlah relationship interaksi merupakan hasil pencatatan sebelum penambahan lima relationship `SIMILAR_TO`. Jumlah akhir dapat berubah jika database diimpor ulang atau diperbarui.

### 4. Pembentukan Matriks User–Item

Matriks User–Item dibentuk berdasarkan interaksi `VIEWED` pada data training. Representasi biner digunakan untuk menunjukkan apakah pengunjung pernah berinteraksi dengan suatu item pada periode tersebut.

Dimensi matriks:

- Jumlah pengunjung: 661.487.
- Jumlah item: 169.622.
- Jumlah pasangan pengunjung–item unik: 994.219.
- Jumlah event `VIEWED` pada data training: 1.244.923.

Beberapa event dapat melibatkan pasangan pengunjung dan item yang sama. Dalam matriks biner, pasangan yang berulang direpresentasikan sebagai satu nilai nonzero sehingga jumlah pasangan unik lebih kecil daripada jumlah event mentah.

Matriks disimpan dalam format sparse untuk mengurangi penggunaan memori karena sebagian besar kombinasi pengunjung dan item tidak memiliki interaksi.

### 5. Analisis Proyeksi Item–Item

Matriks User–Item digunakan untuk membentuk proyeksi Item–Item. Dua item dihubungkan apabila keduanya memiliki pengunjung yang sama.

Operasi perkalian matriks digunakan untuk menghitung jumlah pengunjung bersama bagi pasangan item. Diagonal matriks dihilangkan agar item tidak dibandingkan dengan dirinya sendiri.

Hasil proyeksi pada eksperimen:

| Metrik | Hasil |
|---|---:|
| Jumlah node item | 169.622 |
| Nonzero berarah | 22.118.370 |
| Pasangan unik tak berarah | 11.059.185 |

Proyeksi Item–Item dipilih karena tujuan analisis adalah menemukan item yang memiliki pola interaksi serupa. Struktur ini juga dapat digunakan untuk membangun kandidat rekomendasi berbasis kemiripan item.

### 6. Perhitungan Cosine Similarity

Cosine similarity digunakan untuk mengukur kemiripan dua item berdasarkan pola pengunjung yang berinteraksi dengan keduanya.

Secara umum, nilai cosine similarity berada pada rentang 0 sampai 1 untuk vektor biner nonnegatif:

- Nilai mendekati 1 menunjukkan pola pengunjung yang semakin mirip.
- Nilai mendekati 0 menunjukkan kemiripan yang semakin rendah.

Untuk item acuan `81996`, lima item dengan nilai similarity tertinggi yang tercatat pada eksperimen adalah:

| Item acuan | Item mirip | Cosine similarity |
|---:|---:|---:|
| 81996 | 15878 | 0,707107 |
| 81996 | 430058 | 0,707107 |
| 81996 | 69307 | 0,707107 |
| 81996 | 406715 | 0,707107 |
| 81996 | 119096 | 0,707107 |

Lima relationship `SIMILAR_TO` disimpan ke Neo4j untuk mendemonstrasikan penyimpanan dan pembacaan hasil similarity melalui Cypher.

Jumlah tersebut merupakan contoh hasil untuk satu item acuan, bukan keseluruhan relationship kemiripan dari semua item.

### 7. Implementasi KNN dan Rekomendasi

KNN digunakan untuk mencari pengunjung yang memiliki pola interaksi serupa dengan pengunjung target. Cosine distance digunakan sebagai ukuran jarak, dengan similarity dihitung sebagai:

`similarity = 1 - cosine_distance`

Item yang sudah dilihat oleh pengunjung target dikeluarkan dari kandidat rekomendasi. Item yang dimiliki oleh tetangga kemudian diberi skor berdasarkan similarity tetangga yang menyumbangkannya.

Pada eksperimen dengan user target `128913`, implementasi KNN tidak menghasilkan kandidat item baru. Oleh karena itu, sistem menggunakan fallback popularitas, yaitu memilih item dengan jumlah interaksi tertinggi pada data training yang belum pernah dilihat user target.

Hasil fallback:

| Peringkat | Item ID | Skor popularitas |
|---:|---:|---:|
| 1 | 5411 | 1.726 |
| 2 | 309778 | 1.281 |
| 3 | 370653 | 1.060 |
| 4 | 461686 | 1.023 |
| 5 | 257040 | 1.015 |

Skor popularitas menunjukkan jumlah interaksi item pada data training, bukan nilai cosine similarity maupun skor KNN.

### 8. Evaluasi Rekomendasi

Evaluasi dilakukan menggunakan item relevan dari data testing untuk user target yang sama.

Metrik yang digunakan:

**Hit@5**

Mengukur apakah minimal satu item relevan muncul pada lima rekomendasi teratas. Nilai 1 menunjukkan terdapat item relevan pada Top-5, sedangkan nilai 0 menunjukkan tidak ada item relevan yang berhasil direkomendasikan.

**NDCG@5**

Mengukur kualitas urutan rekomendasi dengan mempertimbangkan posisi item relevan. Item relevan pada posisi lebih tinggi memberikan kontribusi yang lebih besar.

Hasil evaluasi fallback untuk user target `128913`:

| Komponen | Hasil |
|---|---|
| Item relevan testing | 449130 |
| Item rekomendasi Top-5 | 5411, 309778, 370653, 461686, 257040 |
| Hit@5 | 0 |
| NDCG@5 | 0,0000 |

Item relevan `449130` tidak muncul dalam lima rekomendasi teratas. Oleh karena itu, Hit@5 bernilai 0 dan NDCG@5 juga bernilai 0.

Hasil ini merupakan evaluasi untuk satu user target dan menggunakan fallback popularitas. Hasil tersebut tidak dapat langsung digunakan untuk menyimpulkan kinerja keseluruhan sistem atau kinerja KNN murni. Evaluasi yang lebih menyeluruh memerlukan pengujian terhadap lebih banyak user serta pemeriksaan implementasi KNN agar rekomendasi benar-benar dihasilkan oleh metode yang dimaksud.

## Instalasi dan Persiapan

### Prasyarat

Pastikan perangkat telah memiliki:

- Python dengan versi yang kompatibel dengan library yang digunakan.
- Neo4j Desktop atau Neo4j Server.
- Database Neo4j yang aktif.
- Dataset Retail Rocket yang diperlukan.
- Jupyter Notebook atau JupyterLab.

### 1. Clone Repository

```bash
git clone <URL_REPOSITORY>
cd UTS-Graf-Analytics
```

Ganti `<URL_REPOSITORY>` dengan URL repository GitHub setelah repository dibuat.

### 2. Buat Virtual Environment

Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

Linux atau macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Instal Dependensi

Jika `requirements.txt` tersedia:

```bash
pip install -r requirements.txt
```

Jika file tersebut belum tersedia, instal library yang benar-benar digunakan pada notebook dan script, misalnya:

```bash
pip install neo4j pandas numpy scipy scikit-learn networkx matplotlib jupyter
```

### 4. Konfigurasi Koneksi Neo4j

Pastikan Neo4j aktif dan parameter koneksi sesuai dengan konfigurasi lokal.

Contoh parameter:

```python
URI = "bolt://127.0.0.1:7687"
USER = "neo4j"
```

Password sebaiknya dimasukkan melalui `getpass` atau environment variable, bukan ditulis langsung di notebook.

Contoh penggunaan `getpass`:

```python
from getpass import getpass
from neo4j import GraphDatabase

URI = "bolt://127.0.0.1:7687"
USER = "neo4j"
PASSWORD = getpass("Masukkan password Neo4j: ")

driver = GraphDatabase.driver(
    URI,
    auth=(USER, PASSWORD)
)

driver.verify_connectivity()
print("Koneksi Neo4j berhasil.")
```

Jangan mengunggah password, token, atau informasi autentikasi ke repository publik.

## Urutan Menjalankan Proyek

1. Siapkan dataset Retail Rocket.
2. Aktifkan Neo4j dan konfigurasi koneksi Python Driver.
3. Jalankan notebook utama untuk mengikuti proses analisis dari awal hingga akhir.
4. Lakukan import atau verifikasi data sesuai instruksi pada notebook.
5. Jalankan pembentukan matriks User–Item dan Item–User.
6. Jalankan analisis proyeksi Item–Item.
7. Hitung cosine similarity dan simpan hasil contoh ke Neo4j.
8. Jalankan proses KNN dan pembentukan rekomendasi.
9. Jalankan evaluasi Hit@5 dan NDCG@5.
10. Periksa hasil output serta screenshot yang digunakan sebagai bukti pengerjaan.

**Perhatian:** jalankan proses import hanya jika diperlukan. Periksa terlebih dahulu isi script import karena beberapa implementasi dapat menghapus atau membangun ulang relationship. Hindari menjalankan ulang proses yang menghapus data pada database yang sudah berisi hasil analisis tanpa backup.

## Struktur File Pendukung

| File | Fungsi |
|---|---|
| `UTS_Graf.ipynb` | Notebook utama untuk menjalankan dan mendokumentasikan tahapan analisis |
| `vsm_user_item.py` | Membentuk matriks User–Item |
| `vsm_item_user.py` | Membentuk matriks Item–User |
| `analisis_proyeksi.py` | Menganalisis karakteristik interaksi dan proyeksi |
| `proyeksi_item.py` | Membentuk matriks proyeksi Item–Item |
| `similarity_item.py` | Menghitung cosine similarity antaritem |
| `simpan_similarity_neo4j.py` | Menyimpan contoh relationship similarity ke Neo4j |
| `requirements.txt` | Daftar dependensi Python |
| `screenshots/` | Bukti visual hasil query, graf, similarity, dan evaluasi |
| `report/` | Laporan tertulis apabila disertakan |

Daftar ini perlu disesuaikan dengan file aktual yang tersedia di repository.

## Keterbatasan

Beberapa keterbatasan eksperimen yang perlu diperhatikan:

1. Matriks User–Item bersifat sparse karena tidak semua pengunjung berinteraksi dengan semua item.
2. Interaksi berulang pada pasangan pengunjung–item yang sama diringkas dalam representasi matriks biner.
3. Hasil proyeksi dan similarity bergantung pada periode training dan definisi interaksi yang digunakan.
4. Contoh penyimpanan relationship `SIMILAR_TO` hanya mencakup lima item yang mirip dengan satu item acuan.
5. Implementasi KNN pada user target `128913` tidak menghasilkan kandidat item baru sehingga evaluasi contoh menggunakan fallback popularitas.
6. Hasil evaluasi untuk satu user tidak cukup untuk menggambarkan kinerja sistem rekomendasi secara keseluruhan.
7. Reproduksi hasil memerlukan dataset, konfigurasi Neo4j, library, dan urutan eksekusi yang sesuai.

## Kesimpulan

Proyek ini mendemonstrasikan alur analisis graf menggunakan Neo4j dan Python, mulai dari pengelolaan data interaksi hingga pembentukan proyeksi Item–Item dan perhitungan cosine similarity. Hasil similarity dapat disimpan sebagai relationship pada graf Neo4j dan diakses kembali menggunakan Cypher.

Implementasi KNN dan fallback popularitas juga menunjukkan bagaimana rekomendasi dapat dibentuk dan dievaluasi menggunakan Hit@5 serta NDCG@5. Pada eksperimen user target `128913`, fallback popularitas menghasilkan lima item, tetapi tidak mencakup item relevan dari data testing sehingga kedua metrik evaluasi bernilai nol. Hasil ini menjadi dasar untuk perbaikan implementasi dan pengujian lebih lanjut.

## Author

- **Nama:** Nafila Hanum Al Hasaniy
- **Program Studi:** S1 Sains Data
- **Mata Kuliah:** Graf Analytics
- **Institusi:** UNESA
- **Tahun:** 2026

---

*Repository ini dibuat untuk keperluan akademik dan dokumentasi pengerjaan UTS Graf Analytics.*
