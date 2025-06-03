# **Laporan Proyek Machine Learning Terapan - Fakhira Lahen Siregar**

## Project Overview

Pada era digital ini, konsumen memiliki akses ke ribuan film dan serial TV melalui berbagai platform streaming. Tantangan yang muncul adalah bagaimana membantu pengguna menemukan konten yang sesuai dengan preferensi mereka di antara banyaknya pilihan yang tersedia. Oleh karena itu, dibutuhkan sebuah sistem rekomendasi film yang dapat memberikan saran kepada para konsumen berdasarkan preferensi pengguna atau karakteristik film (Content Based Filtering).

MovieLens 100K berisikan dataset sistem rekomendasi yang paling banyak digunakan untuk penelitian dan pembelajaran. Dataset ini berisi data 100.000 rating dari 943 pengguna terhadap 1682 film. Dengan membangun sistem rekomendasi menggunakan pendekatan content-based filtering, proyek ini bertujuan untuk menyarankan film yang serupa dengan film yang pernah disukai pengguna berdasarkan informasi genre.

Referensi:
1. R. Vyshnavi, "Genre Based Movie Recommendation System Using Collaborative Filtering," International Journal of Research Publication and Reviews, vol. 4, no. 12, pp. 4029-4032, Dec. 2023. [Online]. Available: https://ijrpr.com/uploads/V4ISSUE12/IJRPR20642.pdf
   - Penelitian yang dilakukan oleh R. Vyshnavi (2023) mengembangkan sistem rekomendasi film berbasis genre dengan pendekatan collaborative filtering. Dalam penelitian tersebut, data MovieLens digunakan sebagai sumber utama, dan fitur genre dimanfaatkan bersama dengan teknik clustering menggunakan algoritma K-Means untuk mengelompokkan pengguna berdasarkan kesamaan preferensi. Setelah itu, rekomendasi diberikan berdasarkan pengguna lain yang berada dalam klaster yang sama. Evaluasi dilakukan menggunakan metrik precision, recall, dan F1-score. Hasilnya menunjukkan bahwa penambahan fitur genre dapat meningkatkan kualitas rekomendasi karena memberikan hasil yang lebih personal dan relevan.

2. S.-M. Choi, S.-K. Ko, Y.-S. Han, S.-W. Kim, & H.-J. Kim, "A movie recommendation algorithm based on genre correlations," Expert Systems with Applications, vol. 39, no. 9, pp. 8079-8085, Jul. 2012. doi: 10.1016/j.eswa.2012.01.186
   - Penelitian oleh S.-M. Choi dan rekan-rekannya (2012) memfokuskan pada algoritma rekomendasi film dengan memanfaatkan korelasi antar genre. Metode ini bertujuan untuk mengatasi masalah cold-start dan sparsity yang umum terjadi pada sistem collaborative filtering. Dengan mengandalkan korelasi antara genre yang stabil dan tidak berubah-ubah seperti halnya preferensi pengguna, sistem dapat menyarankan film yang tidak hanya memiliki genre identik, tetapi juga genre yang berkaitan erat. Hasil penelitian menunjukkan bahwa pendekatan ini dapat menghasilkan rekomendasi yang lebih akurat dan menyeluruh.

## Business Understanding

### Problem Statements
- Bagaimana membangun sistem rekomendasi film yang dapat menyarankan film berdasarkan genre?
- Bagaimana memanfaatkan data genre untuk menentukan kesamaan antar film?

### Goals
- Mengembangkan sistem rekomendasi berdasarkan genre film.
- Memberikan rekomendasi film serupa berdasarkan input film yang disukai pengguna.

### Solution Statements
- Pendekatan Content-Based Filtering: merekomendasikan film berdasarkan kemiripan fitur konten (genre).
- Pendekatan alternatif (tidak digunakan): Collaborative Filtering yang memanfaatkan kesamaan pola rating antar pengguna.

## Data Understanding

Dataset yang digunakan adalah MovieLens 100K dari GroupLens Research: https://grouplens.org/datasets/movielens/100k/

Dataset ini terdiri dari beberapa file, berikut informasi tentang beberapa file yang digunakan:

- **File u.data (Rating data)**:  
  Setiap baris merepresentasikan satu rating dari satu user terhadap satu film. Total: 100.000 baris.  
  Kolom: user_id, item_id, rating, timestamp.  
  Contoh: `196	242	3	881250949`

- **File u.item (Movie metadata)**:  
  Setiap baris memuat informasi 1 film dengan total 1.682 baris.  
  Termasuk: item_id, title, tanggal rilis, dan kode biner genre.  
  Contoh: `1|Toy Story (1995)|01-Jan-1995|...|0|0|0|1|1|1|...`

- **File u.user (User metadata)**:  
  Informasi dari 943 user yang memberikan rating.  
  Kolom: user_id, age, gender, occupation, zip code.  
  Contoh: `1|24|M|technician|85711`

- **File u.genre**:  
  Daftar nama genre dan ID-nya.  
  Contoh: `unknown|0`

### Penggabungan DataFrame
```
df = pd.merge(ratings, movies, on="item_id")
```
- **ratings**: 100.000 baris, dari `u.data`
- **movies**: 1.682 baris, dari `u.item`
- **df**: hasil gabungan ratings dan movies berdasarkan `item_id` → 100.000 baris dan 23 kolom

### Insight:
- Dataset berisi 100.000 baris dan 23 kolom
- Awaln
- Terdapat 21 kolom numerik (user_id, item_id, rating, dan genre_labels), 1 kolom kategorikal (title), dan 1 kolom datetime (timestamp)
- Nilai agregasi normal dan tidak diidentifikasi data abnormal
- Pemberian rata-rata kolom rating ada di 3.5
- Diidentifikasi ada pebedaan banyak nilai unik pada title dan item_id
- Setelah dicek lebih dalam, terdapat baris duplikassi antara title dan item_id. Tapi tidak saya hapus karena tidak akan mengganggu system rekomendasi (akan ada kemungkinan akan ada film yang 2 kali direkomendasikan).
- Setelah dilihat distribusi rating, rating 4 adalah rating yang paling banyak diberikan user untuk keseluruhan film.
- Film yang paling banyak dirating adalah :
 1. 'Star Wars (1977)',
 2. 'Contact (1997)',
 3. 'Fargo (1996)',
 4. 'Return of the Jedi (1983)',
 5. 'Liar Liar (1997)'
- Film yang paling banyak diberi rating bukan berarti adalah film yang paling tinggi rating nya. Karena Film kedua terbanyak dirating adalah Contact tahun 1997 hanya memiliki rating 3.8
- Visualisasi ditribusi genre :
   ![Distribusi Genre](/genre.png)
   Dari visualisasi diatas :
   - Grafik menunjukan distribusi rating dengan mengurutkan film dengan genre terbanyak terbanyak hingga tersedikit.
   - Film dengan genre terbanyak adalah genre Drama dengan distribusi diatas 700 dari 934 film.
   - Film dengan genre tersedikit adalah genre Western, Film-Noir, dan Fantasy yaitu dibawah 50 film.
-  Visualisasi heatmap untuk melihat hubungan antar variabel:
   ![Heatmap](/heatmap.png)
Setelah dilihat korelasi dengan heatmap. Hubungan korelasi antar genre terlihat seperti beberapa film ada yang action, dan adventure sekaligus. Atau film anak-anak biasanya film animation. Dan film Animation biasanya juga bergenre musical.

## Data Preparation

Langkah-langkah data preparation yang dilakukan adalah sebagai berikut:

1. **Membaca file data**  
   File `u.data` dan `u.item` dibaca ke dalam dua dataframe: `ratings` dan `movies`.

2. **Penghapusan kolom dalam dataframe movies**  
   Dari dataframe `movies`, kolom-kolom seperti `release_date`, `video_release_date`, `IMDb_URL`, serta genre `unknown` dihapus karena tidak diperlukan dalam proses rekomendasi berbasis genre.

3. **Penyesuaian nama kolom**  
   Kolom `movie_id` pada dataframe `movies` diubah menjadi `item_id` agar sama dengan nama kolom di dataframe `ratings`.

4. **Penggabungan dataframe**  
   Dataframe `ratings` dan `movies` digabung berdasarkan kolom `item_id` untuk membentuk dataframe utama `df` yang akan digunakan dalam seluruh proses berikutnya.

5. **Konversi kolom waktu**  
   Dalam tahap Assesing Data kolom `timestamp` dalam dataframe `df` dikonversi ke format datetime menggunakan `pd.to_datetime()` agar kolom timestamp mendapat tipe data yang sesuai.

6. **Pemeriksaan nilai kosong dan duplikasi**  
   Dataset diperiksa untuk memastikan tidak ada missing values atau duplikasi dengan kode berikut:
   ```python
   df.isnull().sum()
   df.duplicated().sum()
   ```

8. **Menyusun fitur genre**:  
   - Membuat variabel genre_cols yang menyimpan daftar nama kolom genre hasil one-hot encoding.<br>
   ```python
   genre_cols = ['Action', 'Adventure', 'Animation', "Children's", 'Comedy', 'Crime', 'Documentary', 'Drama', 'Fantasy', 'Film-Noir', 'Horror', 'Musical', 'Mystery', 'Romance', 'Sci-Fi', 'Thriller', 'War', 'Western']
   ```

9. **Pembangunan item profile**  
   Dibentuk dataframe `movie_features` yang terdiri dari `title` dan kolom-kolom genre. Proses ini juga melibatkan penghapusan baris duplikat berdasarkan `item_id`, dan pengaturan `title` sebagai indeks:
   ```python
   movie_features = df[['title'] + genre_cols].drop_duplicates().set_index('title')
   ```

## Modeling

Model sistem rekomendasi dibuat menggunakan pendekatan Content-Based Filtering. Penggunaan content-based filtering digunakan dengan melihat kemiripan antar item berdasarkan metadata seperti genre. Kelebihannya adalah bisa digunakan bahkan jika pengguna belum memberikan banyak rating (cold-start). Kekurangannya adalah model tidak mempelajari preferensi unik setiap pengguna. Sebaliknya, collaborative filtering lebih mampu menangkap pola selera pengguna berdasarkan rating, tapi akan gagal saat pengguna baru belum pernah menilai apapun.

### Langkah Modeling dengan Content-Based Filtering:
1. **Menghitung Cosine Similarity**  
   Untuk mengukur kemiripan antar film berdasarkan vektor genre, hasilnya disimpan di `cosine_sim_df`.
   
   **Formula Cosine Similarity:**
   
   cosine_similarity(A, B) = A • B / (||A|| * ||B||)

   Nilai berkisar dari 0 (tidak mirip) sampai 1 (identik). Cocok untuk data biner genre.

2. **Fungsi Rekomendasi `recommend_movies`**  
   ```
   def recommend_movies(title, similarity_df, movie_features, n=5):
      sim_scores = similarity_df[title].sort_values(ascending=False)[1:n+1]
      return sim_scores.index.tolist()
   ```
   Fungsi ini menerima nama film sebagai input dan mengembalikan 5 film dengan skor kemiripan tertinggi (kecuali dirinya sendiri) berdasarkan nilai cosine similarity antar fitur genre.


### Contoh Output:
```
Rekomendasi film mirip 'Toy Story (1995)':
1. Aladdin and the King of Thieves (1996)
2. Goofy Movie, A (1995)
3. Aladdin (1992)
4. Heavyweights (1994)
5. Beavis and Butt-head Do America (1996)
```

## Evaluation

Model rekomendasi ini menggunakan pendekatan Content-Based Filtering, yang berarti rekomendasi dihasilkan berdasarkan kemiripan konten (genre) antar film, bukan interaksi pengguna. Oleh karena itu, evaluasi dilakukan secara **kualitatif**, yaitu dengan:

1. **Cetak film acak**: memilih 5 film dari dataset sebagai opsi
2. **Input film referensi**
3. **Menampilkan hasil rekomendasi**
4. **Membuat dataframe `movie_genres_df`** berisi title dan genre_list (gabungan genre)
5. **Verifikasi genre**: melihat genre dari film input dan hasil rekomendasi

Contoh:
Jika pengguna memilih film "Toy Story (1995)", maka sistem merekomendasikan film seperti "Aladdin (1992)", "A Goofy Movie (1995)", dan "Heavyweights (1994)" — semuanya termasuk dalam genre **Animation**, **Children’s**, dan **Comedy**, yang sesuai.

### Alasan Tidak Menggunakan Evaluasi Kuantitatif

Evaluasi kuantitatif seperti **Precision@K**, **Recall@K**, atau **Mean Reciprocal Rank (MRR)** memerlukan **ground truth** — yaitu data yang menunjukkan film mana yang benar-benar relevan untuk masing-masing pengguna. Pada sistem Content-Based Filtering dengan input berupa satu judul film, tidak tersedia ground truth eksplisit, sehingga metrik tersebut tidak bisa dihitung.

Jika dataset memiliki riwayat interaksi pengguna lengkap, saya dapat mengevaluasi dengan menghitung seberapa sering sistem merekomendasikan film yang pernah diberi rating tinggi oleh pengguna tersebut. Namun karena proyek ini menggunakan pendekatan item-item berbasis konten dan bukan riwayat pengguna, maka evaluasi kualitatif berdasarkan genre dianggap **tepat dan cukup representatif**.


## Kesimpulan

Sistem rekomendasi berhasil dibangun dan diuji. Rekomendasi diberikan berdasarkan kemiripan genre menggunakan method Cosine Similarity, dan hasil evaluasi menunjukkan sistem mampu memberikan saran film yang relevan. Pendekatan ini cocok untuk kasus cold-start karena tidak bergantung pada interaksi pengguna.

## Referensi
- MovieLens Dataset: https://grouplens.org/datasets/movielens/
- Scikit-learn Documentation
- Vyshnavi (2023), Choi et al. (2012)

<br>

**---Ini adalah bagian akhir laporan---**

_Catatan:_
- _Anda dapat menambahkan gambar, kode, atau tabel ke dalam laporan jika diperlukan. Temukan caranya pada contoh dokumen markdown di situs editor [Dillinger](https://dillinger.io/), [Github Guides: Mastering markdown](https://guides.github.com/features/mastering-markdown/), atau sumber lain di internet. Semangat!_
- Jika terdapat penjelasan yang harus menyertakan code snippet, tuliskan dengan sewajarnya. Tidak perlu menuliskan keseluruhan kode project, cukup bagian yang ingin dijelaskan saja.
