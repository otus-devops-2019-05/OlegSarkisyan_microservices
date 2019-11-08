# Проектная работа по DevOps. Команда Homecredit.

### О проекте
Наша проектная работа посвящена использованию DevOps практик, в частности настройке CI/CD механизмов, в рамках внутребанковского мобильного приложения Наш Хоум. Это приложение для сотрудников банка, обладающее следующей функциональностью:
1.	Новостные ленты нескольких внутренних порталов
2.	Адресная книга по всем сотрудникам
3.	Модуль Геймификации (игра с реальными KPI)
4.	HR-сервисы (Отпуска, ДМС, заказ справок, отправка документов,  подбор сотрудников, полезная информация)
5.	Ипотечный калькулятор, кредит наличными
6.	Приложение имеет серверную часть и мобильные клиенты (Android, IOS). 

### Участники проекта
Чирков Сергей, Втулкин Дмитрий, Малышев Алексей, Азаркин Иван, Саркисян Олег.

### Этапы
В качестве проектной работы наша команда реализует следующие этапы (согласно требованиям к проектной работе и ограничениям банковской инфраструктуры):
1.	Инфраструктура для CI/CD – у нас используется Jenkins (общебанковское решение)
2.	Подготовка инфраструктуры для сбора обратной связи – Prometheus, grafana
3.	Подготовка инфраструктуры для приложения  – docker, rancher(тестовый и продуктовый стенды по 2 хоста каждый)
4.	Настройка механизмов CI/CD – Jenkins (описание и структура job)
5.	Настройка мониторинга – Prometheus, grafana,Cadvisor, node exporter 
6.	Настройка логирования – elastic, logstash, graylog 
7.	Тесты – планируется написание Smoke и адаптация старых unit тестов
8.	Документация – Readme по проекту, архитектурная схема приложения 
9.	Все файлы, конфиги и описание проекта будет хранится на github – за исключением файлов мобильного приложения (политика службы безопасности). Также часть данных в конфигах будет обезличена (ссылки на внутренние ресурсы и секреты)
10.	Скринкасты следующих частей проекта – выполнения job в Jenkins презентующего сборку сервера приложения, показ инфраструктуры в Rancher, визуализация мониторинга в grafana, результат работы парсера в graylog, собственно работа приложения

### Установка
Установка приложения разделена на три этапа:
1.	Установка сервера приложения (в нашем случае, деплой в ранчер)
2.	Заливка мобильного клиента в app store и play market
3.	Скачивание и установка приложения конечным пользователем

### Файлы в репозитории (по разделам, с описанием):

### Docker 

#### Dockerfile (Используется для сборки приклада).      
FROM containerization/devops-base-java-x86_64:latest   # откуда берем эталонный образ
--------------------------------------  Задаем переменные -------------------------------------------------------
ARG SPRING_ENVIRONMENT_ARG
ENV SPRING_PROFILE=$SPRING_ENVIRONMENT_ARG
ENV SAP_INFO_HOST postgres.sap-info.renv-XXXX.rancher.homecredit.ru
ENV SAP_INFO_PORT 56390
ENV SAP_INFO_DB sap-info
ENV SAP_INFO_USER etl
ENV INTRANET_USERNAME intranetuser
ENV SALESPORTAL_USERNAME salesportal_username
ENV INTRANET_PASSWORD_FILE /var/secret/intranet_password
ENV SALESPORTAL_PASSWORD_FILE /var/secret/salesportal_password
ENV POSTGRES_PASSWORD_FILE /run/secrets/pg_password
ENV SAP_INFO_PASSWORD_FILE /run/secrets/sap_password
ENV SYSTEMAUTH_CLIENTSECRET_FILE /run/secrets/systemauth_password
COPY docker-entrypoint.sh /usr/local/bin/  # копируем файл
RUN chmod 777 /usr/local/bin/docker-entrypoint.sh # Даем права на запуск
RUN ln -s usr/local/bin/docker-entrypoint.sh 
ENTRYPOINT ["docker-entrypoint.sh"] # запускаем скрипт
COPY build/libs/*.jar /data/server.jar  # копируем jar’ник
EXPOSE 8080  # порт который слушает сервис в запущенном контейнере
RUN chmod o+rwx /tmp # даем права
CMD java -Xmx2048M -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8987 -jar /data/server.jar --spring.profiles.active=${SPRING_PROFILE} --server.port=8080 -Duser.timezone=GMT  # запускаем приложение на порту 8080

#### Docker-compose (для деплоя приложения)  
version: '2'
services:
server:
image: artifactory.homecredit.ru:xxxx/xxxx/${homnet_server_image} # какой образ деплоить
environment:
INTRANET_PASSWORD_FILE: /run/secrets/intranet_password   # определяем переменные
INTRANET_USERNAME: intranetuser
POSTGRES_PASSWORD_FILE: /run/secrets/pg_password
SALESPORTAL_PASSWORD_FILE: /run/secrets/salesportal_password
SALESPORTAL_USERNAME: salesportal_username
SAP_INFO_PASSWORD_FILE: /run/secrets/sap_password
SYSTEMAUTH_CLIENTID: home-network
SYSTEMAUTH_CLIENTSECRET_FILE: /run/secrets/systemauth_password
AUTOTESTER_DOMAIN_NAME: XXXX@homecredit.ru
AUTOTESTER_PASSWORD_FILE: /run/secrets/autotester_password
labels:  # задаем метки для приложения и хостов
io.rancher.scheduler.affinity:host_label: operation_support=true 
io.rancher.scheduler.affinity:container_label_ne: io.rancher.stack_service.name=$${stack_name}/$${service_name}
logging:  
driver: gelf  # подключаем драйвер gelf
options:
gelf-address: ${log_collector} # задается переменная для отпраки логов.
tag: ${tag_environment} # задается среда откуда забирать логи
secrets: # пробрасываюся секреты в Rancher
- mode: '674'
uid: '0'
gid: '0'
source: HOMNET_SYSTEM_AUTH_PASSWORD
target: systemauth_password
- mode: 674
uid: '0'
gid: '0'
source: HOMNET_SALESPORTAL_PASSWORD
target: salesportal_password
- mode: 674
uid: '0'
gid: '0'
source: HOMNET_INTRANET_PASSWORD
target: intranet_password
- mode: 674
uid: '0'
gid: '0'
source: HOMNET_POSTGRES_PASSWORD
target: pg_password
- mode: 674
uid: '0'
gid: '0'
source: HOMNET_SAP_DB_PASSWORD
target: sap_password
- mode: '674'
uid: '0'
gid: '0'
source: HOMNET_AUTOTESTER_PASSWORD
target: autotester_password
secrets:
HOMNET_AUTOTESTER_PASSWORD:
external: 'true'
HOMNET_SYSTEM_AUTH_PASSWORD:
external: 'true'
HOMNET_SAP_DB_PASSWORD:
external: 'true'
HOMNET_POSTGRES_PASSWORD:
external: 'true'
HOMNET_SALESPORTAL_PASSWORD:
external: 'true'
HOMNET_INTRANET_PASSWORD:
external: 'true'

#### docker-entrypoint.sh
Нужен для проброса значений секретов в  каждую из переменных сред. 
Это позволит экспортировать значение, хранящееся в секрете, в правильную переменную среды.
#!/usr/bin/env bash
set -e    # если произошла ошибка, скрипт прекращает "бежать"
file_env() {  
local var="$1"
local fileVar="${var}_FILE"
local def="${2:-}"
if [ "${!var:-}" ] && [ "${!fileVar:-}" ]; then
echo >&2 "error: both $var and $fileVar are set (but are exclusive)"
exit 1
fi
local val="$def"
if [ "${!var:-}" ]; then
val="${!var}"
elif [ "${!fileVar:-}" ]; then
val="$(< "${!fileVar}")"
fi
export "$var"="$val"
unset "$fileVar"
}
file_env 'SYSTEMAUTH_CLIENTSECRET'
file_env 'AUTOTESTER_PASSWORD'
file_env 'SAP_INFO_PASSWORD'
file_env 'SALESPORTAL_PASSWORD'
file_env 'INTRANET_PASSWORD'
file_env 'POSTGRES_PASSWORD'
exec "$@"


### Jenkins и Деплой в рачнер 

#### Rancher-compose.yml (Используется для деплоя на несколько хостов, для добавления health check, стратегий развертывания и является неотемлемой частью оркестратора Rancher 1.6 (Cattle)). 
version: '2'
services:
server:
scale: 2  # на сколько хостов скейлить наше приложение
start_on_create: true  # создаем контейнер
upgrade_strategy: # 
start_first: true # сначала стартуем новый контейнер, затем "убиваем" старый
health_check: # проверка здоровья
healthy_threshold: 2 # Число успешных проверочных ответов перед (в настоящее время помеченным как нездоровый) контейнер снова считается исправным.
response_timeout: 2400000 # Количество миллисекунд до того, как мы завершим инициализацию и ждем ответа
recreate_on_quorum_strategy_config:
quorum: 1
port: 8080
unhealthy_threshold: 3 # Количество неудачных проверочных ответов до того, как контейнер (в настоящее время помеченный как исправный) считается нездоровым.
interval: 2000 
strategy: recreateOnQuorum # Что делать если  получен лимит неудачных проверочных ответов (в нашем случае 5).  Recreate -  Означает, что Rancher уничтожит нездоровый контейнер и создаст новый контейнер
request_line: GET "/actuator/health" "HTTP/1.0" # Посылает HTTP запрос методом GET и ждет ответа 200 ОК
reinitializing_timeout: 300000 

#### Jenkinsfile (Этот конвеер использется для сборки, деплоя, прогона тестов).  
node('jenkins-slave2-centos7') {  # Nod'a для запуска 
try {  # блок объявления переменной
env.no_proxy = '*.homecredit.ru'  # не использовать proxy для *.homecredit.ru
def tag = env.BRANCH_NAME # приравниваем тег к ветке
def patternBase = '^v\\d+\\.\\d+\\.\\d+' # формат тегов
def releasePattern = patternBase.concat('$') # формат тегов для боевой среды
def testPattern = patternBase.concat('-(alpha|beta|rc)\\d*$') # формат тегов для тестовой среды
boolean releaseBuild = tag =~ releasePattern #  определяем переменную как boolean для тестовой среды
boolean testBuild = tag =~ testPattern #  определяем переменную как boolean для прод среды
def imageNameForRancher # определяем переменную для ранчера
def imageNameForArtifactory  # определяем переменную для артифактори
if (releaseBuild) {  # условия выбора тегов
imageNameForRancher = "homnet-server:${tag.substring(1)}"
} else if (testBuild) {
imageNameForRancher = "homnet-server-test:${tag.substring(1)}"
} else {
echo "Tag ${tag} does not match expected format. Expected format is : v1.2.3 with optional -alpha -beta -rc suffixes"
return
}
imageNameForArtifactory = 'applications/'.concat(imageNameForRancher) # приравниваем переменные
stage('Clear workspace') {
deleteDir()
}
stage('Git checkout') { # шаг для стягивания кода с gita
checkout scm
}
def branch = sh script: 'git branch -r --contains', returnStdout: true
def isMaster = branch.contains("origin/master") # определяем переменную для ветки origin/master
def isHotfix = branch.contains("origin/hotfix/")  # определяем переменную для ветки origin/hotfix
if (releaseBuild && !isMaster && !isHotfix) { # задаем условие выбора выбора веток в зависимости от тега
throw new Exception("Deployment from ${branch} is not supported.")
}
stage('Gradle build') { # шаг сборки приложения
sh "chmod +x gradlew && ./gradlew clean build" #  даем права на запуск и задаем цель сборки
}
stage('Generate documentation'){   # шаг документируем 
sh "chmod +x gradlew && ./gradlew dokka" 
}
stage('Archive documentation'){
archiveArtifacts artifacts: 'build\\javadoc\\**', onlyIfSuccessful: true
}
stage('Upload to Artifactory') { # заливаем в артифактори
def artifactoryCredentialsId = "jenkins_artifactory" # определяем переменную для креденшелса
def server = Artifactory.newServer url: 'http://artifactory.homecredit.ru:ХХХХ/ХХХХ/', credentialsId: 'artifactoryCredentialsId' # определяем переменную для URL 
server.bypassProxy = true
def uploadSpec = """{
"files": [
{
"pattern": "build/libs/homnet-server*.jar",
"target": "gradle-local/homnet-server/homnet-server-${tag.substring(1)}.jar"
}
]
}"""
def buildInfo = server.upload(uploadSpec)
server.publishBuildInfo(buildInfo)
}
stage('Docker build and push') { # Собираем билд и пушим в артифактори
def java8ArtImage = "artifactory.homecredit.ru:ХХХХ/ХХХХ/devops-base-java-x86_64:latest" #  определяем переменную для артифактори
def java8LocalImage = "containerization/devops-base-java-x86_64:latest" #  определяем переменную для 'эталонной образа 
def springEnvironment  # определяем переменную для среды
if (releaseBuild) {
springEnvironment = 'prod' # приравниваем к Prod среде
} else
springEnvironment = 'staging' # приравниваем к тестовой среде
def systemAuthProps = readProperties file: env.HOMNET_system_auth_properties
docker.withRegistry('https://artifactory.homecredit.ru:ХХХХ', 'cicdhomerdocker') {
sh "docker pull ${java8ArtImage}" # стягиваем образ с артифактори
sh "docker tag ${java8ArtImage} ${java8LocalImage}" # стягиваем эталонный образ
docker.build("${imageNameForArtifactory}", "--build-arg SPRING_ENVIRONMENT_ARG=${springEnvironment} --build-arg SYSTEMAUTH_CLIENTID_ARG=${systemAuthProps['system-auth-client-id']} .").push() # собираем и пушим
sh "docker rmi ${imageNameForArtifactory} ${java8LocalImage} ${java8ArtImage}" # удаляем локально собранный образ
}
}
stage ('Add to rancher') {   # шаг деплоя и тестирования
def smoke # определяем переменную
if (releaseBuild) { # задаем условие выбора параметров
env.credentialsIdVar = '123Dasd2dsad12dasdasd' # переменная (логин и пароль для деплоя)
env.credentialsIdVar1 = 'credentials_SmokeTest_app' # переменная (логин и пароль для деплоя)
env.rancherUrl = 'https://rancher.homecredit.ru/ХХХХ'  # переменная URL'a для прода Ranchera
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ'  # переменная URL'a для logstash
env.tag_environment = 'ourhome-prod'                            # задается среда откуда забирать логи
smoke = "chmod +x smoke_prod.sh && . smoke_prod.sh"   # даем права на запуск и запускаем smoke тест после деплоя 

} else {
env.credentialsIdVar = 'ХХХХХХХХХ' # переменная (логин и пароль для деплоя)
env.credentialsIdVar1 = 'credentials_SmokeTest_app'  # переменная (логин и пароль для деплоя)
env.rancherUrl = 'https://rancher-test.homecredit.ru/ХХХХ' # переменная URL'a для прода Ranchera
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ' # переменная URL'a для logstash
env.tag_environment = 'ourhome-test'                            # задается среда откуда забирать логи
smoke = "chmod +x smoke_test.sh && . smoke_test.sh"  # даем права на запуск и запускаем smoke тест после деплоя 
}
withCredentials([usernamePassword(credentialsId: credentialsIdVar, # вытягиваем логин и пароль и креденшелов Jenkins
usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_KEY')]) {  # парсим логин и пароль
sh "homnet_server_image=${imageNameForRancher} rancher" +   # запускаем делой в зависимости от среды и подставляем вытянутые переменные
' --url ' + rancherUrl + 
' --access-key ' + ACCESS_KEY +
' --secret-key ' + SECRET_KEY +
' up -s home-network -d --force-upgrade --confirm-upgrade'
}
withCredentials([usernamePassword(credentialsId: credentialsIdVar1,  # вытягиваем логин и пароль и креденшелов Jenkins
usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){    # парсим логин и пароль
sh smoke  # запуск скрипта
}
}
emailext attachLog: true, body: "BUILD_URL: ${env.BUILD_URL}", subject: "${env.JOB_NAME} ${env.BUILD_DISPLAY_NAME} completed successfully", to: 'ХХХХ@homecredit.ru'
}    # отправка по почте о результе деплоя. 
catch (Exception e) {  # обработчик ошибок 
emailext attachLog: true, body: "BUILD_URL: ${env.BUILD_URL}", subject: "${env.JOB_NAME} ${env.BUILD_DISPLAY_NAME} failed", to: 'ХХХХ@homecredit.ru'
throw e
}
}

### Мониторинг
Основной скрипт для установки/запуска - RunScripts.sh

#### Установка/запуск Grafana
Выполняем несколько простых комманд, которые описаны на сайте - Grafana

wget <debian package url>
sudo apt-get install -y adduser libfontconfig1
sudo dpkg -i grafana_<version>_amd64.deb

#### Установка/запуск Docker
Выполняем несколько простых комманд, которые установят нам и запустят Docker под Ubuntu - Docker
sudo apt-get update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
Т.к. прямой доступ в интернет запрещен, у нас работает проксирование через Artifactory до GitHub, поэтому нам понадобиться в /etc/docker/ создать файл daemon.json и заполнить его строчкой { "insecure-registries":["artifactory:ХХХХ"] }, чтобы не возникало ошибки при авторизации.

Error response from daemon: Get https://artifactory:ХХХХ/ХХХХ/ХХХХ/:x509:certificate signed by unknown authority
После этого авторизуемся в Docker
docker login -u="login" -p="password"
Устанавливаем/запускаем Prometheus и его exporters
Для проекта мы взяли два основных exporters - CAdvisor и NodeExporter

#### Запуск Prometheus
sudo docker run -d \
 --volume=${PWD}/Prometheus/prometheusotus.yml:/etc/prometheus/prometheus.yml \
 --publish=9095:9090 \
 --name=prometheus \
 --restart=always \
 artifactory:6555/prom/prometheus:latest

#### Запуск CAdvisor
sudo docker run -d \
 --volume=/:/rootfs:ro \
 --volume=/var/run:/var/run:ro \
 --volume=/sys:/sys:ro \
 --volume=/var/lib/docker/:/var/lib/docker:ro \
 --volume=/dev/disk/:/dev/disk:ro \
 --publish=9092:8080 \
 --restart=always \
 --name=cadvisor \
 artifactory:6555/google/cadvisor:latest

#### Запуск NodeExporter
sudo docker run -d \
 --net="host" \
 --pid="host" \
 -v "/:/host:ro,rslave" \
 --name=nodeexporter \
 --restart=always \
 artifactory.ru:6555/prom/node-exporter:latest

В Rancher окружении подняты контейнеры для мониторинга метрик хоста и контейнеров  cadvisor, node-exporter.
Сами метрики собираются Prometheus , 
Нужно открыть доступы до ВМ в ажуре prometheus(AZURE)_to_cadvisor_RENVХХХХ
http://ХХХХ:ХХХХ/ХХХХ   - прометей
http://ХХХХ:ХХХХ/              - графана

#### CAdvisor

Для сборки  и деплоя  приложения необходимы следующие  конфигурационные файлы:

##### Docker-compose.yml (Используется для деплоя готового образа).
 
version: '2'
services:
cadvisor:
image: artifactory.homecredit.ru:ХХХХ/ХХХХ/cadvisor:latest
ports:
- 9092:8080/tcp
labels:
io.rancher.container.agent.role: environmentAdmin
io.rancher.container.create_agent: 'true'
io.rancher.scheduler.affinity:host_label: operation_support=true
io.rancher.scheduler.affinity:container_label_ne: io.rancher.stack_service.name=$${stack_name}/$${service_name}
volumes:
- /:/rootfs:ro
- /var/run:/var/run:ro
- /sys:/sys:ro
- /var/lib/docker/:/var/lib/docker:ro
- /dev/disk/:/dev/disk:ro
logging:
driver: gelf
options:
gelf-address: ${log_collector}
tag: ${tag_environment}
     
##### Rancher-compose.yml
version: '2'
services:
cadvisor:
scale: 1
start_on_create: true
upgrade_strategy:
start_first: true

##### Jenkinsfile
node('jenkins-slave2-centos7') {
try {
def environment = ENVIRONMENT
def test = environment == "TEST"
def prod = environment == "PROD"
stage('Clear workspace') {
deleteDir()
}
stage('Git checkout') {
checkout scm
} 
stage ('Deploy to Rancher') {
def credentialsIdVar
def rancherUrl
if (prod) {
credentialsIdVar = '123Dasd2dsad12dasdasd'
rancherUrl = 'https://rancher.homecredit.ru/ХХХХ'
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ'
env.tag_environment = 'ourhome-prod'
} else if (test) {
credentialsIdVar = 'ХХХХХХХХХХХХ'
rancherUrl = 'https://rancher-test.homecredit.ru/ХХХХ'
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ'
env.tag_environment = 'ourhome-test'
}
withCredentials([usernamePassword(credentialsId: credentialsIdVar,
usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_KEY')]) {
sh "rancher " +
' --url ' + rancherUrl +
' --access-key ' + ACCESS_KEY +
' --secret-key ' + SECRET_KEY +
' up -s cadvisor -d --force-upgrade --confirm-upgrade'
}
}
}
catch(Exception e) {echo e.getMessage()}

#### Node-export

##### Docker-compose.yml (Используется для деплоя готового образа).
version: '2'
services:
nodeexporter:
image: artifactory.homecredit.ru:ХХХХ/prom/node-exporter:latest
user: root
ports:
- 9100:9100/tcp
labels:
io.rancher.container.agent.role: environmentAdmin
io.rancher.container.create_agent: 'true'
io.rancher.scheduler.affinity:host_label: operation_support=true
io.rancher.scheduler.affinity:container_label_ne: io.rancher.stack_service.name=$${stack_name}/$${service_name}
volumes:
- /proc:/host/proc:ro
- /sys:/host/sys:ro
- /:/rootfs:ro
command:
- '--path.procfs=/host/proc'
- '--path.sysfs=/host/sys'
- '--collector.filesystem.ignored-mount-points="^/(sys|proc|dev|host|etc)($$|/)"'
logging:
driver: gelf
options:
gelf-address: ${log_collector}
tag: ${tag_environment}

##### Rancher-compose.yml
version: '2'
services:
nodeexporter:
scale: 2
start_on_create: true
upgrade_strategy:
start_first: true

##### Jenkinsfile
node('jenkins-slave2-centos7') {
try {
def environment = ENVIRONMENT
def test = environment == "TEST"
def prod = environment == "PROD"
stage('Clear workspace') {
deleteDir()
}
stage('Git checkout') {
checkout scm
}
stage ('Deploy to Rancher') {
def credentialsIdVar
def rancherUrl
if (prod) {
credentialsIdVar = 'ХХХХХХХХХХХХХХ'
rancherUrl = 'https://rancher.homecredit.ru/ХХХХ'
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ'
env.tag_environment = 'ourhome-prod'
} else if (test) {
credentialsIdVar = 'ХХХХХХХХХХХХХХ'
rancherUrl = 'https://rancher-test.homecredit.ru/ХХХХ'
env.log_collector = 'udp://os-ХХХХ.homecredit.ru:ХХХХ'
env.tag_environment = 'ourhome-test'
}
withCredentials([usernamePassword(credentialsId: credentialsIdVar,
usernameVariable: 'ACCESS_KEY', passwordVariable: 'SECRET_KEY')]) {
sh "rancher " +
' --url ' + rancherUrl +
' --access-key ' + ACCESS_KEY +
' --secret-key ' + SECRET_KEY +
' up -s nodeexporter -d --force-upgrade --confirm-upgrade'
}
}
}
catch(Exception e) {echo e.getMessage()}
}

### Логирование 

#### Logstash
В docker-compose.yml подключаем драйвер gelf и описаны переменные для подключения адреса и среды деплоя
logging:
driver: gelf
options:
gelf-address: ${log_collector}
tag: ${tag_environment}
На хосте os-ХХХХ сделан инстанс ourhome
/etc/logstash-logstash-ourhome/
/etc/logstash-clientservices/pipelines.yml # объявляются количество точек входа
- pipeline.id: test
path.config: "/etc/logstash-logstash-ourhome/conf.d/logstash-ourhome-test.conf"
pipeline.workers: 1
- pipeline.id: prod
path.config: "/etc/logstash-logstash-ourhome/conf.d/logstash-ourhome.conf"
pipeline.workers: 1
etc/logstash-logstash-ourhome/conf.d/logstash*   # объявляется на какой порт принимать и отправлять логи + фильтр для парса логов.

##### logstash.yml
input {
gelf {
port => ХХХХ     # порт на котором принимаем логи
type => "logstash-ourhome"
}
}
filter {
if [type] == "ourhome-prod" {
mutate {
gsub => ["message", "\x1B\[([0-9]{1,2}(;[0-9]{1,2})?)?[m|K]", ""]
}
grok {
match => { "message" => "^{+.*" }
add_tag => ["json_type"]
tag_on_failure => ["json_parse_fail"]
}
}
if "json_type" in [tags]{
json {
source => "message"
}
}
if "json_parse_fail" in [tags]{
grok {
match => { "message" => "^[0-9a-zA-Z]+.*" }
remove_tag => ["json_parse_fail"]
add_tag => ["string_type"]
}
}
}
output {
gelf {
host => "ХХХ.ХХХ.ХХХ.ХХХ" # port ХХХХ -for "ourhome" stream on Graylog in PROD
port => ХХХХ
}
}

##### elasticsearch.yml
node.name: "os-ХХХХ.ХХХХ.dev.itc.homecredit.ru"
path.logs: "/var/log/ХХХХ/elasticsearch"

network.host: 0.0.0.0
http.port: ХХХХ
transport.tcp.port: ХХХХ


### Ansible

Jenkins устанавливается и настраивается ролью ansible

#### Task/main.yml
---
- name: Install Java # устанавливаем Java
  become: yes
  yum: name={{ java_pkg }}
- name: Install Jenkins # устанавливаем  jenkins
  become: yes
  yum: name=http://repo.jenkins.rpm state=present disable_gpg_check=yes
- name: Get jenkins user home directory # 
  shell: echo ~jenkins
  register: jenkins_user_home
- name: Add to sudoers # Добавляем судо для дженкинса
  become: yes
  copy:
    src=sudo_jenkins
    dest=/etc/sudoers.d/sudo_jenkins
    mode=0440
- name: Create directories for logs # создаем директорию для хранения логов  Jenkins
  become: yes
  file: path="{{project_log}}/jenkins" owner=jenkins group=jenkins state=directory mode=0755
- name: Replace log directories # изменяем вывод лога с дефолтного на /var/log/name_project/Jenkins/
  become: yes
  replace: dest=/etc/init.d/jenkins regexp="/var/log/jenkins" replace="{{project_log}}/jenkins"
  notify:
  - Restart jenkins
- name: Create directories for plugins # создаем дир для плагин
  become: yes
  file: path="{{jenkins_home}}/plugins" owner=jenkins group=jenkins state=directory mode=0755
- name: Install git-client plugin with dependencies # ставим плагин и завсисимости
  become: yes
  get_url: url="http://repo.jenkins-ci.plugins&a={{item.name}}&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/{{item.name}}.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  with_items:
    - name: git
    - name: junit
    - name: mailer
    - name: matrix-project
    - name: scm-api
    - name: script-security
    - name: credentials
    - name: git-client
    - name: ssh-credentials
  notify:
  - Restart jenkins
- name: Install jobConfigHistory plugin with dependencies # ставим плагин
  become: yes
  get_url: url="http://repo.maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a={{item.name}}&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/{{item.name}}.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  with_items:
    - name: jobConfigHistory
    - name: maven-plugin
    - name: javadoc
  notify:
  - Restart jenkins
- name: Install Throttle Concurrent Builds plugin # ставим плагин
  become: yes
  get_url: url="http:/repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=throttle-concurrents&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/throttle-concurrents.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install Crowd2 plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=crowd2&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/crowd2.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install ansicolor plugin # ставим плагин
  become: yes
  get_url: url="http://repo/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=ansicolor&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/ansicolor.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install build-name-setter plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a={{item.name}}&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/{{item.name}}.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  with_items:
    - name: build-name-setter
    - name: token-macro
  notify:
  - Restart jenkins
- name: Install sectioned-view plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=sectioned-view&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/sectioned-view.jpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install parameterized-trigger plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=parameterized-trigger&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/parameterized-trigger.hpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install description-setter plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=description-setter&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/description-setter.hpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install rebuild plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=rebuild&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/rebuild.hpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install build timeout plugin # ставим плагин
  become: yes
  get_url: url="http://repo/maven/redirect?r=thirdparty&g=org.jenkins-ci.plugins&a=build-timeout&v=LATEST&p=hpi" dest={{jenkins_home}}/plugins/build_timeout.hpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Install ldap plugin # ставим плагин ldap
  become: yes
  get_url: url="https://repo/jenkins-ci.plugins/ldap.hpi" dest={{jenkins_home}}/plugins/ldap.hpi owner=jenkins group=jenkins mode=0644 validate_certs=no
  notify:
  - Restart jenkins
- name: Copy config.xml file for setup views # копируем конфиг jenkins
  become: yes
  template: src="config.xml" dest="{{jenkins_home}}/config.xml" owner=jenkins group=jenkins mode=0644
  notify:
  - Restart jenkins
- name: Create directory for prepare jobs # создание директории для джоба
  become: yes
  file: path="{{jenkins_home}}/jobs/{{item}}" state=directory owner=jenkins group=jenkins mode=0755
  with_items:
  - "Homnet.Server.Deploy"
  notify:
    - Reload Jenkins
- name: Create prepare jobs # копирование html джоба расположенного \jenkins\templates\ Homnet.Server.Deploy
  become: yes
  template: src="{{item}}.xml" dest="{{jenkins_home}}/jobs/{{item}}/config.xml" owner=jenkins group=jenkins mode=0644
  with_items:
    - "Homnet.Server.Deploy "
  notify:
    - Reload Jenkins
- name: Start Jenkins # стартуем jenkins
  become: yes
  service: name=jenkins state=started

#### vars\main.tf
java_home: "/usr/lib/jvm/java-1.8.0"
jenkins_home: "/opt/jenkins"
jenkins_default_view_name: "OMNI - {{inventory_file | basename}}"

#### handlers\main.tf
хендлеры для старта, рестарта релоада jenkins
---
- name: Start jenkins
  become: yes
  service: name=jenkins state=started enabled=yes
- name: Reload Jenkins
  shell: curl --request POST http://{{jenkins_username}}:{{jenkins_password}}@{{groups['jenkins'][0]}}:{{port_jenkins}}/jenkins/reload
- name: Restart jenkins
  shell: curl --request POST http://{{jenkins_username}}:{{jenkins_password}}@{{groups['jenkins'][0]}}:{{port_jenkins}}/jenkins/safeRestart

#### files/ sudo_jenkins
содержимое файла /etc/sudoers.d/sudo_jenkins
Defaults:jenkins !requiretty
jenkins ALL=(ALL) NOPASSWD: ALL

