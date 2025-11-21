# Comandos-usados-en-SO3-Adrian-Alcantara-Modulo-7
video 1

sudo dnf install -y nfs-utils

sudo systemctl enable --now nfs-server 

sudo mkdir -p /nfs/OS3

cd /nfs/OS3

touch adrian{1..100}.txt

sudo chmod -R 755 /nfs/OS3

udo chown -R nobody:nobody /nfs/OS3

echo "/nfs/OS CLIENTE_IP(rw,sync,no_root_squash)" | sudo tee -a /etc/exports

showmount -e

maquina cliente

sudo mkdir -p /mnt/nfs_OS3

sudo mount SERVER_IP:/nfs/OS /mnt/nfs_OS

ls /mnt/nfs_OS

mount | grep nfs


video 2

sudo dnf install -y samba* 
sudo systemctl enable --now smb nmb
sudo systemctl start nmb smb 
sudo firewall-cmd --add-service=samba --permanent
sudo firewall-cmd –reload
sudo mkdir -p /samba/compartir
cd /samba/compartir
touch Adrian{1..100}.txt
sudo chmod -R 777 /samba/compartir
sudo chown -R nobody:nobody /samba/compartir
sudo setsebool -P samba_export_all_rw on
sudo useradd zamba
sudo smbpasswd -a zamba

sudo nano /etc/samba/smb.conf
(dentro de sudo nano /etc/samba/smb.conf, agregar al fondo)
[compartir]
   path = /samba/compartir
   browseable = yes
   writeable = yes
   valid users = zamba
   create mask = 0777
   directory mask = 0777
   guest ok = yes


sudo systemctl restart smb nmb

cat /samba/compartir/Adrian99.txt











