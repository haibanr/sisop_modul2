# sisop_modul2
A
Mengambil data dari API Jikan terkait judul manhwa, status, tanggal rilis, genre, tema, dan author. Data disimpan ke file teks sesuai format nama judul yang sudah disesuaikan, lalu diletakkan di folder Manhwa.

Jawab
```bash
void soal_a() {
    bikin_folder("Manhwa");

    for (int i = 0; i < 4; i++) {
        char nama_file[100], url[150], path_json[150], path_txt[150];
        ke_format_file(daftar_manhwa[i].judul, nama_file);

        snprintf(url, sizeof(url), "https://api.jikan.moe/v4/manga/%s", daftar_manhwa[i].id);
        snprintf(path_json, sizeof(path_json), "Manhwa/%s.json", nama_file);
        ambil_data_api(url, path_json);

        snprintf(path_txt, sizeof(path_txt), "Manhwa/%s.txt", nama_file);
        parsing_info(path_json, path_txt);

        remove(path_json);
    }
}
```

Output
```bash
Manhwa/
├── mistaken_as_the_monster_dukes_wife.txt
├── the_villainess_lives_again.txt
├── no_i_only_charmed_the_princess.txt
└── darling_why_cant_we_divorce.txt
```

Penjelasan
1. Membuat folder bernama Manhwa
   ```bash
    void bikin_folder(const char *nama) {
      pid_t pid = fork();
      if (pid == 0) {
          execlp("mkdir", "mkdir", "-p", nama, NULL);
          exit(1);
    } else wait(NULL);
}
   ```
  - Fungsi ini memanggil `fork()` lalu menjalankan perintah `mkdir -p` Manhwa lewat `execvp`.
  - Tujuannya agar folder Manhwa tersedia untuk menyimpan file hasil ekstraksi API.
2. Melakukan loop ke semua data manhwa
  ```bash
  for (int i = 0; i < 4; i++) { ... }
  ```
  - Melakukan iterasi pada 4 judul manhwa yang ada di array  `daftar_manhwa`.
3. Format nama file dari judul
  ```bash
        void ke_format_file(char *asal, char *hasil) {
    int j = 0;
    for (int i = 0; asal[i]; i++) {
        if (isalnum(asal[i])) hasil[j++] = tolower(asal[i]);
        else if (asal[i] == ' ') hasil[j++] = '_';
        // karakter lain di-skip aja biar aman
    }
    hasil[j] = '\0';
}
  ```
  - Mengubah judul menjadi lowercase dan mengganti spasi menjadi underscore.
  - Karakter non-alfanumerik selain spasi dihilangkan.
4. Menyusun URL untuk ambil data JSON dari API Jikan
  ```bash
  snprintf(url, sizeof(url), "https://api.jikan.moe/v4/manga/%s", daftar_manhwa[i].id);
  ```
  - Menggabungkan base URL API dengan ID manga dari array.
5. Download JSON dari API menggunakan curl (via fork + execlp)
  ```bash
    void ambil_data_api(url, path json) {
      pid_t pid = fork();
      if (pid == 0) {
          execlp("curl", "curl", "-s", "-o", output, url, NULL);
          exit(1);
      } else wait(NULL);
  }
  ```
  - Mendownload file JSON hasil dari URL API dan menyimpannya ke folder Manhwa.
  6. Parsing JSON dan simpan informasi penting ke .txt
    ```bash
    void parsing_info(path json, path txt) {
    char cmd[256];
    FILE *hasil = fopen(txt_file, "w");

    snprintf(cmd, sizeof(cmd), "grep 'title\\\"\\|status\\\"\\|published\\\"\\|genres\\\"\\|themes\\\"\\|authors' %s > sementara.txt", json_file);
    system(cmd);

    FILE *tmp = fopen("sementara.txt", "r");
    char baris[256];
    while (fgets(baris, sizeof(baris), tmp)) {
        fputs(baris, hasil);
    }

    fclose(tmp);
    fclose(hasil);
    remove("sementara.txt");
}
    ```
  - Memanggil perintah grep untuk mengambil field penting seperti: title, status, published, genres, themes, authors
  - Output disimpan ke file .txt di folder Manhwa.
  7. Hapus file JSON setelah diambil datanya
    ```bash
    remove(path_json);
    ```
    - File .json tidak diperlukan setelah parsing, jadi dihapus agar folder bersih.

B
Setelah berhasil mengumpulkan data manhwa ke file .txt, sekarang gabungkan seluruh file tersebut ke dalam satu file .csv bernama DataStreamer.csv. Format CSV harus terdiri dari Judul,Status,Rilis,Genre,Tema,Author.

Jawab
```bash
void soal_b() {
    void soal_b() {
    bikin_folder("Archive");

    for (int i = 0; i < 4; i++) {
        char nama_file[100], path_txt[150], path_zip[150];
        ke_format_file(daftar_manhwa[i].judul, nama_file);

        for (int j = 0; nama_file[j]; j++) nama_file[j] = toupper(nama_file[j]);

        snprintf(path_txt, sizeof(path_txt), "Manhwa/%s.txt", nama_file);
        snprintf(path_zip, sizeof(path_zip), "Archive/%s.zip", nama_file);

        pid_t pid = fork();
        if (pid == 0) {
            execlp("zip", "zip", "-j", path_zip, path_txt, NULL);
            exit(1);
        } else wait(NULL);
    }
}
```
Output
```
Archive/
├── MISTAKENASTHEMONSTERDUKESWIFE.zip
├── THEVILLAINESSLIVESAGAIN.zip
├── NOIONLYCHARMEDTHEPRINCESS.zip
└── DARLINGWHYCANTWEDIVORCE.zip
```
Penjelasan
1. Membuat Folder Archive
  - Fungsi `bikin_folder()` membuat folder bernama Archive menggunakan `fork()` dan `execlp("mkdir")`.
2. Iterasi Setiap Manhwa
   ```bash
  for (int i = 0; i < 4; i++) {
    ...
}
  ```
- Perulangan dilakukan untuk memproses 4 manhwa yang ada di array `daftar_manhwa`.
3. Mengubah Judul Menjadi Format File
  ```bash
  ke_format_file(daftar_manhwa[i].judul, nama_file);
  ```
  - Fungsi ini menghapus karakter khusus dan mengganti spasi dengan underscore.
4. Mengubah Semua Huruf ke Kapital
```bash
for (int j = 0; nama_file[j]; j++) nama_file[j] = toupper(nama_file[j]);
```
- `nama_file` diubah jadi huruf kapital semua untuk digunakan sebagai nama .zip
5. Menyusun Path File TXT dan ZIP
  ```bash
  snprintf(path_txt, sizeof(path_txt), "Manhwa/%s.txt", nama_file);
  snprintf(path_zip, sizeof(path_zip), "Archive/%s.zip", nama_file);
  ```
  - path_txt menunjukkan lokasi file teks yang sudah dibuat di soal a.
  - path_zip adalah nama file zip hasil kompresi nanti.
6. Melakukan Kompresi File
  ```bash
  pid_t pid = fork();
if (pid == 0) {
    execlp("zip", "zip", "-j", path_zip, path_txt, NULL);
    exit(1);
} else wait(NULL);
  ```
  - Menggunakan `fork()` untuk membuat proses anak.
  - Proses anak mengeksekusi zip dengan argumen:
    a. `-j`: hanya menyimpan file, tanpa menyimpan path direktori.
    b. `path_zip`: nama file zip tujuan.
    c. `path_txt`: file yang ingin dikompres.
    d. Proses induk menunggu dengan `wait(NULL)`.

C
Tiba-tiba Minji lupa, di album mana lagu yang paling banyak di-streaming tersebut berada. Carikan Minji nama album dari lagu yang paling banyak di-streaming di platform tersebut, beserta tahun rilisnya!
Rule: wajib menggunakan AlbumDetails.csv dan output tidak menggunakan tanda petik " ".

Jawab
```bash
void *ambil_gambar(void *arg) {
    Manhwa *m = (Manhwa *)arg;

    char folder[100];
    snprintf(folder, sizeof(folder), "Heroines/%s", m->heroine);
    bikin_folder(folder);

    for (int i = 1; i <= m->bulan_rilis; i++) {
        char url[200], path[200];
        snprintf(url, sizeof(url), "https://placehold.co/600x400?text=%s_%d", m->heroine, i);
        snprintf(path, sizeof(path), "%s/%s_%d.jpg", folder, m->heroine, i);

        char cmd[300];
        snprintf(cmd, sizeof(cmd), "wget -q \"%s\" -O \"%s\"", url, path);
        system(cmd);
    }

    return NULL;
}

void soal_c() {
    bikin_folder("Heroines");

    for (int i = 0; i < 4; i++) {
        pthread_t tid;
        pthread_create(&tid, NULL, ambil_gambar, (void *)&daftar_manhwa[i]);
        pthread_join(tid, NULL); 
    }
}
```
Output
```bash
Heroines/
├── Alisha/
│   ├── Alisha_1.jpg
│   └── Alisha_2.jpg
├── Dorothea/
│   ├── Dorothea_1.jpg
│   ├── Dorothea_2.jpg
│   ├── Dorothea_3.jpg
│   ├── Dorothea_4.jpg
│   ├── Dorothea_5.jpg
│   └── Dorothea_6.jpg
├── Leslie/
│   ├── Leslie_1.jpg
│   ├── Leslie_2.jpg
│   ├── Leslie_3.jpg
│   ├── Leslie_4.jpg
│   ├── Leslie_5.jpg
│   ├── Leslie_6.jpg
│   ├── Leslie_7.jpg
│   ├── Leslie_8.jpg
│   ├── Leslie_9.jpg
│   ├── Leslie_10.jpg
│   ├── Leslie_11.jpg
│   └── Leslie_12.jpg
└── Riselia/
    ├── Riselia_1.jpg
    ├── Riselia_2.jpg
    ├── Riselia_3.jpg
    ├── Riselia_4.jpg
    ├── Riselia_5.jpg
    ├── Riselia_6.jpg
    ├── Riselia_7.jpg
    ├── Riselia_8.jpg
    ├── Riselia_9.jpg
    └── Riselia_10.jpg
```
Penjelasan
1. Membuat Folder 'Heroines': `bikin_folder("Heroines");`
2. Iterasi Melalui Daftar Manhwa:
   ```bash
    for (int i = 0; i < 4; i++) {
      pthread_t tid;
      pthread_create(&tid, NULL, ambil_gambar, (void *)&daftar_manhwa[i]);
      pthread_join(tid, NULL); 
  }
   ```
3.  Membuat Subfolder Heroine:
```bash
snprintf(folder, sizeof(folder), "Heroines/%s", m->heroine);
bikin_folder(folder);
```
4. Mengunduh gambar
   ```bash
   for (int i = 1; i <= m->bulan_rilis; i++) {
    char url[200], path[200];
    snprintf(url, sizeof(url), "https://placehold.co/600x400?text=%s_%d", m->heroine, i);
    snprintf(path, sizeof(path), "%s/%s_%d.jpg", folder, m->heroine, i);

    char cmd[300];
    snprintf(cmd, sizeof(cmd), "wget -q \"%s\" -O \"%s\"", url, path);
    system(cmd);
}
  ```
  - Untuk setiap bulan hingga bulan_rilis dari manhwa, gambar diunduh dari URL yang dibentuk menggunakan placehold.co dengan teks nama   heroine dan nomor urut gambar. Gambar disimpan dalam subfolder heroine dengan nama Heroine_n.jpg.

D
Setelah semua gambar heroine berhasil diunduh, Cella ingin mengarsipkannya:
- Setiap folder heroine di-zip dengan format:
  [HURUFKAPITALNAMAMANHWA]_[namaheroine].zip
- Disimpan di folder Archive/Images
- Setelah zip selesai, gambar pada masing masing folder Heroine akan dihapus secara *urut dengan abjad*.

Jawab
```bash
void hapus_file_folder(const char *path) {
    DIR *dir = opendir(path);
    struct dirent *ent;
    char full_path[200];

    if (dir) {
        while ((ent = readdir(dir)) != NULL) {
            if (ent->d_type == DT_REG) {
                snprintf(full_path, sizeof(full_path), "%s/%s", path, ent->d_name);
                remove(full_path);
            }
        }
        closedir(dir);
    }
}

void soal_d() {
    bikin_folder("Archive/Images");

    for (int i = 0; i < 4; i++) {
        char folder[100], zipname[150], nama_file[100];
        ke_format_file(daftar_manhwa[i].judul, nama_file);

        for (int j = 0; nama_file[j]; j++) nama_file[j] = toupper(nama_file[j]);

        snprintf(folder, sizeof(folder), "Heroines/%s", daftar_manhwa[i].heroine);
        snprintf(zipname, sizeof(zipname), "Archive/Images/%s_%s.zip", nama_file, daftar_manhwa[i].heroine);

        pid_t pid = fork();
        if (pid == 0) {
            execlp("zip", "zip", "-j", zipname, "-r", folder, NULL);
            exit(1);
        } else wait(NULL);

        hapus_file_folder(folder);
    }
}
```
Output
```bash
Archive/
└── Images/
    ├── MISTAKENASTHEMONSTERDUKESWIFE_Alisha.zip
    ├── THEVILLAINESSLIVESAGAIN_Dorothea.zip
    └── ...
```
Penjelasan
1. `soal_d()` membuat folder Archive/Images menggunakan `bikin_folder()`.
2. Loop sebanyak jumlah manhwa (4x), lalu untuk setiap manhwa:
  - Nama folder heroine diambil sebagai Heroines/NamaHeroine.
  - Nama zip disusun dengan format [HURUFKAPITALJUDUL]_[nama_heroine].zip.
  - File-file di dalam folder heroine dijadikan satu zip menggunakan zip -j.
  - Proses zip dijalankan via `fork()` dan `execlp()`.
3. Setelah zip selesai, isi folder heroine dikosongkan menggunakan `fungsi hapus_file_folder()`:
  - Menggunakan `opendir()` dan `readdir()` untuk melihat isi folder.
  - Menghapus file reguler (DT_REG) satu per satu menggunakan `remove()`


