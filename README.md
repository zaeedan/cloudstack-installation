# Implementasi Apache Cloudstack Private Cloud
![CloudStack Logo](https://upload.wikimedia.org/wikipedia/commons/7/70/Apache_CloudStack_Logo.svg)

# Kelompok 5
Contributors : 
- Ajriya Muhammad Arkan (2206031826)
- Audrina Cristella Hasibuan (2206062926)
- Christopher Sutandar (2206810414)
- Muhammad Abrisam Cahyo Juhartono (2206026050)
- Rizqi Zaidan (2206059742)

## Tujuan Proyek

```
CPU :
RAM :
Penyimpanan :
Network :
Sistem Operasi : Ubuntu Server 24.04
```

## Apa Itu Cloudstack
Apache CloudStack adalah platform open-source untuk membangun, mengelola, dan mengoperasikan infrastruktur cloud computing yang skalabel. Dalam implementasi private cloud, CloudStack memungkinkan organisasi untuk menyediakan sumber daya komputasi (seperti VM, storage, dan jaringan) secara on-demand di dalam lingkungan tertutup dan aman.

## Fitur-Fitur Cloudstack
#### 1. Penyediaan mandiri
Manajemen VM, storage, dan jaringan dilakukan secara terpusat melalui satu sistem kendali.
#### 2. Provisioning dan orkestrasi otomatis
Mendukung otomatisasi dalam penyediaan dan pengelolaan sumber daya infrastruktur secara efisien.
#### 3. Antarmuka web dan API yang kuat
Menyediakan UI berbasis web yang intuitif dan API yang kaya fitur untuk integrasi dan automasi.
#### 4. Integrasi dengan hypervisor populer
Mendukung berbagai hypervisor industri standar seperti:
* KVM (*Kernel-based Virtual Machine*)
* XenServer
* VMware vSphere
#### 5. Multi-tenant dan kontrol akses berbasis peran
Mendukung isolasi pengguna dan kontrol akses berdasarkan peran (role-based access control), ideal untuk lingkungan dengan banyak tim atau unit kerja.

## Teknologi yang Digunakan
#### 1. Hypervisor:
KVM (Kernel-based Virtual Machine) – digunakan untuk menjalankan VM di host Linux.
#### 2. Storage:
NFS (Network File System) – digunakan sebagai primary storage untuk menyimpan disk VM.
#### 3. Network:
Virtual Router – menyediakan layanan DHCP, firewall, NAT, dan VPN untuk jaringan virtual.
#### 4. Database Backend
MySQL atau MariaDB – digunakan untuk menyimpan konfigurasi dan metadata sistem cloud.
#### 5. Orkestrasi & Manajemen:
Management Server – mengatur provisioning, monitoring, dan kontrol seluruh resource cloud.
#### 5. Skalabilitas & High Availability:
Multi-zone architecture – memungkinkan pengelolaan beberapa data center dalam satu platform terpusat.

## Arsitektur Sistem
Apache CloudStack menggunakan struktur hierarkis untuk mengatur infrastruktur cloud secara efisien. Gambar tersebut menggambarkan satu Zone, yang terdiri dari satu atau lebih Pod, yang di dalamnya terdapat Cluster, Host, dan Primary Storage. Selain itu, terdapat Secondary Storage yang terpisah namun terhubung ke seluruh zone.
#### 1. Zone:
* Mewakili satu data center atau lokasi fisik.
* Setiap zone memiliki akses ke Secondary Storage.
* Bisa terdiri dari satu atau beberapa pod.
#### 2. Pod:
NFS (Network File System) – digunakan sebagai primary storage untuk menyimpan disk VM.
#### 3. Network:
Virtual Router – menyediakan layanan DHCP, firewall, NAT, dan VPN untuk jaringan virtual.
#### 4. Database Backend
MySQL atau MariaDB – digunakan untuk menyimpan konfigurasi dan metadata sistem cloud.
#### 5. Orkestrasi & Manajemen:
Management Server – mengatur provisioning, monitoring, dan kontrol seluruh resource cloud.
#### 5. Skalabilitas & High Availability:
Multi-zone architecture – memungkinkan pengelolaan beberapa data center dalam satu platform terpusat.

