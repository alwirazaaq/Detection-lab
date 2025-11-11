XSS (Reflected, Stored, Dom)

## Laporan Pengujian Kerentanan Reflected XSS pada DVWA

Tujuan:
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

Contoh payload dasar:

```html
<script>alert1</script>
```

### 4. Kesimpulan

* Aplikasi dengan penyaringan input yang lemah sangat rentan terhadap serangan XSS.
* Penerapan fungsi sanitasi seperti `htmlspecialchars()` sangat efektif dalam mencegah XSS.
* Pengujian keamanan perlu dilakukan secara berkala selama pengembangan aplikasi.
* Pengetahuan ini digunakan secara **etis**, hanya untuk pembelajaran atau pengujian legal.


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

Hasil pada level **Low**:

* Popup tetap muncul bahkan setelah berpindah halaman → menunjukkan sifat **persisten** dari serangan.

### 4. Teknik Mitigasi yang Ditemukan

* Penggunaan `strip_tags()` dapat mengurangi risiko, namun tidak sepenuhnya aman.
* Membatasi panjang input (`maxlength`) membantu mengontrol data yang masuk.
* `htmlspecialchars()` merupakan metode yang efektif untuk mengonversi karakter khusus menjadi entitas HTML → script tidak akan dieksekusi dan hanya tampil sebagai teks.

### 5. Kesimpulan

* Stored XSS lebih berbahaya dibanding Reflected XSS karena script **disimpan di server** dan dieksekusi ke banyak pengguna.
* Sanitasi input wajib dilakukan **sebelum** data disimpan ke dalam database.
* Implementasi fungsi *escaping* seperti `htmlspecialchars()` dan filter berbasis pola dapat membantu mencegah serangan.
* Semua pengujian dilakukan untuk tujuan pembelajaran dan diterapkan secara **etis** dalam konteks *pentesting* yang legal.



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

```
http://target/vulnerable.php?default=<script>alert('DOM XSS')</script>
```

Pada level **Low**, halaman langsung memproses dan menampilkan nilai tersebut → popup muncul.

### 4. Teknik Mitigasi yang Diterapkan

* Menghindari penggunaan `innerHTML` untuk menampilkan input pengguna.
* Menggunakan `textContent` atau `innerText` untuk mencegah eksekusi script.
* Validasi dan pembatasan nilai parameter terhadap daftar nilai yang diizinkan.
* Encoding karakter menggunakan `encodeURIComponent()` atau `htmlspecialchars()` sebelum diproses di DOM.

### 5. Kesimpulan

* DOM XSS berbeda dari Reflected dan Stored XSS karena eksekusinya terjadi **sepenuhnya di sisi klien**.
* Pengamanan harus diterapkan pada JavaScript yang memproses input pengguna, bukan hanya pada backend.
* Pembatasan input dan penggunaan metode rendering yang aman adalah langkah kunci dalam mencegah serangan DOM XSS.



