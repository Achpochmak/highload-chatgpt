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

Максимум 512 Мб общий размер файлов в диалоге. Возьмем половину от этого значения в качестве среднего размера диалога. Максимум 40-80 запросов к чату в час, поэтому предположим, что на одно сообщение с файлом в среднем приходится 17 Мб, если брать в качестве среднего значения количества сообщений 30 (среднее от половины максимумов). 

Данные необходимые для регистрации:
| Данные    | Длина | Размер | 
|-----------|-----------------------------|-----------|
| Имя     |      25 |        75 байт              |
| Email     |      25 |        75 байт              |
| Пароль     |      10 |        30 байт              |
| Телефон     |      12 |        36 байт              |
| Пароль     |      10 |        30 байт              |
| Страна     |      15 |        45 байт              |

Итого: 291 байт

Средний размер 
| Тип данных     |      Размер на одного пользователя|   
|-----------|-----------------------------|-----------|
| Длина запроса и ответа(бесплатная версия)  |      16Кб |      
| Длина запроса и ответа(платная версия)  |      512Кб |     
| Размер загружаемых файлов     |      17Мб |      
| Данные пользователей     |      300 байт |        

RPS среднее 120_000_000 / 86400 ≈ 1400 запросов/с
Возьмем пиковую нагрузку в 2 раза больше средней. Также предположим, что 10% пользователей отправляют запросы с файлами. Следовательно (процентное соотношение типов запросов примерное)

| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      1170 запросов/с |   2340 запросов/с             |
| Отправка запроса/получение ответа с файлом    |      130 запросов/с |   260 запросов/с             |
| Регистрация/авторизация    |      25 запросов/с |        50 запросов/с              |
| Просмотр истории   |      50 запросов/с |        100 запросов/с             |
| Поиск по истории     |      25 запросов/с |        50 запросов/с             |

Рассчитаем разбивку по типам трафика в Гбит/с(при загрузке истории берем, что в среднем пользователь загружает чат, где ~30 сообщений) для бесплатной версии
| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      18Гбит/с |   36Гбит/с             |
| Отправка запроса/получение ответа  с файлом   |      2Гбит/с |   4Гбит/с             |
| Регистрация/авторизация    |      5*10^(-5)Гбит/с |        1*10^(-4)Гбит/с           |
| Просмотр истории   |       0,36Гбит/с |        0,72Гбит/с            |
| Поиск по истории     |       2*10^(-4)Гбит/с |       4*10^(-4)Гбит/с            |

Рассчитаем разбивку по типам трафика в Гбит/с(при загрузке истории берем, что в среднем пользователь загружает чат, где ~30 сообщений) для платной версии
| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      10 запросов/с |   20 запросов/с             |
| Отправка запроса/получение ответа с файлом    |      1 запросов/с |   2 запросов/с             |
| Регистрация/авторизация    |      0,2 запросов/с |        0,4 запросов/с              |
| Просмотр истории   |      0,4 запросов/с |        0,8 запросов/с             |
| Поиск по истории     |      0,2 запросов/с |        0,4 запросов/с             |

| Тип операции     |     Средняя нагрузка |        Пиковая нагрузка             |
|-----------|-----------------------------|-----------|
| Отправка запроса/получение ответа     |      4,8Гбит/с |   9,6Гбит/с             |
| Отправка запроса/получение ответа    с файлом |      0,6Гбит/с |   1.2Гбит/с             |
| Регистрация/авторизация    |      5*10^(-7)Гбит/с |        1*10^(-6)Гбит/с           |
| Просмотр истории   |       0,096Гбит/с |        0,192Гбит/с            |
| Поиск по истории     |       2*10^(-6)Гбит/с |       4*10^(-6)Гбит/с            |

# 3. Глобальная балансировка нагрузки
### Домены
openai.com: информация о ее продуктах и исследованиях.  
chat.openai.com: Домен для чат-бота ChatGPT  
chat.com:  домен для упрощения доступа к сервису.  
chatgpt.com: домен для упрощения доступа к сервису.  
ai.com: домен для упрощения доступа к сервису.  
oaistatic.com и oaiusercontent.com: доступ к статическим файлам  

### Расположение доменов
Исходя из процентного расположения пользователей, рекомендуется поставить ЦОДы в США, например, в Вирджинии, где расположены крупные облачные хранилища, в Индии, в Бразилии для покрытия Латинской Америки, в Индонезии для покрытия Юго-Восточной Азии и в Великобритании для покрытия Европы. 

### DNS-балансировка
В качестве алгоритма балансировки рекомендуется использовать алгоритм основанный на геолокации и weighted round robin, который будет учитывать также загруженности сервисов.

Используется балансировка нагрузки и anycast балансировка.
***
[1]: https://www.demandsage.com/chatgpt-statistics/
[2]: https://explodingtopics.com/blog/chatgpt-users
[3]: https://increv.co/academy/chatgpt-stats/
[4]: https://aimojo.io/ru/chatgpt-statistics-facts/
