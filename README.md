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