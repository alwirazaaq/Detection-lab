## Laporan Pengujian SQL Injection pada DVWA

**Tujuan:**
Melakukan pengujian kerentanan *SQL Injection* pada aplikasi DVWA pada berbagai level keamanan (Low, Medium, High, Impossible) untuk memahami teknik eksploitasi dan mekanisme mitigasi yang efektif.

### 1. Deskripsi Singkat

SQL Injection adalah teknik serangan di mana penyerang memodifikasi atau menyisipkan fragmen SQL melalui input pengguna untuk mengakses atau memanipulasi data yang seharusnya terlindungi. Pengujian ini memperlihatkan bagaimana perubahan query dapat mengekspos data dan bagaimana prepared statements menutup celah tersebut.

### 2. Tahapan Pengujian & Observasi

| Level Keamanan |                                                Mekanisme yang Diuji | Observasi / Hasil                                                                                                                     |
| -------------- | ------------------------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Low**        |                     Input diproses langsung ke query tanpa validasi | Mudah dieksploitasi — attacker dapat memodifikasi query untuk menampilkan data sensitif (mis. `' OR '1'='1` → menampilkan semua user) |
| **Medium**     | Input diubah menjadi `select`/`option` atau ada filtering sederhana | Beberapa payload dasar diblokir, namun masih ada teknik bypass yang memungkinkan manipulasi query                                     |
| **High**       |  Query diberi pengetatan (mis. `LIMIT 1`) atau validasi lebih ketat | Mengurangi dampak eksfiltrasi data, tetapi tidak sepenuhnya menutup vektor injeksi jika input masih dirangkai langsung ke query       |
| **Impossible** |         Menggunakan **prepared statements / parameterized queries** | Sangat efektif — sintaks SQL dipisahkan dari data input sehingga injeksi praktis tidak mungkin dilakukan dalam skenario pengujian ini |

### 3. Contoh Teknik Pengujian

* Payload klasik yang sering dicoba pada level **Low**:

```sql
' OR '1'='1
```

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection" src="https://github.com/user-attachments/assets/b380721b-ffbf-47bc-a62b-4e94ecb94a93" />



* Contoh perubahan query oleh penyerang untuk menampilkan seluruh tabel user:

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1';

SELECT first_name, last_name FROM users WHERE user_id = '1' or '1'='1';

SELECT first_name, last_name FROM users WHERE user_id = 'a' union select user,password from users #';

'a' union select user,password from users #

```

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection2" src="https://github.com/user-attachments/assets/5da74a48-e5d1-4b91-a32c-6100f94e657d" />


<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection3" src="https://github.com/user-attachments/assets/8c8d1441-567d-49d4-88d1-9381e7414519" />



### 4. Teknik Mitigasi yang Ditemukan

* Gunakan **prepared statements / parameterized queries** untuk memisahkan query dan data.
* Validasi dan sanitasi input di sisi server (whitelisting lebih disarankan daripada blacklisting).
* Batasi hak akses DB (principle of least privilege).
* Jangan menampilkan pesan error database secara rinci ke klien.
* Terapkan pengecekan tipe dan panjang input (mis. integer-only ID, `maxlength`).

### 5. Kesimpulan

* SQL Injection dapat dengan mudah dieksploitasi pada aplikasi yang memproses input pengguna secara langsung dalam query.
* Pengetatan query (LIMIT, validasi) membantu, tetapi **prepared statements** adalah cara yang paling andal untuk mencegah injeksi.
* Pengujian ini menegaskan pentingnya kombinasi validasi input, least-privilege, dan parameterized queries untuk keamanan aplikasi.



------------------------


## Laporan Pengujian Blind SQL Injection pada DVWA

**Tujuan:**
Melakukan pengujian *Blind SQL Injection* pada aplikasi DVWA pada beberapa level keamanan untuk memahami teknik eksploitasi dan mekanisme mitigasi yang diterapkan.

### 1. Deskripsi Singkat

Blind SQL Injection terjadi ketika aplikasi rentan terhadap injeksi SQL tetapi tidak menampilkan hasil query secara langsung — penyerang mengekstrak data secara bertahap melalui respons berbeda (mis. true/false, timing). Pengujian ini menekankan pengujian parameter yang disisipkan melalui cookie dan penggunaan alat otomatis (SQLMap).

### 2. Tahapan Pengujian & Observasi

| Level Keamanan |                                                                                   Mekanisme yang Diuji | Observasi / Hasil                                                                                                                                                                                          |
| -------------- | -----------------------------------------------------------------------------------------------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **High**       |                       Parameter `id` dimasukkan lewat **cookie**; query masih menggunakan string biasa | Pengujian otomatis pakai **SQLMap** (scope MySQL, ignore 404, opsi lain). Proses lama (~30 menit+) namun menunjukkan pola serupa dengan pengujian sebelumnya → masih terdeteksi celah pada konfigurasi ini |
| **Impossible** | Backend memakai **prepared statement** + pengecekan `user token` yang di-**regenerate** setiap request | Prepared statement menutup vektor injeksi; token yang berubah-ubah menghambat tool otomatisasi seperti SQLMap → level ini aman terhadap blind SQLi dalam skenario pengujian ini                            |

**Catatan Teknikal:**

* Pengujian otomatis difokuskan ke MySQL dan menggunakan opsi untuk mengabaikan error 404 untuk mempercepat/menyaring hasil.
* Pengujian pada level *high* memakan waktu signifikan saat menggunakan SQLMap; hasil akhirnya tetap menunjukkan kerentanan bila backend belum memakai prepared statement.
* Pada level *impossible*, kombinasi prepared statements + per-request token (CSRF-like regenerasi) membuat eksploitasi praktis sulit atau tidak mungkin.

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection blind" src="https://github.com/user-attachments/assets/241573c6-79d3-4987-a2d4-c3d4dbbe582a" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection blind2" src="https://github.com/user-attachments/assets/91718c04-bf4b-4598-b42b-608ec1da9561" />



### 3. Teknik Mitigasi yang Ditemukan

* Gunakan **prepared statements / parameterized queries** untuk mencegah injeksi SQL.
* Terapkan validasi dan sanitasi input di sisi server.
* Gunakan mekanisme token yang berubah-ubah (mis. token sesi atau nonce per-request) untuk menghambat otomatisasi serangan.
* Batasi informasi error yang dikembalikan ke klien (hindari leak pesan DB).

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection blind3" src="https://github.com/user-attachments/assets/a428251f-11b7-41fb-9aff-14e951681380" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_sql injection blind4" src="https://github.com/user-attachments/assets/c2585b3f-46cb-4fc2-a44a-5d57295a33ff" />

### 4. Kesimpulan

* Blind SQLi berbahaya karena eksfiltrasi data dapat dilakukan tanpa menampilkan hasil langsung.
* Prepared statements dan mekanisme token per-request terbukti efektif untuk menutup celah pada DVWA di level *impossible*.
* Pengujian otomatis (mis. SQLMap) berguna namun memerlukan waktu dan dapat terhambat oleh mitigasi seperti token dinamis.




------------------------------------



## Laporan Pengujian Kerentanan Reflected XSS pada DVWA

**Tujuan:**
Melakukan pengujian kerentanan *Reflected Cross-Site Scripting (XSS)* pada aplikasi web DVWA dengan berbagai tingkat keamanan (Low, Medium, High, Impossible) untuk memahami bagaimana input pengguna dapat dimanipulasi dan bagaimana mitigasi diterapkan.

### 1. Deskripsi Singkat

Reflected XSS terjadi ketika aplikasi web menampilkan kembali input pengguna tanpa proses penyaringan (filtering) yang tepat. Kondisi ini memungkinkan penyerang menyisipkan *payload* berbahaya, seperti kode JavaScript, sehingga dapat dijalankan di browser korban.

### 2. Tahapan Pengujian

| Level Keamanan | Karakteristik                                                  | Hasil Pengujian                                                                |
| -------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Low**        | Input ditampilkan langsung tanpa filter                        | *Payload* berhasil dieksekusi → rentan terhadap XSS                            |
| **Medium**     | Terdapat *string replace* sederhana untuk menghapus `<script>` | Masih dapat dilewati dengan *payload* alternatif (misalnya event handler HTML) |
| **High**       | Input difilter menggunakan `htmlspecialchars()`                | Script tidak dapat dieksekusi, output ditampilkan sebagai teks biasa           |
| **Impossible** | Validasi dan sanitasi optimal diterapkan                       | Tidak ditemukan celah XSS                                                      |

### 3. Contoh Metode Pengujian

Pengujian dilakukan dengan:

* Memasukkan teks dan *payload* JavaScript pada parameter URL atau form input.
* Mengamati apakah input dipantulkan kembali tanpa sanitasi.
* Membandingkan hasil pada setiap level keamanan.


<img width="954" height="897" alt="VirtualBox_Kali Linux_xss reflected" src="https://github.com/user-attachments/assets/5770d93f-6a3c-4657-9cb3-33d2af8ed791" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss reflected2" src="https://github.com/user-attachments/assets/bc4a0f03-211a-49b5-97d1-73c3ef1ee6ae" />


Contoh payload dasar:

```html
<script>alert('XSS')</script>
```

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss reflected3" src="https://github.com/user-attachments/assets/c3f09ce0-e68c-4454-9dff-1798c778acad" />


### 4. Kesimpulan

* Aplikasi dengan penyaringan input yang lemah sangat rentan terhadap serangan XSS.
* Penerapan fungsi sanitasi seperti `htmlspecialchars()` sangat efektif dalam mencegah XSS.
* Pengujian keamanan perlu dilakukan secara berkala selama pengembangan aplikasi.

---------------------------------------------------------------

## Laporan Pengujian Kerentanan Stored XSS pada DVWA

**Tujuan:**
Melakukan pengujian kerentanan *Stored Cross-Site Scripting (XSS)* pada aplikasi web DVWA di berbagai level keamanan untuk memahami bagaimana data berbahaya dapat disimpan dan dieksekusi secara persisten di server.

### 1. Deskripsi Singkat

Stored XSS adalah jenis serangan di mana penyerang menyisipkan kode berbahaya ke dalam input yang **disimpan di server**, misalnya melalui kolom komentar, buku tamu, atau form input lain. Serangan ini bersifat **persisten**, karena script akan dieksekusi setiap kali halaman dimuat oleh pengguna lain.

### 2. Tahapan Pengujian

| Level Keamanan | Mekanisme Filtering                           | Hasil Pengujian                                                                                |
| -------------- | --------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Low**        | Tidak ada penyaringan input                   | *Payload* berhasil disimpan dan menampilkan pop-up setiap halaman dimuat → sangat rentan       |
| **Medium**     | Menghapus tag HTML menggunakan `strip_tags()` | Beberapa *payload* masih dapat melalui filter menggunakan teknik *event handler* atau encoding |
| **High**       | Filtering pola kata kunci yang mirip *script* | Sebagian besar payload diblokir, namun masih dapat diuji terhadap bypass berbasis encoding     |
| **Impossible** | Validasi dan sanitasi optimal diterapkan      | Serangan tidak dapat dilakukan, input aman sebelum disimpan ke database                        |

### 3. Contoh Payload Pengujian

Payload dasar yang digunakan:

```html
<script>alert('Stored XSS')</script>
```

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss stored" src="https://github.com/user-attachments/assets/447b9652-181b-46a8-8785-b610d728f1ab" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss stored2" src="https://github.com/user-attachments/assets/3d6b743d-7d28-45ca-9bf1-c418c88314c2" />



Hasil pada level **Low**:

* Popup tetap muncul bahkan setelah berpindah halaman → menunjukkan sifat **persisten** dari serangan.

### 4. Teknik Mitigasi yang Ditemukan

* Penggunaan `strip_tags()` dapat mengurangi risiko, namun tidak sepenuhnya aman.
* Membatasi panjang input (`maxlength`) membantu mengontrol data yang masuk.
* `htmlspecialchars()` merupakan metode yang efektif untuk mengonversi karakter khusus menjadi entitas HTML → script tidak akan dieksekusi dan hanya tampil sebagai teks.

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss stored3" src="https://github.com/user-attachments/assets/ad2659ab-8c90-445c-b1c8-fe89d7cfb6dd" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss sroted4" src="https://github.com/user-attachments/assets/9cf2bff4-859f-4e1a-913e-bed5ebb89d17" />



### 5. Kesimpulan

* Stored XSS lebih berbahaya dibanding Reflected XSS karena script **disimpan di server** dan dieksekusi ke banyak pengguna.
* Sanitasi input wajib dilakukan **sebelum** data disimpan ke dalam database.
* Implementasi fungsi *escaping* seperti `htmlspecialchars()` dan filter berbasis pola dapat membantu mencegah serangan.


-----------------------------------------------------------------

## Laporan Pengujian Kerentanan DOM-Based XSS pada DVWA

**Tujuan:**
Melakukan pengujian kerentanan *DOM-Based Cross-Site Scripting (XSS)* pada aplikasi DVWA untuk memahami bagaimana manipulasi *Document Object Model (DOM)* oleh JavaScript dapat dimanfaatkan untuk menjalankan *payload* berbahaya di sisi klien.

### 1. Deskripsi Singkat

DOM-Based XSS adalah serangan yang terjadi ketika data berbahaya **diproses langsung di browser**, bukan melalui server. Script berbahaya disisipkan melalui parameter URL atau elemen yang dapat dimodifikasi di sisi klien, lalu dieksekusi ketika JavaScript memproses nilai tersebut ke dalam halaman.

Karena serangan terjadi **tanpa interaksi server**, filter server-side *tidak cukup* untuk mencegahnya — perlindungan harus dilakukan di sisi klien (JavaScript).

### 2. Tahapan Pengujian

| Level Keamanan | Mekanisme Client-Side                                                         | Hasil Pengujian                                                                     |
| -------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Low**        | Nilai parameter langsung dimasukkan ke DOM                                    | *Payload* JavaScript berhasil dijalankan → rentan terhadap DOM XSS                  |
| **Medium**     | Nilai parameter diperiksa untuk kata `script`; jika ditemukan → nilai diganti | Memblokir script dasar, namun bypass masih memungkinkan dengan encoding/obfuscation |
| **High**       | Parameter dibatasi hanya pada nilai bahasa yang valid (mis. EN, FR, DE)       | Eksekusi script diblokir karena input tidak digunakan langsung dalam DOM            |
| **Impossible** | Pengolahan parameter dilakukan menggunakan ID dan encoding aman               | Script tidak dapat dieksekusi, celah DOM XSS tertutup                               |

### 3. Contoh Payload Pengujian

Payload dasar dimasukkan melalui parameter URL:

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss dom" src="https://github.com/user-attachments/assets/6458464d-23cc-4017-82c7-0bdcdec5a577" />

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss dom2" src="https://github.com/user-attachments/assets/6c913457-5679-4866-a35b-a8258d8f034b" />



```
http://target/vulnerable.php?default=<script>alert('DOM XSS')</script>
```

Pada level **Low**, halaman langsung memproses dan menampilkan nilai tersebut → popup muncul.

### 4. Teknik Mitigasi yang Diterapkan

* Menghindari penggunaan `innerHTML` untuk menampilkan input pengguna.
* Menggunakan `textContent` atau `innerText` untuk mencegah eksekusi script.
* Validasi dan pembatasan nilai parameter terhadap daftar nilai yang diizinkan.
* Encoding karakter menggunakan `encodeURIComponent()` atau `htmlspecialchars()` sebelum diproses di DOM.

<img width="954" height="897" alt="VirtualBox_Kali Linux_xss dom3" src="https://github.com/user-attachments/assets/456b9825-adc7-4050-b7cf-68d0a8fbfa84" />


<img width="954" height="897" alt="VirtualBox_Kali Linux_xss dom4" src="https://github.com/user-attachments/assets/3a9ed81b-9583-4dd6-9d74-bda5912020c8" />


### 5. Kesimpulan

* DOM XSS berbeda dari Reflected dan Stored XSS karena eksekusinya terjadi **sepenuhnya di sisi klien**.
* Pengamanan harus diterapkan pada JavaScript yang memproses input pengguna, bukan hanya pada backend.
* Pembatasan input dan penggunaan metode rendering yang aman adalah langkah kunci dalam mencegah serangan DOM XSS.




