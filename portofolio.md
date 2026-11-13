# Portofolio Praktikum Sistem Operasi

## Identitas

- Nama: Alya Dyah Sari
- NIM: 2510512036
- Kelompok: 6 (Kelas A)
- Judul proyek: Rancangan dan Implementasi Sistem Otomasi Backup Data Linux Berbasis Checksum dan Pembatasan Akses Docker
- Pilihan proyek: 5A Otomasi Backup dan Pemulihan Data
- Peran: Anggota 3, Verifikasi Integritas Data dan Checksum

## Asal Proyek

Proyek ini dikerjakan secara berkelompok. Repository ini adalah *fork* dari repository utama kelompok:
https://github.com/SO-Kelompok6/kelompok6_5A_SistemOperasi

| Nama | NIM | Komponen |
| --- | --- | --- |
| Dimah Alika Cahyani | 2510512014 | Validasi sumber dan parameter |
| Audira Syifa Amalia Rasyid | 2510512011 | [ISI] |
| Lucky Ardillah | 2510512029 | [ISI] |
| Dinda Azzahra | 2510512035 | [ISI] |
| Alya Dyah Sari | 2510512036 | Verifikasi integritas data dan checksum |

Proyek ini bukan karya pribadi sepenuhnya. Bagian yang saya kerjakan dijelaskan di bawah.

## Kontribusi Pribadi

**Skrip**

- `scripts/checksum.sh`: membuat berkas `.sha256` dari arsip (mode `generate`) dan membandingkan hash arsip dengan nilai tersimpan (mode `verify`), dengan exit status 0 (cocok), 1 (tidak cocok), dan 2 (arsip atau berkas checksum tidak ada, atau cara pakai salah).

**Pengujian**

- Skenario Uji 3 (KU-3.1 sampai KU-3.5): uji positif, uji negatif, dan uji hak akses dengan akun `backupop` dan `alya`, beserta bukti log dan screenshot.

**Dokumentasi (laporan)**

- Bagian 3.3, 4.3, 5.3, 7.1, 7.2, dan 7.3 (bagian Anggota 3).

## Bukti Kontribusi

| Pekerjaan | Tautan |
| --- | --- |
| `scripts/checksum.sh` | https://github.com/SO-Kelompok6/kelompok6_5A_SistemOperasi/commit/ef0c49a |
| Bukti pengujian (`evidence/kasus_uji_3.txt`) | https://github.com/SO-Kelompok6/kelompok6_5A_SistemOperasi/commit/ff8bf23 |
| Commit akhir repository pribadi | [ISI: link commit] |

## Skenario Individu: Verifikasi Integritas dan checksum

**Tujuan.** Membuktikan bahwa `checksum.sh` meloloskan arsip asli dan mendeteksi arsip yang diubah dengan pesan jelas dan exit status non-zero.

**Kondisi awal.** Dijalankan sebagai `backupadm` dari `/opt/backup-sistem`. Arsip dummy `dataBackup/tes.tar.gz` dibuat dari `dataSumber/tes.txt`, dan skrip memiliki izin eksekusi. Uji hak akses dijalankan sebagai `backupop` dan `alya`.

| ID | Perintah | Diharapkan | Aktual | Status |
| --- | --- | --- | --- | --- |
| KU-3.1 | `./scripts/checksum.sh verify dataBackup/tes.tar.gz` | exit 0 | exit 0 | Lulus |
| KU-3.2 | `./scripts/checksum.sh verify dataBackup/uji.tar.gz` (salinan diubah) | exit 1 | exit 1 | Lulus |
| KU-3.3 | `verify` arsip asli setelah KU-3.2 | exit 0 | exit 0 | Lulus |
| KU-3.4 | `verify` dijalankan sebagai `backupop` | exit 0 | exit 0 | Lulus |
| KU-3.5 | `ls dataBackup` sebagai `alya` | ditolak | Permission denied | Lulus |

**Verifikasi.** Exit status dibaca dengan `echo $?` tepat setelah skrip. Hasil uji tersimpan di `evidence/kasus_uji_3.txt`.

**Pemulihan.** Menghapus berkas uji dengan `rm -f dataBackup/uji*`. Arsip asli dan berkas `.sha256`-nya tidak diubah.

## Analisis Konsep Sistem Operasi

- **Integritas data.** SHA-256 membaca seluruh isi berkas dan menghasilkan nilai 256 bit sebagai sidik jari. Berkas yang sama selalu menghasilkan nilai yang sama, sedangkan perubahan sekecil apa pun mengubah nilainya secara drastis. Sistem berkas ext4 hanya memberi checksum pada metadata, bukan isi berkas pengguna, sehingga pemeriksaan isi arsip dilakukan di tingkat aplikasi.
- **Perizinan.** Kernel mencocokkan UID dan GID proses dengan pemilik, grup, dan mode pada inode. `backupop` (anggota `backupgrp`) dapat mengakses arsip, sedangkan `alya` (bukan anggota grup) sudah ditolak sejak `cd /opt/backup-sistem`, sebelum sampai ke `dataBackup`.
- **Proses dan exit status.** Skrip memanggil `sha256sum` sebagai proses anak. Hasilnya dikembalikan ke shell dan dibaca lewat `$?`, sehingga skrip lain (misalnya restore) dapat mengambil keputusan tanpa membaca pesan.
- **Proteksi dan integritas.** Perizinan mencegah akses pihak tak berwenang, sedangkan checksum mendeteksi perubahan. Keduanya dipakai bersama.
- **Sinkronisasi.** Mode `verify` hanya membaca sehingga aman dijalankan bersamaan. Mode `generate` menulis berkas `.sha256`, sehingga penulisan bersamaan perlu dilindungi lock pada tingkat backup (dikerjakan Anggota 5).

Keterbatasan yang saya sadari: berkas `.sha256` disimpan di folder yang sama dengan arsip sehingga checksum hanya mendeteksi kerusakan atau perubahan tidak disengaja, bukan membuktikan keaslian; akun dengan `sudo` atau root dapat melewati perizinan; `verify` harus dijalankan dari `/opt/backup-sistem` karena jalur arsip ikut tercatat di berkas `.sha256`; dan skrip baru diuji pada arsip dummy. Perbaikan yang terpikir: menyimpan hash di lokasi terpisah dengan akses tulis terbatas, atau memakai tanda tangan digital agar keaslian arsip dapat dibuktikan.

## Kendala dan Penyelesaian

| Kendala | Penyelesaian |
| --- | --- |
| Akun belum punya hak tulis ke repository kelompok, sehingga unggah berkas ditolak | Meminta pemilik organisasi memberi akses, lalu melakukan commit lewat GitHub |
| Skrip terunggah ke folder `scrips` (salah ketik) dan bukan `scripts` | Menghapus berkas di folder yang salah dan membuatnya ulang di `scripts/` |
| Skrip diketik manual di terminal VM sehingga muncul kesalahan sintaks (nama perintah dan tanda kutip salah) | Memeriksa dengan `bash -n` dan memperbaiki baris yang salah |
| Uji negatif pertama gagal karena nama berkas salah ketik (`uji.tzr.gz`), bukan karena arsip berubah | Mengulang pengujian dengan nama yang benar dan memastikan tidak ada pesan `No such file or directory` |
| Uji hak akses awalnya masih dijalankan sebagai `backupop`, sehingga tidak ditolak | Keluar dengan `exit`, lalu mengulang pengujian sebagai `alya` |

## Refleksi

**Yang sudah saya kuasai.** Saya sudah bisa membuat dan menjalankan skrip Bash yang menghitung dan membandingkan checksum dengan `sha256sum`, membaca exit status lewat `$?`, dan menguji skenario positif dan negatif. Saya juga bisa menjelaskan perbedaan perizinan, yang mencegah akses, dengan checksum, yang mendeteksi perubahan. Selain itu, saya mulai terbiasa memakai GitHub untuk commit dan bekerja di repository kelompok.

**Yang masih perlu saya pelajari.** Penulisan skrip Bash yang teliti (sintaks, tanda kutip, dan jalur berkas), karena beberapa kali terjadi salah ketik. Selain itu, cara menyimpan checksum secara lebih aman (di lokasi terpisah atau dengan tanda tangan digital), dan cara skrip checksum terhubung dengan skrip pengarsipan, restore, dan locking milik anggota lain.

## Sumber dan Atribusi

Proyek dikerjakan berkelompok. Referensi: dokumentasi GNU Coreutils (`sha256sum`), dokumentasi kernel Linux tentang checksum ext4, standar FIPS 180-4, dan buku *Operating System Concepts* (Silberschatz dkk.). Menggunakan bantuan AI Claude untuk membantu menyusun dan memperbaiki skrip, draf laporan, dan portofolio; seluruh pengujian dijalankan dan diperiksa sendiri di mesin virtual.
