---
Инфо:

---Логика приложения:

Приложение - бот для указания оставшихся дней отпуска, на любой день начиная с текущего.
При обращении пользователя к боту с запросом, он залезает в БД проверяет есть ли его id в "белом" списке и если он есть, то выдает информацию высчитывая сколько их осталось
Паралельно периодически обновляется информация из таблицы HR с отпусками из которой в БД приложения подгружаются данные

---Инфраструктурные моменты:

Сервер инфраструктуры на котором подняты gitlab-ci, gitlab-runner, ELK. Prometheus в docker контейнерах
Два сервера приложения - test и prod, на которых подняты по два контейнера с postgresql и приожением на python

Мониториг через prometheus, оповещение через телеграм бота

Приложения запущенны на портах:
Gitlab - 20
ELK - 5044 \ 9200 \ 5601
Grafana - 3000
Prometheus - 9090
Postgres_exporter - 9187
Alertmanager - 9093
prometheus_bot - 9087

---Обновление:

После коммита с тегом удаляется старый тест-сервер, передаются параметры и запускается новый
Для обновления на прод необходимо запустить обновление вручную

---
Установка (ручками-ручками):

1. Настроить переменные для infrastructure/terraform.tfvars по аналогии с infrastructure/terraform.tfvars.example

2. Получить от GCP секретный docker.json и положить его в папки infrastructure и devops

3. Из папки infrastructure создать виртуалку командой:
`terraform apply`

4. Дождаться загрузки gitlab по полученному ip от terraform

5. В gitlab, завести пароль для root, создать проект services (internal)

6. Получить два токена от ботов для теста и прода

7. Создать секретные переменные в gitlab CI\CD

POSTGRES_PASSWORD_TEST - пароль от postgresql для тест

POSTGRES_PASSWORD - пароль от postgresql для прода

VACANCY_TOKEN_TEST - токен телеграм бота для теста

VACANCY_TOKEN - токен телеграм бота для прода

BIRTHDAY_TOKEN_TEST - токен телеграм бота для теста

BIRTHDAY_TOKEN - токен телеграм бота для прода

DEVELOPER_ID_TEST - id телеграм разработчика для теста

DEVELOPER_ID - id телеграм разработчика для прода

HR_ID_TEST - id телеграмм HR для теста

HR_ID - id телеграмм HR для прода

HR_STAFFF - "Ник HR в телеграмме"

PRIVATE_KEY_ID_VARIABLE - секретный ид для подключения к гугл таблице HR

PRIVATE_KEY_VARIABLE - секретный rsa ключ для подключения к гугл таблице HR

ELK_HOST - "ip сервера инфраструктуры"

DEVOPS_PROJECT_ID - id проекта GCP в котором разворачаваются сервера

DEVOPS_PRIVATE_KEY_ID_VARIABLE - секретный ид для управления GCP

DEVOPS_PRIVATE_KEY_VARIABLE - секретный rsa ключ для управления GCP

DEVOPS_CLIENT_EMAIL - секретный email для управления GCP

DEVOPS_CLIENT_ID - секретный клиент ид для управления GCP

DEVOPS_CLIENT_CERT_URL - секретный урл клиента с сертификатом для управления GCP

PROJECT_DUMP - ID проекта со стартовым дампом

8. Выполнить на сервере инфраструктуры скрипт используя полученный токен
cd /opt && sudo ./register-gitlab-runner.sh <token>

9. Положить ключ из вывода скрипта в GCP для доступа

10. В gitlab залить код и поставить новый тэг

11. В kibana (:5601) надо сконфигурировать index pattern:
`filebeat-*`

12. На сервере инфраструктуры запустить скрипт для включения мониторинга:
`add_prod_to_monitoring.sh <ip prod>`

13. Добавить в Grafana дашборд из папки infrastructure/monitoring/grafana

14. Профит


P.S. После обновления приложения на проде необходимо в мистеме мониторинга подставить новый ip и пересобрать приложения