# ChatGPT

# 1. Тема, MVP, анализ трафика
ChatGPT — это чат-бот на основе нейросети, генерирующий тексты, похожие на написанные человеком

### MVP
1. Регистрация/авторизация
2. Написание и отправка запроса в текстовом формате
3. Отправка изображений
4. Генерация текстового ответа
5. Поиск по истории запросов
6. Просмотр истории запросов

### Анализ трафика
- MAU 180M [1]
- DAU 120M [1]
- Среднее время посещения - 6 минут 14 сек [2]
- В среднем 1 млрд запросов в день [2]
- ~ 10 млн пользователей преимиум подписки [3]


#### Использование ChatGPT по странам
Процент стран по количеству пользователей [[2]]

| Страна    | Количество пользователей, % |
|-----------|:-----------------------------:|
| США     |              14,07              |
| Индия  |              9,5             |
| Бразилия |              4,73              |
| Великобритания       |              3,88              |
| Индонезия |              3,84               |
|   Другие |              63,98               |

Аудитория по возрасту:
- 15% в возрасте 18-29 лет
- 17% в возрасте 30-44 года
- 9% в возрасте 45-64 года
- 5% в возрасте 65+

![image](https://github.com/user-attachments/assets/d21c0384-253e-4a32-9f1b-390b2562cefa)

# 2. Расчет нагрузки

- MAU 180M [1]
- DAU 120M [1]

В чатгпт максимальная длина токенов на вопрос/ответ 4096 для стандартного пользователя, один токен это где-то 4 символа. Возьмем в качестве среднего значения половину от максимального значения - 2048 токенов = 8192 символа. Для пользователя премиум максимум 128 тыс токенов, аналогично возьмем для него половину получим 256 тыс символов в среднем.
В качестве кодировки символов выберем кодировку UTF-8, следовательно, 1 символ весит в среднем 2 байта. Тогда на одного пользователя потребуется: Бесплатный тариф - 16кб байт, платный - 512 кб

Максимум 512 Мб общий размер файлов в диалоге. Возьмем половину от этого значения в качестве среднего размера диалога. Максимум 40-80 запросов к чату в час, поэтому предположим, что на одно сообщение с файлом в среднем приходится примерно 8 Мб, если брать в качестве среднего значения количества сообщений 30 (среднее от половины максимумов)(то есть берем 256 Мб среднего значения размера файлов и делим на количество сообщений). 

Данные необходимые для регистрации:
| Данные    | Длина | Размер | 
|-----------|-----------------------------|-----------|
| Имя     |      25 |        50 байт              |
| Email     |      25 |        50 байт              |
| Пароль     |      10 |        20 байт              |
| Телефон     |      12 |        24 байт              |
| Пароль     |      10 |        20 байт              |
| Страна     |      15 |        30 байт              |

Итого: 194 байт(возьмем как 200 байт)

Средний размер 
| Тип данных     |      Размер на одного пользователя|   
|-----------|-----------------------------|
| Длина запроса и ответа(бесплатная версия)  |      16Кб |      
| Длина запроса и ответа(платная версия)  |      512Кб |     
| Размер загружаемых файлов     |      8Мб |      
| Данные пользователей     |      200 байт |        

RPS среднее 1_000_000_000 / 86400 ≈ 12000 запросов/с
Возьмем пиковую нагрузку в 2 раза больше средней. Также предположим, что 10% пользователей отправляют запросы с файлами. Следовательно (процентное соотношение типов запросов примерное)

| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      10200 запросов/с |   20400 запросов/с             |
| Отправка запроса/получение ответа с файлом    |      1100 запросов/с |   2300 запросов/с             |
| Регистрация/авторизация    |      23 запросов/с |        45 запросов/с              |
| Просмотр истории   |      45 запросов/с |        90 запросов/с             |
| Поиск по истории     |      23 запросов/с |        45 запросов/с             |

Рассчитаем разбивку по типам трафика в Гбит/с(при загрузке истории берем, что в среднем пользователь загружает чат, где ~30 сообщений) для бесплатной версии
| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      1,3Гбит/с |   2,6Гбит/с             |
| Отправка запроса/получение ответа  с файлом   |      7,4Гбит/с |   15Гбит/с             |
| Регистрация/авторизация    |      4*10^(-5)Гбит/с |        8*10^(-5)Гбит/с           |
| Просмотр истории   |       0,2Гбит/с |        0,4Гбит/с            |
| Поиск по истории     |       3*10^(-3)Гбит/с |       64*10^(-3)Гбит/с            |

Рассчитаем разбивку по типам трафика в Гбит/с(при загрузке истории берем, что в среднем пользователь загружает чат, где ~30 сообщений) для платной версии
| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      600 запросов/с |   1200 запросов/с             |
| Отправка запроса/получение ответа с файлом    |      67 запросов/с |   135 запросов/с             |
| Регистрация/авторизация    |      1 запросов/с |        2 запросов/с              |
| Просмотр истории   |      3 запросов/с |        5 запросов/с             |
| Поиск по истории     |      1 запросов/с |        3 запросов/с             |

| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      2,5Гбит/с |   5Гбит/с             |
| Отправка запроса/получение ответа    с файлом |      3,4Гбит/с |   6,8Гбит/с             |
| Регистрация/авторизация    |      2*10^(-7)Гбит/с |        4*10^(-7)Гбит/с           |
| Просмотр истории   |       0,04Гбит/с |        0,108Гбит/с            |
| Поиск по истории     |       0,004Гбит/с |       0,008Гбит/с            |

# 3. Глобальная балансировка нагрузки
### Домены
openai.com: информация о ее продуктах и исследованиях.  
chat.openai.com: Домен для чат-бота ChatGPT  
chat.com:  домен для упрощения доступа к сервису.  
chatgpt.com: домен для упрощения доступа к сервису.  
ai.com: домен для упрощения доступа к сервису.  
oaistatic.com и oaiusercontent.com: доступ к статическим файлам  

### Расположение доменов
Итого планируется 5 основных дата-центров:

США (Вирджиния) — покрытие Северной Америки.

Великобритания (Лондон) — покрытие Европы.

Индия (Мумбаи) — покрытие Южной Азии.

Бразилия (Сан-Паулу) — покрытие Латинской Америки.

Индонезия (Джакарта) — покрытие Юго-Восточной Азии.

Резервные дополнительные ДЦ:

Западное побережье США (Орегон) — на случай проблем в Вирджинии и балансировки побережий США.

Франкфурт (Германия) — дополнительная точка для Европы.

Токио (Япония) — дополнительная точка для Восточной Азии

### DNS-балансировка

Трафик перенаправляется на ближайший доступный ДЦ через систему DNS-балансировки (с использованием Health Checks).

Для минимизации потерь используется кэширование в CDN и anycast маршрутизация для статических файлов

|Домен | Балансировка | Обоснование|
|-----------|-----------------------------|---------------------------|
|openai.com | GeoDNS + Weighted Round Robin |  Нужно быстро отдать сайт с минимальной задержкой, распределять нагрузку между регионами|
|chat.openai.com | GeoDNS + Session Affinity Load Balancer внутри региона | Поддерживаем сессию пользователя в одном ДЦ для состояния чата|
|chatgpt.com, chat.com, ai.com | GeoDNS (перенаправляют на chat.openai.com) |  Быстрая переадресация на основной сервис |
|oaistatic.com, oaiusercontent.com | Anycast через CDN | Статический контент,  максимально быстрая загрузка из ближайшей точки. Anycast позволяет всегда попасть на ближайший узел. |

# 4. Локальная балансировка нагрузки
![image](https://github.com/user-attachments/assets/e355c66e-2599-4071-ab5b-8526c845f5d8)

Запрос от клиента проходит через интернет, где GeoDNS определяет ближайший дата-центр. Через IP-туннелирование (GRE/VXLAN) трафик поступает на L4-балансировщики (HAProxy/Keepalived) с VIP-адресами, которые обеспечивают базовую балансировку TCP-трафика и отказоустойчивость через VRRP. Далее запрос передаётся на кластер L7-балансировщиков (Nginx) с SSL-терминацией, WAF и rate-limiting, работающих в режиме Active-Active с BGP-роутингом. 

От L7 трафик идёт в Ingress Controller (NGINX Ingress/Envoy) в Kubernetes-кластере, который определяет целевой pod на GPU-нодах по правилам маршрутизации. Pod с ML-моделью обрабатывает запрос. 

# 5. Логическая схема БД
![image](https://github.com/user-attachments/assets/9d1db344-7a77-4cd9-8fec-d48e0ce4091b)
![image](https://github.com/user-attachments/assets/3c7543cb-97fe-4af1-a1b6-d3e08ab74fd3)

| Таблица |	Размер на пользователя |
|-----------|-----------------------------|
|users | Личные данные пользователей |
| requests | Данные о запросах |
| chats | Данные о чатах |
| user_files |	Данные о загружаемых файлах |
|payments |	Данные об оплате подписки |

Расситаем нагрузку на чтение и запись, возьмем, что примерно 10% пользователей загружают файлы:  

| Таблица |	Размер строки | Количество записей | Общий объем в сутки |
|-----------|-----------------------------|------------|-------------|
| users | 155 байт | 400 млн | 58Тб |
| chats | 36Кб | 3600 млн | 121 Пб |
| requests | 36Кб | 3600 млн | 121 Пб |
| user_files |	240 байт | 12 млн | 28 Гб |
| payments |	57 байт | 300 тыс. | 18 Гб |

Из п.2 возьмем среднее значение файлов 17Мб, поэтому в сутки нужно еще 200Тб в облачном хранилище.

Особенности каждой таблицы:  
Users:
* Шардирование: проводится основываясь на географическом признаке по полю country
* Кеширование: часто запрашиваемые данные(email, subscription) храняться в кеше
* Консистентность: необходима строгая консистентность данных, так как это влияет на бехопасность личных данных пользователя

Requests:
* Шардирование: по user_id
* Кеширование: последние 30 запросов(примерное количество запросов за сутки)
* Консистентность: допустимы задержки

Chats:
* Шардирование: по user_id, чтобы распределить чаты пользователей равномерно по серверам.
* Кеширование: последние активные чаты пользователя могут храниться в кеше для быстрого доступа.
* Консистентность: возможны небольшие задержки при обновлении данных, но порядок сообщений должен сохраняться.

User_files:
* Шардирование: предполагается использование облачного хранилища S3, поэтому шардирование не нужно
* Кеширование: не требуется
* Консистентность: рекомендуется обеспечение строгой консистентности, файлы не должны теряться

Payments:
* Шардирование: по user_id
* Кеширование: нет необходимости
* Консистентность: строгая консистентность, т.к. финансовые данные


# 6. Физическая схема БД
![image](https://github.com/user-attachments/assets/815346b5-d192-4d79-a7bd-189819df3f3b)

### 1. Индексы  
PostgreSQL:
- Users: 
  - email: UNIQUE INDEX (для быстрого поиска и защиты от дубликатов)  
  - phone: UNIQUE INDEX (для быстрого поиска и защиты от дубликатов)  

- Payments:  
  - user_id: B-TREE INDEX (для быстрого поиска платежей по пользователю)  
  - status: B-TREE INDEX (для поиска по статусу)  
  - subscription_till: B-TREE INDEX (для поиска истекающих подписок)  

MongoDB:
- Requests: 
  - created_at: B-TREE-индекс  

- Chats:  
  - user_id: B-TREE INDEX (для поиска чатов конкретного пользователя)  

- Files:
  - request_id: B-TREE INDEX (для связи файлов с запросами)  


### 2. Денормализация
- Users → Payments: храним subscription_till в Users, чтобы не делать JOIN.  



### 3. Выбор СУБД (потаблично)  
| Таблица       | СУБД           | Причины выбора |
|--------------|---------------|----------------|
| Users       | PostgreSQL    | Строгая консистентность, поддержка транзакций, удобные индексы |
| Payments    | PostgreSQL    | ACID-транзакции, работа с финансовыми операциями |
| Requests    | MongoDB       | Высокая нагрузка на запись, JSON-структура, удобство хранения запросов |
| Chats       | MongoDB       | Гибкость, JSON-структура, горизонтальное масштабирование |
| Files       | MongoDB + S3  | Метаданные в MongoDB, файлы в S3 (эффективное хранение) |



### 4. Шардирование и резервирование (потаблично)  

| Таблица      | Шардирование | Репликация / Резервирование |
|-------------|-------------|---------------------------|
| Users       | По `country`  | 2+ реплики |
| Payments    | По `user_id` (PostgreSQL партицирование) | Синхронная репликация |
| Requests    | По `user_id` (MongoDB Sharding) | Реплика-сет на 3+ ноды |
| Chats       | По `user_id` (MongoDB Sharding) | Реплика-сет на 3+ ноды |
| Files       | По `request_id` (MongoDB) | S3 версионирование и бэкапы |



### 6. Балансировка запросов / мультиплексирование подключений  
- PostgreSQL:  
  - PgBouncer для соединений  
  -  Nginx для балансировки нагрузки  

- MongoDB:  
  - MongoDB драйвер автоматически балансирует между шардами  

- S3:  
  - Использование CDN (CloudFront) для скачивания файлов  


### 7. Схема резервного копирования  
PostgreSQL:  
- pg_dump (логический бэкап)  
- pg_basebackup (физический бэкап)  
- Репликация на резервный сервер  

MongoDB:  
- mongodump (логический бэкап)  
- Репликация через Replica Set  
- Snapshots на уровне файловой системы  

S3:  
- Включение версионирования файлов  
- Репликация между регионами

# 7. Алгоритмы
Какие алгоритмы/технологии необходимо использовать:
| Алгоритм / Технология       | Область применения                     | Зачем |
|---------------------------------|-------------------------------------------|----------------------------------------------------------|
| Трансформеры (Transformer)  | Генерация и понимание текста              | Основа современных LLM, обеспечивает качественное NLP.   |
| Attention Mechanism         | Улучшение контекстного понимания текста   | Позволяет эффективно обрабатывать длинные последовательности. |
| BPE / WordPiece            | Токенизация текста                       | Оптимизирует обработку слов, уменьшает размер словаря.    |
| KV-кэширование             | Ускорение инференса LLM                   | Снижает вычисления при генерации последовательностей.     |
| Distributed Inference      | Распределенная обработка запросов         | Позволяет масштабировать инференс на множество серверов.  |
| Quantization (INT8, FP16)   | Сжатие и ускорение моделей                | Уменьшает объем памяти и ускоряет вычисления.            |
| LoRA / P-Tuning            | Эффективная дообучка моделей              | Позволяет кастомизировать модель без полного переобучения. |
| Retrieval-Augmented Gen. (RAG) | Доступ к внешним знаниям                | Улучшает точность ответов, снижает "галлюцинации".        |
| Caching ответов             | Повторное использование частых ответов    | Снижает нагрузку на модель, ускоряет ответы.             |
| Rate Limiting & QoS         | Балансировка нагрузки                     | Защищает от перегрузки, обеспечивает fair use.           |
| Batch Processing           | Пакетная обработка запросов               | Увеличивает пропускную способность GPU/TPU.              |
| Model Sharding            | Распределение больших моделей по устройствам | Позволяет запускать LLM на нескольких GPU/TPU.          |
| Speculative Decoding       | Ускорение генерации текста                | Предсказывает токены заранее, снижая latency.            |
| RLHF (Reinforcement Learning from Human Feedback) | Улучшение качества ответов | Делает ответы более релевантными и безопасными.          |
| Deduplication              | Устранение дублирования данных            | Экономит ресурсы при хранении и обработке.               |
| Compressed Storage (Huffman, LZ4) | Эффективное хранение моделей         | Уменьшает дисковое пространство и время загрузки.        |

Рассмотрим как производится AI Pipline
| Этап                | Что происходит                         | Артефакты                     | Как доставляется/хранится                     |
|-------------------------|--------------------------------------------|-----------------------------------|--------------------------------------------------|
|  Сбор данных      | Краулинг, парсинг, фильтрация сырых данных | Сырые тексты, изображения, логи  |  Объектные хранилища (S3, GCS), Kafka  |
|  Предобработка    | Очистка, дедупликация, токенизация         | Очищенные датасеты, токены       | Колоночные БД (Parquet) |
|  Обучение модели  | Distributed training на GPU/TPU            | Чекпоинты, логи обучения         |  FSx Lustre (для high-speed access), ML Metadata Store (MLMD) |
|  Валидация       | Тесты качества, bias-анализ                | Отчеты, метрики   |  Prometheus + Grafana |
|  Развертывание    | Конвертация в serving-формат  | Оптимизированные модели          |  Model Registry, CDN для весов моделей |
|  Инференс        | Генерация ответов в реальном времени       | Кэшированные ответы, эмбеддинги  |  Redis/Memcached, KV-хранилища (FAISS для поиска) |
|  Мониторинг       | Сбор feedback, анализ аномалий             | Логи запросов, юзер-фидбек       |  Elasticsearch + Kibana, ClickHouse для аналитики |
|  Дообучение       | Incremental learning с новыми данными      | Дельты-моделей (LoRA-адаптеры)   |  Версионирование (DVC),Git-LFS для малых артефактов |



# 8. Технологии 
| Технология            | Область применения         | Мотивация выбора / зачем используется |
|----------------------|----------------------------|----------------------------------------|
| PostgreSQL           | Хранение данных | Надежная реляционная СУБД с поддержкой транзакций и строгой консистентности. Оптимальна для хранения персональных и финансовых данных. Хорошо масштабируется |
| Redis                | Кеширование часто запрашиваемых данных | Быстрый in-memory storage. Позволяет ускорить доступ к часто используемым данным (профили пользователей, активные чаты, сессии). |
| Amazon S3            | Хранение файлов пользователей | Облачное объектное хранилище с высокой доступностью и встроенной репликацией. Отлично подходит для user_files с высокой надежностью хранения. |
| Kafka                | Очереди событий / логирование действий пользователей | Высоконагруженная система доставки сообщений. Позволяет строить event-driven архитектуру и асинхронно обрабатывать действия пользователя. |
| PyTorch / DeepSpeed | Обучение и оптимизация LLM модели | PyTorch как фреймворк обучения ИИ, а DeepSpeed позволяет экономно расходовать память и ускорять обучение больших моделей. |
| HuggingFace Hub      | Хранение и доставка моделей | Платформа для хранения, версионирования и публикации моделей машинного обучения. Упрощает доставку модели в разные окружения. |
| Prometheus + Grafana | Мониторинг состояния сервисов | Сбор и визуализация метрик работы сервисов и инфраструктуры. Позволяет быстро находить узкие места и ошибки. |
| Airflow              | Управление ML Pipeline | Планировщик задач для построения AI пайплайна: загрузка данных, препроцессинг, обучение модели |


# 9. Обеспечение надёжности

Восстановление при сбоях:

| Сценарий сбоя                             | Механизм восстановления                                                  |
|------------------------------------------|--------------------------------------------------------------------------|
| Сбой мастер-ноды PostgreSQL              | Автоматическое переключение на реплику + восстановление WAL       | 
| Сбой ноды AI Pipeline                    | Возобновление DAG в Airflow с последнего чекпоинта                      |
| Потеря модели после обучения             | Используется S3 для хранения всех версий             |
| Недоступность CDN/поставщика S3          | Fallback на локальный MinIO             |
| Проблемы с Redis                         | Подключение к реплике, восстановление из AOF/RDB                        | 
| Массовый сбой в датацентре               | Переключение на бэкап-кластер в другом регионе + DNS failover |


Обеспечение доступности:

| Компонент               |  Обеспечивающие механизмы                             |
|-------------------------|------------------------------------------------------|
| PostgreSQL              | Репликация, автоматический failover, PgBouncer     |
| Redis                   |  Master-Replica, Sentinel                            |
| Kafka                   | Реплики, min.insync.replicas                        |
| S3 / MinIO              |  Версионирование, мульти-региональная репликация     |
| API Gateway / Nginx     |  HA-масштабирование, health-checks                   |
| Модель (HuggingFace)    |  Локальный кэш, версионирование                      |
| Kubernetes              |  Авто-перезапуск, горизонтальное масштабирование     |
| MongoDB                 | ReplicaSet + шардирование|

# 10. Схема проекта
![image](https://github.com/user-attachments/assets/be99aa25-8fa2-4136-83cc-1647550af3bf)


- Round-Robin + Health Checks (проверка доступности бекендов)
- PostgreSQL: Чтение из реплик, запись в мастер, автоматический фейловер на реплику
- Redis: Шардирование через Redis Cluster
- Kafka: Запросы пользователей распределяются по партициям.

# 11. Список серверов
|Компонент | RAM (на сервер) | CPU (на сервер) | RPS | Количество серверов |
|----------------------------|------------------|-----------------|--------------|-------------------|
|API Gateway | 8 ГБ | 4 ядра | 1000 | 2 |
|Backend / FastAPI | 16 ГБ | 8 ядер | 3000 | 3 |
|Redis | 8 ГБ | 4 ядра | 10000 | 2 |
|PostgreSQL | 64 ГБ | 16 ядер | 1000 | 2 |
|Kafka | 32 ГБ | 16 ядер | 20000 | 3 |
|Inference Worker / PyTorch | 128 ГБ | 32 ядра | 50 | 3 |
|Локальное хранилище моделей | 32 ГБ | 8 ядер | 5000 | 2 |
|Airflow | 32 ГБ | 8 ядер | - | 2 |
|Data Preprocessing Worker | 128 ГБ | 32 ядра | - | 3 |
|Prometheus | 16 ГБ | 8 ядер | 20000 метрик | 1 |
|Grafana | 16 ГБ | 8 ядер | - | 1 |
***
[1]: https://www.demandsage.com/chatgpt-statistics/
[2]: https://explodingtopics.com/blog/chatgpt-users
[3]: https://increv.co/academy/chatgpt-stats/
[4]: https://aimojo.io/ru/chatgpt-statistics-facts/
[5]: https://itspeaker.ru/news/chislo-polzovateley-chatgpt-uzhe-bolshe-400-mln/
[6]: https://www.kommersant.ru/doc/7517265
