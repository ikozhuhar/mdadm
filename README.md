### Работа с mdadm

### <a name='toc'>Содержание</a>
1. [Смотрим блочные устройства](#look_blk)
2. [Собрать RAID-массив](#create_raid)
3. [Создание конфигурационного файла mdadm.conf](#conf_file)
4. [Создание файловой системы и монтирование](#create_fs)
5. [Сломать/починить RAID-массив](#break_fix)
6. [Удалить RAID-массив](#delete_raid)
7. [Создать GPT раздел, пять партиций и смонтировать их на диск](#create_gpt)

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

##### 4. [[⬆]](#toc) <a name='create_fs'>Создание файловой системы и монтирование</a>
```
sudo mkfs.ext4 -F /dev/md0  
```
![image](https://github.com/user-attachments/assets/e2bbb149-daee-4000-b51e-180c9d850ec2)

##### Смотрим результат
```
sudo mkdir /mnt/md0  
sudo mount /dev/md0 /mnt/md0
sudo lsblk -o +UUID,NAME,FSTYPE
```
![image](https://github.com/user-attachments/assets/01a17b1a-9e41-4346-83e6-70827d06fc01)


#### 5. [[⬆]](#toc) <a name='create_fs'>Сломать/починить RAID</a>

##### Можно, искусственно “зафейлить” одно из блочных устройств
```
sudo mdadm /dev/md0 --fail /dev/sdd
```
![image](https://github.com/user-attachments/assets/a6368df7-cb6c-4ea9-a5ee-f85a2af87e20)

##### Посмотрим, как это отразилось на RAID
```
cat /proc/mdstat
sudo mdadm -D /dev/md0
```
![image](https://github.com/user-attachments/assets/12036513-737d-48e2-8fe7-25d6451a9bf0)

##### Удалить “сломанный” диск из массива
```
sudo mdadm /dev/md0 --remove /dev/sdd
```
![image](https://github.com/user-attachments/assets/c471e0ef-2f2c-434e-8743-ad353be97f52)

##### Представим, что мы вставили новый диск в сервер и теперь нам нужно добавить его в RAID. Диск должен пройти стадию rebuilding. Например, если это был RAID 1 (зеркало), то данные должны скопироваться на новый диск
```
sudo mdadm /dev/md0 --add /dev/sdd  
cat /proc/mdstat
sudo mdadm -D /dev/md0  
```
![image](https://github.com/user-attachments/assets/76f52687-0309-4159-963a-ae1f7bac5997)


#### 6. [[⬆]](#toc) <a name='delete_raid'>Как удалить RAID-массив</a>

##### Отмонтировать RAID
```
sudo umount /mnt/md0
```

##### Останавливаем RAID-массив
```
sudo mdadm -S /dev/md0
```

##### Чтобы в дальнейшем система не пыталась автоматически собрать массив (например после перезагрузки) из дисков, которые участвовали в RAID-массиве, необходимо очистить супер-блоки на этих дисках
```
sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
```
![image](https://github.com/user-attachments/assets/c494e347-8cf2-4614-a4e9-44e8d6bc9973)


##### В завершении, убираем ссылки на разобранный RAID-массив в /etc/mdadm/mdadm.conf (в Debian) или в /etc/mdadm.conf (в CentOS), если они делались там ранее

#### 7. [[⬆]](#toc) <a name='create_gpt'>Создать GPT раздел, пять партиций и смонтировать их на диск</a>

##### Создаем раздел GPT и партиции на RAID
```
parted -s /dev/md0 mklabel gpt
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%
```
![image](https://github.com/user-attachments/assets/dc3fadcb-7a04-4e95-9ef7-f6b2bdecec93)

##### Далее можно создать на этих партициях ФС





