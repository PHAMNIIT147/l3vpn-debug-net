This is guidance about configuration for working telnet *Guest* from *Host* run SAI SDK
# QEMU/KVM - Virt-Manager | Folder sharing
  [Drive](https://drive.google.com/file/d/1ocAOcnqC9UAYE0dvxP54FlAyS0yw0T_1/view?usp=sharing)
  
# Ubuntu Server 
Install NFS Kernel
```
sudo apt update
sudo apt install nfs-kernel-server
```
## Create an NFS Export Directory
```
sudo mkdir -p /mnt/nfs_share
sudo chown -R nobody:nogroup /mnt/nfs_share/
sudo chmod 777 /mnt/nfs_share/
```
## Grant NFS Share Access to Client Systems
```
sudo nano /etc/exports
/mnt/nfs_share  192.168.122.0/24(rw,sync,no_subtree_check)
/mnt/nfs_share  client_IP_1 (re,sync,no_subtree_check)
```
## Export the NFS Share directory
```
 sudo exportfs -a
 sudo systemctl restart nfs-kernel-server
```
## Allow NFS Access through the Firewall
```
sudo ufw allow from 192.168.122.0/24 to any port nfs
sudo ufw enable
sudo ufw status
```
# Install the NFS Client on the Client Systems
## Install the NFS-Common Package
```
sudo apt udpate
sudo apt install nfs-common/mn
```
## Create a NFS Mount Point on Client
```
sudo mkdir -p /mnt/nfs_clientshare
sudo mount 172.16.2.167:/mnt/nfs_share  /mnt/nfs_clientshare
df
172.16.2.167:/mnt/share ... ... ... 45% /mnt/clientshare
```
