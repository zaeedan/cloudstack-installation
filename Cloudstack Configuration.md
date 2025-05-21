# Membangun Infrastruktur Cloud Service
![image](https://github.com/user-attachments/assets/68bc24c6-5799-47d5-9f3b-f839d1b6e0bd)

## Mengakses Web Cloudstack
Setelah berhasil melakukan instalasi Cloudstack, buka browser dan akses url web cloudstack.
Lakukan default login menggunakan informasi username dan password default. Untuk keamanan, user bisa mengubah passwordnya nanti di bagian Setting.

Di dalam dashboard, user dapat melihat beberapa menu seperti:
- Zone: wilayah data center.
- Template: tempat ISO/image dari OS yang akan digunakan.
- Instance : VM yang berjalan.
- Service Offering : konfigurasi CPU/RAM untuk VM.
- Network : pengaturan jaringan VM.
Untuk membuat VM, user harus menyiapkan struktur VM-nya dari menu-menu yang disebutkan. Dimulai dari membuat zone (Add Zone) terlebih dahulu.

## Add Zone
Di menu Zone, user akan melihat tampilan pertama sebagai berikut. User harus menentukan tipe zone yang akan digunakan sesuai kebutuhan:
- Core: ditujukan untuk data center, memiliki range jaringan yang cukup luas dan memberikan akses pada fungsionalitas lainnya di Apache Cloudstack.
- Edge: disebut lightweight zone, digunakan untuk layanan publik atau aplikasi yang diakses dari luar. Fungsionalitas terbatas dibandingkan tipe Core.
Pada langkah ini, pilih yang tipe Core.
![image](https://github.com/user-attachments/assets/18755c8a-4fd5-4d41-aa6e-d32cf4e72235)
Selanjutnya memilih Core Zone Type.
- Advanced: lebih fleksibel dan bisa bikin banyak jaringan (public/private).
- Basic: lebih sederhana, semua VM di 1 jaringan dan cocok untuk testing.
Pada langkah ini, pilih yang tipe Basic karena user akan deploy VM yang memang ditujukan untuk melakukan testing saja.
![image](https://github.com/user-attachments/assets/8f68c20b-5d44-4747-ab85-d937a5f3b88b)
Pada bagian Zone details, user akan diminta untuk mengatur informasi dasar mengenai zone (data center) yang dibentuk.
Informasi yang diisi adalah:
```
Name: Kelompok5
IPv4 DNS1: 8.8.8.8
IPv4 DNS2: 1.1.1.1
Internal DNS 1: 192.168.18.1
Hypervisor: KVM
```

Dilanjut ke bagian Network, disini user akan menentukan topologi jaringan dan membuat traffic type yang dibutuhkan oleh CloudStack.
Namun karena tadi di bagian Core Zone type sudah memilih 'Basic', maka hanya akan ditampilkan 1 physical network yang berfungsi merepresentasikan jaringan fisik tunggal yang digunakan oleh semua jenis trafik.
![image](https://github.com/user-attachments/assets/03ba50e3-9118-4395-91c8-b57d2eaf4f92)
Setiap zone harus memiliki satu atau lebih pod. Di bagian pod ini, user akan mengkonfigurasi informasi yang dibutuhkan oleh sebuah pod, seperti pod name (nama untuk identifikasi pod); reserved system gateway (IP address router jaringan local yang menghubungkan VM ke luar jaringan); reserved system netmask (ukuran subnet); start & end reserved system IP (IP address khusus CloudStack system VM). Berikut adalah detail dari pengisian informasi pada Pod:
```
Pod name: kelompok5
Reserved system gateway: 192.168.18.1
Reserved system netmask: 255.255.255.0
Start reserved system IP: 192.168.18.11
End reserved system IP: 192.168.18.20
```
![image](https://github.com/user-attachments/assets/eb27938b-b994-4c46-9c6a-894201ade06a)
Setelah Pod, user diarahkan ke bagian Guest Network Traffic. Bagian ini menentukan jenis trafik untuk VM (dari dan antar VM). IP address yang diambil berasal dari range IP address yang sudah ditentukan di Pod.
```
Guest gateway: 192.168.18.1
Guest netmask: 255.255.255.0
Guest start IP: 192.168.18.21
Guest end IP: 192.168.18.10
```
Langkah selanjutnya adalah Add resources. Di bagian ini, terdapat beberapa sub-step yang harus dilakukan.
a. Cluster: kumpulan host (server KVM) yang pakai jenis hypervisor sama. Semua host di dalam cluster ini akan berada di subnet yang sama dan mengakses shared storage yang sama pula.
```
Cluster name: kelompok5
```
b. IP address: mengkonfigurasi informasi dari host (assign alamat ip) agar host dapat bekerja di dalam CloudStack.
```
Host name: 192.168.18.250
Username: root
Authentication Method: Password | System SSH Key
Password: 
```
c. Primary Storage: tempat nyimpan disk utama VM yang berjalan pada semua host yang ada di dalam cluster.
```
Name: primaryStorage_kelompok5
Scope: Zone
Protocol: nfs
Server: 192.168.18.250
Path: /export/primary
Provide: DefaultPrimary
```
d. Secondary Storage: tempat untuk menyimpan template, ISO, dan Snapshot.
```
Provider: NFS
Name: secondaryStorage_kelompok5
Server: 192.168.18.250
Path: /export/secondary
```
Terakhir, launch zone yang sudah dibentuk. Dengan begitu, zone akan aktif dan dapat digunakan untuk menjalankan VM.

## Insert Image Ubuntu ke VM

Langkah upload image Ubuntu ini penting agar CloudStack bisa membuat VM dengan sistem operasi yang ditentukan, dan itu bisa terwujud dengan menyediakan sumber instalasi OS. Bentuknya bisa berupa ISO (CD installer) atau Template (berupa image disk yang siap dipakai). Tanpa image Ubuntu ini, VM tidak akan tahu sistem operasi apa yang harus dijalankan atau digunakan. Berikut adalah detail dari pengisian informasi yang dibutuhkan pada proses insert image Ubuntu:
![image](https://github.com/user-attachments/assets/3a68131f-266b-4d10-b404-4b13ab7ce9cf)

Jika berhasil, maka tampilannya akan menjadi seperti ini.
![image](https://github.com/user-attachments/assets/ffe3ae20-b359-4112-a4ab-fa978167b0c1)

## Add Compute Offering

Compute Offering merupakan tempat konfigurasi CPU dan RAM untuk VM. Konfigurasi spesifikasi VM ini penting karena jika tidak dilakukan, CloudStack tidak akan tahu harus mengalokasikan resource sebesar apa untuk VM yang dibuat oleh user. Menu ini bisa dilihat dari menu utama Service Offerings. 
Berikut adalah pengisian bagian compute offering:
![image](https://github.com/user-attachments/assets/febde1f0-280a-4b15-89fa-4c98cba7bcad)
![image](https://github.com/user-attachments/assets/e06ea6f1-6d96-46c1-bb82-a64e8b5375ec)

Jika sudah berhasil, maka tampilannya akan menjadi seperti ini.
![image](https://github.com/user-attachments/assets/8a126786-5fdb-4367-af04-c2d0c21f8131)

## Add instance

Langkah ini dilakukan setelah semua konfigurasinya selesai, mulai dari zone, network, storage, dan sebagainya (bisa dilihat lagi dari atas). Intinya, di bagian ini user akan membuat VM berdasarkan spesifikasi zone yang telah dibentuk agar tahu bagaimana VM-nya dijalankan dan diakses.
Beberapa spesifikasi yang diminta pada bagian ini adalah sebagai berikut:
1. Assign Instance to another Account
```
Domain: ROOT
Account: admin
```
2. Select deployment infrastructure: isi dengan informasi data center yang telah dibuat di langkah-langkah sebelumnya.
```
Zone: zone_kelompok5
Pod: pod_kelompok5
Cluster: cluster_kelompok5
Host: cloudstack5
```
3. Template/ISO: pilih informasi template/ISO yang telah di-input sebelumnya. Disini, kami menggunakan ISO Ubuntu 22 (Ubuntu_22) dengan Hypervisor jenis KVM.
![image](https://github.com/user-attachments/assets/b012861e-fde3-40f1-9751-546b2f572cbd)

4. Compute offering: pilihlah hasil konfigurasi VM sesuai dengan spesifikasi yang diinginkan (yang telah dibuat sebelumnya).
![image](https://github.com/user-attachments/assets/fd73bb7a-dd5b-4258-93ef-c11ff83b58a2)

5. Disk size: 30 GB
![image](https://github.com/user-attachments/assets/e9959335-1cd9-441d-a4fb-1b9386a73fcc)

[Bagian SSH key pairs, Advanced mode, dan Details bersifat opsional]
![image](https://github.com/user-attachments/assets/4de0a605-c870-4d9c-aa4e-87d355c2df87)

Jika sudah selesai mengisi dan berhasil add, maka tampilannya akan menjadi seperti ini. VM yang sudah berhasil dibuat sekarang dapat diakses. 
![image](https://github.com/user-attachments/assets/8dc0b3c0-6138-46ed-bc85-3c6cb1815e34)
![image](https://github.com/user-attachments/assets/d02e1b98-84ac-46cf-ba14-8b9f40f8bf90)

Melakukan akses ke VM:
![image](https://github.com/user-attachments/assets/a9f5f9a4-ac25-468c-b78c-b1d4f2cd3882)

## Akses VM Lewat SSH (Port Forwarding)

Agar bisa melakukan login ke VM lewat laptop secara remote, user harus melakukan langkah berikut:
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 192.168.18.29:22
sudo iptables -t nat -A POSTROUTING -p tcp -d 192.168.18.29 --dport 22 -j MASQUERADE
```
Di langkah ini, informasi perangkat laptop yang memiliki port 2222 di host akan diteruskan ke port 22 di VM dengan IP address 192.168.18.29. Dengan begitu, laptop dapat melakukan SSH ke VM.

Namun, user masih harus melakukan beberapa step tambahan ini setelah melakukan SSH dengan tujuan dapat mengakses VM melalui browser.
1. Install apache server
```
sudo apt install apache2
```
2. Membuat website pada apache server
```
sudo mkdir /var/www/gci/
```
Lalu, isi file html-nya dengan informasi berikut.
![image](https://github.com/user-attachments/assets/9584aaa5-12fc-4d67-87d8-e6bf246e39d5)

3. Lakukan autentikasi
```
cd /etc/apache2/sites-available/
sudo cp 000-default.conf gci.conf
sudo a2ensite gci.conf
```
Tampilan:
![image](https://github.com/user-attachments/assets/be547b85-32f5-4143-a535-008a2ce04436)

4. Ping website untuk memeriksa apakah website berhasil dibuat.
```
ping kelompok5.com
```
![image](https://github.com/user-attachments/assets/7dda2471-031b-4603-b09d-3d45cfb44c1d)

Website berhasil di-ping, sekarang website dapat diakses melalui browser.
