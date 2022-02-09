# loging
### Сбор и анализ логов. Настраиваем центральный сервер для сбора логов.
### Задание
1. В вагранте поднимаем 2 машины web и log 
2. На web поднимаем nginx 
3. На log настраиваем центральный лог сервер на любой системе на выбор
- journald
- rsyslog
- elk \
Настраиваем аудит, следящий за изменением конфигов nginx \
Все критичные логи с web должны собираться и локально и удаленно.
Все логи с nginx должны уходить на удаленный сервер (локально только критичные).
Логи аудита должны также уходить на удаленную систему.

## Выполнение
Сформировать ssh ключ \
`ssh-keygen -t rsa -f ~/.ssh/vagrant-key -b 2048` \
Добавить ssh ключ в папку в корне проекта \
`provision/ssh/vagrant-pub.key`

Для конфигурирования серверов используется `ansible`

Запускаем стенд: \
`ansible-playbook play.yaml`

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
### Все настройки добавляются при помощи шаблонов.
Все настройки Rsyslog хранятся в файле /etc/rsyslog.conf

Для того, чтобы наш сервер мог принимать логи, нам необходимо внести следующие изменения в файл: \
Открываем порт 514 (TCP и UDP): \
Находим закомментированные строки: \
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

В конец файла /etc/rsyslog.conf добавляем правила приёма сообщений от хостов: \
`#Add remote logs` \
`$template RemoteLogs,"/var/log/rsyslog/%HOSTNAME%/%PROGRAMNAME%.log"` \
`*.* ?RemoteLogs` \
`& ~` 

Данные параметры будут отправлять в папку `/var/log/rsyslog` логи, которые будут приходить от
других серверов. Например, Access-логи
