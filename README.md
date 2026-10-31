# Analisis dan Penanganan Temuan EDA pada Dataset DVRP Kilby

Repositori/notebook ini berisi proses *Exploratory Data Analysis* (EDA) dan pemeriksaan penanganan temuan pada dataset **Kilby Dynamic Vehicle Routing Problem (DVRP)**. Fokus analisis adalah memahami karakteristik spasial, permintaan pelanggan (*demand*), waktu ketersediaan pelanggan (`Time_Avail`), kualitas data, serta implikasinya terhadap pemrosesan VRP statis dan DVRP berbasis waktu.

## Daftar Notebook

| Notebook | Keterangan |
|---|---|
| [`EDA_Final.ipynb`](EDA_Final.ipynb) | Eksplorasi dataset, pemeriksaan kualitas data, statistik deskriptif, visualisasi distribusi dan hubungan variabel, serta deteksi outlier/anomali. |
| [`Penanganan_Temuan_EDA.ipynb`](Penanganan_Temuan_EDA.ipynb) | Pemeriksaan koordinat ganda, pengujian fungsi jarak aman, uji normalitas, deteksi outlier demand, analisis perbedaan perlakuan waktu, perbandingan skala instance, serta analisis komparatif/sensitivitas. |

## Dataset

Dataset yang digunakan adalah **Kilby instances for Dynamic Vehicle Routing Problem** dari Mendeley Data:

- Sumber: <https://data.mendeley.com/datasets/3fwc3twwn6/4>
- Format file: `.dat`
- Contoh instance yang digunakan: `c50D.dat`, `c100D.dat`, dan `c199D.dat`.
- Informasi yang dibaca dari file meliputi koordinat lokasi, demand, durasi (jika tersedia), dan waktu ketersediaan pelanggan.

Notebook menggunakan parser untuk membaca bagian-bagian file `.dat`, antara lain:

- `LOCATION_COORD_SECTION`: ID dan koordinat X-Y setiap node.
- `DEMAND_SECTION`: kebutuhan/muatan setiap node.
- `DURATION_SECTION`: durasi layanan, apabila tersedia dan diproses.
- `TIME_AVAIL_SECTION`: waktu ketika pelanggan/order tersedia untuk dilayani.

**Catatan:** Depot menggunakan ID `0`. Dalam beberapa DataFrame analisis, depot dipisahkan dari tabel pelanggan sehingga jumlah baris pelanggan tidak mencakup depot.

## Lingkungan dan Dependensi

Notebook dijalankan melalui Google Colab dan menggunakan Python dengan pustaka berikut:

- `pandas`
- `numpy`
- `matplotlib`
- `seaborn`
- `scipy`
- `google.colab.drive`

Sebagian besar pustaka tersedia di Google Colab. Jika menjalankan di lingkungan lokal, instal dependensi dengan:

```bash
pip install pandas numpy matplotlib seaborn scipy
```

## Cara Menjalankan

1. Unduh dataset Kilby dari tautan Mendeley Data.
2. Ekstrak file dataset sehingga file `.dat` dapat diakses.
3. Buka notebook di Google Colab.
4. Unggah notebook ke Google Colab atau buka file `.ipynb` dari Google Drive.
5. Sesuaikan variabel `file_path` atau `base_path` pada notebook dengan lokasi penyimpanan dataset.
6. Jika notebook menggunakan Google Drive, jalankan autentikasi saat diminta.
7. Jalankan seluruh sel secara berurutan (*Runtime → Run all*).

Contoh struktur direktori Google Drive yang digunakan pada notebook penanganan:

```text
MyDrive/
└── DATASET TUGAS/
    └── DVRP Kilbys instances for Dynamic Vehicle Routing Problem/
        ├── c50D.dat
        ├── c100D.dat
        └── c199D.dat
```

Jika nama folder atau lokasi file berbeda, ubah `base_path` pada notebook. Notebook `Penanganan_Temuan_EDA.ipynb` menggunakan path Google Drive tersebut pada beberapa bagian analisis komparatif.

---

## 1. Tahapan EDA (`EDA_Final.ipynb`)

### 1.1 Deskripsi Dataset

Notebook memuat dan mem-parsing file instance, kemudian membentuk DataFrame yang memuat informasi node. Kolom yang dibentuk/diturunkan dalam proses analisis mencakup:

| Kolom | Keterangan |
|---|---|
| `ID` | Identitas node/pelanggan; ID `0` digunakan sebagai depot. |
| `Type` | Jenis node, yaitu `Depot` atau `Customer`. |
| `X`, `Y` | Koordinat lokasi node. |
| `Demand` | Besarnya permintaan pelanggan. |
| `Duration` | Durasi layanan jika tersedia pada file dan parser. |
| `Time_Avail` | Waktu ketersediaan/kemunculan order. |
| `Distance_To_Depot` | Jarak Euclidean pelanggan ke depot yang dihitung dari koordinat. |

Ringkasan yang tercatat pada output notebook untuk instance yang diberi label `c199D.dat`:

- Total node: 101 (1 depot dan 100 pelanggan).
- Koordinat depot (ID 0): X = 35 dan Y = 35.
- Total demand pelanggan: 1.458 unit.
- Rata-rata demand: 14,58 unit.
- Rentang demand: 1–41 unit.
- Rentang `Time_Avail`: 2–374.
- Rata-rata jarak pelanggan ke depot: 24,95 satuan jarak.
- Pelanggan terjauh yang tercatat: ID 65, dengan jarak 49,93 satuan.

Angka di atas adalah ringkasan output notebook yang diunggah. Pastikan instance yang dipakai ketika menjalankan ulang notebook sama, karena hasil dapat berbeda antar-instance.

### 1.2 Pemeriksaan Struktur dan Kualitas Data

Pemeriksaan meliputi:

- Ukuran DataFrame (jumlah baris dan kolom).
- Tipe data setiap kolom.
- Jumlah nilai kosong (*missing values*).
- Duplikasi seluruh baris.
- Duplikasi ID.
- Duplikasi koordinat X-Y.
- Pemeriksaan nilai demand, waktu ketersediaan, dan jarak terhadap kondisi yang dianggap tidak valid oleh kode.

### 1.3 Statistik Deskriptif

Analisis statistik dilakukan pada variabel utama:

- `Demand`
- `Time_Avail`
- `Distance_To_Depot`

Statistik yang dihitung meliputi mean, standar deviasi, median, kuartil pertama (Q1), kuartil ketiga (Q3), IQR, minimum, maksimum, dan skewness. Notebook juga menjalankan uji normalitas Shapiro-Wilk.

### 1.4 Visualisasi Distribusi dan Hubungan Variabel

Visualisasi digunakan untuk melihat:

- Sebaran spasial pelanggan berdasarkan koordinat X dan Y.
- Posisi depot terhadap pelanggan.
- Distribusi waktu ketersediaan pelanggan.
- Distribusi demand.
- Hubungan antara variabel utama.

Pada peta sebar, warna merepresentasikan `Time_Avail` dan ukuran titik merepresentasikan `Demand`.

### 1.5 Deteksi Outlier dan Anomali

Notebook membandingkan deteksi outlier dengan metode:

- **IQR (Interquartile Range):** nilai di luar batas Q1 − 1,5 × IQR dan Q3 + 1,5 × IQR.
- **Z-score:** observasi dengan nilai absolut Z-score lebih dari 3.

Analisis difokuskan pada `Demand`, `Time_Avail`, dan `Distance_To_Depot`.

---

## 2. Penanganan Temuan EDA (`Penanganan_Temuan_EDA.ipynb`)

Notebook kedua menguji beberapa temuan dan implikasinya terhadap pengolahan data DVRP.

### 2.1 Pemeriksaan Koordinat Ganda

Notebook memeriksa apakah terdapat node dengan koordinat X-Y yang sama. Pemeriksaan ini penting karena dua node yang berimpit dapat menghasilkan jarak Euclidean nol.

Pada output instance `c100D.dat` yang disertakan, tidak ditemukan koordinat ganda.

### 2.2 Pengujian Jarak Euclidean Aman

Notebook membandingkan rumus jarak Euclidean biasa dengan fungsi `safe_euclidean_distance()`.

Jika jarak dua titik bernilai nol, fungsi tersebut mengembalikan nilai epsilon (`1e-4`) agar jarak tidak digunakan sebagai pembagi bernilai nol pada perhitungan lanjutan.

```python
def safe_euclidean_distance(x1, y1, x2, y2, epsilon=1e-4):
    dist = np.sqrt((x1 - x2)**2 + (y1 - y2)**2)
    if dist == 0.0:
        return epsilon
    return dist
```

Pada instance yang diuji, tidak ditemukannya koordinat ganda berarti kasus jarak nol tidak muncul pada data tersebut. Fungsi pengaman tetap disediakan untuk mengantisipasi instance lain atau data dengan titik berimpit.

### 2.3 Uji Normalitas dan Pemilihan Statistik

Uji Shapiro-Wilk dilakukan pada `Demand`, `Time_Avail`, dan `Distance_To_Depot`. Output notebook `c100D.dat` menunjukkan:

| Variabel | p-value Shapiro-Wilk | Status pada output | Ringkasan yang digunakan |
|---|---:|---|---|
| `Demand` | 0,00182 | Tidak normal | Median dan IQR |
| `Time_Avail` | 0,00072 | Tidak normal | Median dan IQR |
| `Distance_To_Depot` | 0,79684 | Normal | Mean/standar deviasi dapat dipertimbangkan; notebook juga menampilkan median dan IQR |

Aturan interpretasi yang digunakan adalah p-value < 0,05 menunjukkan bukti untuk menolak asumsi normalitas. Hasil uji ini digunakan untuk menentukan ringkasan statistik yang sesuai, bukan untuk mengubah nilai data.

### 2.4 Deteksi Outlier pada Demand

Pada output `c100D.dat`, metode IQR menghasilkan:

- Q1 = 8,00
- Q3 = 19,25
- IQR = 11,25
- Batas bawah = −8,875
- Batas atas = 36,125
- Outlier yang terdeteksi: pelanggan ID 85 dengan `Demand` = 41.

Outlier tersebut ditandai untuk dianalisis. Notebook juga membuat perbandingan sensitivitas dengan dan tanpa baris outlier. Penghapusan outlier pada analisis sensitivitas **bukan berarti pelanggan tersebut otomatis dihapus dari skenario operasional**; keputusan mempertahankan atau mengeluarkannya perlu mengikuti tujuan eksperimen dan batasan masalah VRP.

### 2.5 Perbandingan Perlakuan Waktu: VRP Statis dan DVRP

Notebook memvisualisasikan dua skenario:

| Aspek | Tanpa penanganan (VRP statis) | Dengan penanganan waktu (DVRP/event-driven) |
|---|---|---|
| `Time_Avail` | Diabaikan; seluruh node diasumsikan tersedia pada waktu awal (`t = 0`). | Nilai `Time_Avail` digunakan untuk menentukan kapan pelanggan/order tersedia. |
| Pemrosesan order | Seluruh pelanggan diperlakukan tersedia serentak. | Pelanggan diproses sesuai waktu ketersediaannya. |
| Respons terhadap order baru | Tidak merepresentasikan kedatangan order secara dinamis. | Kedatangan order dapat menjadi pemicu proses optimasi ulang dalam rancangan event-driven. |

Visualisasi ini merupakan ilustrasi perlakuan data dan waktu. Implementasi *background job scheduler* dan pengukuran kinerja scheduler secara nyata perlu dilakukan pada tahap implementasi/eksperimen sistem; notebook ini belum dengan sendirinya membuktikan bahwa scheduler produksi telah berjalan.

### 2.6 Perbandingan Antar-Skala Instance

Notebook membandingkan instance `c50D`, `c100D`, dan `c199D`. Output yang tersimpan:

| Instance | Jumlah pelanggan | Jumlah waktu unik (`Time_Avail`) | Median demand | IQR demand | Korelasi Spearman Demand-Time |
|---|---:|---:|---:|---:|---:|
| `c50D` | 50 | 46 | 15,00 | 11,25 | −0,1123 |
| `c100D` | 100 | 87 | 13,00 | 11,25 | 0,0317 |
| `c199D` | 199 | 156 | 15,00 | 12,00 | 0,2156 |

Hasil tersebut menunjukkan bahwa jumlah pelanggan dan variasi waktu ketersediaan berbeda antar-instance. Korelasi Spearman yang ditampilkan menggambarkan hubungan monotonik antara demand dan waktu ketersediaan pada masing-masing instance; korelasi tidak dengan sendirinya menunjukkan hubungan sebab-akibat.

### 2.7 Uji Wilcoxon dan Analisis Sensitivitas

Notebook membuat dua array metrik berbasis demand dengan faktor pengali acak:

- `baseline_metric`: demand dikalikan faktor acak 1,3–1,6.
- `proposed_metric`: demand dikalikan faktor acak 1,0–1,25.

Dengan seed 42, output yang tersimpan adalah:

- Statistik Wilcoxon (W) = 0,0000.
- p-value = 3,89656 × 10⁻¹⁸.

**Batasan interpretasi penting:** angka tersebut berasal dari metrik simulasi/sintetis yang dibentuk dengan faktor pengali acak di dalam notebook, bukan dari pengukuran langsung jarak, response time, CPU, atau memori dua implementasi algoritma/scheduler. Karena itu, hasil ini hanya menunjukkan perbedaan pada array sintetis yang diuji dan belum dapat dijadikan bukti empiris bahwa *Background Job Scheduler* meningkatkan efisiensi sistem nyata.

Analisis sensitivitas outlier pada output yang tersimpan menghasilkan:

| Skenario | Jumlah node | Estimasi beban rute |
|---|---:|---:|
| Dengan outlier | 100 | 1.632,96 |
| Tanpa outlier | 99 | 1.487,85 |

Estimasi tersebut dihitung menggunakan faktor pengali 1,12 dan 1,05 pada jumlah demand sesuai kode. Nilainya merupakan estimasi ilustratif, bukan total jarak rute hasil algoritma VRP.

---

## 3. Ringkasan Temuan

Berdasarkan output notebook yang diunggah:

1. Data memuat atribut spasial, demand, dan waktu ketersediaan yang relevan untuk analisis DVRP.
2. Pemeriksaan instance `c100D.dat` tidak menemukan koordinat ganda.
3. Pada `c100D.dat`, `Demand` dan `Time_Avail` tidak berdistribusi normal menurut uji Shapiro-Wilk pada taraf 0,05, sedangkan `Distance_To_Depot` tercatat normal.
4. Metode IQR mendeteksi satu outlier demand pada ID 85 dengan nilai 41.
5. `Time_Avail` membedakan skenario pemrosesan seluruh pelanggan secara serentak dari skenario pemrosesan berdasarkan waktu ketersediaan.
6. Jumlah pelanggan dan variasi waktu unik berbeda pada instance `c50D`, `c100D`, dan `c199D`.
7. Uji Wilcoxon dan estimasi beban pada notebook menggunakan nilai sintetis/faktor asumsi. Keduanya perlu dibedakan dari evaluasi kinerja algoritma atau scheduler yang diukur secara langsung.

## 4. Batasan dan Hal yang Perlu Diperhatikan

- Beberapa bagian notebook menggunakan path Google Drive yang spesifik. Path perlu disesuaikan dengan lokasi dataset masing-masing pengguna.
- Output yang tercantum di README merupakan output yang tersimpan pada file notebook saat diunggah. Hasil dapat berubah jika instance atau data yang dijalankan berbeda.
- Nama `c199D.dat` pada ringkasan EDA menghasilkan 101 node (1 depot + 100 pelanggan) pada output notebook. Sementara notebook penanganan secara eksplisit memuat `c100D.dat` dengan 100 pelanggan. Periksa kembali nama file dan instance saat mereproduksi hasil agar pelaporan tidak tertukar.
- Fungsi jarak aman mengubah jarak nol menjadi epsilon. Ini mencegah pembagian dengan nol, tetapi tidak mengubah fakta bahwa dua node memiliki koordinat yang sama.
- Outlier demand tidak otomatis merupakan kesalahan data. Dalam VRP, demand besar dapat menjadi karakteristik pelanggan yang valid dan harus dipertimbangkan terhadap kapasitas kendaraan.
- Uji Wilcoxon pada notebook saat ini menggunakan metrik sintetis, bukan hasil pengukuran eksperimen sistem.
- Evaluasi kinerja DVRP yang kuat sebaiknya mengukur langsung total jarak, response time, penggunaan CPU, dan memori pada kondisi eksperimen yang setara untuk pendekatan statis dan dinamis.

## 5. Rencana Pengembangan Eksperimen

Untuk melanjutkan analisis menuju evaluasi algoritma DVRP, eksperimen dapat dikembangkan dengan:

1. Mempertahankan `Time_Avail` sebagai atribut waktu kemunculan/ketersediaan order.
2. Menentukan aturan simulasi waktu dan kapan order baru dimasukkan ke sistem.
3. Menjalankan baseline VRP statis dan pendekatan DVRP pada instance, kendaraan, kapasitas, serta parameter yang sebanding.
4. Memicu optimasi ulang ketika event kedatangan pelanggan terjadi.
5. Mencatat total jarak tempuh, response time (ms), penggunaan CPU, dan memori dari eksekusi aktual.
6. Mengulang eksperimen dengan seed dan konfigurasi yang terdokumentasi agar hasil dapat direproduksi.

## Referensi

- Kilby, P. et al. *Kilby instances for Dynamic Vehicle Routing Problem*. Mendeley Data, Version 4. <https://data.mendeley.com/datasets/3fwc3twwn6/4>
- Notebook EDA: `EDA_Final.ipynb`.
- Notebook penanganan: `Penanganan_Temuan_EDA.ipynb`.
