# Comment-faire-snapshot-ZFS


## Étapes pour créer un instantané et le sauvegarder sur un disque dur externe

1. Créer un instantané ZFS :
    - Utilisez la commande zfs snapshot pour créer un instantané de votre pool de données.
    ```bash
    zfs snapshot -r datapool@snapshot_name
  
    ```
    Par exemple :
    ```bash
    zfs snapshot -r datapool@backup2024-07-16
    ```
    - Vérifiez les instantanés pour vous assurer qu'ils ont été créés correctement.
    ```bash
    zfs list -t snapshot
    ```

2. Connecter et préparer le disque dur externe :
    - Connectez votre disque dur externe à votre serveur Proxmox.
    - Trouvez l'identifiant du disque (par exemple, /dev/sde) en utilisant lsblk ou fdisk -l.
    ```bash
    lsblk
    ```
    - Créez un système de fichiers sur le disque dur externe si nécessaire (par exemple, ext4).
    ```bash
    mkfs.ext4 /dev/sde1
    ```
    - Montez le disque dur externe.
    ```bash
    mkdir /mnt/external
    mount /dev/sde1 /mnt/external
    ```

3. Envoyer l'instantané vers le disque dur externe :
    - Utilisez la commande zfs send pour envoyer l'instantané vers le disque dur externe. Vous pouvez combiner cela avec zfs receive si vous avez un pool ZFS sur le disque dur externe, ou simplement envoyer le flux de données vers un fichier.
    - Pour envoyer l'instantané vers un fichier sur le disque dur externe :
    ```bash
    zfs send -R datapool@snapshot_name | gzip > /mnt/external/backupfile.gz
    ```
    Par exemple :
    ```bash
    zfs send -R datapool@backup2024-07-16 | gzip > /mnt/external/backup2024-07-16.gz
    ```

4. Vérifier la sauvegarde :
    - Vérifiez que le fichier de sauvegarde a été créé correctement sur le disque dur externe.
    ```bash
    ls -lh /mnt/external
    ```

5. Démonter le disque dur externe :
    - Une fois la sauvegarde terminée, démontez le disque dur externe en toute sécurité.
    ```bash
    umount /mnt/external
    ```


## Restaurer les données depuis la sauvegarde

1. Vérifier si il y a déjà des snapshots:
    ```bash
    zfs list
    ```
2. Si vous avez besoin de supprimer des snapshots:
    ```bash
    zfs destroy datapool@snapshot_name
    ```
    Par exemple :
    ```bash
    zfs destroy -r datapool@backup2024-07-16
    ```
1. Connecter et monter le disque dur externe :
    - Montez le disque dur externe de la même manière que décrit précédemment.
2. Restaurer le snapshot depuis le fichier de sauvegarde :
    - Utilisez la commande zfs receive pour recevoir le flux de données du fichier de sauvegarde.
    ```gunzip -c /mnt/pve/BACKUP/backup2024-07-16.gz | zfs receive -vn datapool@restored_snapshot```
4. Utiliser le snapshot restauré :
    - Vous pouvez alors utiliser le snapshot restauré comme vous le feriez avec n'importe quel autre snapshot.

## Restaurer les données quand la partition ZFS contient l'OS hôte
1. Booter à partir d'un live cd (j'utilise personnellement manjaro_kde).
2. Sur la machine physique taper la commande suivante pour pouvoir avoir un accès à distance.
   ```bash
   sudo systemctl start sshd.service
   ```
   le mot de passe par défaut sudo est ```manjaro```
3. Il faut maintenant repérer le nom des partitions sur lequelles nous allons intervenir, grâce à la commande ```lsblk```.
   voici un extrait de la commande (le nom des disques et des partitions pourront être différente veiller à adapter les futures commande pour votre cas):
   ```bash
   sde           8:64   0 238,5G  0 disk 
    └─sde1        8:65   0 238,5G  0 part 
    sdf           8:80   1 119,3G  0 disk 
    ├─sdf1        8:81   1 119,2G  0 part 
    │ └─ventoy  254:0    0   3,3G  1 dm   /run/miso/bootmnt
    └─sdf2        8:82   1    32M  0 part 
    nvme0n1     259:0    0 238,5G  0 disk 
    ├─nvme0n1p1 259:1    0  1007K  0 part 
    ├─nvme0n1p2 259:2    0     1G  0 part 
    ├─nvme0n1p3 259:3    0    63G  0 part 
    ├─nvme0n1p4 259:4    0  93,4G  0 part 
    └─nvme0n1p5 259:5    0  81,1G  0 part 
    nvme1n1     259:6    0 238,5G  0 disk 
    ├─nvme1n1p1 259:7    0  1007K  0 part 
    ├─nvme1n1p2 259:8    0     1G  0 part 
    ├─nvme1n1p3 259:9    0    63G  0 part 
    ├─nvme1n1p4 259:10   0  93,4G  0 part 
    └─nvme1n1p5 259:11   0  81,1G  0 part 
   ```
   Pour ma part je vais intervenir sur la partition ```nmve0n1p3``` & ```nmve1n1p3``` car ma configuration ZFS est en mirroring (RAID 1).
   La partitions ```sde1``` sera quant à elle la partition qui contient ma backup snapshot.
### Importer le pool ZFS
1. Assurez-vous que les modules ZFS sont chargés :
   ```bash
    sudo modprobe zfs
   ```
   - si vous avez des difficulté avec la commande précédente, essayé ceci pour vous aider:
     ```bash
     sudo pacman-mirrors --fasttrack
     sudo pacman -Syy; sudo pacman -Syu
     sudo pacman -S linux$(uname -r | grep -oP '^\d+\.\d+' | tr -d .)-zfs # cela va choisir automatiquement la bonne version pour votre noyau
     sudo pacman -S openssl
     ```
   
   
## Remarque
  Assurez-vous d'avoir suffisamment d'espace sur le disque dur externe pour stocker la sauvegarde. Vous pouvez également utiliser des outils de gestion de sauvegarde comme rsync pour synchroniser les données sur le disque dur externe de manière incrémentielle.
