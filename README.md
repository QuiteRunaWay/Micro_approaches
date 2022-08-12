# Домашнее задание к занятию "11.03 Микросервисы: подходы"

Вы работаете в крупной компанию, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps специалисту необходимо выдвинуть предложение по организации инфраструктуры, для разработки и эксплуатации.


## Задача 1: Обеспечить разработку

Предложите решение для обеспечения процесса разработки: хранение исходного кода, непрерывная интеграция и непрерывная поставка. 
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- Облачная система;
- Система контроля версий Git;
- Репозиторий на каждый сервис;
- Запуск сборки по событию из системы контроля версий;
- Запуск сборки по кнопке с указанием параметров;
- Возможность привязать настройки к каждой сборке;
- Возможность создания шаблонов для различных конфигураций сборок;
- Возможность безопасного хранения секретных данных: пароли, ключи доступа;
- Несколько конфигураций для сборки из одного репозитория;
- Кастомные шаги при сборке;
- Собственные докер образы для сборки проектов;
- Возможность развернуть агентов сборки на собственных серверах;
- Возможность параллельного запуска нескольких сборок;
- Возможность параллельного запуска тестов;

Обоснуйте свой выбор.

## Ответ: 

Как нам ранее на других курсах по обучению в данном направлении DevOps говорили, то непрерывная установка (в подавляющем большинстве случаев) осуществляется командой разработки. Крайне мало команд разработки, которые позволяют полный процесс сборки, тестирования, выкладку - отдать на автоматизацию. Практически всегда используется процесс непрерывной поставки, когда в итоговом счете всё же для выкладки кто-то должен нажать "заветную кнопку" чтобы продукт пошёл в прод.
Это я к тому, что процесс выбора продукта для разработки необходимо выбирать тот, который больше знает команда и где большинство из сотрудников обладает необходимыми навыками.

Ранее так же тут в курсе мы проходили такие продукты как GitLab и Jenkins.

Jenkins как раз имеет необходимые нам агенты, которые ставятся на сервера, и в зависимости от настроек, мы можем запускать даже на одной тестовой машине несколько сборок необходимого нам кода из нашего репозитория. В частности при обучении мы собирали и прокатывали нашу роль при помощь ansible (через Jenkins), которая хранилась в нашем репозитории в GitHub. 

Jenkins позволяет нам запустить как автоматический pipeline по событию, так и  инициировать задачу по сборке по кнопке. При этом, мы можем даже предварительно задать какие то условия, перед запуском сборки (я лично делал при выполнении задачи агентов Jenkins предварительное копирование файлов из одной директории в другую, с изменением прав доступа (обычный скрипт на bash)).

Jenkins позволяет так же для каждой нашей сборки указать дополнительные параметры, что нам и надо.

GitLab в свою очередь позволит нам хранить наш код, т.к. имеет встроенный Git. Большой плюс GitLab в том, что его можно развернуть на своих мощностях. Большим плюсом для наших разработчиков будет наличие компонента Own Wiki в GitLab, позволяющий создавать свои странички для проектов с произвольным содержимым. Ведь документация очень важный момент при разработке продукта.

В GitLab есть Packages & Registries - компонента для хранения артефактов и образов. В частности мы использовали во время учебы так же Maven как репозиторий для хранения. А Подсистема Registries будет использоваться для хранения docker-образов - тем самым выполняется еще одно из требований подбора среды для разработки.

В итоге, для решения поставленной задачи для разработки я бы предложил использовать следующий продукт (в связке): GitLab и Jenkins. Эти 2 продукта будут полностью покрывать все наши потребности, на мой взгляд.

## Задача 2: Логи

Предложите решение для обеспечения сбора и анализа логов сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- Сбор логов в центральное хранилище со всех хостов обслуживающих систему;
- Минимальные требования к приложениям, сбор логов из stdout;
- Гарантированная доставка логов до центрального хранилища;
- Обеспечение поиска и фильтрации по записям логов;
- Обеспечение пользовательского интерфейса с возможностью предоставления доступа разработчикам для поиска по записям логов;
- Возможность дать ссылку на сохраненный поиск по записям логов;

Обоснуйте свой выбор.

## Ответ:

Я опять, наверно, не буду "изобретать колесо", и прибегну к уже полученным знаниям в рамках курса DevOps-инженер. Ранее мы разбирали систему для сбора логов на основе ELK.

Следовательно выбор будет такой: будем использовать стэк ***Elasticsearch-Logstash-Kibana (ELK).***

Что это нам даст: 

В основе ELK лежит архитектура Hot-Warm, при которой кластерные узлы hot осуществляют сбор быстрых логов за короткий промежуток времени (час, день и пр.), а Warm узлы осуществляют хранение данных за пределами выбранного промежутка времени. Есть еще возможность использования Cold узлов, но это если требуется хранить данные очень старые, например месяц и ранее. Тем самым - мы получаем ***долговременное***  и ***централизованное*** хранение наших данных, можно сказать за любой требуемый период времени.

В качестве агента сбора логов будет использоваться ***Filebeat***, что даст минимальную нагрузку на целевую информационную систему. Он будет передавать данные в Logstash. Элементарная настройка Filebeat (которая заключается просто в указании источника данных и конечной точки сбора в конфигурационном файле) будет большим преимуществом в отношении других систем сбора логов.

Logstash за счет универсального обработчика данных grok будет парсить входные данные и складывать их в централизованную БД Elasticsearch.

Elasticsearch будет обеспечивать поиск и фильтрацию по записям логов. Elasticsearch обладает отличной отказоустойчивостью за счет логирования на каждое изменение данных в хранилище сразу на нескольких узлах.

В качестве графического интерфейс для отображения данных для пользователя будет использоваться Kibana. Она позволяет осуществлять вывод как текстовых данных, так и происзводить сортировку данных по определенным значениям. Данные можно представить в виде диаграмм или графиков.


## Задача 3: Мониторинг

Предложите решение для обеспечения сбора и анализа состояния хостов и сервисов в микросервисной архитектуре.
Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:
- Сбор метрик со всех хостов, обслуживающих систему;
- Сбор метрик состояния ресурсов хостов: CPU, RAM, HDD, Network;
- Сбор метрик потребляемых ресурсов для каждого сервиса: CPU, RAM, HDD, Network;
- Сбор метрик, специфичных для каждого сервиса;
- Пользовательский интерфейс с возможностью делать запросы и агрегировать информацию;
- Пользовательский интерфейс с возможность настраивать различные панели для отслеживания состояния системы;

Обоснуйте свой выбор.

## Ответ:

Для мониторинга каждый будет выбирать тот иснтрумент, или с которым он работал или с тем, с которым он сталкивался.
Текущие современные системы, такие как Zabbix, Prometheus - они все позволяют собирать требуемые нам параметры, будь то информация о железе или информация о работе самих сервисов.

Более наглядное представление, всё же, будет представлено в связке ***Prometheus + Grafana***.
Prometheus позволяет собирать все требуемые у нас метрики, а Grafana будет их визуализировать как нам надо. Плюс Grafana, мы её так же в рамках курса разворачивали и смотрели - это возможность создания для себя именно тех дашбордов с данными - которые, дейтсивтельно нужны. Можно создать данные для руководства, которые не должны вникать в суть технических моментов (количество запросов к сайту с разбивками успешности или нет, количетсво кликов на сайте и входа пользователей, наприемр, в корзину, чтобы понимать пользуется функционалом кто-то или нет и пр.) и до тех. параметров, которые нужны непосредственно техническому специалисту, будь то нагрузка на сеть, железо, статусы ответа с разбивкой по кодам от сервисов и пр.

#### Так же если подвести итог всем 3 выше заданным вопросам "какой же инструмент выбрать" - надо будет смотреть, на 1. А что текущая команда уже знает и умеет? 2. Сколько ресурсов есть у организации и есть ли возможность расширения (как железа, так и рабочей силы)? 3. Какие задачи преследуются: сделать хорошо и "навсегда" или "быстро, дешево и временно"?

#### Это я к тому, что есть как условно-бесплатные решения, так и платные решения с расширенным функционалом и поддержкой от разработчика продукта. Всё относительно и всё необходимо выбирать по нуждам и требованиям организации.
