# Дипломный практикум в YandexCloud
  * [Цели:](#цели)
  * [Этапы выполнения:](#этапы-выполнения)
      * [Регистрация доменного имени](#регистрация-доменного-имени)
      * [Создание инфраструктуры](#создание-инфраструктуры)
          * [Установка Nginx и LetsEncrypt](#установка-nginx)
          * [Установка кластера MySQL](#установка-mysql)
          * [Установка WordPress](#установка-wordpress)
          * [Установка Gitlab CE, Gitlab Runner и настройка CI/CD](#установка-gitlab)
          * [Установка Prometheus, Alert Manager, Node Exporter и Grafana](#установка-prometheus)
  * [Что необходимо для сдачи задания?](#что-необходимо-для-сдачи-задания)
  * [Полезные ссылки](#полезные-ссылки)

---
## Цели:

1. Зарегистрировать доменное имя (любое на ваш выбор в любой доменной зоне); ☑ 
2. Подготовить инфраструктуру с помощью Terraform на базе облачного провайдера YandexCloud; ☑
3. Настроить внешний Reverse Proxy на основе Nginx и LetsEncrypt; ☑
4. Настроить кластер MySQL; ☑
5. Установить WordPress; ☑
6. Развернуть Gitlab CE и Gitlab Runner; ☑
7. Настроить CI/CD для автоматического развёртывания приложения; ☑
8. Настроить мониторинг инфраструктуры с помощью стека: Prometheus, Alert Manager и Grafana. ☑

---
## Решение:

### Регистрация доменного имени

Рекомендуемые регистраторы:
  - [reg.ru](https://reg.ru) - (лучше по соответствию цена/функционал)
  - [nic.ru](https://nic.ru)

Цель:

1. Получить возможность выписывать [TLS сертификаты](https://letsencrypt.org) для веб-сервера. ✓

Результаты:

1. Есть доступ к личному кабинету на сайте регистратора; ✓
2. Зарегистрировали домен и можете им управлять (kuberwars.online). ✓

### Создание инфраструктуры

Для начала необходимо подготовить инфраструктуру в YC при помощи [Terraform](https://www.terraform.io/).

Особенности выполнения:

- Бюджет ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов;
- Следует использовать последнюю стабильную версию [Terraform](https://www.terraform.io/).

Предварительная подготовка:

1. Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя ✓
2. Подготовьте [backend](https://www.terraform.io/docs/language/settings/backends/index.html) для Terraform:
   S3 bucket в созданном YC аккаунте. ✓
3. Настройте [workspaces](https://www.terraform.io/docs/language/state/workspaces.html)
   Создайте два workspace: *stage* и *prod*. В случае выбора этого варианта все последующие шаги должны учитывать факт существования нескольких workspace. ✓
   Для stage определен уровень 50% процессорной мощности. Для prod полноценные 100%.
4. Создайте VPC с подсетями в разных зонах доступности. ✓
5. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` без дополнительных ручных действий. ✓

Цель:

1. Повсеместно применять IaaC подход при организации (эксплуатации) инфраструктуры. ✓
2. Иметь возможность быстро создавать (а также удалять) виртуальные машины и сети. С целью экономии денег на вашем аккаунте в YandexCloud. ✓

Ожидаемые результаты:

1. Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий. ✓
2. Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.

---
### Установка Nginx и LetsEncrypt

Необходимо разработать Ansible роль для установки Nginx и LetsEncrypt.

**Для получения LetsEncrypt сертификатов во время тестов своего кода пользуйтесь [тестовыми сертификатами](https://letsencrypt.org/docs/staging-environment/), так как количество запросов к боевым серверам LetsEncrypt [лимитировано](https://letsencrypt.org/docs/rate-limits/).**

Рекомендации:
  - Имя сервера: `kuberwars.online`
  - Характеристики: 2vCPU, 2 RAM, External address (Public) и Internal address.

Цель:

1. Создать reverse proxy с поддержкой TLS для обеспечения безопасного доступа к веб-сервисам по HTTPS. ✓

Ожидаемые результаты:

1. В вашей доменной зоне настроены все A-записи на внешний адрес этого сервера:
    - `https://www.kuberwars.online` (WordPress)
    - `https://gitlab.kuberwars.online` (Gitlab)
    - `https://grafana.kuberwars.online` (Grafana)
    - `https://prometheus.kuberwars.online` (Prometheus)
    - `https://alertmanager.kuberwars.online` (Alert Manager)
2. Настроены все upstream для выше указанных URL, куда они сейчас ведут на этом шаге не важно, позже вы их отредактируете и укажите верные значения. ✓

___
### Установка кластера MySQL

Необходимо разработать Ansible роль для установки кластера MySQL.

Рекомендации:
  - Имена серверов: `db01.kuberwars.online` и `db02.kuberwars.online`
  - Характеристики: 4vCPU, 4 RAM, Internal address.

Цель:

1. Получить отказоустойчивый кластер баз данных MySQL.

Ожидаемые результаты:

1. MySQL работает в режиме репликации Master/Slave.
2. В кластере автоматически создаётся база данных c именем `wordpress`.
3. В кластере автоматически создаётся пользователь `wordpress` с полными правами на базу `wordpress` и паролем `wordpress`.

**Вы должны понимать, что в рамках обучения это допустимые значения, но в боевой среде использование подобных значений не приемлимо! Считается хорошей практикой использовать логины и пароли повышенного уровня сложности. В которых будут содержаться буквы верхнего и нижнего регистров, цифры, а также специальные символы!**

___
### Установка WordPress

Необходимо разработать Ansible роль для установки WordPress.

Рекомендации:
  - Имя сервера: `app.kuberwars.online`
  - Характеристики: 4vCPU, 4 RAM, Internal address.

Цель:

1. Установить [WordPress](https://wordpress.org/download/). Это система управления содержимым сайта ([CMS](https://ru.wikipedia.org/wiki/Система_управления_содержимым)) с открытым исходным кодом.


По данным W3techs, WordPress используют 64,7% всех веб-сайтов, которые сделаны на CMS. Это 41,1% всех существующих в мире сайтов. Эту платформу для своих блогов используют The New York Times и Forbes. Такую популярность WordPress получил за удобство интерфейса и большие возможности.

Ожидаемые результаты:

1. Виртуальная машина на которой установлен WordPress и Nginx/Apache (на ваше усмотрение). ✓
2. В вашей доменной зоне настроена A-запись на внешний адрес reverse proxy: ✓
    - `https://www.kuberwars.online` (WordPress)
3. На сервере `kuberwars.online` отредактирован upstream для выше указанного URL и он смотрит на виртуальную машину на которой установлен WordPress. ✓
4. В браузере можно открыть URL `https://www.kuberwars.online` и увидеть главную страницу WordPress. ✓
---
### Установка Gitlab CE и Gitlab Runner

Необходимо настроить CI/CD систему для автоматического развертывания приложения при изменении кода.

Рекомендации:
  - Имена серверов: `gitlab.kuberwars.online` и `runner.kuberwars.online`
  - Характеристики: 4vCPU, 4 RAM, Internal address.

Цель:
1. Построить pipeline доставки кода в среду эксплуатации, то есть настроить автоматический деплой на сервер `app.kuberwars.online` при коммите в репозиторий с WordPress. ✓

Подробнее об [Gitlab CI](https://about.gitlab.com/stages-devops-lifecycle/continuous-integration/)

Ожидаемый результат:

1. Интерфейс Gitlab доступен по https. ✓
2. В вашей доменной зоне настроена A-запись на внешний адрес reverse proxy: ✓
    - `https://gitlab.kuberwars.online` (Gitlab)
3. На сервере `kuberwars.online` отредактирован upstream для выше указанного URL и он смотрит на виртуальную машину на которой установлен Gitlab. ✓
3. При любом коммите в репозиторий с WordPress и создании тега (например, v1.0.0) происходит деплой на виртуальную машину. ✓

___
### Установка Prometheus, Alert Manager, Node Exporter и Grafana

Необходимо разработать Ansible роль для установки Prometheus, Alert Manager и Grafana. ✓

Рекомендации:
  - Имя сервера: `monitoring.kuberwars.online`
  - Характеристики: 4vCPU, 4 RAM, Internal address.

Цель:

1. Получение метрик со всей инфраструктуры. ✓

Ожидаемые результаты:

1. Интерфейсы Prometheus, Alert Manager и Grafana доступены по https. ✓
2. В вашей доменной зоне настроены A-записи на внешний адрес reverse proxy: ✓
  - `https://grafana.kuberwars.online` (Grafana)
  - `https://prometheus.kuberwars.online` (Prometheus)
  - `https://alertmanager.kuberwars.online` (Alert Manager)
3. На сервере `kuberwars.online` отредактированы upstreams для выше указанных URL и они смотрят на виртуальную машину на которой установлены Prometheus, Alert Manager и Grafana. ✓
4. На всех серверах установлен Node Exporter и его метрики доступны Prometheus. ✓
5. У Alert Manager есть необходимый [набор правил](https://awesome-prometheus-alerts.grep.to/rules.html) для создания алертов. ✓
6. В Grafana есть дашборд отображающий метрики из Node Exporter по всем серверам. ✓

---
## Что необходимо для сдачи задания?

1. Репозиторий со всеми Terraform манифестами и готовность продемонстрировать создание всех ресурсов с нуля. ✓
2. Репозиторий со всеми Ansible ролями и готовность продемонстрировать установку всех сервисов с нуля. ✓
3. Скриншоты веб-интерфейсов всех сервисов работающих по HTTPS на вашем доменном имени. ✓
  - `https://www.kuberwars.online` (WordPress)
  - `https://gitlab.kuberwars.online` (Gitlab)
  - `https://grafana.kuberwars.online` (Grafana)
  - `https://prometheus.kuberwars.online` (Prometheus)
  - `https://alertmanager.kuberwars.online` (Alert Manager)
4. Все репозитории рекомендуется хранить на одном из ресурсов ([github.com](https://github.com) или [gitlab.com](https://gitlab.com)). 

---
#### Полезные ссылки

1. https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-state-storage
2. https://terraform-eap.website.yandexcloud.net/docs/providers/yandex/r/compute_instance.html
3. https://www.cyberciti.biz/faq/how-to-install-and-configure-latest-version-of-ansible-on-ubuntu-linux/
4. https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html
5. https://galaxy.ansible.com
6. https://www.8host.com/blog/poluchenie-sertifikata-lets-encrypt-s-pomoshhyu-ansible-v-ubuntu-18-04/
7. https://www.8host.com/blog/kak-rabotat-s-ansible-prostaya-i-udobnaya-shpargalka/


---