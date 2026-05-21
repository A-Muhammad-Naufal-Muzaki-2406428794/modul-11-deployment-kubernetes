# Module 11 - Deployment on Kubernetes

## Tutorial: Hello Minikube

Pada tutorial ini saya menjalankan Kubernetes cluster lokal menggunakan Minikube. Setelah menjalankan `minikube start`, saya membuat Deployment bernama `hello-node` menggunakan image `registry.k8s.io/e2e-test-images/agnhost:2.39`.

Deployment tersebut membuat Pod yang berjalan dengan status `Running`. Saya juga membuat Service dengan tipe `LoadBalancer` agar aplikasi dapat diakses dari luar cluster melalui bantuan `minikube service hello-node`.

Setelah aplikasi dibuka melalui browser, log aplikasi dapat dilihat menggunakan `kubectl logs`.