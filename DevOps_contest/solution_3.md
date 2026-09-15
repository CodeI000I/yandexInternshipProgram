# Цели
1) Сгруппировать все записи по полям: **номер кластера** и **номер ноды**
2) Обработать все группы в которых есть обе записи (А и АААА)
3) Для группы с обеими запись сформировать точный ipv6 адрес с помощью функции *hex*
4) Сравнить сформированный массив с записанным

# Реализация на Python
```
import ipaddress
import re


pattern = re.compile(
    r"^.+-(\d+)-(\d+)\.kit\.yandex\.net\s+\d+\s+IN\s+(A|AAAA)\s+(\S+)$"
)

hosts = {}

with open("input.txt", encoding="utf-8") as file:
    for line in file:
        line = line.strip()

        match = pattern.match(line)
        if not match:
            continue

        cluster, node, record_type, address = match.groups()
        key = (cluster, node)

        if key not in hosts:
            hosts[key] = {}

        hosts[key][record_type] = address

result = []
for (cluster, node), records in hosts.items():
    if "A" not in records or "AAAA" not in records:
        continue

    ipv4 = records["A"]

    ipv6 = f"2b4e:{hex(int(cluster))[2:]}:{hex(int(node))[2:]}::{ipv4.replace(".",":")}"

    if records["AAAA"].lower() != ipv6:
        result.append((int(cluster), int(node), cluster, node, ipv6))


result.sort()

for _, _, cluster, node, ipv6 in result:
    print(cluster, node, ipv6)

```
