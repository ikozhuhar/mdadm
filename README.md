# mdadm

## Собрать RAID0/1/5/10

##### Смотрим блочные устройства
```
$ sudo apt install lsscsi  
$ sudo lshw -short | grep disk
``` 
##### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```
$ sudo mdadm --zero-superblock --force /dev/sd{b,c}
```
##### Создать рейд
```
$ sudo mdadm --create --verbose /dev/md0 --level 1 --raid-device=2 /dev/sd{d,c}  
$ cat /proc/mdstat  
$ sudo mdadm -D /dev/md0
``` 
##### Создание файловой системы и монтирование
```
$ sudo mkfs.ext4 -F /dev/md0  
$ sudo mkdir /mnt/md0  
$ sudo mount /dev/md0 /mnt/md0
```

#### Сломать/починить RAID

##### Можно, искусственно “зафейлив” одно из блочных устройств
```
$ sudo mdadm /dev/md0 --fail /dev/sdd
```
##### Посмотрим, как это отразилось на RAID:
```
$ cat /proc/mdstat
$ sudo mdadm -D /dev/md0
```
##### Удалить “сломанный” диск из массива
```
$ sudo mdadm /dev/md0 --remove /dev/sdd
```
##### Представим, что мы вставили новый диск в сервер и теперь нам нужно добавить его в RAID. Диск должен пройти стадию rebuilding. Например, если это был RAID 1 (зеркало), то данные должны скопироваться на новый диск.
```
$ sudo mdadm /dev/md0 --add /dev/sdd  
$ sudo mdadm -D /dev/md0  
$ cat /proc/mdstat
```
