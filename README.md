# install-debian

Creer la clé USB d'install a partir de l'iso 
(#  usb-creator-gtk  ## marche pas sur une image debian ?)

    dd bs=4M if=~/Downloads/debian-live-9.4.0-amd64-gnome.iso of=/dev/sdb status=progress

BIOS : desactiver le secure boot UEFI ; en legacy, on voit passer le "boot on usb"
Si le BIOS ne veut pas booter sur la clef, vérifier qu'un boot USB est bien autorisé (désactivé parfois)

Installer une Debian avec les options par défaut, et un utilisateur de départ "bob", dans /home
(on ajoutera le "vrai" user plus tard ; bob permet de bosser tranquillement jusqu'au cryptage)
Faire les partitions :

    sda1    swap     mem+10%
    sda5    /        40G
    sda6    /home  le reste ; ne pas a crypter, on le fait plus tard

    git clone ...

    apt-get update -y
    apt-get install -y  vim openssh-client openssh-server git emacs ecryptfs-utils cryptsetup apt-file synaptic manpages

ATTENTION : vérifier le que /home est bien sur sda6, par ex avec:

```
fdisk -l
cat /etc/fstab

umount /home
# Ici, on met la passphrase que seuls les admins connaissent. L'utilisateur choisira la sienne plus tard (cf. luksAddKey ci-dessous). 
cryptsetup luksFormat /dev/sda6 -c aes -s 256 -h sha256
cryptsetup luksOpen /dev/sda6 vault
mke2fs -t ext4 -j /dev/mapper/vault -L vault
mount /dev/mapper/vault /home


# * Finir la config : 
# ** dans /etc/fstab, remplacer la ligne /home par : 
/dev/mapper/vault /home         ext4    defaults        0       2
# ** dans /etc/crypttab : 
echo "vault             /dev/sda6        none        luks" >> /etc/crypttab
```

#### Crypter la swap : 
 A FAIRE 


# Créer le compte utilisateur 

```
uidNumber=1001
givenName=jd

adduser --home /home/$givenName --shell /bin/bash --uid $uidNumber $givenName
adduser $givenName sudo

# pour vérifier : 
id $givenName

sudo -i OU su  # suivant qu'on a mis $givenName les droits de sudo
deluser --remove-home bob

    sed -e "s/#.*$//" extra_packages_debian |xargs echo 
    apt-get install -y #[copier-coller tous les packages de la ligne prédécente]
```

#### Tests performances : 

```
cd /root
hdparm -Tt /dev/sda6  # une première fois à blanc
hdparm -Tt /dev/sda6 | tee -a PERFORMANCES.txt
glxgears             | tee -a PERFORMANCES.txt
```

## Suite de l'install des packages

```
apt-get autoremove


##NVIDIA :: https://wiki.debian.org/fr/NvidiaGraphicsDrivers#Debian_9_.2BAKs_Stretch_.2BALs-
# 1: ajouter contrib et non-free dans /etc/apt/sources.list
deb http://httpredir.debian.org/debian/ stretch main contrib non-free
# 2: 
apt-get update
apt-get install  nvidia-driver
Mmmmmh, ca marche pas non plus sur le precision 5520 ; pour annuler le massacre :
apt-get install --reinstall xserver-xorg 
apt-get install --reinstall xserver-xorg-nouveau


# Imprimante et scanner HP : 

    apt install hplip
    bash Downloads/hplip-3.22.4.run

# En cas de pb, pour la reconfigurer : 

    hp-setup
```

Pour le scanner (HP Envy 6032e), telecharger le plugin non-free associé
 https://www.openprinting.org/download/printdriver/auxfiles/HP/plugins/
(cf. https://answers.launchpad.net/hplip/+question/693488) 

# grub et windows

Depuis mi-2023, grub ne détecte plus automatiquement les partitions windows. Il faut décommenter la ligne 

    GRUB_DISABLE_OS_PROBER=false

dans le fichier `/etc/default/grub`, puis relancer 

    sudo update-grub
    
