# Module 11 - Deployment on Kubernetes

## Tutorial: Hello Minikube

Pada tutorial ini saya menjalankan Kubernetes cluster lokal menggunakan Minikube. Setelah menjalankan `minikube start`, saya membuat Deployment bernama `hello-node` menggunakan image `registry.k8s.io/e2e-test-images/agnhost:2.39`.

Deployment tersebut membuat Pod yang berjalan dengan status `Running`. Saya juga membuat Service dengan tipe `LoadBalancer` agar aplikasi dapat diakses dari luar cluster melalui bantuan `minikube service hello-node`.

Setelah aplikasi dibuka melalui browser, log aplikasi dapat dilihat menggunakan `kubectl logs`.

## Monitoring Resource Usage

Saya mencoba menjalankan command `kubectl top nodes` dan `kubectl top pods` untuk melihat penggunaan resource CPU dan memory. Pada awalnya command tersebut belum dapat digunakan karena Metrics API belum tersedia. Untuk mengaktifkannya, saya menjalankan command `minikube addons enable metrics-server`.

Setelah metrics-server aktif dan Pod metrics-server berjalan di namespace `kube-system`, command `kubectl top nodes` dan `kubectl top pods` dapat digunakan untuk menampilkan penggunaan CPU dan memory dari node dan Pod yang berjalan.

## Reflection on Hello Minikube

### 1. Compare the application logs before and after you exposed it as a Service.

Sebelum aplikasi diekspos sebagai Service, log hanya menunjukkan bahwa server aplikasi berhasil dijalankan. Contohnya, container menampilkan log bahwa HTTP server berjalan pada port 8080 dan UDP server berjalan pada port 8081.

Setelah Deployment diekspos sebagai Service bertipe LoadBalancer dan aplikasi dibuka menggunakan `minikube service hello-node`, log mulai menampilkan tambahan aktivitas yang berkaitan dengan request masuk. Ketika saya membuka atau me-refresh aplikasi beberapa kali di browser, jumlah log bertambah. Hal ini terjadi karena setiap akses dari browser menghasilkan request ke aplikasi yang berjalan di dalam Pod, lalu request tersebut tercatat di log aplikasi.

Dari percobaan ini, saya memahami bahwa Service berhasil meneruskan traffic dari browser lokal ke Pod yang berada di dalam Kubernetes cluster.

### 2. What is the purpose of the `-n` option and why did the output not list the pods/services that you explicitly created?

Option `-n` pada `kubectl` digunakan untuk menentukan namespace tempat command dijalankan. Namespace adalah pemisahan logis di dalam Kubernetes cluster yang digunakan untuk mengelompokkan resource.

Ketika saya menjalankan `kubectl get pods` tanpa option `-n`, Kubernetes menggunakan namespace default. Karena itu, output menampilkan resource yang saya buat sendiri, seperti Pod dan Service dari `hello-node`.

Namun, ketika saya menjalankan `kubectl get pods,services -n kube-system`, Kubernetes menampilkan resource dari namespace `kube-system`. Namespace ini berisi komponen internal Kubernetes dan Minikube, seperti CoreDNS, metrics-server, dan service internal lainnya. Output tersebut tidak menampilkan Pod atau Service `hello-node` karena resource tersebut dibuat di namespace default, bukan di namespace `kube-system`.

## Tutorial: Rolling Update Deployment

Pada bagian ini saya membuat Deployment baru bernama `spring-petclinic-rest` menggunakan image `docker.io/springcommunity/spring-petclinic-rest:3.0.2`. Setelah Deployment berhasil dibuat, saya mengecek Pod dan log aplikasi untuk memastikan aplikasi berjalan dengan baik. Dari log aplikasi, web server Spring Boot berjalan pada port 9966.

Saya kemudian membuat Service bertipe `LoadBalancer` untuk Deployment tersebut menggunakan port 9966. Service ini memungkinkan aplikasi Spring Petclinic REST diakses dari luar cluster melalui command `minikube service spring-petclinic-rest`.

Setelah aplikasi berhasil dijalankan, saya mencoba melakukan scaling Deployment menjadi 4 replica menggunakan command `kubectl scale deployment spring-petclinic-rest --replicas=4`. Setelah scaling, jumlah Pod Spring Petclinic REST bertambah menjadi 4 dan semuanya berjalan dengan status `Running`.

Kemudian saya melakukan rolling update dengan mengganti image dari versi `3.0.2` menjadi `3.2.1` menggunakan command `kubectl set image`. Kubernetes mengganti Pod lama secara bertahap dengan Pod baru yang menggunakan image versi terbaru. Setelah proses rollout selesai, Deployment berhasil menggunakan image `3.2.1`.

Saya juga mencoba mengganti image ke versi yang tidak tersedia, yaitu `4.0`. Beberapa Pod gagal berjalan dan menampilkan status seperti `ImagePullBackOff`. Hal ini terjadi karena Kubernetes tidak dapat menarik image dengan tag tersebut dari container registry. Setelah itu, saya menggunakan `kubectl rollout undo` untuk mengembalikan Deployment ke versi terakhir yang berhasil, yaitu `3.2.1`.

## Kubernetes Manifest Files

Pada bagian ini saya mengekspor konfigurasi Deployment dan Service dari cluster Kubernetes menjadi file manifest YAML. File `deployment.yaml` berisi definisi Deployment untuk aplikasi Spring Petclinic REST, sedangkan file `service.yaml` berisi definisi Service bertipe LoadBalancer.

Setelah file manifest dibuat, saya mencoba menghapus cluster Minikube menggunakan `minikube delete`, lalu membuat ulang cluster dengan `minikube start`. Kemudian saya menjalankan ulang aplikasi menggunakan `kubectl apply -f deployment.yaml` dan `kubectl apply -f service.yaml`.

Dengan menggunakan manifest file, proses deployment menjadi lebih mudah diulang karena konfigurasi aplikasi sudah terdokumentasi dalam file YAML. Saya tidak perlu lagi menjalankan semua command manual satu per satu dari awal.

## Reflection on Rolling Update & Kubernetes Manifest File

### 1. What is the difference between Rolling Update and Recreate deployment strategy?

Rolling Update dan Recreate adalah dua strategi deployment yang memiliki cara kerja berbeda dalam mengganti versi aplikasi. Pada strategi Rolling Update, Kubernetes mengganti Pod lama dengan Pod baru secara bertahap. Artinya, sebagian Pod lama tetap berjalan ketika Pod baru sedang dibuat, sehingga aplikasi masih dapat melayani request selama proses update berlangsung. Strategi ini cocok digunakan untuk aplikasi yang membutuhkan ketersediaan tinggi dan ingin mengurangi downtime saat deployment.

Sementara itu, pada strategi Recreate, Kubernetes akan menghentikan semua Pod lama terlebih dahulu sebelum membuat Pod baru. Karena semua Pod lama dimatikan sebelum versi baru dijalankan, terdapat periode ketika aplikasi tidak tersedia. Hal ini dapat menyebabkan downtime, tetapi strategi ini lebih sederhana dan berguna ketika versi lama dan versi baru aplikasi tidak boleh berjalan bersamaan. Dari percobaan ini, saya memahami bahwa Rolling Update lebih aman untuk menjaga availability aplikasi, sedangkan Recreate lebih cocok untuk kondisi yang membutuhkan restart bersih atau perubahan besar yang tidak kompatibel dengan versi sebelumnya.

### 2. Try deploying the Spring Petclinic REST using Recreate deployment strategy and document your attempt.

Saya mencoba melakukan deployment Spring Petclinic REST menggunakan strategi Recreate dengan membuat manifest Deployment terpisah bernama `deployment-recreate.yaml`. Pada manifest tersebut, saya mengatur strategi deployment menjadi `type: Recreate`. Saya juga membuat manifest Service terpisah bernama `service-recreate.yaml` agar deployment dengan strategi Recreate dapat diakses tanpa bentrok dengan deployment Spring Petclinic REST sebelumnya yang menggunakan Rolling Update.

Setelah manifest dibuat, saya menjalankan `kubectl apply -f deployment-recreate.yaml` dan `kubectl apply -f service-recreate.yaml`. Kemudian saya memverifikasi hasilnya dengan menjalankan `kubectl get deployments`, `kubectl get pods`, dan `kubectl get services`. Deployment baru berhasil dibuat, Pod berhasil berjalan dengan status `Running`, dan Service berhasil tersedia dengan tipe `LoadBalancer`. Aplikasi juga dapat diakses menggunakan `minikube service spring-petclinic-rest-recreate`.

Untuk mengamati perilaku Recreate, saya mencoba mengganti image aplikasi menggunakan `kubectl set image`. Ketika proses update berjalan, saya melihat bahwa Pod lama dihentikan terlebih dahulu sebelum Pod baru dibuat. Perilaku ini berbeda dari Rolling Update yang mengganti Pod secara bertahap. Dari percobaan ini, saya memahami bahwa strategi Recreate memang lebih sederhana, tetapi memiliki risiko downtime karena tidak ada Pod yang berjalan saat proses pergantian versi berlangsung.

### 3. Prepare different manifest files for executing Recreate deployment strategy.

Saya menyiapkan manifest file yang berbeda untuk menjalankan strategi Recreate. File yang saya buat adalah `deployment-recreate.yaml` dan `service-recreate.yaml`. File `deployment-recreate.yaml` berisi konfigurasi Deployment untuk Spring Petclinic REST dengan strategi deployment Recreate. Perbedaan utama dengan `deployment.yaml` adalah pada bagian strategi deployment, yaitu manifest Rolling Update menggunakan `strategy.type: RollingUpdate`, sedangkan manifest Recreate menggunakan `strategy.type: Recreate`.

Saya juga menggunakan nama resource dan label yang berbeda, yaitu `spring-petclinic-rest-recreate`, agar tidak terjadi konflik dengan Deployment dan Service Spring Petclinic REST yang sudah dibuat sebelumnya. Service pada `service-recreate.yaml` menggunakan selector yang sesuai dengan label Pod dari Deployment Recreate. Dengan begitu, Service dapat meneruskan traffic ke Pod yang benar. Pemisahan manifest ini membuat konfigurasi Rolling Update dan Recreate lebih jelas, mudah diuji, dan mudah dibandingkan.

### 4. What do you think are the benefits of using Kubernetes manifest files? Recall your experience in deploying the app manually and compare it to your experience when deploying the same app by applying the manifest files (i.e., invoking `kubectl apply -f` command) to the cluster.

Menurut saya, manfaat utama menggunakan Kubernetes manifest files adalah deployment menjadi lebih terstruktur, konsisten, dan mudah diulang. Saat melakukan deployment secara manual, saya harus menjalankan banyak command seperti `kubectl create deployment`, `kubectl expose deployment`, `kubectl scale`, dan `kubectl set image`. Cara manual ini membantu saya memahami proses dasar Kubernetes, tetapi berisiko menyebabkan kesalahan jika ada command yang terlupa atau salah diketik.

Dengan manifest file, konfigurasi aplikasi seperti nama Deployment, jumlah replica, image container, port, Service, dan strategi deployment dapat ditulis secara deklaratif di dalam file YAML. Setelah itu, saya hanya perlu menjalankan `kubectl apply -f deployment.yaml` dan `kubectl apply -f service.yaml` untuk membuat resource yang sama. Hal ini membuat proses deployment lebih praktis dan repeatable.

Manifest file juga memudahkan dokumentasi karena seluruh konfigurasi deployment tersimpan di repository Git. Jika ada perubahan konfigurasi, perubahan tersebut dapat dilacak melalui commit. Selain itu, TA atau orang lain dapat mencoba menjalankan aplikasi di Minikube mereka hanya dengan menggunakan file manifest yang sama. Dibandingkan deployment manual, penggunaan manifest file terasa lebih cocok untuk proyek nyata karena lebih mudah dibagikan, diuji ulang, dan dipelihara.

### 5. (Optional) Do the same tutorial steps, but on a managed Kubernetes cluster (e.g., GCP). You need to provision a Kubernetes cluster on Google Cloud Platform. Then, re-run the tutorial steps (Hello Minikube and Rolling Update) on the remote cluster. Document your attempt and highlight the differences and any issues you encountered.

Saya tidak mengerjakan bagian opsional menggunakan managed Kubernetes cluster seperti Google Kubernetes Engine. Pada pengerjaan modul ini, saya berfokus pada bagian wajib menggunakan Minikube, yaitu menjalankan Hello Minikube, membuat Deployment dan Service, mengaktifkan metrics-server, melakukan Rolling Update, mencoba rollback, membuat Kubernetes manifest files, serta mencoba strategi Recreate.

Meskipun tidak menjalankan bagian opsional, saya memahami bahwa deployment pada managed Kubernetes cluster akan memiliki beberapa perbedaan dibandingkan Minikube. Pada Minikube, cluster berjalan secara lokal di komputer saya, sedangkan pada managed Kubernetes cluster, cluster berjalan di infrastructure cloud. Service bertipe LoadBalancer pada cloud juga dapat memperoleh external IP sungguhan, sedangkan pada Minikube akses biasanya dilakukan melalui bantuan `minikube service`. Selain itu, penggunaan managed cluster kemungkinan membutuhkan konfigurasi tambahan seperti akun cloud, billing, provisioning cluster, pengaturan kubectl context, dan pengelolaan resource cloud.