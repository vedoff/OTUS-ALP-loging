# loging
### Сбор и анализ логов. Настраиваем центральный сервер для сбора логов.
### Задание
1. В вагранте поднимаем 2 машины web-server и log-server
2. На web-server поднимаем nginx 
3. На log-server настраиваем центральный лог сервер на любой системе на выбор
- journald
- rsyslog
- elk \
Настраиваем аудит, следящий за изменением конфигов nginx \
Все критичные логи с web должны собираться и локально и удаленно.
Все логи с nginx должны уходить на удаленный сервер (локально только критичные).
Логи аудита должны также уходить на удаленную систему.

  - PS 
      > для сбора логов будем использовать `rsyslog` \
      > в данной работе `elk` не используется. \
Карта: \
[Конфигурирование `auditd`](https://github.com/vedoff/loging/blob/main/README.md#%D0%B2%D1%81%D0%B5-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B8-%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%BB%D1%8F%D1%8E%D1%82%D1%81%D1%8F-%D0%BF%D1%80%D0%B8-%D0%BF%D0%BE%D0%BC%D0%BE%D1%89%D0%B8-%D1%88%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD%D0%BE%D0%B2-jinja) \
[Настройка отправки логов с сервера](https://github.com/vedoff/loging#-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D0%BC-%D0%BE%D1%82%D0%BF%D1%80%D0%B0%D0%B2%D0%BA%D1%83-%D0%BB%D0%BE%D0%B3%D0%BE%D0%B2-%D1%81-web-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0-) \
[Настройка аудита, контролирующего изменения конфигурации nginx](https://github.com/vedoff/loging#-%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B0%D1%83%D0%B4%D0%B8%D1%82%D0%B0-%D0%BA%D0%BE%D0%BD%D1%82%D1%80%D0%BE%D0%BB%D0%B8%D1%80%D1%83%D1%8E%D1%89%D0%B5%D0%B3%D0%BE-%D0%B8%D0%B7%D0%BC%D0%B5%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3%D1%83%D1%80%D0%B0%D1%86%D0%B8%D0%B8-nginx-) \
[Настройка пересылки логов на удаленный сервер](https://github.com/vedoff/loging#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D0%BC-%D0%BF%D0%B5%D1%80%D0%B5%D1%81%D1%8B%D0%BB%D0%BA%D1%83-%D0%BB%D0%BE%D0%B3%D0%BE%D0%B2-%D0%BD%D0%B0-%D1%83%D0%B4%D0%B0%D0%BB%D0%B5%D0%BD%D0%BD%D1%8B%D0%B9-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80)\
[Настройка Log-сервера для приема логов с удаленных машин.](https://github.com/vedoff/loging#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B8%D0%BC-log-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80-%D0%B4%D0%BB%D1%8F-%D0%BF%D1%80%D0%B8%D0%B5%D0%BC%D0%B0-%D0%BB%D0%BE%D0%B3%D0%BE%D0%B2-%D1%81-%D1%83%D0%B4%D0%B0%D0%BB%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D0%BC%D0%B0%D1%88%D0%B8%D0%BD)

## Выполнение
Сформировать ssh ключ \
`ssh-keygen -t rsa -f ~/.ssh/vagrant-key -b 2048` \
Добавить ssh ключ в папку в корне проекта \
`provision/ssh/vagrant-pub.key`

Для конфигурирования серверов используется `ansible`

Запускаем стенд: \
`vagrant up` \
`ansible-playbook play.yml`

### Как это работает:
Будет развернут стенд из двух машин `web-server` и `log-server` \
На `web-server` будет установлен `nginx` и сконфигурирован при помощи шаблонов `J2` \
Для развертывания испрльзуются роли:
- nginx
- audit-rules
- rsyslog-configure
### Что будет сконфигурировано
- Установлен `nginx` полсе развертывания доступен по адресу `http://192.168.56.161:8088`
- Сконфигурирован `rsyslog` 
### Все настройки добавляются при помощи шаблонов Jinja.
Настройки rsyslog хранятся в файле /etc/rsyslog.conf

Для того, чтобы наш сервер мог принимать логи, нам необходимо внести следующие изменения в файл: \
Открываем порт 514 (TCP и UDP): \
Находим закомментированные строки: 
- В нашем случае я уже их раскоментировал 

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-09%2016-21-45.png)

- Возможный вариант

`# provides UDP syslog reception`\
`#module(load="imudp")` \
`#input(type="imudp" port="514")` \
`# provides TCP syslog reception` \
`#module(load="imtcp")` \
`input(type="imtcp" port="514")` 

И приводим их к виду: \
`# provides UDP syslog reception` \
`module(load="imudp")` \
`input(type="imudp" port="514")` \
`# provides TCP syslog reception` \
`module(load="imtcp")` \
`input(type="imtcp" port="514")` 

В конец файла `/etc/rsyslog.conf` добавляем правила приёма сообщений от хостов: \
`#Add remote logs` \
`$template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"` \
`*.* ?RemoteLogs` \
`& ~` 

Данные параметры будут отправлять в папку `/var/log/rsyslog` логи, которые будут приходить от
других серверов. Например, Access-логи nginx от сервера `web-server`, будут идти в файл
`/var/log/rsyslog/web-server/nginx_access.log`

### === Настроим отправку логов с web-сервера ===
Находим в файле /etc/nginx/conf.d/mydomain.conf раздел с логами и приводим их к следующему виду: 

`error_log /var/log/nginx/error.log;` \
`error_log syslog:server=192.168.56.162:514,tag=nginx_error;` \
`access_log syslog:server=192.168.56.162:514,tag=nginx_access,severity=info combined;`

Для Access-логов указыаем удаленный сервер и уровень логов, которые нужно отправлять. \
Для error_log добавляем удаленный сервер. \
Если требуется чтобы логи хранились локально и отправлялись на удаленный сервер, требуется указать 2 строки. \
Tag нужен для того, чтобы логи записывались в разные файлы. \
По умолчанию, error-логи отправляют логи, которые имеют `severity: error, crit, alert` и `emerg`. 

Попробуем несколько раз зайти по адресу http://192.168.56.161:8088 \
Далее заходим на log-сервер и смотрим информацию об nginx:

cat /var/log/rsyslog/web-server/nginx_access.log \
cat /var/log/rsyslog/web-server/nginx_error.log

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-09%2017-08-41.png)

### === Настройка аудита, контролирующего изменения конфигурации nginx ===

За аудит отвечает утилита `auditd`, в `RHEL-based` системах обычно он уже предустановлен.\
Проверим это: \
`rpm -qa | grep audit` \
Настроим аудит изменения конфигурации nginx: \
Добавим правило, которое будет отслеживать изменения в конфигруации nginx. Для этого в конец \
файла /etc/audit/rules.d/audit.rules добавим следующие строки: \
`-w /etc/nginx/nginx.conf -p wa -k nginx_conf` \
`-w /etc/nginx/default.d/ -p wa -k nginx_conf` \
`-w /etc/nginx/conf.d/ -p wa -k nginx_conf`

Данные правила позволяют контролировать запись (w) и измения атрибутов (a) в: \
В файле: \
`/etc/nginx/nginx.conf` 

Всех файлов каталогов: \
`/etc/nginx/default.d/` \
`/etc/nginx/conf.d/` 

Для более удобного поиска к событиям добавляется метка `nginx_conf`
Перезапускаем службу `auditd`: \
`service auditd restart`

### После данных изменений у нас начнут локально записываться логи аудита.
Чтобы проверить, что логи аудита начали записываться локально, нужно внести изменения в файл или поменять его атрибут. \
`/etc/nginx/conf.d/mydomain.conf` \
 Потом посмотреть информацию об изменениях: \
`ausearch -f /etc/nginx/conf.d/mydomain.conf` 

Также можно воспользоваться поиском по файлу `/var/log/audit/audit.log`, указав тэг: \
`grep nginx_conf /var/log/audit/audit.log`

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-09%2017-52-23.png)

## Настроим пересылку логов на удаленный сервер
### Как это работает
Все нижеперечисленные манипуляции производятся при помощи `ansible` \
Используются роли:
- audit-rules-to-remote-server
- audit-log-prepare

Выполним: \
`ansible-playbook rmt-audit-play.yml`

Установим audispd-plugins

Найдем и поменяем следующие строки в файле: \
`/etc/audit/auditd.conf:` \
`log_format = RAW` \
`name_format = HOSTNAME` \
В `name_format` \
указываем `HOSTNAME` параметры берутся из переменных в папке `vars`, \
чтобы в логах на удаленном сервере отображалось имя хоста.

В файле \
`/etc/audisp/plugins.d/au-remote.conf` поменяем параметр `active` на `yes`: \
`active = yes` \
`direction = out` \
`path = /sbin/audisp-remote` \
`type = always` \
`#args =` \
`format = string` 

В файле: \
`/etc/audisp/audisp-remote.conf` \
требуется указать адрес сервера и порт, на который будут \
отправляться логи: \
В файле: \
`/etc/audisp/audisp-remote.conf` \
`remote_server = 192.168.56.162` \
`port = 60`

Далее перезапускаем службу auditd: \
`service auditd restart`

На этом настройка web-сервера завершена. 
## Настроим Log-сервер для приема логов с удаленных машин.
Отроем порт TCP 60 \
В файле \
`/etc/audit/auditd.conf:` \
`tcp_listen_port = 60` \
Перезапускаем службу `auditd`: \
`service auditd restart`

Проверяем что это работаает: \
Выполним \
`ss -tunlp`

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-10%2010-18-01.png)

Видим, что `log-server` слушает на `60` порту. \
Проверим, что идет пересылка логов `audit` на лог сервер. \
Для этого изменим атрибуты конфиг файла домена на `nginx` \
`vagrant ssh web-server` \
`chmod g+w /etc/nginx/conf.d/medomain.conf`

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-10%2010-20-01.png)

Перейдем на `log-server` \
`vagrant ssh log-server` \
Выполним поиск логов \
`grep web-server /var/log/audit/audit.log`

![](https://github.com/vedoff/loging/blob/main/pict/Screenshot%20from%202022-02-10%2011-33-06.png)

Наблюдаем сбор логов с удаленного хоста `web-server`




