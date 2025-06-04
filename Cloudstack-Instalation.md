# Instalasi Dan Konfigurasi Apache Cloudstack Private Cloud

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

### Menambahkan Command Dibawah mysqld.cnf

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
