# 2. Не получилось записать файлы

У тебя есть сервер, он отлично работает и используется другими людьми. Один из твоих коллег попытался записать в директорию /data/lake три файла размером по 8GiB, но не смог этого сделать. Успешно записался только один файл, а два других не записались с какой-то ошибкой. Текст ошибки он не сохранил, но попросил тебя разобраться с проблемой. При этом он попросил не трогать успешно записанный файл, так как с ним на сервере уже работает важный и очень долгий аналитический процесс. Коллега попросил сообщить ему, когда можно будет дозаписать оставшиеся два файла.

Вот что тебе известно о состоянии сервера:
```console
# free --mega --lohi
              total        used        free      shared  buff/cache   available
Mem:         131984      110995        4021        3804       16967       16110
Low:         131984      127962        4021
High:             0           0           0
Swap:             0           0           0

# df -B32
Filesystem      32B-blocks       Used   Available Use% Mounted on
udev            2110399232          0  2110399232   0% /dev
/dev/md0       20076492288 6019910400 13035901312  32% /
tmpfs            335544320  301989888    33554432  90% /data/lake
tmpfs           2111751168          0  2111751168   0% /sys/fs/cgroup
/dev/md1        1974737280  391432704  1482169472  21% /var
/dev/loop5       536870912  335544320   201326592  63% /data

# df -i
Filesystem       Inodes   IUsed    IFree IUse% Mounted on
udev           16487494     472 16487022    1% /dev
/dev/md0       39854080 2256887 37597193    6% /
tmpfs                32      31        1   97% /data/lake
tmpfs          16498056      17 16498039    1% /sys/fs/cgroup
/dev/md1        3932160  116812  3815348    3% /var
/dev/loop5           32      30        2   94% /data

# mount | grep "/data "
/var/datawarehouse on /data type ext4 (ro,relatime)
```
Можно ли решить проблему коллеги на этом сервере, и если да, то какую команду нужно выполнить?

## Варианты ответа:
- resize2fs /data 24576т
- growfs --size 24576m /data
- mount -o remount, size=24576m /data
- umount /data/lake; mount -t tmpfs -о size=24g,nr_inodes=48 none /data/lake
- mount -o remount, size=25165824k,nr_inodes=64 /data/lake && umount /data && mkfs.ext4 -N 64 /var/ datawarehouse && mount -t ext4 -o rw /var/datawarehouse /data
- mount -о remount, size=24g,nr_inodes=48 /data/lake
- mount -о remount,w /data; mount -o remount,size=25165824k /data/lake
- mount -o remount,nr_inodes=42 /data/lake; mount -o remount, size=26214600k /data/lake
- lvextend -L +10G /dev/loop5 && resize2fs /var/datawarehouse && mount -o remount,w /data && mount -o remount,nr_inodes=33,size=24576m /data/lake
- mount -o remount, size=24578m /data/lake
- можно, HO в списке нетверной команды
- нельзя 