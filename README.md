### Работа с mdadm

#### <a name='toc'>Содержание</a>
1. [Смотрим блочные устройства](#look_blk)
2. [Собрать RAID-массив](#create_raid)
3. [Сломать/починить RAID](#break_fix)
4. [Удалить RAID-массив](#delete_raid)

#### 1. [[⬆]](#toc) <a name='look_blk'>Смотрим блочные устройства</a>
```
sudo lsblk 
sudo lshw -short | grep disk
```
![image](https://github.com/user-attachments/assets/32556c3a-4117-4db8-9531-6da890478f15)

##### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```
mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g}
```
![image](https://github.com/user-attachments/assets/536cde52-6250-4874-b10b-61b4c3ecaaf4)

##### 2. [[⬆]](#toc) <a name='create_raid'>Собрать RAID0/1/5/10</a>
```
$ sudo mdadm --create --verbose /dev/md0 --level 1 --raid-device=2 /dev/sd{d,c}  
$ cat /proc/mdstat  
$ sudo mdadm -D /dev/md0
``` 
###### Создание файловой системы и монтирование
```php
$ sudo mkfs.ext4 -F /dev/md0  
$ sudo mkdir /mnt/md0  
$ sudo mount /dev/md0 /mnt/md0
```

#### Сломать/починить RAID

###### Можно, искусственно “зафейлить” одно из блочных устройств
```
$ sudo mdadm /dev/md0 --fail /dev/sdd
```
###### Посмотрим, как это отразилось на RAID
```
$ cat /proc/mdstat
$ sudo mdadm -D /dev/md0
```
###### Удалить “сломанный” диск из массива
```php
$ sudo mdadm /dev/md0 --remove /dev/sdd
```
###### Представим, что мы вставили новый диск в сервер и теперь нам нужно добавить его в RAID. Диск должен пройти стадию rebuilding. Например, если это был RAID 1 (зеркало), то данные должны скопироваться на новый диск
```php
$ sudo mdadm /dev/md0 --add /dev/sdd  
$ sudo mdadm -D /dev/md0  
$ cat /proc/mdstat
```
#### Как удалить RAID-массив

###### Отмонтировать RAID
```php
$ sudo umount /mnt/md0
```

###### Останавливаем RAID-массив
```php
$ sudo mdadm -S /dev/md0
```

###### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```php
$ sudo mdadm --zero-superblock --force /dev/sd{b,c}  
или  
$ sudo mdadm --zero-superblock /dev/sda1  
$ sudo mdadm --zero-superblock /dev/sdb1  
$ sudo mdadm --zero-superblock /dev/sdc1  
```
###### В завершении, убираем ссылки на разобранный RAID-массив в /etc/mdadm/mdadm.conf (в Debian) или в /etc/mdadm.conf (в CentOS), если они делались там ранее
