# Google Maps

# 1. Тема и целевая аудитория

### **Тема проекта**

Google Maps — это сервис картографии и навигации, предоставляющий пользователям информацию о геолокации, маршрутах, пробках и панорамах улиц.

### Продуктовые метрики

- **2B+ MAU**[^2]
- **95% трафика на веб-сайт приходится на мобильные устройства, 4% — на пк**[^1]
- **средняя продолжительность посещения веб-версии — 24 секунды** [^1]
- **доступно в 250+ странах**[^2]
- **250M+ бизнесов и мест**[^2]
- **100M+ обновлений информации каждый день** [^2]
- **(по США) приложением Google Maps используется пользователем 50 раз в месяц, в среднем сессия длится 3 минуты**[^3]
- **500M пользователей добавляют новую информацию каждый год**[^4]
- **За май 2023 года сайт Google Maps посетили 196.8 млн раз**[^5]
- **В среднем за день в Google Play приложение скачивают около 7 млн раз**[^6]
- **Google Maps в среднем использует около 5-10 MB данных за каждый час поездки**[^7]

### **Целевая аудитория**

#### Трафик по странам

##### Mobile

![Трафик по странам на мобильных устройствах](assets/mobile-traffic.png)

##### Desktop

![Трафик по странам на ПК](assets/desktop-traffic.png)

---

## Функционал MVP

### **Ключевые функции MVP**

1. **Отображение карты**
2. **Поиск мест и организаций**
3. **Построение маршрутов**
4. **Определение местоположения пользователя**
5. **Информация о местах**

---

## Ключевые продуктовые решения

1. **Кэширование (загрузка) карт** — оффлайн-доступ и оптимизация трафика.

---

# 2. Расчет нагрузки

## Продуктовые метрики

- 2B MAU[^2]
- Зная Stickiness = 30%[^3], посчитаем DAU = MAU \* 0,3 = 600M

Среднее количество действий пользователя по типам в день:

| **Действие**                        | **Среднее число повторений за день** | **Описание**                                                                                                                                                                                                                                                                           |
| ----------------------------------- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Отображение карты**               | **2,5**                              | Карта отображается при каждом запуске приложения. В среднем пользователь Google Maps открывает приложение 50 раз в месяц[^3], т. е. ≈ 1,67 раза в день. Учитываем, что некоторые пользователи открывают карту несколько раз за одну сессию, поэтому принимаем **2,5 действия** в день. |
| **Отображение информации о местах** | **3,34**                             | В среднем пользователи запрашивают данные о местах **2 раза за сессию**. Если **1,67 сессии в день**, то это **≈3,34 раза в день**.                                                                                                                                                    |
| **Поиск мест и организаций**        | **2,37**                             | В **71% случаев использования**[^9] пользователи совершают поиск. Предположим, что в среднем за сессию поиск совершают 2 раза. **1,67 \* 71% \* 2 ≈ 2,37**.                                                                                                                            |
| **Построение маршрутов**            | **1,25**                             | **75% пользователей**[^9] строят маршрут при каждом открытии. **1,67 \* 0,75 ≈ 1,25** маршрута в день.                                                                                                                                                                                 |
| **Загрузка оффлайн карт**           | **0,017**                            | Пользователь загружает карты своего региона, после чего обновляет их. Предположим, что выполняется **≈1 раз в 2 месяца**, то есть в день **≈1/60 = 0,017**.                                                                                                                            |

## Технические метрики

### Хранилище

#### Карта

Карты хранятся в виде тайлов (фрагментов изображений или векторных данных), которые загружаются при перемещении по карте. Причем тайлы не рендерятся предварительно, потому что а) объем получается слишком большим; б) доля просмотренных тайлов из всех составляет малые 1.79%. Поэтому тайлы рендерятся динамически, а наиболее использованные тайлы кэшируются.

Согласно информации с сайта OSM[^8] размер кэша составляет 1250 ГБ.

#### Информация о местах

| Поле                     | Описание                      | Средний объем на 1 объект |
| ------------------------ | ----------------------------- | ------------------------- |
| **ID места**             | Уникальный идентификатор      | 16 байт                   |
| **Название**             | Текст (UTF-8)                 | 50 байт                   |
| **Категория**            | Тип места (кафе, отель, парк) | 16 байт                   |
| **Координаты (lat/lon)** | Два float-числа               | 16 байт                   |
| **Часы работы**          | JSON-объект                   | 200 байт                  |
| **Отзывы**               | Список текстовых данных       | ~1 КБ (по 10 отзывов)     |
| **Рейтинг**              | Число с плавающей точкой      | 8 байт                    |
| **Доп. данные**          | Телефон, сайт, соцсети        | 200 байт                  |

## **Итоговая таблица хранения данных**

| **Тип данных**          | **Кол-во объектов** | **Размер 1 объекта** | **Общий объем** |
| ----------------------- | ------------------- | -------------------- | --------------- |
| **Информация о местах** | ~250 млн[^2]        | ~3 КБ                | ~720 ГБ         |
| **Размер кэша**         |                     |                      | ~1250 ГБ        |

### Сетевой трафик

| **Тип трафика**          | **Размер 1 запроса (МБ)**                                                                           | **Average RPS**                                                        | Peak RPS  | **Пиковый трафик** |
| ------------------------ | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | --------- | ------------------ |
| **Информация о местах**  | 0,003 МБ                                                                                            | 2,5 (действий в сутки) / 86400 (секунд в сутках) \* 600M DAU ≈ `17.4K` | **34.8K** | **835 Мбит/с**     |
| **Поиск мест**           | 100 байт (запрос) \* 10 результатов = 1 КБ                                                          | 2,37 / 86400 \* 600M ≈ `16.5K`                                         | **33K**   | **258 Мбит/с**     |
| **Построение маршрутов** | Возьмем маршрут в 10 км: возможно 200 точек \* 16 байт = 3,2 КБ + метаданные маршрута 2 КБ = 5,2 КБ | 1,25 / 86400 \* 600M ≈ `8.7K`                                          | **17.4K** | **707 Мбит/с**     |
| **Отображение карты**    | при экране 1080p при масштабировании 16 уровня загружается 9-12 тайлов за раз. 12 \* 0,6 = 7,2 КБ   | 2,5 / 86400 \* 600M ≈ `17.4K`                                          | **34.8K** | **19 Гбит/с**      |

# 3. Глобальная балансировка нагрузки

## Обоснования расположения ДЦ (влияние на продуктовые метрики)

Возьмем распределение пользователей по странам с сайта hypestat[^10]:

Из этих данных, делаем вывод, что больше всего трафика приходится на пользователей из США. После идут Индия, Китай, Бразилия и Япония.

Таким образом, логичнее всего расположить ДЦ в крупных городах перечисленных выше стран:

- США, Нью-Йорк (Восток США)
- США, Лос-Анджелес (Запад США)
- США, Чикаго (Северо-Запад США)
- США, Техас (Даллас) – баланс между западом и востоком США
- Индия, Мумбаи
- Китай, Пекин
- Бразилия, Сан-Паулу
- Япония, Токио

![alt text](assets/data-centers-on-map.png)

| Тип запроса                                 | Нью-Йорк | Лос-Анджелес | Чикаго | Даллас | Мумбаи | Пекин | Сан-Паулу | Токио |
| ------------------------------------------- | -------- | ------------ | ------ | ------ | ------ | ----- | --------- | ----- |
| **Отображение карты** (1,67M)               | 250.5K   | 250.5K       | 250.5K | 250.5K | 167K   | 167K  | 167K      | 167K  |
| **Отображение информации о местах** (1,67M) | 250.5K   | 250.5K       | 250.5K | 250.5K | 167K   | 167K  | 167K      | 167K  |
| **Поиск мест и организаций** (1.58M)        | 237K     | 237K         | 237K   | 237K   | 158K   | 158K  | 158K      | 158K  |
| **Построение маршрутов** (0.83M)            | 124.5K   | 124.5K       | 124.5K | 124.5K | 83K    | 83K   | 83K       | 83K   |

## Схема DNS балансировки

Для балансировки по миру будет использоваться **Geo-Based DNS**. Он будет перенаправлять пользователей в нужные регионы.

## Схема Anycast балансировки

Anycast — это метод маршрутизации, при котором один IP-адрес назначен нескольким серверам в разных местах.

Anycast сети будут использоваться в США, так как в штатах расположено несколько дата-центов. Маршрутизация на уровне сети будет направлять запрос к ближайшему узлу, помимо этого будет распределяться трафик тем самым справляться с DDoS атаками.

# 4. Локальная балансировка нагрузки

В качестве балансировщика выбран балансировщик уровня L7, так как он позволяет кэшировать карты и данные, а также сжимать данные.

В качестве решения будет использоваться NGINX благодаря его высокой производительности, гибкости настройки и поддержке множества протоколов. В качестве стратегии балансировки предлагается использовать round robin.

### Обеспечение отказоустойчивости

1. Kubernetes при отказе одного пода автоматически запускает новый.

2. Для большей отказойустойчивости Kubernetes будут масштабироваться пропорционально нагрузке

3. Kubernetes использует Service Discovery для определения доступных сервисов, и при отказе одного сервера другие продолжают работать, а клиент автоматически направляет запросы на работающие.

4. Nginx распределяет нагрузку между оставшимися серверами, чтобы снизить влияние отказа одного из них на пользователя.

# 5. Логическая схема БД[^11][^12]

![ER Diagram](assets/ER.png)

## Размеры таблиц

| **Таблица**         | **Описание**                                                                                     | Размер записи (байт) | Число записей                       | Объем   |
| ------------------- | ------------------------------------------------------------------------------------------------ | -------------------- | ----------------------------------- | ------- |
| **category**        | Содержит категории мест (например, рестораны, магазины, парки)                                   | 18                   | ~100                                | 1.8 KB  |
| **place**           | Описывает места (название, описание, рейтинг, контакты, категория и т. д.)                       | 663                  | 250M[^2]                            | 154 GB  |
| **review**          | Отзывы пользователей о местах (текст, рейтинг)                                                   | 152                  | число мест \* 5 отзывов = 1.25 B    | 177 GB  |
| **user**            | Информация о пользователях (ID, имя, email, хеш пароля)                                          | 364                  | 2B                                  | 678 GB  |
| **node**            | Представляет географические точки (широта, долгота)                                              | 48                   | 9B[^13]                             | 402 GB  |
| **node_tag**        | Дополнительные атрибуты узлов (например, название улицы, высота и т. д.)                         | 232                  | 906M[^13]                           | 196 GB  |
| **way**             | Объединяет узлы в линии (например, дороги, реки)                                                 | 16                   | 1B[^13]                             | 15 GB   |
| **way_node**        | Связь между узлами и путями (промежуточная таблица, определяет последовательность узлов в пути). | 50                   | Число путей \* 5 узлов на путь = 5B | 233 GB  |
| **way_tag**         | Хранит дополнительные атрибуты путей                                                             | 232                  | 2,5M[^13]                           | 553 MB  |
| **relation**        | Определяет сложные связи между объектами (например, маршруты, границы)                           | 16                   | 13M[^13]                            | 198 MB  |
| **relation_tag**    | Дополнительные свойства для связей                                                               | 232                  | 53M[^13]                            | 11.5 GB |
| **relation_member** | Описывает участников отношений (узлы, пути, другие связи)                                        | 54                   | Число relation \* 7 members = 91M   | 4.5 GB  |
| **member_type**     | Тип участника в отношении (узел, путь, связь)                                                    | 102                  | 3                                   | 306 B   |
| **tile_cache**      | Информация о кэше                                                                                | 44                   | 1.6B[^8]                            | 65.6 GB |

## Нагрузки (QPS) на чтение/запись

### Кэш

<details>
  <summary>🧠 Логика рассуждений кэширования тайлов</summary>

| Zoom  | Что видно             | Типичный юзкейс                                        | Поведение                 | Доля запросов | Взятое значение |
| ----- | --------------------- | ------------------------------------------------------ | ------------------------- | ------------- | --------------- |
| 0–5   | Весь мир / континенты | Почти неиспользуемы. Только при первом открытии карты. | Очень редкие              | ~1%           | 1%              |
| 6–8   | Страны / регионы      | Люди рассматривают страны, маршруты между городами     | Немного больше            | ~4%           | 4%              |
| 9–12  | Города, районы        | **Основной уровень навигации, просмотра, поиска**      | Основа кэша               | **65–75%**    | 70%             |
| 13–16 | Кварталы, улицы       | Подробный просмотр улиц, работа с POI                  | Очень часто при навигации | ~20–25%       | 23%             |
| 17–20 | Дома, здания          | Почти только при встраивании, просмотре входов/точек   | Узкие сценарии            | ~2–4%         | 2%              |

**Итого:** По информации с OSM Wiki[^8], с 0 по 12 уровни мы покроем кэшом на 100%. То есть `cache-hit = 75%` запросов пойдет в кэш вместо бд, а 25% — в БД.

</details>

Для снижения нагрузки на базу данных в картографическом сервисе применяется кэширование тайлов. Будем применять lazy-render с кэшированием тайлов. Сразу после рендера, тайл кэшируется. Тайлы сохраняются на диске в виде файлов `/tiles/{z}/{x}/{y}.png`, а информация о них хранится в служебной таблице `tile_cache`, где фиксируются дата рендера (`rendered_at`), срок актуальности (`expires_at`).

#### Логика кэширования:

- Если тайл актуален — он отдается из кэша.

- Если тайл отсутствует или устарел — он рендерится, сохраняется и кешируется.

#### Инвалидация кэша:

При изменении геоданных вычисляется `bounding box` изменения, и для всех тайлов, которые пересекаются с этой областью, в `tile_cache` проставляется метка об устаревании. При следующем запросе эти тайлы будут перерендерены.

#### Консистентность тайлов:

Чтобы избежать ситуации, когда один тайл отображает новые данные, а соседний — старые, при каждом запросе дополнительно проверяются восемь соседних тайлов. Если хотя бы один из них был перерендерен на более новой версии данных, текущий тайл также считается устаревшим и рендерится заново.

Такой подход позволяет:

- избежать несостыковок между тайлами,

- отказаться от полной предгенерации,

- и снизить нагрузку на базу данных при массовых чтениях.

### Таблицы с метаданными

| **Таблица**  | **Описание**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Read QPS | Write QPS |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | --------- |
| **user**     | Предположим, что у Google Maps MAU 40% от общего числа пользователей, то есть общее число пользователей `2B / 0,4 = 5B`.<br/>**R:** Пользователи \~ входят в аккаунт раз в 3 месяца, т.е. на это действие RPS = `5B / 3 мес = 643`. Помимо этого,  при загрузке **информации о местах** (34.8K RPS) загружаются отзывы, соответственно и данные об авторе отзыва: `34.8K RPS * 5 отзывов = 174K QPS` <br/>Также для отображения имени пользователя при открытии приложения приходится `2B MAU * 50 сессий в месяц[3] / 1 месяц = 38.6K QPS`<br/>**W:** Логично предположить, что пользователи редко меняют данные о себе и по большей части данные о пользователях читаются. В основном меняется пароль пользователя. Пусть каждые 2 года они должны менять пароль => `5B / 2 года = 80 QPS` | 213.2K   | 80        |
| **place**    | **R:** **Информация о местах** + **Поиск мест** = `67.8K` <br/>**W**:`100M` обновлений карты в день[^1][^16] / `86 400` = `1 157` QPS                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 67.8K    | 1 157     |
| **category** | Категории мест не меняются. Складывается из **Информация о местах** + **Поиск мест**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | 67.8K    | 0         |
| **review**   | **R:** Информация о местах `34.8K * 5 отзывов на место = 174K`<br/>**W:** По информации от Google[^17] пользователи оставляют миллионы отзывов каждый день. В Яндекс Картах оставляют `37K` отзывов в день[^18]. Учитывая, что у Яндекс Карт \~ 40 млн MAU[^19], то посчитав линейную пропорцию `2B Google Maps / 40M Яндекс.Карт * 37K отзывов в день = 1.85M отзывов/день`. Учитывая, что в источнике Google указывается, что оставляются миллионы отзывов, примем значение в `2M/day ≈ 23 QPS`                                                                                                                                                                                                                                                                                            | 174K     | 23        |

### Таблицы с геоданными

Так как таблицы `node`, `node_tag`, `way`, `way_node`, `way_tag`, `relation`, `relation_tag`, `relation_member`, `member_type` все вместе используются при отображении информации о местах, поиске мест, построении маршрутов и отображении карты. Поэтому R/W QPS у них одинаковый.

| **Таблица**                                                                                         | Read QPS                                                                                                                                                          | Write QPS                                                                       |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `node`, `node_tag`, `way`, `way_node`, `relation`, `relation_tag`, `relation_member`, `member_type` | (`34.8K`) Информация о местах + <br>(`33K`) Поиск мест + <br>(`17.4K`) Построение маршрутов +<br>(`34.8K` \* 25% (75% попадает в кэш))  Отображение карты = `94K` | `100M` обновлений карты в день[^1][^16] / `86 400` (секунд в дне) = `1 157` QPS |

## Требования консистентности

- В таблицах `place` и `relation_member` поле `member_id` может ссылаться как на node, так и на way, или relation. Поэтому `relation`, `node` и `way` обязаны иметь **уникальные PK, которые не повторяются друг с другом**!
- Последовательность: в таблицах `way_node` и `relation_member` значения поля `sequence_id` должны быть уникальными для одного `way_id` и `relation_id` соответственно. Также они должны быть в порядке возрастания.
- Атомарность операций: при сложных операциях (`добавление места` = `добавление place` + `добавление node / way / relation`) все изменения должны быть выполнены успешно или отменены полностью.

# 6. Физическая схема БД

![Физ схема БД ER](assets/er-physical.png)

## Выбор СУБД

#### Для геоданных

Для таблиц `node`, `node_tag`, `way`, `way_node`, `way_tag`, `relation`, `relation_tag`, `relation_member`, `member_type` выбрана СУБД PostgreSQL, так как в ней имеется поддержка `PostGIS`, которая ориентирована на хранение, индексирование и получение географических данных, и `pgRouting`, которая предоставляет эффективные инструменты для построения маршрутов.

#### Для метаданных

Для таблиц `place`, `category`, `review`, `user` будет использоваться СУБД Apache Cassandra, так как основные данные, которые обновляются в картах -- это информация о местах и отзывы. Эта СУБД имеет отличную горизонтальную масштабируемость, оптимизации для огромного числа операций записи.

## Индексы

- **Пространственные запросы**:
  - Для ускорения работы с пространственными запросами создадим пространственные индексы для таблицы `place_on_map`: `CREATE INDEX place_on_map_coord_idx ON place_on_map USING GIST (coord);`
  - Так как `node` огромное количество (9B записей), и требуется искать все `node`, которые попадают в viewport для рендеринга, то обязательно нужен индекс для поиска по координатам `node`: `CREATE INDEX node_coord_gist_idx ON node USING GIST (coord)`
- **От `node` к сложным объектам (`way` & `relation`)**
  - Для быстрого поиска `way_node` (5B записей) по `node_id` создаем индекс: `CREATE INDEX way_node_node_id_idx ON way_node (node_id);
  - Аналогично для быстрого поиска `way` из `way_node` и `relation_member` создаем индекс по `PK`: `CREATE INDEX way_id_idx ON way (id)`
  - На `node` и `way` ссылается `relation_member.member_id`, поэтому нужно создать индекс по этому полю для эффективного поиска `relation` и `relation_member` по `node_id` и `way_id`: `CREATE INDEX relation_member_member_id_idx ON relation_member (member_id);`
- **Для ускорения текстового поиска**:
  - по названию **GIN индекс** для поиска (даже с опечатками): `CREATE INDEX place_on_map_title_trgm_idx ON place_on_map USING GIN (title gin_trgm_ops)
  - по категории места B-tree индекс для **точного** совпадения: `CREATE INDEX place_on_map_category_idx ON place_on_map USING BTREE (category)`
- **От сложных объектов (`way` & `relation`) к простым (`node`)**
  - Поиск всех `node` внутри `way`: Индекс на `way_node.way_id`: `CREATE INDEX way_node_way_idx ON way_node (way_id)
  - Быстрый поиск `way` по PK: уже созданных индекс `way_id_idx`
  - Быстрый поиск `node` по PK: `CREATE INDEX node_id_idx ON node (id)`
- **Теги**:
  - Быстрый поиск тегов для точки (наибольший объем по тегам у таблицы `node` в сравнении с `way` & `relation`): индекс на `node_tag.node_id`: `CREATE INDEX node_tag_node_idx ON node_tag (node_id)`
  - Поиск тегов по ключу (B-Tree):
    - `CREATE INDEX node_tag_key_idx ON node_tag (key)`
    - `CREATE INDEX way_tag_key_idx ON way_tag (key)`
    - `CREATE INDEX relation_tag_key_idx ON relation_tag (key)`

## Репликация и шардирование

- Для таблиц `node`, `node_tag`, `way`, `way_node`, `relation`, `relation_tag`, `relation_member`, `member_type`
  - Шардирование по странам (малые страны будем объединять в группы, чтобы было распределено все равномерно)
  - Будем использовать 50 реплик Master-Slave на чтение

### Партиционирование

- Как можно было заметить, структура таблиц `way_tag`, `relation_tag`, `node_tag` идентична. По сути это 1 таблица, но ее логически разделили на три для ускорения операций получения тегов. Таким образом при получении тегов `way`, мы не будем проходиться по тегам `relation` & `node`.

## Денормализация

- Для большей эффективности вместо использования таблицы `category`, категорию места будем писать напрямую в таблицу `place`
- Таким же образом избавимся от таблицы `member_type`, а `member_type` будем напрямую писать в `relation_member`

## Клиентские библиотеки / интеграции

Бэкэнд будет написан на Go из-за его высокой производительности и популярности.

- PostgreSQL - [pgx](https://github.com/jackc/pgx)
- Apache Cassandra - [Apache Cassandra GoCQL Driver](https://github.com/apache/cassandra-gocql-driver)

## Балансировка запросов / мультиплексирование подключений

- PostgreSQL: PgBouncer
- Apache Cassandra: балансировка на уровне драйвера

## Схема резервного копирования

- PostgreSQL: каждые сутки `pg_basebackup`, каждые 5 минут `WAL-G`
- Apache Cassandra: каждые сутки `nodetool snapshot`, каждые 5 минут `Incremental Commit Logs`

## 7. Алгоритмы

### Metatile[^14]

**MetaTile** — это техника для оптимизации генерации тайлов карты. Вместо того чтобы рендерить **каждый 256×256 тайл отдельно**, рендерится **большая область** (например, 4×4 тайла), а затем она разрезается.

**Проблема:** так как запросов может быть много и если тайлы рендерятся по отдельности, то это может нагружать сервер огромным числом запросов + линии, шрифты на границах тайла могут не совпадать

**Решение:** Вместо того чтобы рендерить каждый тайл отдельно, сервер рендерит сразу большую область (например, 4×4 тайла) и потом разрезает на маленькие части.

**Как работает:**

1. Сервер рендерит **MetaTile (1024x1024 px)**
2. MetaTile **разрезается** на 16 обычных тайлов (256×256 px)
3. Эти тайлы **отдаются клиенту**

В итоге:

- Снижается нагрузка на сервер
- Быстрее происходит рендеринг
- Нет артефактов на границах тайлов

### Работа offline

1. Карта в виде тайлов хранится в БД MBTiles[^15] (SQLite)
2. Информация о местах (`places` и `places_on_map`) хранится в виде отдельной SQLite БД

Когда пользователь загружает карту, сервер сжимает данные и отправляет на устройство в формате MBTiles.
Девайс индексирует карту и сохраняет в кэш.
Без интернета клиент использует локальную БД.

## 8. Технологии

| Технология       | Область применения              | Мотивационная часть                                                                                                                                                                                                                                                         |
| ---------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mapnik           | Backend                         | Используется для генерации тайлов                                                                                                                                                                                                                                           |
| TypeScript       | Frontend                        | Скриптовый язык со статической типизацией. Используется для написания интерфейсов                                                                                                                                                                                           |
| React            | Frontend                        | Используется для эффективного написания интерфейсов с применением реактивности                                                                                                                                                                                              |
| Leaflet          | Frontend                        | Используется для отображения карт                                                                                                                                                                                                                                           |
| Kotlin           | Мобильное приложение            | зык программирования, используемый для создания мобильного приложения на Android                                                                                                                                                                                            |
| Swift            | Мобильное приложение            | Язык программирования, используемый для создания мобильного приложения на iOS                                                                                                                                                                                               |
| Golang           | Backend                         | Популярный компилируемый многопоточный ЯП для создания высокоэффективных программ, работающих на современных распределённых системах и многоядерных процессорах. Имеет множество библиотек, которые помогут настроить взаимодействие как с клиентом, так и с базами данных. |
| PostgreSQL       | Хранение гео данных             | СУБД поставляет большой функционал, высокую надежность, а также имеет расширения для геоданных                                                                                                                                                                              |
| Apache Cassandra | Хранение мета данных            | СУБД позволит обеспечить быструю скорость чтения и записи, поддерживает автошардирование                                                                                                                                                                                    |
| Prometheus       | Хранение метрик                 | Гибкая система для сбора и хранения метрик приложения                                                                                                                                                                                                                       |
| Grafana          | Визуализация метрик             | Позволяет создавать большое количество дашбордов для лучшей визуализации метрик и текущего состояния приложения                                                                                                                                                             |
| Nginx            | Reverse-proxy, load balancer    | Быстрое и популярное решение в области веб-серверов. Позволит обеспечить стабильную работу приложения благодаря возможности балансировать нагрузку между серверами.                                                                                                         |
| Git              | Система контроля версий         | Версионирование кода, совместная разработка приложения                                                                                                                                                                                                                      |
| GitLab           | Хранение удаленных репозиториев | Открытая и свободная система управления репозиториями кода. Также поддерживает удобную настройку CI/CD.                                                                                                                                                                     |
| Kubernetes       | Развертывание приложения        | Инструмент для оркестрации, масштабирования и поддержания отказоустойчивости приложения.                                                                                                                                                                                    |

# Список источников

[^1]: [Анализ веб-трафика maps.google.com](https://www.similarweb.com/website/maps.google.com/#geography)
[^2]: [Почему Google? Сайт Google с описанием преимуществ Google Maps](https://mapsplatform.google.com/why-google/)
[^3]: [Leading US Map and Navigation Smartphone Apps, Ranked by Monthly Unique Users, Aug 2019](https://www.emarketer.com/chart/234831/leading-us-map-navigation-smartphone-apps-ranked-by-monthly-unique-users-aug-2019)
[^4]: [20 things you didn’t know you could do with Google Maps](https://blog.google/products/maps/20-years-google-maps-20-features/)
[^5]: [Google Maps Statistics 2024 By Usage, Revenue, Accuracy, Traffic Data, Trends, Web Usage and API Usage](https://www.enterpriseappstoday.com/stats/google-maps-statistics.html#:~:text=Google%20Maps%20website%20had%20a%20record%20number%20of%20visits.%20Google%20Maps%20website%20had%20around%20196.8%20million%20visits%20total%20in%20May%202023.)
[^6]: [Google Maps on AppBrain](https://www.appbrain.com/app/google-maps/com.google.android.apps.maps)
[^7]: [Сколько данных используют Google Maps?](https://truely.com/blog/how-much-data-does-google-maps-use#:~:text=with%20Standard%20Mode-,5%2D10%20MB,-Navigation%20with%20Traffic)
[^8]: [OSM Wiki: Tile disk usage](https://wiki.openstreetmap.org/wiki/Tile_disk_usage)
[^9]: [Как часто россияне используют геосервисы](https://iom.anketolog.ru/2020/02/12/geoservisy)
[^10]: [HypeStat: Google Maps](https://hypestat.com/info/maps.google.com)
[^11]: [OSM: DB Schema](https://wiki.openstreetmap.org/wiki/Openstreetmap-website/Database_schema)
[^12]: [OSM: Элементы](https://wiki.openstreetmap.org/wiki/RU:%D0%AD%D0%BB%D0%B5%D0%BC%D0%B5%D0%BD%D1%82%D1%8B)
[^13]: [OSM: taginfo](https://taginfo.openstreetmap.org/sources/db)
[^14]: [OSM Wiki: Meta Tiles](https://wiki.openstreetmap.org/wiki/Meta_tiles)
[^15]: [OSM Wiki: MBTiles](https://wiki.openstreetmap.org/wiki/MBTiles)
[^16]: [Google Maps Platform: Build with the power and scale of Google Maps](https://mapsplatform.google.com/resources/blog/places-insights-api-now-in-preview/#:~:text=With%20100%20million%20updates%20to%20the%20map%20every%20day)
[^17]: [How reviews on Google Maps work](https://blog.google/products/maps/how-google-maps-reviews-work/#:~:text=With%20millions%20of%20reviews%20posted%20every%20day%20from%20people%20around%20the%20world)
[^18]: [Исследования Яндекса — Как хвалят и ругают рестораны и бары](https://yandex.ru/company/researches/2023/orgs-reviews#:~:text=%D0%AF%D0%BD%D0%B4%D0%B5%D0%BA%D1%81%20%D0%9A%D0%B0%D1%80%D1%82%D1%8B%20%D0%B5%D0%B6%D0%B5%D0%B4%D0%BD%D0%B5%D0%B2%D0%BD%D0%BE%20%D1%81%D0%BE%D0%B1%D0%B8%D1%80%D0%B0%D1%8E%D1%82%2037%20%D1%82%D1%8B%D1%81%D1%8F%D1%87%20%D0%BE%D1%82%D0%B7%D1%8B%D0%B2%D0%BE%D0%B2)
[^19]: [По пути с рекламой: новые возможности геомедийных форматов для ещё большего охвата — Новости рекламных технологий Яндекса](https://yandex.ru/adv/news/po-puti-s-reklamoy-novye-vozmozhnosti-geomediynykh-formatov-dlya-esche-bolshego-okhvata#:~:text=%D0%9F%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D1%8F%20%D0%9A%D0%B0%D1%80%D1%82%20%D0%B8%C2%A0%D0%9D%D0%B0%D0%B2%D0%B8%D0%B3%D0%B0%D1%82%D0%BE%D1%80%D0%B0%C2%A0%E2%80%94%20%D1%81%D0%B0%D0%BC%D1%8B%D0%B5%20%D0%BE%D1%85%D0%B2%D0%B0%D1%82%D0%BD%D1%8B%D0%B5%20%D0%B8%D0%B7%C2%A0%D0%93%D0%B5%D0%BE%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D0%BE%D0%B2.%20%D0%98%D1%85%C2%A0%D0%B5%D0%B6%D0%B5%D0%BC%D0%B5%D1%81%D1%8F%D1%87%D0%BD%D0%B0%D1%8F%20%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%B0%D1%8F%20%D0%B0%D1%83%D0%B4%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D1%8F%20%D1%81%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D1%8F%D0%B5%D1%82%2037)
