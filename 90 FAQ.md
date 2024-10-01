### Что такое репозиторий в yum?
Это сервер в интернете, с которого можно скачать пакеты для установки в yum. Так же там появляются обновления для пакетов, и можно в случае чего обновиться на новую версию.
### Как посмотреть какие репозитории добавлены в yum?
```bash
rpm -qa
```
### Как найти репозиторий с определенным названием в yum?
```bash
rpm -qa | grep pgdg
```
### Как удалить репозиторий из yum?
```bash
yum -y remove pgdg-redhat-repo-42.0-45PGDG.noarch
yum clean all
```
### Как установит новый репозиторий в yum?
```bash
yum -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

### Где найти актуальные репозитории для PostgresSQL?
https://yum.postgresql.org/repopackages/

P.S если что для версии CentOS Stream 9 - лучше брать Red Hat Enterprise Linux 8 - x86_64. RHEL 9 репозиторий не заработает.

Еще стоит обращать на то, для какой архитектуры репозиторий
* x86_64 - архитектура всех обычных компьютеров и серверов (Если у тебя Windows, то у тебя точно она)
* aarch64(arm64) - архитектура новых маков на процессорах М1