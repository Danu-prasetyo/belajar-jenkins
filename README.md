## **CI/CD & Jenkins**

_Continuous Integration_ (CI) dan _Continuous Delivery/Deployment_ (CD)! Konsep CI/CD adalah praktik dalam pengembangan aplikasi modern yang bertujuan untuk mempercepat pengiriman aplikasi yang berkualitas tinggi dengan ==mengotomatiskan== proses build, testing, dan deployment.

- **Continuous Integration (CI):** Praktik di mana pengembang secara teratur mengintegrasikan perubahan kode mereka ke repositori pusat. Setiap integrasi kemudian diverifikasi oleh build otomatis dan tes otomatis untuk mendeteksi masalah integrasi sedini mungkin.
- **Continuous Delivery (CD):** Sebuah ekstensi dari CI yang memastikan bahwa kode dapat dirilis ke produksi kapan saja setelah melewati semua tahapan build dan testing.
- **Continuous Deployment (CD):** Otomatisasi penuh dari proses pengiriman, di mana setiap perubahan kode yang melewati semua pengujian secara otomatis dideploy ke produksi tanpa intervensi manual.

**Apa itu Jenkins?**

Jenkins adalah automation server ==open-source== yang paling banyak digunakan untuk membuat pipeline CI/CD. Jenkins menyediakan ribuan plugin yang memungkinkan integrasi dengan hampir semua _tools_ dan teknologi dalam ekosistem pengembangan perangkat lunak(software).

**Mengapa Menggunakan Jenkins di Docker?**

Menjalankan Jenkins di Docker memiliki beberapa keuntungan signifikan, terutama di lingkungan pengembangan lokal seperti Windows:

- **Isolasi:** Jenkins berjalan dalam lingkungannya sendiri, terisolasi dari sistem host Kita. Ini mencegah konflik dependensi dengan aplikasi lain di komputer Kita.
- **Portabilitas:** Kita dapat dengan mudah memindahkan instalasi Jenkins (dengan data dan konfigurasinya) antar mesin.
- **Reproducibility:** Memastikan lingkungan Jenkins selalu konsisten dan mudah dibuat ulang.
- **Manajemen Versi:** Mudah untuk mencoba versi Jenkins yang berbeda atau melakukan _rollback_ ke versi sebelumnya.
- **Pembersihan Mudah:** Menghapus instalasi Jenkins semudah menghapus kontainer dan volumenya.

---

**1. Persiapan Lingkungan (Windows)**

Sebelum menjalankan Jenkins, pastikan lingkungan Docker Desktop Kita di Windows sudah siap.

- Verifikasi Docker Desktop & WSL2:
  Pastikan Docker Desktop sudah terinstal dan berjalan. Periksa ikon paus di system tray Kita. Jika solid dan tidak ada peringatan, berarti sudah siap.
  Jenkins akan menggunakan Docker Desktop dan WSL2 yang sudah Kita persiapkan sebelumnya.
- Persiapan Direktori untuk Data Jenkins:
  Sangat penting untuk menyimpan data Jenkins secara persisten. Jika tidak, semua konfigurasi, job, dan log Kita akan hilang setiap kali kontainer Jenkins dihentikan atau dihapus. Kita akan menggunakan Docker Volume untuk ini.
  Buat sebuah direktori di komputer Windows Kita yang akan digunakan untuk menyimpan data Jenkins. Contoh: `D:\DIGIFORM\devops\learn_jenkins\jenkins`.
  ```
  # Di Command Prompt atau PowerShell
  mkdir D:\DIGIFORM\devops\learn_jenkins\jenkins
  ```
  Pastikan direktori ini memiliki izin baca/tulis yang cukup untuk Docker Desktop.

---

**2. Instalasi & Konfigurasi Awal Jenkins di Docker**

Sekarang kita akan menjalankan Jenkins sebagai kontainer Docker.

- **Menarik(Pull) Image Jenkins**:
  Jenkins memiliki image resmi di Docker Hub. Kita akan menarik(pull) versi LTS (Long Term Support) yang direkomendasikan karena stabilitasnya.
  ```
  docker pull jenkins/jenkins:lts
  ```
![[Pasted_image_20250616212413.png]](images/Pasted_image_20250616212413.png)

- **Jalankan Kontainer Jenkins dengan Volume Persisten**:
  Ini adalah langkah krusial. Kita akan me-mount direktori `D:\DIGIFORM\devops\learn_jenkins\jenkins` yang kita buat sebelumnya ke dalam kontainer Jenkins.
  ```
  docker run -d -p 8090:8080 -p 50000:50000 --name jenkins-web -v D:/DIGIFORM/devops/learn_jenkins/jenkins:/var/jenkins_home jenkins/jenkins:lts
  ```
![[Pasted_image_20250616220905.png]](images/Pasted_image_20250616220905.png)  

Mari kita bedah perintah ini:
  - `docker run -d`: Menjalankan kontainer di latar belakang (`detached` mode).
  - `-p 8090:8080`: **Port Forwarding**. Me-_mappingkan_ port `8090` di host Kita ke port `8080` di dalam kontainer agar Jenkins UI berjalan di port ini.
  - `-p 50000:50000`: **Port Forwarding** tambahan. Port `50000` digunakan oleh Jenkins Agent untuk komunikasi. Meskipun mungkin belum diperlukan segera, baik untuk dipersiapkan.
  - `--name my-jenkins`: Memberikan nama `my-jenkins` pada kontainer. Ini memudahkan untuk mengidentifikasi dan mengelolanya.
  - `-v D:/DIGIFORM/devops/learn_jenkins/jenkins:/var/jenkins_home`: **Docker Volume (Bind Mount)**.
    - `D:/DIGIFORM/devops/learn_jenkins/jenkins`: Direktori di host Windows Kita yang kita buat tadi. **Penting: Gunakan _forward slashes_ `/` untuk path Windows dalam perintah Docker.**
    - `/var/jenkins_home`: Direktori default di dalam kontainer Jenkins tempat semua data (konfigurasi, _job_, _plugin_, _log_) disimpan.
  - `jenkins/jenkins:lts`: Image Docker yang akan digunakan.
- **Mengakses Jenkins UI & Proses Unlock:**
  Setelah kontainer berjalan (mungkin perlu waktu beberapa menit saat pertama kali), Kita bisa mengakses Jenkins UI.
  1. **Verifikasi Kontainer Yang Berjalan:**
     ```
     docker ps
     ```
![[Pasted_image_20250616221026.png]](images/Pasted_image_20250616221026.png)

Pastikan `jenkins-web` terdaftar dan statusnya `Up`.

  2. **Akses Jenkins UI:**

     Buka browser web Kita dan navigasikan ke:

     http://localhost:8080

  3. **Proses Unlock Jenkins:**
		![[Pasted_image_20250616221113.png]](images/Pasted_image_20250616221113.png)
     Saat pertama kali diakses, Jenkins akan meminta Kita untuk memasukkan "Administrator password". Password ini dihasilkan secara otomatis di dalam kontainer.

     Untuk mendapatkannya, Kita bisa melihat log kontainer Jenkins:

     ```
     docker logs jenkins-web
     ```

     Cari baris yang mirip dengan:
		```
	     *************************************************************
	     *************************************************************
	     ************************************************************* 
	     Jenkins initial setup is required. An admin user has been created and a password generated. 
	     Please use the following password to proceed to installation:
	      
	     12345678900987654321123456789 <-- Ini adalah password Kita
	      
	     This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
	     *************************************************************
	     *************************************************************
	     *************************************************************
		```
		![[Pasted_image_20250616222122.png]](images/Pasted_image_20250616222122.png)

		![[Pasted_image_20250616222555.png]](images/Pasted_image_20250616222555.png)
		
	 Salin password tersebut (string panjang setelah `Please use the following password to proceed to installation:`) dan tempelkan ke kolom "Administrator password" di UI Jenkins. Klik "Continue".

  4. **Instalasi Plugin Dasar:**

     Jenkins akan menawarkan dua opsi instalasi plugin:
		![[Pasted_image_20250616222654.png]](images/Pasted_image_20250616222654.png)
		
     - **Install suggested plugins:** Ini direkomendasikan untuk pemula karena akan menginstal plugin-plugin dasar yang paling sering digunakan (Git, Pipeline, Maven Integration, dll.).
     - **Select plugins to install:** Memberi Kita kontrol penuh untuk memilih plugin.

     Pilih **"Install suggested plugins"**. Tunggu hingga proses instalasi selesai. Ini mungkin memakan waktu beberapa menit.
		![[Pasted_image_20250616222858.png]](images/Pasted_image_20250616222858.png)
		
  5. **Konfogurasi Awal**:

     Setelah plugin terinstal, Kita akan diminta untuk membuat first admin user.
		![[Pasted_image_20250616223513.png]](images/Pasted_image_20250616223513.png)

		![[Pasted_image_20250616223644.png]](images/Pasted_image_20250616223644.png)
		
     Isi kolom-kolom yang diperlukan (Username, Password, Full name, Email address, dan URL jenkins) dan klik "Save and Finish".

  6. **Jenkins is Ready!**
	![[Pasted_image_20250616223749.png]](images/Pasted_image_20250616223749.png)
     Klik "Start using Jenkins". Kita akan diarahkan ke Dashboard Jenkins.
- Menghentikan dan Memulai Ulang Kontainer Jenkins:
  Untuk menghentikan:
  
  ```
  docker stop my-jenkins
  ```
  Untuk memulai:
  
  ```
  docker start my-jenkins
  ```
  Karena kita menggunakan volume, semua konfigurasi dan data Kita akan tetap ada setelah kontainer dihentikan dan dimulai ulang.

---

**3. Dasar-dasar Jenkins UI & Manajemen**

Mari kita kenali antarmuka Jenkins.
![[Pasted_image_20250616224159.png]](images/Pasted_image_20250616224159.png)

- **Dashboard:** Halaman utama setelah login. Menampilkan daftar _job_ yang ada, status build terakhir, dan navigasi utama.
- **Build History:** Di sidebar kiri, menunjukkan riwayat build semua _job_.
- **Manage Jenkins:** Ini adalah pusat administrasi.
	![[Pasted_image_20250616224615.png]](images/Pasted_image_20250616224615.png)
	
  - **System:** Konfigurasi global Jenkins (URL Jenkins, variabel lingkungan global, dsb.).
  - **Plugins:** Mengelola plugin (instal, hapus, update).
  - **Nodes(Server):** Mengelola _build agents_ (mesin tempat _job_ benar-benar dijalankan).
  - **Credentials:** Mengelola kredensial (username/password, SSH keys, secret text) yang digunakan oleh Jenkins untuk mengakses sumber daya eksternal (misalnya, repositori Git).
  - **Tools:** Mengelola instalasi otomatis alat seperti JDK, Maven, Gradle, dll.
  - **Users:** Mengelola akun pengguna Jenkins.
  - **Security:** Mengatur otentikasi dan otorisasi.
- **New Item:** Untuk membuat _job_ atau _pipeline_ baru.
- **My Views:** Untuk mengelola dan memantau _job_ di Jenkins.

**Konfigurasi Global Tools (JDK & Maven):**

Untuk membangun proyek Java/Maven Kita, Jenkins perlu mengetahui di mana menemukan JDK (Java Development Kit) dan Maven. Kita akan menggunakan fitur "Global Tool Configuration" di Jenkins.

1. Di Dashboard Jenkins, klik **"Manage Jenkins"** di sidebar kiri.
2. Gulir ke bawah dan klik **"Tools"**.
	![[Pasted_image_20250616225046.png]](images/Pasted_image_20250616225046.png)
	
3. **Konfigurasi JDK:**

	![[Pasted_image_20250616230006.png]]
	(images/Pasted_image_20250616230006.png)
   - Cari bagian **"JDK Installations"**.
   - Klik **"Add JDK"**.
   - Centang **"Install automatically"**.
   - Beri nama pada JDK (misalnya, `JDK_17` atau `JDK_21`). Penting: Nama ini akan Kita gunakan di `Jenkinsfile`.
   - Pilih versi OpenJDK yang sesuai (misalnya, `OpenJDK 17` atau `OpenJDK 21`). Pastikan versi ini kompatibel dengan proyek Java Kita.
   - Klik "Save".

4. **Konfigurasi Maven:**

	![[Pasted_image_20250616230121.png]](images/Pasted_image_20250616230121.png)
	
   - Cari bagian **"Maven Installations"**.
   - Klik **"Add Maven"**.
   - Centang **"Install automatically"**.
   - Beri nama pada instalasi Maven (misalnya, `Maven_V3`). Ini juga akan Kita gunakan di `Jenkinsfile`.
   - Pilih versi Maven yang sesuai (misalnya, `Maven 3.9.9`).
   - Klik "Save".

Jenkins sekarang akan secara otomatis mengunduh dan mengelola instalasi JDK dan Maven di dalam kontainer Jenkins saat diperlukan oleh sebuah _job_.

---

**4. Memahami Jenkins Pipeline & Jenkinsfile**

Konsep Pipeline:

Jenkins Pipeline adalah rangkaian plugin yang memungkinkan Kita untuk mengimplementasikan dan mengintegrasikan pipeline continuous delivery ke Jenkins. Pipeline menyediakan serangkaian langkah yang dapat diperluas untuk menjalankan build otomatis, tes, dan deploy aplikasi Kita. Definisi Pipeline ditulis dalam file teks bernama Jenkinsfile, yang di-check-in ke repositori kontrol source code Kita. Ini dikenal sebagai "Pipeline as Code".

**Manfaat "Pipeline as Code":**

- **Auditabilitas:** Perubahan pada Pipeline dapat dilacak melalui version control.
- **Konsistensi:** Pipeline yang sama dapat digunakan untuk semua cabang dan lingkungan.
- **Kolaborasi:** Pengembang dapat berkontribusi pada definisi Pipeline.

**Struktur Dasar `Jenkinsfile` (Declarative Pipeline):**

Ada dua sintaks Pipeline: Declarative dan Scripted. Kita akan fokus pada **Declarative Pipeline** karena lebih mudah dibaca dan dipelajari untuk pemula.

Groovy

```
// Jenkinsfile (Declarative Pipeline)

// pipeline : Blok kode yang mendefinisikan seluruh Pipeline.
// Pipeline ini berisi serangkaian langkah(stages) yang otomatis, seperti build, testing, dan deploy aplikasi, yang dieksekusi secara berurutan atau paralel.

pipeline {
    // 1. agent: Mendefinisikan di mana pipeline akan berjalan
    agent any // Atau agent { docker { image 'maven:3.8.6-openjdk-11-slim' } } untuk spesifik

    // 2. tools: Mendefinisikan alat yang akan digunakan (misalnya, Maven, JDK)
    tools {
        // Nama alat harus sesuai dengan yang dikonfigurasi di "Manage Jenkins" -> "Tools"
        jdk 'JDK_21'
        maven 'Maven_V3'
    }

    // 3. stages: Blok utama yang berisi satu atau lebih stage
    stages {
        // Stage 1: Checkout source codw
        stage('Checkout') {
            steps {
                // Pull kode dari repositori Git
                git branch: 'main', url: 'https://github.com/Danu-prasetyo/simple-java-maven-app.git'
            }
        }

        // Stage 2: Membangun aplikasi (compile & package)
        stage('Build') {
            steps {
                // Menjalankan perintah Maven untuk membersihkan dan menginstal build
                sh 'mvn clean install'
            }
        }

        // Stage 3: Menjalankan unit tes
        stage('Test') {
            steps {
                // Menjalankan perintah Maven untuk menjalankan tes
                sh 'mvn test'
                // Opsi: Menerbitkan laporan tes JUnit
                // junit '**/target/surefire-reports/*.xml'
            }
        }

        // Stage 4: Deploy (simulasi)
        stage('Deploy') {
            steps {
                echo 'Simulating deployment...'
                // Kita bisa menambahkan langkah deployment yang sebenarnya di sini,
                // seperti menyalin JAR ke server lain, membangun Docker image, dll.
                sh 'cp target/*.jar /path/to/deployment/folder || true' // Contoh copy (akan gagal di Docker kontainer, hanya ilustrasi)
                echo 'Deployment simulated successfully!'
            }
        }
    }

    // 4. post: Tindakan yang akan dijalankan setelah pipeline selesai (selalu, berhasil, gagal, dll.)
    post {
        always {
            echo 'Pipeline finished.'
            // cleanWs() // Membersihkan workspace setelah build selesai
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

**Penjelasan Blok dalam `Jenkinsfile`:**

- **`pipeline { ... }`**: Blok terluar yang mendefinisikan seluruh Pipeline.
- **`agent { ... }`**:
  - Menentukan di mana pipeline atau _stage_ akan dijalankan.
  - `agent any`: Akan berjalan di agen mana pun yang tersedia.
  - `agent none`: Pipeline tidak akan secara otomatis mengalokasikan agen; setiap _stage_ harus menentukan agennya sendiri.
  - `agent { docker { image '...' } }`: Akan menjalankan _stage_ dalam kontainer Docker baru yang dibuat dari image tertentu. Ini sangat kuat untuk memastikan lingkungan build yang terisolasi dan konsisten. **(Ini adalah pendekatan yang sangat direkomendasikan untuk produksi, tetapi untuk awal kita akan gunakan `agent any` dan konfigurasi Tools di Jenkins UI).**
- **`tools { ... }`**: Mendefinisikan alat-alat yang dikonfigurasi di "Manage Jenkins" -> "Tools" yang akan tersedia di jalur eksekusi pipeline.
- **`stages { ... }`**: Berisi semua _stage_ dari pipeline. Setiap _stage_ mewakili langkah utama dalam proses CI/CD Kita (misalnya, Checkout, Build, Test, Deploy).
- **`stage('Nama Stage') { ... }`**: Mendefinisikan satu langkah besar dalam pipeline. Setiap _stage_ harus memiliki nama yang unik.
- **`steps { ... }`**: Berisi satu atau lebih perintah atau fungsi Pipeline yang akan dieksekusi dalam _stage_ tersebut.
  - `git branch: 'main', url: '...'`: Mengambil kode sumber dari repositori Git.
  - `sh 'command'`: Menjalankan perintah _shell_ di agen tempat pipeline berjalan.
  - `echo 'message'`: Mencetak pesan ke _console output_ build.
  - `junit 'path/to/reports/*.xml'`: Menganalisis laporan tes JUnit.
- **`post { ... }`**: Bagian ini berisi tindakan yang akan dieksekusi setelah pipeline selesai, tergantung pada status akhir pipeline (berhasil, gagal, selalu, dll.).

---

**5. Membangun Proyek Java/Maven Pertama (CI)**

Sekarang kita akan membuat _job_ Pipeline pertama kita di Jenkins untuk membangun dan menguji repositori GitHub Kita.

1. Siapkan Proyek GitHub User Kita:

   Pastikan repositori https://github.com/Danu-prasetyo/simple-java-maven-app.git Kita sudah publik atau dapat diakses oleh Jenkins.

   - Kloning repositori ini ke komputer lokal Kita untuk membiasakan diri dengan strukturnya:
     
     ```
     git clone https://github.com/Danu-prasetyo/simple-java-maven-app.git
     cd simple-java-maven-app
     ```
   ![[Pasted_image_20250616231256.png]](images/Pasted_image_20250616231256.png)
   
   - Buat file `Jenkinsfile` di **root** repositori ini. Salin konten `Jenkinsfile` yang saya berikan di atas ke file baru bernama `Jenkinsfile` (pastikan huruf besar J dan tanpa ekstensi `.txt`).
   - Commit dan Push `Jenkinsfile` ke repositori Kita:
     Bash
     ```
     git add Jenkinsfile
     git commit -m "Add Jenkinsfile for CI/CD pipeline"
     git push origin main # Atau nama branch default Kita
     ```
![[Pasted image 20250617105040.png]]

2. **Membuat Item Pipeline Baru di Jenkins:**

   3. Di Dashboard Jenkins, klik **"New Item"** di sidebar kiri.
   4. Masukkan nama Item (misalnya, `Simple-Java-Maven-CI`).
   5. Pilih **"Pipeline"** sebagai tipe proyek.
   6. Klik **"OK"**.
	![[Pasted image 20250617105317.png]]
	
   7. Konfigurasi Pipeline:

      Kita akan diarahkan ke halaman konfigurasi job.
	![[Pasted image 20250617105710.png]]
	
      - **General:** (Opsional) Tambahkan deskripsi.
      - **Build Triggers:** Biarkan kosong dulu, kita akan memicu secara manual.
      - **Pipeline:** Ini adalah bagian terpenting.
        - Pilih **"Definition"** sebagai `Pipeline script from SCM`.
        - Pilih **"SCM"** sebagai `Git`.
        - **Repository URL:** Masukkan URL repositori GitHub Kita: `https://github.com/Danu-prasetyo/simple-java-maven-app.git`
        - **Credentials:** Biarkan `<none>` dulu jika repo Kita publik. Jika repo privat, Kita perlu menambahkan kredensial Git (akan dibahas di bagian selanjutnya).
        - **Branches to build:** `*/main` (atau `*/master` tergantung nama branch default Kita).
        - **Script Path:** `Jenkinsfile` (ini adalah nama file kita di root repositori).
        - Klik **"Save"**.

8. **Menjalankan Build Pertama:**

   9. Kita akan kembali ke halaman _job_ `Simple-Java-Maven-CI`.
	![[Pasted image 20250617105747.png]]
	
   10. Di sidebar kiri, klik **"Build Now"**.
	![[Pasted image 20250617105843.png]]
	![[Pasted image 20250617105905.png]]
	
   11. **Menganalisis Hasil Build:**

      - Di bagian "Build History" di sidebar kiri, Kita akan melihat build baru muncul (misalnya, `#1`).
	    ![[Pasted image 20250617110518.png]]
	    
      - Klik pada nomor build tersebut (misalnya, `#1`).
      - Klik **"Console Output"** di sidebar kiri.
		![[Pasted image 20250617110543.png]]
		
      Kita akan melihat output konsol yang merinci semua langkah yang dieksekusi oleh Jenkins sesuai `Jenkinsfile` Kita:

      - Pengambilan kode Git.
      - Output `mvn clean install` (kompilasi, unduh dependensi, _package_ JAR).
      - Output `mvn test` (menjalankan tes).
      - Pesan `echo` dari stage `Deploy`.
      - Jika semua berjalan lancar, Kita akan melihat status **"SUCCESS"** di akhir.

      Jika ada kegagalan, periksa _stack trace_ di `Console Output` untuk mengidentifikasi masalahnya (misalnya, kesalahan kompilasi, tes gagal, atau masalah dengan alat Java/Maven).

---

**6. Integrasi Git dengan Jenkins (Kredensial)**

Jika repositori GitHub Kita privat, Jenkins tidak akan bisa mengaksesnya tanpa kredensial. Ada dua metode utama:

- **GitHub Access Token (Disarankan untuk GitHub pribadi/repositori publik yang sering diakses):**
  1. **Buat Personal Access Token di GitHub:**

     - Pergi ke GitHub Kita -> Settings -> Developer settings -> Personal access tokens -> Tokens (classic) -> Generate new token (classic).
	    ![[Pasted image 20250617111423.png]]
	    
     - Berikan nama yang deskriptif (misalnya, `jenkins-token`).
	    ![[Pasted image 20250617111558.png]]
	    
     - Berikan izin (`scopes`) yang diperlukan, minimal `repo` (untuk mengakses repositori).

		![[Pasted image 20250617111636.png]]
		
     - Generate token dan **salin tokennya segera**. Kita tidak akan bisa melihatnya lagi.

  2. **Tambahkan Kredensial di Jenkins:**

	![[Pasted image 20250617111911.png]]
     - Dashboard Jenkins -> Manage Jenkins -> Credentials.
     
	    ![[Pasted image 20250617111938.png]]
     - Klik **"(global)"** di bawah "Stores scoped to Jenkins".
     
	    ![[Pasted image 20250617112009.png]]
     - Klik **"Add Credentials"** di sidebar kiri.
     
	    ![[Pasted image 20250617112032.png]]
     - **Kind:** Pilih `Secret text`.
     ![[Pasted image 20250617113055.png]]
     - **Secret:** Tempelkan Personal Access Token GitHub yang tadi Kita salin.
     - **ID:** Berikan ID yang unik dan mudah diingat (misalnya, `github-personal-access-token`). Ini akan digunakan di Pipeline Kita.
     - **Description:** (Opsional) Deskripsi singkat.
     - Klik **"Create"**.

  3. **Perbarui Konfigurasi Job Pipeline:**

     - Kembali ke konfigurasi _job_ `Simple-Java-Maven-CI` Kita.
     - Di bagian **"Pipeline" -> "SCM" -> "Git" -> "Credentials"**, pilih kredensial yang baru Kita tambahkan berdasarkan ID-nya (`github-personal-access-token`).
     - Klik **"Save"**.
- **SSH Key (Disarankan untuk server produksi dan keamanan lebih tinggi):**
  1. **Buat SSH Key Pair di Host Jenkins (atau di mesin Kita):**
     - Buka Git Bash atau terminal Linux/macOS.
	    ![[Pasted image 20250617115915.png]]
  2. 
     - `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
     - Ikuti instruksi. Ini akan membuat `id_rsa` (private key) dan `id_rsa.pub` (public key) di `~/.ssh/`.
  3. **Tambahkan Public Key ke GitHub Repo:**
  ![[Pasted image 20250617114506.png]]
     - Pergi ke repositori GitHub Kita -> Settings -> Deploy keys -> Add deploy key.
     
     ![[Pasted image 20250617120018.png]]
     - Tempelkan isi file `id_rsa.pub`. Beri judul. Centang "Allow write access" jika Jenkins perlu melakukan push.
  4. **Tambahkan Private Key ke Jenkins Credentials:**
	  ![[Pasted image 20250617111938.png]]
     - Dashboard Jenkins -> Manage Jenkins -> Credentials -> (global) -> Add Credentials.
     
     ![[Pasted image 20250617120150.png]]
     - **Kind:** `SSH Username with private key`.
     
     ![[Pasted image 20250617120346.png]]
     - **ID:** Berikan ID (misalnya, `github-ssh-key`).
     - **Username:** `git` (ini standar untuk Git via SSH).
     - **Private Key:** Pilih "Enter directly" -> "Add" dan tempelkan isi dari file `id_rsa`.
     - **Passphrase:** Jika Kita mengatur passphrase saat membuat key, masukkan di sini.
     - **Create**.
  5. **Perbarui Konfigurasi Job Pipeline:**
     - Ganti Repository URL menjadi format SSH: `git@github.com:Danu-prasetyo/simple-java-maven-app.git`
     - Pilih kredensial SSH yang Kita tambahkan.
6. Perbarui Jenkinsfile
	Ganti stage checkout menggunakan SSH
	```
		// Stage 1: Checkout source code dari repositori Git(Comment/hapus baris sebelumnya)
	        // stage('Checkout/Pulling Code') {
	        //     steps {
	        //         // pull kode dari repositori Git
	        //         git branch: 'master', url: 'https://github.com/Danu-prasetyo/simple-java-maven-app.git'
	        //     }
	        // }

		// Ganti dengan baris ini
	        stage('Prepare SSH Host Keys') {
	            steps {
	                // Menambahkan kunci host GitHub ke known_hosts di workspace Jenkins
	                sh 'ssh-keyscan -H github.com >> ~/.ssh/known_hosts'
	                // Opsional: Untuk melihat isi known_hosts
	                // sh 'cat ~/.ssh/known_hosts'
	            }
	        }
	        stage('Checkout') {
	            steps {
	                // Pastikan Anda menggunakan URL SSH Git
	                git branch: 'main', url: 'git@github.com:Danu-prasetyo/simple-java-maven-app.git', credentialsId: 'github-ssh-key' // Sesuaikan credentialsId
	            }
	        }
	    // proses lainnya
	```

	Jangan lupa push changes ini ke repository. lalu jalankan ulang build jenkins.
---

**7. Membangun Docker Image Aplikasi (CD - Lanjutan)**

Ini adalah langkah kunci dalam _Continuous Deployment_ di mana Pipeline Kita akan membangun _Docker image_ dari aplikasi Kita dan mungkin mendorongnya ke _Docker Registry_.

Prasyarat:

Untuk membangun Docker image di dalam Pipeline Jenkins, kontainer Jenkins Kita perlu memiliki akses ke Docker daemon host. Ini adalah konfigurasi yang lebih kompleks dan berpotensi kurang aman (dikenal sebagai "Docker-in-Docker" atau "Dind").

Pendekatan Sederhana untuk Demo Lokal (dan lebih aman/direkomendasikan untuk produksi):

Alih-alih menjalankan Docker daemon di dalam kontainer Jenkins, pendekatan yang lebih disukai adalah:

1. **Menggunakan Jenkins Agent dengan Docker Client:** Kita memiliki agen terpisah (bisa kontainer Docker lain atau VM) yang memiliki Docker terinstal dan dapat menjalankan perintah Docker. Jenkins master akan mendelegasikan tugas Docker ke agen ini.
2. **Mount Docker Socket:** Ini adalah cara termudah untuk demo lokal. Kita akan me-mount _Docker socket_ dari host ke kontainer Jenkins. Ini memungkinkan Jenkins _master_ untuk menggunakan _Docker daemon_ di host. **Peringatan: Ini memberikan kontainer Jenkins akses penuh ke Docker daemon host Kita, yang bisa menjadi risiko keamanan di lingkungan produksi.**

**Mari kita gunakan metode "Mount Docker Socket" untuk demo ini.**

1. **Hentikan dan Hapus Kontainer Jenkins Kita Saat Ini:**

   Bash

   ```
   docker stop my-jenkins
   docker rm my-jenkins
   ```

2. **Jalankan Kembali Kontainer Jenkins dengan Docker Socket Mount:**

   Bash

   ```
   docker run -d -p 8080:8080 -p 50000:50000 \
   --name my-jenkins \
   -v C:/jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock \ # Mount Docker socket
   -v C:/ProgramData/docker:/var/lib/docker \ # Mount Docker data directory (optional, but can help with some issues)
   jenkins/jenkins:lts
   ```

   - `-v /var/run/docker.sock:/var/run/docker.sock`: Ini adalah _socket_ Unix yang digunakan Docker CLI untuk berkomunikasi dengan _Docker daemon_. Di Windows (dengan WSL2), Docker Desktop mengeksposnya melalui WSL2 sehingga path ini dapat diakses.
   - `-v C:/ProgramData/docker:/var/lib/docker`: Ini adalah direktori tempat Docker Desktop menyimpan data-datanya (gambar, kontainer, volume). Memasang ini dapat membantu beberapa skenario, tetapi `docker.sock` adalah yang utama.

3. Buat Dockerfile untuk Aplikasi Java/Maven Kita:

   Di root repositori GitHub Kita (simple-java-maven-app), buat file baru bernama Dockerfile.

   Dockerfile

   ```
   # Dockerfile

   # Stage 1: Build Java application
   FROM maven:3.8.6-openjdk-17-slim AS builder
   WORKDIR /app
   COPY pom.xml .
   COPY src ./src
   RUN mvn clean package -DskipTests # Build the JAR, skip tests during Docker build as Jenkins already tested

   # Stage 2: Create a lightweight runtime image
   FROM openjdk:17-jre-slim
   WORKDIR /app
   # Copy the JAR from the builder stage
   COPY --from=builder /app/target/simple-java-maven-app-1.0.jar app.jar
   EXPOSE 8080 # Expose the port your Spring Boot/Java app might run on

   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```

   - **Penting:** Aplikasi Kita `simple-java-maven-app` adalah aplikasi Java Maven dasar. Untuk membuatnya berjalan sebagai aplikasi web (misalnya, Spring Boot), Kita mungkin perlu menambahkan dependensi web di `pom.xml` dan sedikit kode untuk _server_ HTTP sederhana. Namun, `Dockerfile` di atas akan membuat _image_ yang berisi JAR yang dapat dieksekusi.

4. Perbarui Jenkinsfile untuk Membangun Docker Image:

   Tambahkan stage baru di Jenkinsfile Kita.

   Groovy

   ```
   // Jenkinsfile

   pipeline {
       agent any

       tools {
           jdk 'JDK_11' // Sesuaikan dengan nama JDK Kita
           maven 'M3'   // Sesuaikan dengan nama Maven Kita
       }

       stages {
           stage('Checkout') {
               steps {
                   git branch: 'main', url: 'https://github.com/Danu-prasetyo/simple-java-maven-app.git'
               }
           }

           stage('Build') {
               steps {
                   sh 'mvn clean install'
               }
           }

           stage('Test') {
               steps {
                   sh 'mvn test'
                   junit '**/target/surefire-reports/*.xml' // Publikasikan laporan tes
               }
           }

           // --- Stage Baru: Build Docker Image ---
           stage('Build Docker Image') {
               steps {
                   script {
                       // Membangun image Docker
                       // Tag image dengan nomor build Jenkins untuk keunikan
                       def dockerImage = docker.build("simple-java-app:${env.BUILD_NUMBER}")
                       echo "Docker image built: ${dockerImage.id}"

                       // Opsional: push ke Docker Hub atau registry lainnya
                       // Pastikan Kita sudah login ke Docker Hub di Jenkins kontainer
                       // atau sediakan kredensial Docker Hub di Jenkins
                       // docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials-id') { // Ganti ID kredensial Docker Hub Kita
                       //     dockerImage.push()
                       // }
                   }
               }
           }

           // --- Stage Baru: Deploy (Simulasi atau Nyata) ---
           stage('Deploy Application') {
               steps {
                   echo 'Simulating deployment...'
                   // Contoh simulasi deployment dengan menjalankan kontainer dari image yang baru dibuat
                   script {
                       def appContainer = docker.image("simple-java-app:${env.BUILD_NUMBER}").run("-p 8081:8080") // Jalankan di port 8081 host
                       echo "Application deployed to http://localhost:8081"
                       // Untuk demo, kita mungkin ingin menghentikan kontainer ini setelah beberapa waktu atau secara manual.
                       // Di produksi, ini akan menjadi bagian dari orchestrator (Kubernetes/Docker Swarm)
                   }
                   echo 'Deployment simulated successfully!'
               }
           }
       }

       post {
           always {
               echo 'Pipeline finished.'
               cleanWs() // Membersihkan workspace setelah build selesai
           }
           success {
               echo 'Pipeline executed successfully!'
           }
           failure {
               echo 'Pipeline failed!'
           }
       }
   }
   ```

   - **`docker.build(...)`**: Ini adalah sintaks Pipeline Docker yang memungkinkan Jenkins berinteraksi dengan Docker daemon.
   - `docker.withRegistry(...)`: Blok opsional untuk _push_ image ke registry. Kita perlu membuat kredensial Jenkins jenis "Username with password" untuk Docker Hub dan memberikan ID kredensialnya di sini.
   - **Pastikan Kita telah meng-commit `Dockerfile` dan `Jenkinsfile` yang diperbarui ke repositori GitHub Kita!**

5. Jalankan Build Kembali:

   Pergi ke Jenkins UI, ke job Simple-Java-Maven-CI Kita, dan klik "Build Now".

   Amati Console Output. Kita akan melihat langkah-langkah untuk membangun Docker image dan menjalankan kontainer (simulasi deployment).

---

**8. Trigger Otomatisasi CI/CD**

Untuk otomatisasi penuh, Jenkins dapat memicu _build_ secara otomatis.

- **SCM Polling:**
  - Jenkins akan secara berkala (misalnya, setiap 5 menit) memeriksa repositori Git Kita untuk perubahan. Jika ada perubahan, _build_ akan dipicu.
  - Di konfigurasi _job_ Pipeline Kita: **"Build Triggers" -> Centang "Poll SCM"**.
  - Di kolom "Schedule", masukkan jadwal Cron (misalnya, `H/5 * * * *` untuk polling setiap 5 menit).
- **Webhook (GitHub Webhook - Lebih Efisien):**
  - GitHub akan memberi tahu Jenkins secara instan setiap kali ada perubahan (commit, push) di repositori Kita. Ini jauh lebih efisien daripada polling.
  - **Langkah-langkah:**
    1. **Aktifkan GitHub Hook di Jenkins:**
       - Di konfigurasi _job_ Pipeline Kita: **"Build Triggers" -> Centang "GitHub hook trigger for GITScm polling"**.
       - Simpan konfigurasi job.
    2. **Konfigurasi Webhook di GitHub:**
       - Pergi ke repositori GitHub Kita -> Settings -> Webhooks.
       - Klik **"Add webhook"**.
       - **Payload URL:** Masukkan URL Jenkins Kita diikuti dengan `/github-webhook/`. Contoh: `http://YOUR_JENKINS_IP:8080/github-webhook/`
         - **Penting:** Jika Jenkins Kita berjalan di `localhost`, GitHub tidak bisa langsung mengaksesnya. Untuk menguji webhook secara lokal, Kita perlu menggunakan alat seperti `ngrok` atau membuat Jenkins dapat diakses dari internet. Untuk demo lokal, SCM Polling lebih mudah.
         - Jika Kita menggunakan `ngrok`, jalankan `ngrok http 8080` dan gunakan URL yang diberikan `ngrok`.
       - **Content type:** `application/json`
       - **Secret:** (Opsional, tapi disarankan) Masukkan _secret_ acak. Kita juga perlu mengkonfigurasi _secret_ ini di Jenkins (di konfigurasi _job_ atau global).
       - **Which events would you like to trigger this webhook?**: Pilih `Just the push event.`
       - Centang "Active".
       - Klik **"Add webhook"**.
  Sekarang, setiap kali Kita `git push` ke repositori, GitHub akan mengirim notifikasi ke Jenkins, dan Jenkins akan memicu build Pipeline Kita secara otomatis.

---

**9. Troubleshooting Umum Jenkins**

- **Izin Volume (Permission Denied):**
  - Error: `Permission denied` saat Jenkins mencoba menulis ke `/var/jenkins_home`.
  - Solusi: Pastikan user yang menjalankan kontainer Jenkins memiliki izin tulis di `C:\jenkins_home` di host Kita. Jenkins di Docker biasanya berjalan sebagai user `jenkins` dengan UID `1000`. Di Windows, ini biasanya tidak jadi masalah besar karena WSL2 menangani _mapping_ izin. Namun, jika Kita mengalami masalah, coba ubah kepemilikan atau izin direktori `C:\jenkins_home` di WSL2/Linux.
- **Konektivitas Jaringan (Can't connect to GitHub/Docker Hub):**
  - Error: `Could not connect to github.com` atau `Failed to pull image`.
  - Solusi: Pastikan kontainer Jenkins Kita memiliki akses internet. Periksa pengaturan _firewall_ Windows Kita atau konfigurasi proxy Docker Desktop.
- **Plugin Hilang atau Rusak:**
  - Error: `No such DSL method 'git' found` atau error terkait fitur Pipeline yang tidak dikenal.
  - Solusi: Pergi ke "Manage Jenkins" -> "Plugins" -> "Installed plugins". Pastikan plugin yang diperlukan (misalnya, Git Plugin, Pipeline Plugin, Pipeline: Stage View Plugin) sudah terinstal dan diaktifkan. Coba restart Jenkins kontainer jika baru saja menginstal plugin.
- **Masalah Dependensi (Java/Maven Path):**
  - Error: `mvn not found` atau `java not found`.
  - Solusi: Pastikan Kita telah mengkonfigurasi JDK dan Maven dengan benar di "Manage Jenkins" -> "Tools" dan nama yang Kita gunakan di `Jenkinsfile` (misalnya, `jdk 'JDK_11'`, `maven 'M3'`) cocok persis dengan yang dikonfigurasi.
- **Kontainer Nginx/Apache dari Docker Compose Tidak Bisa Diakses:**
  - Solusi: Pastikan port forwarding (`-p 80:80`) sudah benar di `docker-compose.yml` atau `docker run`. Periksa juga _firewall_ Windows.

---

Sekian!
