## Работа с mdadm

#### Собрать RAID0/1/5/10

###### Смотрим блочные устройства
```php
$ sudo apt install lsscsi  
$ sudo lshw -short | grep disk
``` 
###### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```php
$ sudo mdadm --zero-superblock --force /dev/sd{b,c}
```
###### Создать рейд
```php
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
###### Посмотрим, как это отразилось на RAID:
```
$ cat /proc/mdstat
$ sudo mdadm -D /dev/md0
```
###### Удалить “сломанный” диск из массива
```php
$ sudo mdadm /dev/md0 --remove /dev/sdd
```
###### Представим, что мы вставили новый диск в сервер и теперь нам нужно добавить его в RAID. Диск должен пройти стадию rebuilding. Например, если это был RAID 1 (зеркало), то данные должны скопироваться на новый диск.
```php
$ sudo mdadm /dev/md0 --add /dev/sdd  
$ sudo mdadm -D /dev/md0  
$ cat /proc/mdstat
```
#### Как удалить RAID-массив

###### Отмонтировать RAID
```php
$ sudo umount /mount/md0
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
###### В завершении, убираем ссылки на разобранный RAID-массив в /etc/mdadm/mdadm.conf (в Debian) или в /etc/mdadm.conf (в CentOS), если они делались там ранее.
