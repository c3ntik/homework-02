# homework-02
# **Дисковая подсистема**

## **Homework**

- Работа с mdadm.
- Добавить в Vagrantfile еще дисков.
- Собрать R0/R5/R10 на выбор.
- Сломать/починить raid.
- Прописать собранный рейд в конфигурационный файл, чтобы рейд собирался при загрузке.
- Создать GPT раздел и 5 партиций.
- Vagrantfile, который сразу собирает систему с подключенным рейдом.

#vagrant up


**Далее проверим поднялась ли машина:**

#vagrant status

**Все работает корректно, можно подлючиться к ней:**

#vagrant ssh

## **2.Работа с mdadm, сборка RAID**

## **4.Прописываем собранный рейд в конфигурационный файл, чтобы рейд собирался при загрузке.**

**Для того, чтобы быть уверенным что ОС запомнила какой RAID массив требуется создать и какие компоненты в него входят создадим файл mdadm.conf.
Сначала убедимся, что информация верна:**

sudo mdadm --detail --scan --verbose

**Теперь создадим mdadm.conf:**

echo "DEVICE partitions" > /etc/mdadm.conf

mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm.conf


**Созданный файл:**

cat /etc/mdadm.conf

DEVICE partitions

ARRAY /dev/md0 level=raid5 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=b4d3da63:dc6dd4d1:493d0a01:3bded655

## **5.Создание GPT раздела и 5 партиций**

**Создаем раздел GPT на RAID:**

parted -s /dev/md0 mklabel gpt


**Создаем партиции:**

parted /dev/md0 mkpart primary ext4 0% 20%

parted /dev/md0 mkpart primary ext4 20% 40%

parted /dev/md0 mkpart primary ext4 40% 60%

parted /dev/md0 mkpart primary ext4 60% 80%

parted /dev/md0 mkpart primary ext4 80% 100%


**Создаем на этих партициях фс:**

for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done


**Монтируем их по каталогам:**

mkdir -p /raid/part{1,2,3,4,5}

for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
for i in $(seq 1 5); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /raid/part$i ext4 defaults 0 0 >> /etc/fstab; done
