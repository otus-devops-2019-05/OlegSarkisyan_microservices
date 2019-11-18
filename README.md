# OlegSarkisyan_microservices
OlegSarkisyan microservices repository


# Выполнено ДЗ №21 - Kubernetes. Networks ,Storages.

##  Основное ДЗ
Создали ветку kubernetes-3
Настроили и понаблюдали за работой ClusterIP, NodePort и LoadBalancer
Познакомились с Ingress - настроили правила внутри кластера
Загрузили сертификаты в кластер кубера и настроили Ingress на приём только https траффика
Использовали инструмент NetworkPolicy для разделния сервсов БД и фронта 
Настроили Хранилище - подключили Volume, создали и смонтировали выделенный диск, пересоздали pod
Научились создавать хранилища в автоматическом режиме с помощью StorageClass - настроили динамическое выделение Volume 
Запушили ветку на github


# Выполнено ДЗ №20 - Kubernetes. Запуск кластера и приложения. Модель безопасности.

##  Основное ДЗ
Создали ветку kubernetes-2
Развернули локальный Kubernetes
Создали ресурсы для Deployment 
Проверили работу UI и пробросили сетевые порты
Настроили comment, самостоятельно настроили post
Настроили Mongodb и объекты Service - post, ui, mongodb и comment
Для правильного поиска адресов настроим comment-mongodb-service.yml и post-mongodb-service.yml
Посмотрели на работу minikube
Посмотрели на вывод Namespace
Развернули Kubernetes в GСP в GKE
Запустили приложение в GKE
Запушили ветку на github


# Выполнено ДЗ №19 - Введение в Kubernetes

##  Основное ДЗ
Создали ветку kubernetes-1
Создали директорию reddit
Создали файлы с манифестами
Прошли задание The Hard Way
Запушили ветку на github

# Выполнено ДЗ №18 - Логирование и распределенная трассировка

##  Основное ДЗ
Создали ветку logging-1
Обновили код src
С помощью Docker-machine развернули новую VM
Создали docker-compose-logging.yml
Добавили EFK стек
Настроили и потестировали Fluentd
Рассмотрели работу структурированных логов
Сконфигурировали Кибану
Использовали функционал фильтров для парсинга логов
Рассмотрели парсинг неструктурированных логов
Интегрировали Zipkin, взглянули на трейсинг
В процессе работы периодически правили и пересобирали образы
Запушили ветку на github


# Выполнено ДЗ №17 - Мониторинг приложения и инфраструктуры

##  Основное ДЗ
Создали ветку monitoring-2
С помощью Docker-machine развернули новую VM
Установили и запустили prometheus
Установили настроили и проверили работу cAdvisor
Визуализировали метрики с помощью Grafana 
Добавили готовый шаблон дашборда
Создали свой собственный дашборд
Настроили Алертинг - установили allertmanager
В процессе работы периодически правили и пересобирали образы
Запушили все образы на dockerhub
Ссылка на докер хаб - https://cloud.docker.com/u/grimasnik777/repository/docker/grimasnik777
Запушили ветку на github


# Выполнено ДЗ №16 - Введение в мониторинг. Системы мониторинга.

##  Основное ДЗ
Создали ветку monitoring-1
С помощью Docker-machine развернули новую VM
Установили и запустили prometheus
Посмотрели базовые метрики
Переупорядочили структуру директорий
Создали докер образ
Собрали images сервисов
Подняли мониторинг вместе с микросервисами
Проверили состояние сервисов согласно мониторингу
Добавили node exporter
Пересобрали сервисы и убедились в добавлении новых метрик
Ссылка на докер хаб - https://cloud.docker.com/u/grimasnik777/repository/docker/grimasnik777
Запушили ветку на github


# Выполнено ДЗ №15 - Gitlab:построение процесса непрерывной поставки 

##  Основное ДЗ
Создали ветку gitlab-ci-1
С помощью Docker-machine развернули новую VM 
Установили Docker затем docker-compose.yml
Скачали образы и запустили gitlab CI 
Зарегестрировались, создали группу, создали проект
Подключили проект microservices
Создали и запушили .gitlab-ci.yml
Создали и запустили runner
Запустили пайплайн
Добавили тесты
Описали окружение, добавили два этапа staging и production
Добавили тэгирование 
Определили динамические окружения
Запушили сборку в origin

# Выполнено ДЗ №14 - Docker: сети, docker-compose

##  Основное ДЗ
Создали ветку docker-4
Подключились к ранее созданному Docker host
Потестировали работу с сетью в Docker
none, host, bridge
Установили Docker-compose
Создали docker-compose.yml
Поработали с переменными окрeжения
Базовое имя формируется из названия папки + название сервиса
Выполнили самостоятельные задания
запушили ветку на github

# Выполнено ДЗ №13 - Docker Микросервисы

##  Основное ДЗ
Создали ветку docker-3
Подключились к ранее созданному Docker host
Разбили приложение на несколько компонентов
Создали Dockerfile для сервисов post-py, comment, ui
Собрали, запустили и протестировали приложение
Выполнили самостоятельные задания
запушили ветку на github

# Выполнено ДЗ №12 - Docker Технологии контейнеризации

##  Основное ДЗ
Создали ветку docker-2
Установили docker и docker-machine
Проверили работу сервера и клиента
Выполнили простые действия в docker
Docker images, docker run, docker ps,Docker start & attach
Docker exec, docker commit, Docker kill & stop
Docker rm & rmi
Установили и настроили gcolud
Попрактиковались с docker machine
Поправили конфиги и собрали образ reddit
Запустили контейнер, настроили firewall
Зарегестрировались в dicker hub, запушили свой образ
Выполнили самостоятельные задания
запушили ветку на github

