### Задание 
Для выполнения домашнего задания используйте методичку

Что нужно сделать?

в вагранте поднимаем 2 машины web и log
на web поднимаем nginx
на log настраиваем центральный лог сервер на любой системе на выбор
journald;
rsyslog;
elk.
настраиваем аудит, следящий за изменением конфигов нжинкса

Все критичные логи с web должны собираться и локально и удаленно.

Все логи с nginx должны уходить на удаленный сервер (локально только критичные).

Логи аудита должны также уходить на удаленную систему.

развернуть еще машину elk*

таким образом настроить 2 центральных лог системы elk и какую либо еще;
в elk должны уходить только логи нжинкса;
во вторую систему все остальное.


### Решение

Для решения задачи созданы 4 плейбука 
1) Create_vm.yml -  создание виртуальных  машин и  предварительная настройка 
2) Config_vm_web.yml - Настройка виртуальной машины с Nginx и удаленной отправки логов
3) Config_vm_log.yml - Настройка машины централизованного храннеия логов 
4) Config_vm_elk.yml - Настройка машины с ELK



В файле /vars/hosts Указаны основные переменные  для конфигурирования VM
Файл secrets.yml содержит чувствительную информацию для авторизации на хостах виртуализации и в связи с этим защищен

После выполнения плейбуков у нас будут настроены 3 машины 
web
log
elk

c web  с помощью journald-upload  будет  отправляться журнал на мащину log 
Согласно заданию  в journal  будут вносится данные только уровня error и ниже, так как в конфигурационном файле nginx.conf мы указали - 
```bash 
error_log syslog:server=unix:/dev/log warn;
access_log syslog:server=unix:/dev/log;

```

Логируем ошибки и обращения к nginx
![смотрим за nginx](https://raw.githubusercontent.com/jecka2/elk/refs/heads/main/journal-nginx.png)



C помощью auditd и настроеных ruls  мы  можем наблюдать за изменениями в директории /etc/nginx 

```bash
 name: create rule for audit
      ansible.builtin.file:
       path: /etc/audit/rules.d/nginx.rules
       state: touch
       mode: '0640'

    - name: add rule
      ansible.builtin.lineinfile:
        path: /etc/audit/rules.d/nginx.rules
        line: -w /etc/nginx -p wax -k nginx_mon
```
Пытаемся создать файл test_audiit в директории /etc/nginx
```bash
sudo touch /etc/nginx/test_audit
``` 
Aудит присматривает за /etc/nginx/
![Аудит за /etc/nginx](https://raw.githubusercontent.com/jecka2/elk/refs/heads/main/ansible-audit.png)


Проводим проверку что наш журнал успешно передается на центральный сервер логов

```bash 

journalctl --file /var/log/journal/remote/remote-192.168.1.101.journal --since="2 minutes ago"

```

Результат запроса
![Журнал с хоста в Nginx](https://raw.githubusercontent.com/jecka2/elk/refs/heads/main/journal-remote.png)


Задание с * 


Дополнительно мы развернули  elk и лог с Nginx так же отправляются туда 


elk
![Журнал с хоста в elk](https://raw.githubusercontent.com/jecka2/elk/refs/heads/main/elk.png)
