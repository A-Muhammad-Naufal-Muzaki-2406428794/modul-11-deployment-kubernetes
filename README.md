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