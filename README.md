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

4. 