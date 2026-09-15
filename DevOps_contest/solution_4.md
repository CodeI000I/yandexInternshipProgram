# Цели
1) /proc/partions - все распознанные разделы диска и основные сведения распределения
2) /proc/meminfo - сообщает об использовании и статистики RAM в реальном времени 
3) Размер tmpfs по умолчанию есть половина от всей RAM
# Решение на Python
```
result = 0

with open("/proc/partitions") as file:
    for line in file:
        fields = line.split()

        if len(fields) != 4 or not fields[2].isdigit():
            continue

        minor = int(fields[1])
        size_kib = int(fields[2])
        name = fields[3]

        if minor != 0 and not name.startswith("loop"):
            result += size_kib

print(result * 1024)

with open("/proc/meminfo", "r") as f:
    for line in f:
        fields = line.split()

        field_name = fields[0]
        field_size = int(fields[1])
        
        if field_name == "MemTotal:":
            print(field_size * 512)
            break

```