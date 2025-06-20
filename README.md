# Implementasi Apache Cloudstack Private Cloud
![CloudStack Logo](https://upload.wikimedia.org/wikipedia/commons/7/70/Apache_CloudStack_Logo.svg)

# Kelompok 5
Contributors : 
- Ajriya Muhammad Arkan (2206031826)
- Audrina Cristella Hasibuan (2206062926)
- Christopher Sutandar (2206810414)
- Muhammad Abrisam Cahyo Juhartono (2206026050)
- Rizqi Zaidan (2206059742)

# I. Mengenai Cloudstack
## Tujuan Proyek
#### 1. Membangun dan Mengkonfigurasi Private Cloud Berbasis Apache CloudStack
#### 2. Memahami Arsitektur dan Komponen Cloud Computing Secara Praktis
#### 3. Mengembangkan Kemampuan Deploy dan Manajemen VM melalui Web UI dan API
#### 4. Mempelajari Integrasi Komponen Virtualisasi dan Storage dalam Lingkungan Cloud

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
#### 4. Database Backend:
MySQL atau MariaDB – digunakan untuk menyimpan konfigurasi dan metadata sistem cloud.
#### 5. Orkestrasi & Manajemen:
Management Server – mengatur provisioning, monitoring, dan kontrol seluruh resource cloud.
#### 5. Skalabilitas & High Availability:
Multi-zone architecture – memungkinkan pengelolaan beberapa data center dalam satu platform terpusat.

## Arsitektur Sistem
<img src="https://github.com/user-attachments/assets/62652671-4837-4fec-a694-190a61a7f24a" alt="arsitektur" width="400"/>

Apache CloudStack menggunakan struktur hierarkis untuk mengatur infrastruktur cloud secara efisien. Gambar di atas menggambarkan satu Zone, yang terdiri dari satu atau lebih Pod, yang di dalamnya terdapat Cluster, Host, dan Primary Storage. Selain itu, terdapat Secondary Storage yang terpisah namun terhubung ke seluruh zone.
#### 1. Zone:
* Mewakili satu data center atau lokasi fisik.
* Setiap zone memiliki akses ke Secondary Storage.
* Bisa terdiri dari satu atau beberapa pod.
#### 2. Pod:
* Unit dalam zone yang biasanya mewakili satu rak fisik atau segmen jaringan.
* Terdiri dari satu atau lebih Cluster.
#### 3. Cluster:
* Sekumpulan host (server fisik) yang menjalankan hypervisor yang sama.
* Mengakses Primary Storage bersama.
* Digunakan untuk menjalankan VM dan mengelola resource compute.
#### 4. Host:
* Server fisik yang menjalankan VM melalui hypervisor (misalnya KVM).
* Terhubung ke Primary Storage untuk menyimpan disk image VM.
#### 5. Primary Storage:
* Menyimpan disk utama untuk VM yang berjalan di cluster.
* Bersifat shared di antara host dalam satu cluster.
#### 6. Secondary Storage:
* Digunakan untuk menyimpan: ISO (installer VM), Template VM, Snapshot

## Referensi
* P. Loshin, "Network File System (NFS)" TechTarget. [Online]. Available: https://www.techtarget.com/searchenterprisedesktop/definition/Network-File-System. [Accessed: May 21, 2025].
* "What is Apache CloudStack?" Apache Cloudstack. [Online]. Available: https://docs.cloudstack.apache.org/en/4.11.1.0/conceptsandterminology/concepts.html. [Accessed: May 21, 2025].
* "Apa Itu KVM (Mesin Virtual Berbasis Kernel)?" AWS. [Online]. Available: https://aws.amazon.com/id/what-is/kvm/. [Accessed: May 21, 2025].



# II. Instalasi Dan Konfigurasi Apache Cloudstack Private Cloud

## Operating System: Ubuntu Server 24.04.2 LTS

## Network Address

```
Alamat jaringan : 192.168.18.0/24
Alamat IP Host : 192.168.18.X/24
Gateway : 192.168.18.1
IP Publik : 100.96.X.X (Menggunakan TailScale)
```

## Network Configuration

### Modifikasi Berkas Konfigurasi Jaringan di Direktori /netplan
Masuk sebagai root dan buka direktori konfigurasi jaringan dengan perintah berikut:

```
sudo -i 
cd /etc/netplan
nano ./*.yaml
```

Kemudian, modifikasi isi berkas konfigurasi menjadi seperti berikut:

```
network:
  ethernets:
    enp1s0:
      dhcp4: false
      dhcp6: false
      optional: true
  wifis:
    wlp0s20f3:
      optional: true
      access-points:
        "XXX": # Access Point name
          password: "XXX" # Password
      dhcp4: true
  bridges:
    cloudbr0:
      addresses: [192.168.18.250/24]  # Alamat IP host
      routes:
        - to: default
          via: 192.168.18.1 # Gateaway
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [enp1s0]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
```

### Terapkan Konfigurasi Jaringan

```
netplan generate        # Menghasilkan berkas konfigurasi untuk renderer
netplan apply           # Menerapkan konfigurasi jaringan ke sistem
reboot                  # Memulai ulang sistem
```

## Menginstall Software Cloudstack Management Server Ke Ubuntu 

### Konfigurasi MariaDB

```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

### Menambahkan Command

```
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```

### Restart MariaDB

```
sudo systemctl restart mariadb
```

### Setup Database Dan Management Service.

```
sudo cloudstack-setup-databases cloud:password@localhost --deploy-as=root
sudo cloudstack-setup-management
```

### Konfigurasi Storage Primer dan Sekunder

```
apt install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

## Konfigurasi Server NFS
NFS dipakai sebagai Shared Storage sebagai penyimpunan Primer dan Sekunder, hal ini penting untuk dapat bisa diakses oleh banyak host pada waktu yang sama.

### Command:

```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

## Konfigurasi Host Cloudstack dengan Hypervisor KVM

### Install KVM dan Agen Cloudstack

```
apt install qemu-kvm cloudstack-agent -y
```

### Konfigurasi Manajemen Virtualisasi KVM
Mengubah isi Konfigurasi dengan:
```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# Tambahkan baris berikut ke /etc/default/libvirtd (Untuk Ubuntu 24.04 ke atas)
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

### Tambahkan Command berikut:
```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

### Restart libvirtd
```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

## Konfigurasi Docker dan Layanan Lainnya
```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

## Generate Host UUID
```
apt install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

## Konfigurasi Firewall
```
NETWORK=192.168.1.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT     # Portmap service (needed for NFS)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT    # NFS server port
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT   # NFS mountd (dynamic port)
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT   # NFS nlockmgr
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT     # NFS rpc.mountd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT     # NFS rquotad (quota)
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT     # NFS statd
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT    # CloudStack Agent
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT    # CloudStack Management Server
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT    # CloudStack Console Proxy (VM console)	
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT   # libvirt (KVM communication)

apt install iptables-persistent
```

## Install Cloudstack Agent
```
systemctl unmask cloudstack-agent
apt update -y
apt install cloudstack-agent -y
```

## Run Cloudstack Management Server
```
cloudstack-setup-management
systemctl status cloudstack-management
```

## Akses Dashboard Web (Menggunakan TailScale)
```
http://100.96.X.X/client
```

## Akses Dashboard Web (Menggunakan TailScale)
```
https://youtu.be/8Q_Ryfe1goc
```

# III. Membangun Infrastruktur Cloud Service
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
![image](https://github.com/user-attachments/assets/bfd5e1f1-b77e-4812-9423-2a4388995bcb)

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
![image](https://github.com/user-attachments/assets/206ee0ae-fdfa-4f98-800a-42224a734c85)


Langkah selanjutnya adalah Add resources. Di bagian ini, terdapat beberapa sub-step yang harus dilakukan.

a. Cluster: kumpulan host (server KVM) yang pakai jenis hypervisor sama. Semua host di dalam cluster ini akan berada di subnet yang sama dan mengakses shared storage yang sama pula.
```
Cluster name: kelompok5
```
![image](https://github.com/user-attachments/assets/23394d3c-7a8f-4c62-9e72-85f9e7f8c0dd)


b. IP address: mengkonfigurasi informasi dari host (assign alamat ip) agar host dapat bekerja di dalam CloudStack.
```
Host name: 192.168.18.250
Username: root
Authentication Method: Password | System SSH Key
Password: 
```
![image](https://github.com/user-attachments/assets/f0b56cbf-2d6d-48e3-86a7-47728d06c3c7)


c. Primary Storage: tempat nyimpan disk utama VM yang berjalan pada semua host yang ada di dalam cluster.
```
Name: primaryStorage_kelompok5
Scope: Zone
Protocol: nfs
Server: 192.168.18.250
Path: /export/primary
Provide: DefaultPrimary
```
![image](https://github.com/user-attachments/assets/340f1c1b-676b-4639-90f6-99eb92cd4745)


d. Secondary Storage: tempat untuk menyimpan template, ISO, dan Snapshot.
```
Provider: NFS
Name: secondaryStorage_kelompok5
Server: 192.168.18.250
Path: /export/secondary
```
![image](https://github.com/user-attachments/assets/c3a6b3bc-4a7b-47ff-80ff-ebb038637e4d)


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

6. [Bagian SSH key pairs, Advanced mode, dan Details bersifat opsional]

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
