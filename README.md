# Ответы на тестовые задачи

1. https://github.com/mst1711/k8tests/tree/main/issue1

2. Самый простой вариант при условии, что все хосты имеют одинаковые fqdn имена, различающиеся только порядковым индексом:
```
for i in $(seq 1 120); do \
    scp -i ~/.ssh/key /path/to/new/config/nginx.conf root@our-host-$i.our-domain.com:/etc/nginx/nginx.conf; \
    ssh -i ~/.ssh/key root@our-host-$i.our-domain.com nginx -s reload; \
done
```
В более сложном случае можно написать bash скрипт с проверками на корректность конфигруационного файла на конкретном хосте либо написать ansible playbook с доставкой нового конфига и перезапуском nginx

3. Воспользуемся fail2ban
/etc/fail2ban/filter.d/nginx-errors.conf
```
[Definition]
failregex = <HOST> .* 410 .*
            <HOST> .* 403 .*
ignoreregex =
```
/etc/fail2ban/jail.d/nginx-errors.conf
```
[nginx-errors]
enabled = true
filter = nginx-errors
logpath = /var/log/nginx/access.log
maxretry = 60
bantime = 2700
findtime = 180
action = iptables-multiport[name=nginx, port="80,443", protocol=tcp]
```

4. Предварительно условимся, что mount point прописан в fstab
- Локально создаем файл со следующим содержимым
**check_mount.sh**
```
#!/bin/bash

CHECK_PATH="/opt/data"
mount | grep $CHECK_PATH 2>&1 > /dev/null
if [ $(echo $?) == "0" ]; then
    exit 0
fi

res=1
until [ $res == "0" ]; do
    mount $CHECK_PATH 2>&1 > /dev/null
    res=$(echo $?)
done

```
- а потом запускаем его на всех 120 машинах
```
for i in $(seq 1 120); do cat check_mount.sh | ssh -i ~/.ssh/key root@our-host-$i.our-domain.com bash; done

5. 