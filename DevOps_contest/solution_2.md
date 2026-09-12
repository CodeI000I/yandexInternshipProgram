# Разбор команд
## free --mega --lohi
```console
              total        used        free      shared  buff/cache   available
Mem:         131984      110995        4021        3804       16967       16110
Low:         131984      127962        4021
High:             0           0           0
Swap:             0           0           0
```

Вывод свободной памяти в мегабайтах и зоны LowMem/highMem, LowMemory - область физической памяти, которую ядро может постоянно отображать в виртуальном адресном пространстве. HighMemory - оставшайся память, не имеющая постоянного отображения (редкость для современных 64-битных компьютеров)

## df -B32
Выводит информацию о занятом и свободном месте файловых систем, но в блоках по 32 байта.
```console
Filesystem      32B-blocks       Used   Available Use% Mounted on
udev            2110399232          0  2110399232   0% /dev
/dev/md0       20076492288 6019910400 13035901312  32% /
tmpfs            335544320  301989888    33554432  90% /data/lake
tmpfs           2111751168          0  2111751168   0% /sys/fs/cgroup
/dev/md1        1974737280  391432704  1482169472  21% /var
/dev/loop5       536870912  335544320   201326592  63% /data
```

## df -i
Filesystem       Inodes   IUsed    IFree IUse% Mounted on
udev           16487494     472 16487022    1% /dev
/dev/md0       39854080 2256887 37597193    6% /
tmpfs                32      31        1   97% /data/lake
tmpfs          16498056      17 16498039    1% /sys/fs/cgroup
/dev/md1        3932160  116812  3815348    3% /var
/dev/loop5           32      30        2   94% /data

Выводит информацию о занятом и свободном месте файловых систем, но в inode (структура данных, хранящая всю метаинформацию о файле).
Общая информация о файловых системах:
- udev - виртуальная ФС, обслуживает /dev - устройства ядра создаются здесь автоматически
- tmpfs (temporary filesystem) - временная файловая система, которая хранится в оперативной памяти или свапе
- /dev/loop - файл-образ, подключенное как блочное устройство

Раз наша файловая система tmpfs в /data/lake, то нам нужно ее перемонтировать, увеличив размер. 335544320 блока по 32 байта = 10 737 418 240 байт = 10 GiB, при этом занято 301989888*32 = 9 GiB, то есть чтобы дописать еще два файла по 8 GiB, нужно 25 GiB Также нужно увеличить число доступных inode хотя бы на 1, чтобы вместилось два файла.
# Разбор вариантов ответа
- resize2fs - применяется для ext2/3/4
- growfs - старая утилита из solaric/sunos, для линукса аналог xfs_growfs
- mount -o remount, size=24576m /data - перемонтирует не ту директорию
- umount /data/lake; mount -t tmpfs -о size=24g,nr_inodes=48 none /data/lake - перемонтируем tmpfs, поэтому не сохранится успешно записанный файл
- mount -o remount, size=25165824k,nr_inodes=64 /data/lake && umount /data && mkfs.ext4 -N 64 /var/datawarehouse && mount -t ext4 -o rw /var/datawarehouse /data - зачем-то размонтируем data и создаем новую ext4 файловую систему
- mount -о remount, size=24g,nr_inodes=48 /data/lake - перемонтируем на 24 GiB, но нужно 25 GiB
- mount -о remount,w /data; mount -o remount,size=25165824k /data/lake - при перемонтировании не увеличили количество доступных inode, третий файл не запишется
- mount -o remount,nr_inodes=42 /data/lake; mount -o remount, size=26214600k /data/lake - увеличили объем до 25 GiB и количество доступных inode до 42, подходит
-  lvextend -L +10G /dev/loop5 && resize2fs /var/datawarehouse && mount -o remount,w /data && mount -o remount,nr_inodes=33,size=24576m /data/lake - зачем-то увеличиваем /data (и скорее всего с ошибкой, так как lvextend применяется для логических томов), зачем-то применяем resize2fs для /dev. что сразу приведет к ошибке
- mount -o remount, size=24578m /data/lake - увеличиваем до 24 GiB, что недостаточно, а также не увеличиваем inode

# Ответ: mount -o remount,nr_inodes=42 /data/lake; mount -o remount, size=26214600k /data/lake