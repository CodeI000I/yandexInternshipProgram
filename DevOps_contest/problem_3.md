# Странный генератор ipv6
В нашем проекте используется большое количество серверов, и иногда нам приходится проверять DNS-выгрузки на корректность указанных адресов для хостов. Выгрузка всегда имеет следующий формат:
```console
super-cluster-<номер кластера>-<номер ноды>.kit.yandex.net 600 IN A    <IPv4 адрес>
super-cluster-<номер кластера>-<номер ноды>.kit.yandex.net 600 IN AAAA <IPv6 адрес>
```
Существует 3 конфигурации серверов:
- v4only - только IPv4
- v6only - только IPv6
- dualstack - IPv4 и IPv6

## Проблема

В конфигурации dualstack IPv6 адрес сервера генерируется автоматически на основе выданного ему IPv4 адреса, но этот адрес не всегда корректно прописывается в DNS. Вам нужно проверить и исправить некорректные AAAA записи.

## Алгоритм генерациии ipv6 адреса:

Если для хоста есть обе записи (A и AAAA), то AAAA запись должна иметь следующий формат:
2b4e:<номер кластера>:<номер ноды>::<ipv4>

Где:
- Номер кластера и номер ноды берутся из FQDN и представляются в 16-ричном формате
- IPv4 адрес разделяется точками и добавляется в конец

Например для хоста contest-123-456.kit.yandex.net 600 IN A 10.10.10.10
Будет валидный ipv6 адрес такой: 2b4e:7b:1c8::10:10:10:10

## Формат ввода
Ваше приложение будет запущено в каталоге с файлом input.txt. Вы можете читать данные как из файла, так и из стандартного ввода.

## Формат вывода
Отсортированный по номеру кластера и номеру ноды список записей с исправленными v6 адресами <номер кластера> <номер ноды> <v6 адрес>

P.S v6 адрес должен быть записан в максимально коротком виде, учитывая правила записи ipv6 (https://www.ibm.com/docs/ru/i/7.4.0?topic=concepts-ipv6-address-formats)

P.P.S номер кластера и номер ноды должны быть записаны также как и в fqdn

## Пример

```Ввод
contest-001-007.kit.yandex.net 600 IN A 193.3.180.187
contest-001-010.kit.yandex.net 600 IN A 124.178.206.124
contest-001-005.kit.yandex.net 600 IN A 131.247.195.48
contest-001-005.kit.yandex.net 600 IN AAAA 2b4e:1:5::131:247:195:48
contest-001-009.kit.yandex.net 600 IN AAAA 2b4e:1:9:1c50:373b:3db9:1b9f:1aaf
contest-001-002.kit.yandex.net 600 IN AAAA 2b4e:1:2:1755:4d6:2aad:de0:1cb5
contest-001-008.kit.yandex.net 600 IN A 126.132.142.197
contest-001-001.kit.yandex.net 600 IN AAAA 2b4e:1:1:3656:3518:121b:1081:3a21
contest-001-006.kit.yandex.net 600 IN A 57.203.44.214
contest-001-006.kit.yandex.net 600 IN AAAA None
contest-001-003.kit.yandex.net 600 IN A 129.231.101.104
contest-001-003.kit.yandex.net 600 IN AAAA 2b4e:1:00003::0000
contest-001-004.kit.yandex.net 600 IN A 89.254.135.150
contest-002-009.kit.yandex.net 600 IN A 217.109.150.247
contest-002-005.kit.yandex.net 600 IN A 21.161.239.114
contest-002-005.kit.yandex.net 600 IN AAAA 2b4e:212:5:212:21:161:239:114
contest-002-006.kit.yandex.net 600 IN AAAA 2b4e:2:6:128e:3c9f:23ab:3fa:3b3b
contest-002-003.kit.yandex.net 600 IN A 247.218.87.177
contest-002-004.kit.yandex.net 600 IN A 135.187.109.94
contest-002-002.kit.yandex.net 600 IN A 114.92.42.204
contest-002-008.kit.yandex.net 600 IN A 95.233.92.206
contest-002-010.kit.yandex.net 600 IN A 64.45.251.185
contest-002-010.kit.yandex.net 600 IN AAAA 2b4e:2:a::64:45:251:185
contest-002-007.kit.yandex.net 600 IN A 41.74.31.91
contest-002-001.kit.yandex.net 600 IN A 87.168.134.117
contest-002-001.kit.yandex.net 600 IN AAAA 2b4e:2:1::87:168:134:117
contest-003-001.kit.yandex.net 600 IN A 213.110.134.156
contest-003-008.kit.yandex.net 600 IN A 87.157.16.155
contest-003-009.kit.yandex.net 600 IN A 203.211.83.98
contest-003-010.kit.yandex.net 600 IN A 144.218.22.195
contest-003-006.kit.yandex.net 600 IN A 208.201.148.20
contest-003-006.kit.yandex.net 600 IN AAAA 2b4e:3:6::208:201:148:20
contest-003-005.kit.yandex.net 600 IN AAAA 2b4e:3:5:2471:23a8:3d53:959:1b5a
contest-003-004.kit.yandex.net 600 IN AAAA 2b4e:3:4:1028:32c2:5c0:1701:95a
contest-003-007.kit.yandex.net 600 IN A 215.45.70.120
contest-003-002.kit.yandex.net 600 IN AAAA 2b4e:3:2:fe2:8cf:180c:bad:31ea
contest-003-003.kit.yandex.net 600 IN AAAA 2b4e:3:3:2907:10c1:1a37:39a8:1352
```

```Вывод
001 003 2b4e:1:3::129:231:101:104
001 006 2b4e:1:6::57:203:44:214
002 005 2b4e:2:5::21:161:239:114
```