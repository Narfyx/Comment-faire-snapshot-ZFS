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

  Pour restaurer les données à partir de la sauvegarde :
  
  1. Connecter et monter le disque dur externe :
    - Montez le disque dur externe de la même manière que décrit précédemment.
  2. Restaurer le snapshot depuis le fichier de sauvegarde :
    - Utilisez la commande zfs receive pour recevoir le flux de données du fichier de sauvegarde.
    ```bash

    gunzip -c /mnt/pve/BACKUP/backup2024-07-16.gz | zfs receive -vn datapool@restored_snapshot
    
    ```

  4. Utiliser le snapshot restauré :
    - Vous pouvez alors utiliser le snapshot restauré comme vous le feriez avec n'importe quel autre snapshot.


## Remarque
  Assurez-vous d'avoir suffisamment d'espace sur le disque dur externe pour stocker la sauvegarde. Vous pouvez également utiliser des outils de gestion de sauvegarde comme rsync pour synchroniser les données sur le disque dur externe de manière incrémentielle.
