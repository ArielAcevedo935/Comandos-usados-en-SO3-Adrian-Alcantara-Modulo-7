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

practica 3

1.	Preparación del sistema
---------------------------------------------------------
hostnamectl set-hostname samba.inet
 sudo su
echo "10.0.0.92 OS3.inet" >> /etc/hosts 
dnf -y update
dnf install -y oracle-epel-release-el10
sudo dnf config-manager --set-enabled ol9_codeready_builder 
dnf update 
sudo dnf makecache 
dnf -y install wget tar gcc make python3-devel 

---------------------------------------------------------
2. Descarga e instalación de samba
mkdir -p /samba && cd /samba 
wget https://download.samba.org/pub/samba/stable/samba-4.16.2.tar.gz 
tar -zxvf samba-4.16.2.tar.gz && cd samba-4.16.2/ 

sudo dnf -y install docbook-style-xsl python3-markdown bison dbus-devel flex gdb \ 
gnutls-devel jansson-devel keyutils-libs-devel krb5-workstation \ 
libacl-devel libaio-devel libarchive-devel libattr-devel libblkid-devel \ 
libtasn1 libtasn1-tools libxml2-devel libxslt lmdb-devel openldap-devel \ 
pam-devel perl perl-ExtUtils-MakeMaker perl-Parse-Yapp popt-devel \ 
python3-cryptography python3-dns python3-gpg readline-devel rpcgen \ 
systemd-devel zlib-devel perl-JSON gpgme-devel screen 

./configure --prefix=/usr/local/samba --enable-fhs --with-ads --with-systemd 
make -j$(nproc)
make install 
export PATH=/usr/local/samba/bin:/usr/local/samba/sbin:$PATH 

nano ~/.bash_profile                                                        
PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH 
export PATH                                                                                    
nano ../.bash_profile                                                                                                                                                                                                                            
PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH                                      
export PATH                                                                                                
                                                                                                           
ERROR DE PROVISION                                                                                    
echo "/usr/local/samba/lib/samba" | sudo tee /etc/ld.so.conf.d/samba.conf                           
echo "/usr/local/samba/lib64" | sudo tee -a /etc/ld.so.conf.d/samba.conf                            
echo "/usr/local/samba/lib" | sudo tee -a /etc/ld.so.conf.d/samba.conf                             
                                                                                                 
sudo ldconfig 
                                                                                               
samba-tool domain provision --use-rfc2307 --interactive --option="interfaces= lo ens160" --option="bind interfaces only=yes" 
                                                                                                     
#Dominio: OS3.inet Add NFS and Samba setup instructions                                            
#Nombre de dominio: OS3                                                                            
#Nombre de controlador: dc                                                                         
#Backend DNS: SAMBA_INTERNAL                                                                     
#Contraseña: Teletubies123                                                                         
# inicia el servicio de samba con el siguiente comando                                           
samba                                                                                                    
                                                                                                           
#Pruebas de DS                                                                                        
ping google.com                                                                                        
ping OS3.inet                                                                                          
                                                                                                   
#Copiar el Archivo de Kerberos                                                                
cp /usr/local/samba/var/lib/samba/private/krb5.conf /etc/krb5.conf                                     
#Agregar reglas al firewall                                                     
firewall-cmd --permanent --add-service={ldap,ldaps,kerberos,dns} 
firewall-cmd --permanent --add-port={53,88,135,139,389,445,464,636,3268,3269}/tcp 
firewall-cmd --permanent --add-port={53,88,123,138,389,464}/udp 
firewall-cmd --reload 
samba-tool domain passwordsettings set --complexity=off
samba-tool user create lanegracubana 20251402
samba-tool group addmembers "Domain Admins" lanegracubana
samba-tool group listmembers "Domain Admins"
samba-tool domain info 10.0.0.92
samba-tool user list
samba-tool computer list

# Agregar Registro DN maquina cliente #########IP DEl SERVER ######### IP VM WINDOWS ################ 
                              samba-tool dns add 10.0.0.92 OS3.inet www A 10.0.0.90 -U lanegracubana 

#### en la VM de windows###: 
##En la configuracion de al tarjeta de Red cambiar el DNS de la maquina a que apunte al servidor de linux 
#haga ping al dominio (OS3.inet) el ping debe de responder con al Ip der servidor 

Presiona Win + R, escribe sysdm.cpl y presiona Enter. 
Ir a "Nombre del equipo" > "Cambiar": 

Selecciona Miembro de un dominio e ingresa: os3.inet 
Usuario: lanegracubana 
Contraseña: 20251251




##### RESPALDO DE CONFIGURACION  #####
1. Preparación del sistema
--------------------------
hostnamectl set-hostname dc1.inet (NO PUEDES TENER EL MISMO HOSTNAME QUE EL DOMINIO O DARÁ ERROR)
echo "192.168.0.16 dc1.inet" >> /etc/hosts   # poner la IP de su server

#Verificar si se cambió el hostname#
hostname
hostname -f

#Poner al día el sistema e instalar paquetes necesarios#
dnf -y update
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y
sudo subscription-manager repos --enable codeready-builder-for-rhel-8-$(arch)-rpms
dnf update
sudo dnf makecache
dnf -y install wget tar gcc make python3-devel

2. Descarga e instalación de Samba
----------------------------------
mkdir -p /samba && cd /samba
wget https://download.samba.org/pub/samba/stable/samba-4.16.2.tar.gz
tar -zxvf samba-4.16.2.tar.gz && cd samba-4.16.2/

sudo dnf -y install \
  docbook-style-xsl python3-markdown bison dbus-devel flex gdb \
  gnutls-devel jansson-devel keyutils-libs-devel krb5-workstation \
  libacl-devel libaio-devel libarchive-devel libattr-devel libblkid-devel \
  libtasn1 libtasn1-tools libxml2-devel libxslt lmdb-devel openldap-devel \
  pam-devel perl perl-ExtUtils-MakeMaker perl-Parse-Yapp popt-devel \
  python3-cryptography python3-dns python3-gpg readline-devel rpcgen \
  systemd-devel zlib-devel perl-JSON gpgme-devel screen

./configure --prefix=/usr/local/samba --enable-fhs --with-ads --with-systemd
make -j$(nproc)
make install

Poner este comando primero
export PATH=/usr/local/samba/bin:/usr/local/samba/sbin:$PATH

NOTA: Si te aparece contenido en el primer nano (~/.bash_profile) solo reemplaza el path que tenga ese archivo con el nuevo y listo, el otro archivo (../.bash_profile) no existe.
nano ~/.bash_profile 
        PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH
        export PATH  


nano ../.bash_profile
        PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH
        export PATH

NOTA: Asegurate que en --option="interfaces= lo ens160" tenga el verdadero nombre de tu interfaz de red, o sea que en vez de ens160 tenga el nombre de la interfaz que usas (si no sabes, pon ip a en la terminal y el nombre que vas a usar será la interfaz que tenga la ip de tu maquina).
samba-tool domain provision --use-rfc2307 --interactive --option="interfaces= lo ens160" --option="bind interfaces only=yes"

#Dominio: OS3.inet  
#Nombre de dominio: OS3 
#Nombre de controlador: dc  
#Backend DNS: SAMBA_INTERNAL  
#Contraseña: [CONTRASEÑA] Te va a pedir una contraseña de minimo 7 caracteres y que cumpla con los requerimentos de complejidad 

# inicia el servicio de samba con el siguiente comando
samba
NOTA: Si de casualidad se reinicia el sistema, no va a iniciar samba con ese comando, así que tendrás que usar "sudo /usr/local/samba/sbin/samba" para ejecutarlo, que es su ruta absoluta.

#Pruebas de DNS
ping google.com
ping OS3.inet
ping dc1.inet (si no funciona el OS3.inet en Linux, prueba tu hostname, pero debe funcionar como OS3.inet en Windows) 

#Copiar el Archivo de Kerberos
cp /usr/local/samba/var/lib/samba/private/krb5.conf /etc/krb5.conf

#Agregar reglas al firewall
firewall-cmd --permanent --add-service={ldap,ldaps,kerberos,dns}
firewall-cmd --permanent --add-port={53,88,135,139,389,445,464,636,3268,3269}/tcp
firewall-cmd --permanent --add-port={53,88,123,138,389,464}/udp
firewall-cmd --reload

# Configura el DNS para que sea solo la IP del propio servidor
sudo nmcli con mod ens160 ipv4.dns 192.168.0.16

# Le dice que ignore cualquier DNS automático (como el de DHCP)
sudo nmcli con mod ens160 ipv4.ignore-auto-dns yes

# Aplica los cambios
sudo nmcli con up ens160

NOTA: Necesitas cambiarle el DNS a tu maquina SERVER para que las CLIENTE que estén en el dominio puedan acceder a internet 



# Agregar Registro DN maquina cliente
###################IP DEl SERVER ################ IP VM WINDOWS ################                
samba-tool dns add 192.168.0.16 OS3.inet www A 192.168.0.11 -U Administrator

#### en la VM de windows###:

## En la configuracion de al tarjeta de Red cambiar el DNS de la maquina a que apunte al servidor de Linux ##
Presione Win + R y escriba ncpa.cpl, luego le da click derecho al adaptador de red, propiedades y seleccione "Protocolo de Internet versión 4 (TCP/IPv4)" y propiedades.
Dejas la IP como esté y cambias el DNS a manual, y en servidor dns preferido pones la ip de tu SERVER y en servidor alternativo lo dejas vacío o va a fallar.
Luego abres CMD y escribes: ipconfig /flushdns

#haga ping al dominio (OS3.inet) el ping debe de responder con al Ip del servidor
ping OS3.inet
ping www.OS3.inet

Presiona Win + R, escribe sysdm.cpl y presiona Enter.
Ir a "Nombre del equipo" > "Cambiar":

Selecciona Miembro de un dominio e ingresa: os3.inet
Le va a pedir un reinicio en la maquina de windows, y luego de eso, le das a other users y pones:

Usuario: OS3\Administrator (NOTA: Si pones solo Administrator, windows lo va a tomar como un usuario local y te va a decir contraseña incorrecta aunque esté bien)
Contraseña: [Contraseña puesta]

#Agregar usuario lanegracubana

sudo /usr/local/samba/bin/samba-tool user create lanegracubana “20251402” --given-name=”La Negra” --surname=”Cubana” --mail=lanegracubana@so3.inet

Si sigue sin funcionar el inicio de sesión pero todo está correcto, revisa el firewall y el estado de SELinux:

# Rango estándar de puertos dinámicos para RPC
sudo firewall-cmd --permanent --add-port=49152-65535/tcp

# Recarga el firewall para aplicar los cambios
sudo firewall-cmd --reload

# Pone SELinux en modo permisivo temporalmente
sudo setenforce 0

# Para verificar el estado (debería decir "Permissive")
sestatus

INSTALACIÓN Y CONFIGURACIÓN DE SAMBA COMO CONTROLADOR DE DOMINIO ACTIVE DIRECTORY (OS3.INET)

1. Preparación del sistema
--------------------------
hostnamectl set-hostname OS3.inet
echo "192.168.100.73 OS3.inet" >> /etc/hosts   # poner la IP de su server
dnf -y update
dnf -y install epel-release
### En rhel usar : (sudo dnf config-manager --set-enabled powertools) 
### En Rocky linux usar: (sudo dnf config-manager --set-enabled crb) 
### En oracle-linux usar:(sudo dnf config-manager --set-enabled ol9_codeready_builder)
dnf update
sudo dnf makecache
dnf -y install wget tar gcc make python3-devel

2. Descarga e instalación de Samba
----------------------------------
mkdir -p /samba && cd /samba
wget https://download.samba.org/pub/samba/stable/samba-4.16.2.tar.gz
tar -zxvf samba-4.16.2.tar.gz && cd samba-4.16.2/

sudo dnf -y install \
  docbook-style-xsl python3-markdown bison dbus-devel flex gdb \
  gnutls-devel jansson-devel keyutils-libs-devel krb5-workstation \
  libacl-devel libaio-devel libarchive-devel libattr-devel libblkid-devel \
  libtasn1 libtasn1-tools libxml2-devel libxslt lmdb-devel openldap-devel \
  pam-devel perl perl-ExtUtils-MakeMaker perl-Parse-Yapp popt-devel \
  python3-cryptography python3-dns python3-gpg readline-devel rpcgen \
  systemd-devel zlib-devel perl-JSON gpgme-devel screen

./configure --prefix=/usr/local/samba --enable-fhs --with-ads --with-systemd
make -j$(nproc)
make install

export PATH=/usr/local/samba/bin:/usr/local/samba/sbin:$PATH

nano ~/.bash_profile 
        PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH
        export PATH  
nano ../.bash_profile
        PATH=$PATH:$HOME/bin:/usr/local/samba/bin:/usr/local/samba/sbin:$PATH
        export PATH

samba-tool domain provision --use-rfc2307 --interactive --option="interfaces= lo ens160" --option="bind interfaces only=yes"

#Dominio: OS3.inet  
#Nombre de dominio: OS3 
#Nombre de controlador: dc  
#Backend DNS: SAMBA_INTERNAL  
#Contraseña: Teletubies123

# inicia el servicio de samba con el siguiente comando
samba

#Pruebas de DS
ping google.com
ping OS3.inet

#Copiar el Archivo de Kerberos
cp /usr/local/samba/var/lib/samba/private/krb5.conf /etc/krb5.conf

#Agregar reglas al firewall
firewall-cmd --permanent --add-service={ldap,ldaps,kerberos,dns}
firewall-cmd --permanent --add-port={53,88,135,139,389,445,464,636,3268,3269}/tcp
firewall-cmd --permanent --add-port={53,88,123,138,389,464}/udp
firewall-cmd --reload

# Agregar Registro DN maquina cliente
###################IP DEl SERVER ################ IP VM WINDOWS ################                
samba-tool dns add 192.168.100.73 OS3.inet www A 192.168.100.74 -U Administrator

#### en la VM de windows###:

## En la configuracion de al tarjeta de Red cambiar el DNS de la maquina a que apunte al servidor de linux
#haga ping al dominio (OS3.inet) el ping debe de responder con al Ip der servidor

Presiona Win + R, escribe sysdm.cpl y presiona Enter.
Ir a "Nombre del equipo" > "Cambiar":

Selecciona Miembro de un dominio e ingresa: os3.inet

Usuario: Administrator
Contraseña: Teletubies123













