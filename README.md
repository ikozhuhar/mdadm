### Работа с mdadm

#### <a name='toc'>Содержание</a>
1. [Смотрим блочные устройства](#look_blk)
2. [Собрать RAID-массив](#create_raid)
3. [Создание конфигурационного файла mdadm.conf](#conf_file)
4. [Сломать/починить RAID-массив](#break_fix)
5. [Удалить RAID-массив](#delete_raid)

#### 1. [[⬆]](#toc) <a name='look_blk'>Смотрим блочные устройства</a>
```
sudo lsblk 
sudo lshw -short | grep disk
```
![image](https://github.com/user-attachments/assets/32556c3a-4117-4db8-9531-6da890478f15)

##### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```
sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
```
![image](https://github.com/user-attachments/assets/536cde52-6250-4874-b10b-61b4c3ecaaf4)

##### 2. [[⬆]](#toc) <a name='create_raid'>Собрать RAID0/1/5/10</a>
```
sudo mdadm --create --verbose /dev/md0 --level 6 --raid-device=5 /dev/sd{b,c,d,e,f}  
```
![image](https://github.com/user-attachments/assets/c77a01b0-438a-47ff-83ff-d0014b6e714f)

##### Проверяем, что RAID собрался нормально:
```
cat /proc/mdstat  
sudo mdadm -D /dev/md0
```
![image](https://github.com/user-attachments/assets/ae8bf07a-0f96-42b7-b7ec-3a594c1f25f6)

##### 3. [[⬆]](#toc) <a name='conf_file'>Создание конфигурационного файла mdadm.conf</a>

##### Для того, чтобы быть уверенным, что ОС запомнила, какой RAID массив требуется создать и какие компоненты в него входят, создадим файл mdadm.conf. Убедимся, что информация верна:
```
sudo mdadm --detail --scan --verbose
```
![image](https://github.com/user-attachments/assets/844377e9-83b6-46db-861c-2854f6a14fb5)

##### А затем в две команды создадим файл mdadm.conf
```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```
![image](https://github.com/user-attachments/assets/60c52ddb-f31a-4d60-a43d-6cea30ae8a7e)


















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
