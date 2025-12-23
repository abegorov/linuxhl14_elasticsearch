# Настройка централизованного сбора логов в кластер Elasticsearch

## Задание

1. Создать и настроить кластер **Elasticsearch**. Настроить узлы кластера.
2. Выбрать инструмент для сбора логов. Установить и настроить инструмент для сбора логов. Сконфигурировать шаблоны и индексы.
3. Проверить сбор логов. Настроить визуализацию.

## Реализация

Задание сделано так, чтобы его можно было запустить как в **Vagrant**, так и в **Yandex Cloud**. После запуска происходит развёртывание следующих виртуальных машин:

- **es-backend-01** - **NetBox**, **patroni**, **redis**, **angie**;
- **es-backend-02** - **NetBox**, **patroni**, **redis**, **angie**;
- **es-backend-03** - **NetBox**, **patroni**, **redis**, **angie**;
- **es-es-01** - **ElasticSearch**;
- **es-es-02** - **ElasticSearch**;
- **es-es-03** - **ElasticSearch**.

В независимости от того, как созданы виртуальные машины, для их настройки запускается **Ansible Playbook** [provision.yml](provision.yml) который запускает следующие роли:

- **angie** - устанавливает и настраивает **angie**;
- **apt_sources** - настраивает репозитории для пакетного менеджера **apt** (используется [mirror.yandex.ru](https://mirror.yandex.ru)).
- **bach_completion** - устанавливает пакет **bash-completion**.
- **chrony** - устанавливает **chrony** для синхронизации времени между узлами.
- **etcd** - устанавливает и настраивает кластер **etcd** для его дальнейшего использования **patroni**.
- **haproxy** - устанавливает и настраивает **haproxy** для проксирования запросов к **redis** и **postgresql**.
- **hosts** - прописывает адреса всех узлов в `/etc/hosts`.
- **keepalived** - устанавливает и настраивает **keepalived** при разворачивании в **vagrant**.
- **locale_gen** - генерит локаль **ru_RU.UTF-8** для последующего использования в **postgresql**.
- **netbox** - устанавливает и настраивает **netbox**.
- **patroni** - устанавливает и настраивает кластер **patroni**.
- **patroni_db** - создаёт базу данных в кластере **patroni** (определяет лидера и создаёт её на лидере).
- **patroni_facts** - собирает информацию о членах кластера **patroni** (определяет лидера).
- **patroni_privs** - настраивает права доступа к базам данных в кластере **patroni**.
- **patroni_user** - создаёт пользователей в кластере **patroni**.
- **pgbouncer** - устанавливает и настраивает **pgbouncer**.
- **pgdg_repo** - устанавливает репозиторий **pgdb** для **patroni** и **postgresql**.
- **redis** - устанавливает и настраивает **redis** (с репликацией на другие узлы).
- **redis_repo** - устанавливает и настраивает репозиторий для **redis**.
- **redis_sentinel** - устанавливает и настраивает **redis sentinel** для автоматического переключения мастера в кластере **redis**.
- **system_groups** - создаёт группы пользователей **Linux**.
- **system_users** - создаёт группы пользователей **Linux**.
- **tls_ca** - создаёт сертификаты для корневых центров сертификации.
- **tls_certs** - создаёт сертификаты для узлов.
- **tls_copy** - копирует серитификаты на узел.
- **wait_connection** - ожидает доступность виртуальных машин.

Данные роли настраиваются с помощью переменных, определённых в следующих файлах:

- [group_vars/all/angie.yml](group_vars/all/angie.yml) - общие настройки **angie**;
- [group_vars/all/ansible.yml](group_vars/all/ansible.yml) - общие переменные **ansible** для всех узлов;
- [group_vars/all/certs.yml](group_vars/all/certs.yml) - настройки генерации сертификатов для СУБД и **angie**;
- [group_vars/all/hosts.yml](group_vars/all/hosts.yml) - настройки для роли **hosts** (список узлов, которые нужно добавить в `/etc/hosts`);
- [group_vars/backend/angie.yml](group_vars/backend/angie.yml) - настройки **angie** для узлов **backend**;
- [group_vars/backend/certs.yml](group_vars/backend/certs.yml) - настройки генерации сертификатов для **backend**;
- [group_vars/backend/haproxy.yml](group_vars/backend/haproxy.yml) - настройки **haproxy**  для узлов **backend** (проксирования не лидера **patroni** и **redis**);
- [group_vars/backend/keepalived.yml](group_vars/backend/keepalived.yml) - настройки **keepalived** для узлов **backend**;
- [group_vars/backend/netbox.yml](group_vars/backend/netbox.yml) - настройки **netbox** для узлов **backend**;
- [group_vars/backend/patroni.yml](group_vars/backend/patroni.yml) - настройки **patroni** для узлов **backend**;
- [group_vars/backend/pgbouncer.yml](group_vars/backend/pgbouncer.yml) - настройки **pgbouncer** для узлов **backend**;
- [group_vars/backend/redis.yml](group_vars/backend/redis.yml) - настройки **redis** для узлов **backend**;
- [group_vars/backend/users.yml](group_vars/backend/users.yml) - настройки создания пользователей и групп на узлах **backend**;
- [host_vars/es-backend-01/redis.yml](host_vars/es-backend-01/mariadb.yml) - настройки **redis** для **es-backend-01**;
- [host_vars/es-backend-01/keepalived.yml](host_vars/es-backend-01/keepalived.yml) - настройки **keepalived** для **es-backend-01**;
- [host_vars/es-backend-02/keepalived.yml](host_vars/es-backend-02/keepalived.yml) - настройки **keepalived** для **es-backend-02**;
- [host_vars/es-backend-03/keepalived.yml](host_vars/es-backend-03/keepalived.yml) - настройки **keepalived** для **es-backend-03**.

## Запуск

### Запуск в Yandex Cloud

1. Необходимо установить и настроить утилиту **yc** по инструкции [Начало работы с интерфейсом командной строки](https://yandex.cloud/ru/docs/cli/quickstart).
2. Необходимо установить **Terraform** по инструкции [Начало работы с Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart).
3. Необходимо установить **Ansible**.
4. Необходимо перейти в папку проекта и запустить скрипт [up.sh](up.sh).

### Запуск в Vagrant (VirtualBox)

Необходимо скачать **VagrantBox** для **bento/ubuntu-24.04** версии **202510.26.0** и добавить его в **Vagrant** под именем **bento/ubuntu-24.04/202510.26.0**. Сделать это можно командами:

```shell
curl -OL https://app.vagrantup.com/bento/boxes/ubuntu-24.04/versions/202510.26.0/providers/virtualbox/amd64/vagrant.box
vagrant box add vagrant.box --name "bento/ubuntu-24.04/202510.26.0"
rm vagrant.box
```

После этого нужно сделать **vagrant up** в папке проекта.

## Проверка

Протестировано в **OpenSUSE Tumbleweed**:

- **Vagrant 2.4.9**
- **VirtualBox 7.2.4_SUSE r170995**
- **Ansible 2.20.1**
- **Python 3.13.9**
- **Jinja2 3.1.6**

После запуска **NetBox** должен открываться по **IP** балансировщика. Для **Yandex Cloud** адрес можно узнать в выводе **terraform** в поле **load_balancer** (смотри [outputs.tf](outputs.tf)). Для **vagrant** это (можно использовать любой адрес):

- [https://192.168.56.51](https://192.168.56.51) - узел **es-backend-01**.
- [https://192.168.56.52](https://192.168.56.52) - узел **es-backend-02**.
- [https://192.168.56.53](https://192.168.56.53) - узел **es-backend-03**.

Однако **keepalived** настроен таким образом, что при недоступности одного из узлов, его адрес переезжает на один из доступных.
