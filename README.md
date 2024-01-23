# LVM sous Linux

## Introduction

<br/>

LVM, ou _logical Volume Manager_, est un outil de gestion de stockage sous Linux qui permet de creer plusieurs Volume Logique (LV) a partir de plusieurs disques physiques

Il offre une flexibilte en permettant de redimensionnement dynamique des LV, la gestion des snapshots etc...

<u>**Avantages de LVM :**</u>
<ul>

<li>Permet d'ajuster la taille des volumes Logiques sans avoir a demonter le system de fichier</li>

<li>Permet de regrouper plusieurs disques physiques en un seul volume Logique pour augumenter la capacite de stockage.</li>

<li>Permet de creer des snapshots</li>

</ul>

## Configuration

```sh
$ lsblk -o NAME,SIZE,FSTYPE
```
```
NAME    SIZE FSTYPE
loop0  55.5M squashfs
loop1  32.3M squashfs
loop2  70.4M squashfs
sda      40G
└─sda1   40G ext4
sdb      10M iso9660
sdc       5G
sdd       5G
sde       5G
```

#### 1. volume Pyhisque

##### Creation d'un PV (Physical Volume)

```bash
$ pvcreate /dev/sdc
```
```
Physical volume "/dev/sdc" successfully created.
```

maintenant je veux partitionner le disque `/dev/sdc` avec la commande `fdisk`

```bash
$ lsblk -o name,size | grep sdd
```

```
sdd       5G
├─sdd1  2.5G
└─sdd2  2.5G
```

créer deux volumes physiques supplémentaires à partir de ces deux partitions

```bash
$ pvcreate /dev/sdd1 /dev/sdd2
```
```
Physical volume "/dev/sdd1" successfully created.
Physical volume "/dev/sdd2" successfully created.
```

Lister les volumes physiques disponibles

**pvscan**

```bash
$ pvscan
```
```
PV /dev/sdc                       lvm2 [5.00 GiB]
PV /dev/sdd1                      lvm2 [2.50 GiB]
PV /dev/sdd2                      lvm2 [<2.50 GiB]
Total: 3 [<10.00 GiB] / in use: 0 [0   ] / in no VG: 3 [<10.00 GiB]
```

**pvs**

```bash
$ pvs
```
```
PV         VG Fmt  Attr PSize  PFree
/dev/sdc      lvm2 ---   5.00g  5.00g
/dev/sdd1     lvm2 ---   2.50g  2.50g
/dev/sdd2     lvm2 ---  <2.50g <2.50g
```

**pvdisplay**

```bash
$ pvdisplay
```

```
"/dev/sdc" is a new physical volume of "5.00 GiB"
--- NEW Physical volume ---
PV Name               /dev/sdc
VG Name               
PV Size               5.00 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               SzSkdD-xKYa-4y7P-teyU-481p-uiQ8-qieMJJ

"/dev/sdd1" is a new physical volume of "2.50 GiB"
--- NEW Physical volume ---
PV Name               /dev/sdd1
VG Name               
PV Size               2.50 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               553Iy4-JJ21-LfIw-udtO-j9Cd-7gFS-iXXFVS

"/dev/sdd2" is a new physical volume of "<2.50 GiB"
--- NEW Physical volume ---
PV Name               /dev/sdd2
VG Name               
PV Size               <2.50 GiB
Allocatable           NO
PE Size               0   
Total PE              0
Free PE               0
Allocated PE          0
PV UUID               bf7ghn-QkPm-EUdp-GdyW-shMG-5sMn-VhNtYB
```

##### La Suppression d'un PV (Physical Volume)

```bash
$ pvremove /dev/sdd2
```
```
Labels on physical volume "/dev/sdd2" successfully wiped.
```

Verifier

```bash
$ pvs
```
```
PV         VG Fmt  Attr PSize PFree
/dev/sdc      lvm2 ---  5.00g 5.00g
/dev/sdd1     lvm2 ---  2.50g 2.50g
```

#### 1. volume Groupe

##### 1.1. La Creation d'un volume Groupe (VG)

Les groupes de volumes sont créés à l'aide de la commande `vgcreate`. Le premier argument de vgcreate est le nom que vous souhaitez donner à ce groupe de volumes, et le reste est la liste des volumes physiques

```bash
$ vgcreate  vg1 /dev/sdc1 /dev/sdd1
```
```
Volume group "vg1" successfully created
```

Lister les Volumes Groupes disponibles

**vgs**

```bash
$ vgs
```
```
VG       #PV    #LV   #SN   Attr    VSize   VFree
vg1      2      0     0     wz--n-  7.49g   7.49g
```

**vgscan**

```bash
$ vgscan
```
```
Found volume group "vg1" using metadata type lvm2
```
**vgdisplay**

```bash
$ vgdisplay
```
```
  --- Volume group ---
  VG Name               : vg1
  System ID             
  Format                : lvm2
  Metadata Areas        : 2
  Metadata Sequence No  : 1
  VG Access             : read/write
  VG Status             : resizable
  MAX LV                : 0
  Cur LV                : 0
  Open LV               : 0
  Max PV                : 0
  Cur PV                : 2
  Act PV                : 2
  VG Size               : 7.49 GiB
  PE Size               : 4.00 MiB
  Total PE              : 1918
  Alloc PE / Size       : 0 / 0   
  Free  PE / Size       : 1918 / 7.49 GiB
  VG UUID               : LYVE9P-vY0G-OAW6-an8q-yfBx-rrB1-YU61m1
```

##### 1.2. Étendre un volume Groupe (VG)

Étendre un groupe de volumes signifie ajouter des volumes physiques supplémentaires à un groupe de volumes. Pour ce faire, la commande vgextend est utilisée. La syntaxe est simple :

```bash
$ pvcreate /dev/sde
Physical volume "/dev/sde" successfully created.
$ vgextend vg1 /dev/sde
Volume group "vg1" successfully extended
```

##### 1.3 Réduire un volume Groupe

Tout comme étendre un groupe de volumes signifie ajouter un autre volume physique, le réduire signifie supprimer un ou plusieurs volumes physiques.

```bash
$ pvmove /dev/sde
$ vgreduce vg1 /dev/sde
Removed "/dev/sde" from volume group "vg1"
```

##### 1.4 Supprimer un volume Groupe

```bash
$ vgremove vg1
```

#### Les Volumes Logiques

Les volumes logiques sont créés à l'aide de la commande `lvcreate`. La syntaxe couramment utilisée se présente comme suit :

```bash
lvcreate -L <size> -n <LV_Name> <VG_Name>
```
* l'option `-L` : La taille du LV <br/><br/>
* L'option `-n` : Le nom du LV

**Exemple**
```bash
$ lvcreate -L 5G -n lv1 vg1
```
```
 Logical volume "lv1" created.
```
Vous pouvez désormais l'utiliser comme n'importe quelle partition. Formatez-le avec ext4

```bash
$ mkfs.ext4 /dev/vg1/lv1
```

Montez-le
```sh
$ mkdir /mnt/lv1
$ mount /dev/vg1/lv1 /mnt/lv1
```

##### 1.1 Snapshots

Creer une Snapshot d'un Volume Logique a l'aide de commande `lvcreate`

```bash
$ lvcreate -L 5G -S -n "SnapShot01" /dev/vg1/lv1
```
Une nouvelle Volume Logique de 5Gib sera creer ---> `/dev/vg1/SnapShot01`
* L'option `-s` : Creer une Snapshot <br/><br/>
* L'option `-n` : Le nom du Snapshot

Monter

```bash
$ mount /dev/vg1/SnapShot01 /mnt/Snap-Of-lv1
```

##### 1.2 Etendre une LV

```bash
$ lvrextend -L 20G /dev/vg1/lv1

# Si le Volume Logique est formatees par ext* :
$ resize2fs /dev/vg1/lv1

# pour xfs
$ xfs_growfs /dev/vg1/lv1
```

##### 1.2 Reduire une LV

```bash
# demonter
$ umount /dev/vg1/lv1
$ lvreduce -L 2G /dev/vg1/lv1
```
