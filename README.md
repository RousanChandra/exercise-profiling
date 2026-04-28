
Test Plan Screenshoot via GUI
<img width="1919" height="645" alt="image" src="https://github.com/user-attachments/assets/47f73447-1ad4-41aa-a3ca-53178a6db246" />
<img width="1916" height="534" alt="image" src="https://github.com/user-attachments/assets/feb2dd54-7219-4739-a8f3-20ff14b1d608" />
<img width="1919" height="664" alt="image" src="https://github.com/user-attachments/assets/14dca25e-caf1-460b-b976-6809b04e4fb4" />
Test Plan Screenshoot via Command line
<img width="1648" height="618" alt="image" src="https://github.com/user-attachments/assets/f4213fe1-480d-4fa7-803f-71c62463830c" />
<img width="1509" height="693" alt="image" src="https://github.com/user-attachments/assets/187bf411-95b0-452b-8d9e-a7c09a02b1c0" />
<img width="1495" height="624" alt="image" src="https://github.com/user-attachments/assets/5309ba62-5c61-414c-b7b9-b8fec47691d3" />

Before Optimize:
<img width="1535" height="412" alt="image" src="https://github.com/user-attachments/assets/ac6a4292-6db6-4835-86e8-b55bdee1f907" />
<img width="1919" height="1064" alt="Screenshot 2026-04-28 203848" src="https://github.com/user-attachments/assets/ffaa2e31-6fb9-4f71-aca1-64e4b6567050" />
<img width="1919" height="982" alt="image" src="https://github.com/user-attachments/assets/0df630f6-44b1-4fa9-b588-a185e52c10c5" />

After Optimize:

<img width="1485" height="790" alt="image" src="https://github.com/user-attachments/assets/45cf89e8-97f4-4b4f-8e71-1ac680c97893" />
<img width="1904" height="937" alt="Screenshot 2026-04-28 195844" src="https://github.com/user-attachments/assets/131107e0-16ba-40dc-9132-076502d049f8" />
<img width="1919" height="1038" alt="Screenshot 2026-04-28 200711" src="https://github.com/user-attachments/assets/82f25622-404a-4e61-b0d3-6518391e2d22" />

**Conclusion**
Setelah melakukan profiling dan pengujian menggunakan JMeter, saya melakukan beberapa optimasi pada kode sumber untuk meningkatkan performa aplikasi. Berikut adalah perbandingannya:

- Endpoint /all-student:
Sebelumnya, method getAllStudentsWithCourses mengalami masalah N+1 Query, di mana aplikasi melakukan query ke database berulang kali untuk setiap baris data. Saya mengatasinya dengan menggunakan JOIN FETCH pada repository, sehingga data mahasiswa dan kursus dapat diambil sekaligus dalam satu eksekusi query yang efisien.

- Endpoint /all-student-name:
Pada bagian joinStudentNames, sebelumnya aplikasi melakukan penggabungan string menggunakan operator += di dalam looping. Hal ini menyebabkan tingginya penggunaan memori karena terciptanya banyak objek string baru. Saya menggantinya dengan StringBuilder yang secara signifikan mengurangi beban kerja garbage collector dan mempercepat waktu pemrosesan.

- Endpoint /highest-gpa:
Sebelumnya, aplikasi mengambil seluruh data mahasiswa ke memori Java untuk kemudian dicari GPA tertingginya melalui looping manual. Ini tidak efisien terutama jika jumlah data mahasiswa terus bertambah. Saya mengubah logika tersebut dengan mengandalkan perintah SQL (ORDER BY gpa DESC LIMIT 1) di sisi database, sehingga aplikasi hanya menerima satu data mahasiswa yang memang paling relevan.

Kesimpulan:
Berdasarkan hasil pengujian JMeter, optimasi ini berhasil menurunkan durasi respons secara signifikan. Perubahan yang dilakukan tidak hanya sekadar membuat aplikasi berjalan lebih cepat, tetapi juga mengurangi beban kerja database dan penggunaan memori pada JVM. Dengan strategi ini, aplikasi menjadi lebih scalable dan efisien dalam menangani request dari pengguna.


1. Apa perbedaan pendekatan pengujian performa dengan JMeter dan profiling dengan IntelliJ Profiler dalam konteks optimasi performa aplikasi?
JMeter digunakan untuk black-box testing (pengujian dari luar). Fokusnya adalah mensimulasikan beban pengguna (seperti 10 atau 100 user secara bersamaan) untuk melihat bagaimana aplikasi merespons dari sisi throughput dan response time (seberapa cepat aplikasi membalas request).
IntelliJ Profiler digunakan untuk white-box testing (pengujian dari dalam). Fokusnya adalah melihat ke dalam kode sumber, memantau penggunaan CPU, memori, dan durasi eksekusi tiap method. Ini membantu kita menemukan baris kode mana yang spesifik menjadi penyebab aplikasi menjadi lambat.

2. Bagaimana proses profiling membantu Anda dalam mengidentifikasi dan memahami titik lemah (weak points) dalam aplikasi Anda?
Proses profiling membantu memvisualisasikan alur eksekusi aplikasi menggunakan Flame Graph dan Method List. Dengan ini, saya bisa melihat secara langsung method mana yang memiliki waktu eksekusi paling lama (bottleneck). Dalam kasus aplikasi ini, saya jadi tahu bahwa penggunaan looping untuk query database (masalah N+1) adalah penyebab utama lambatnya aplikasi.

3. Apakah menurut Anda IntelliJ Profiler efektif dalam membantu Anda menganalisis dan mengidentifikasi bottleneck dalam kode aplikasi?
Ya, sangat efektif. Tanpa profiler, kita hanya bisa menebak-nebak bagian mana yang lemot. Dengan profiler, data yang disajikan sangat akurat, mulai dari CPU time, execution time, hingga call tree, sehingga perbaikan kode bisa dilakukan dengan presisi tinggi.

4. Apa tantangan utama yang Anda hadapi saat melakukan pengujian performa dan profiling, dan bagaimana Anda mengatasinya?
Tantangan utamanya adalah konsistensi hasil tes. Kadang hasil profiling berbeda-beda karena faktor latar belakang proses OS yang berjalan. Cara mengatasinya adalah dengan memastikan aplikasi di-run beberapa kali (warm-up) agar JIT compiler JVM bekerja optimal, serta menutup aplikasi lain yang tidak diperlukan agar sumber daya CPU lebih fokus ke aplikasi yang dites.

5. Apa manfaat utama yang Anda peroleh dari penggunaan IntelliJ Profiler untuk melakukan profiling kode aplikasi?
Manfaat utamanya adalah kita bisa mengoptimasi kode berdasarkan data nyata (data-driven optimization), bukan asumsi. Kita jadi belajar teknik refactoring yang benar, seperti mengganti N+1 query menjadi Join Fetch dan mengganti string concatenation yang tidak efisien dengan StringBuilder.

6. Bagaimana Anda menangani situasi di mana hasil profiling dengan IntelliJ Profiler tidak sepenuhnya konsisten dengan temuan dari pengujian performa menggunakan JMeter?
Jika hasil tidak konsisten, saya akan menganalisis di mana letak perbedaannya. Profiling fokus pada internal logic kode, sedangkan JMeter juga dipengaruhi faktor jaringan (network latency), koneksi database pool, dan load pada sistem operasi. Saya akan melakukan kalibrasi dengan menjalankan tes beberapa kali dan mengambil rata-ratanya untuk mendapatkan hasil yang representatif.

7. Strategi apa yang Anda terapkan dalam mengoptimasi kode aplikasi setelah menganalisis hasil pengujian performa dan profiling? Bagaimana Anda memastikan perubahan yang Anda buat tidak mempengaruhi fungsionalitas aplikasi?
Strategi yang saya terapkan adalah: 1) Optimasi query database (mengurangi jumlah query ke database), 2) Optimasi manajemen memori (menggunakan StringBuilder), dan 3) Menggunakan sorting di sisi database alih-alih di sisi aplikasi. Untuk memastikan fungsionalitas tetap terjaga, saya melakukan pengecekan data di browser untuk memastikan hasil output tetap sama sebelum dan sesudah optimasi.
