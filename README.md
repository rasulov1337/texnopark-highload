# Google Maps

# 1. Тема и целевая аудитория

### **Тема проекта**

Google Maps — это сервис картографии и навигации, предоставляющий пользователям информацию о геолокации, маршрутах, пробках и панорамах улиц.

### Продуктовые метрики

-   **100M+ MAU (сайт maps.google.com)**[^1]
-   **1B+ MAU в целом на всех устройствах**[^2]
-   **95% трафика на веб-сайт приходится на мобильные устройства, 4% — на пк**[^1]
-   **средняя продолжительность посещения веб-версии — 24 секунды** [^1]
-   **доступно в 250+ странах**[^2]
-   **250M+ бизнесов и мест**[^2]
-   **100M+ обновлений информации каждый день** [^2]
-   **(по США) приложением Google Maps используется пользователем 50 раз в месяц, в среднем сессия длится 3 минуты**[^3]
-   **500M пользователей добавляют новую информацию каждый год**[^4]
-   **За май 2023 года сайт Google Maps посетили 196.8 млн раз**[^5]
-   **В среднем за день в Google Play приложение скачивают около 7 млн раз**[^6]
-   **Google Maps в среднем использует около 5-10 MB данных за каждый час поездки**[^7]

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

-   1B+ MAU[^2]
-   MAU / 30 ≈ 33,3M DAU

Среднее количество действий пользователя по типам в день:

| **Действие**                        | **Среднее число повторений за день** | **Описание**                                                                                                                                     |
| ----------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Отображение карты**               | **2,5**                              |                                                                                                                                                  |
| **Отображение информации о местах** | **2,5**                              | В среднем пользователи запрашивают данные о местах **1–3 раза за сессию**. Если **1,67 сессии в день**, то это **≈2,5 раза в день**.             |
| **Поиск мест и организаций**        | **1,5**                              | В **50% случаев использования** пользователи делают поиск. **1,67 \* 50% ≈ 0,83**, округляем с учетом многократного поиска за сессию до **1,5**. |
| **Построение маршрутов**            | **1,2**                              | **70% пользователей** строят маршрут при каждом открытии. **1,67 \* 0,7 ≈ 1,2** маршрута в день.                                                 |
| **Загрузка оффлайн карт**           | **0,03**                             | Выполняется **≈1 раз в месяц**, то есть в день **≈1/30 = 0,03**.                                                                                 |

## Технические метрики

### Хранилище

#### Карта

Карты хранятся в виде тайлов (фрагментов изображений или векторных данных), которые загружаются при перемещении по карте.

| Поле                 | Описание                                 | Средний объем на 1 объект |
| -------------------- | ---------------------------------------- | ------------------------- |
| **Тайл изображения** | JPEG                                     | 0,5 КБ [^8]               |
| **Метаданные тайла** | Координаты, зум-уровень, дата обновления | 1 КБ                      |

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
| **Фото**                 | Ссылки на изображения         | ~2 КБ                     |
| **Доп. данные**          | Телефон, сайт, соцсети        | 200 байт                  |

## **Итоговая таблица хранения данных**

| **Тип данных**          | **Кол-во объектов** | **Размер 1 объекта** | **Общий объем** |
| ----------------------- | ------------------- | -------------------- | --------------- |
| **Карты (тайлы)**       | ~100 млрд           | ~0,5 КБ              | ~47 ГБ          |
| **Информация о местах** | ~250 млн            | ~3 КБ                | ~750 ТБ         |

### Сетевой трафик

#### Пиковое потребление в течение суток (в Гбит/с)

Пиковая нагрузка: 40% пользователей активны одновременно
| **Тип трафика** | **Размер 1 запроса (МБ)** | **Число запросов в секунду (RPS) в пике** | **Пиковый трафик (Гбит/с)** |
| ------------------------- | ------------------------- | ----------------------------------------- | --------------------------- |
| **Загрузка карт (тайлы)** | 0,5 КБ | (33.3M \* 2,5 \* 40%) / (3\*60) ≈ 184 тыс | **0,7 Гбит/с** |
| **Информация о местах** | 0,05 МБ | (33.3M \* 2,5 \* 40%) / (3\*60) ≈ 184 тыс | **72 Гбит/с** |
| **Поиск мест** | 0,1 МБ | (33.3M \* 1,5 \* 40%) / (3\*60) ≈ 108 тыс | **84 Гбит/с** |
| **Построение маршрутов** | 0,15 МБ | (33.3M \* 1,2 \* 40%) / (3\*60) ≈ 88 тыс | **103,2 Гбит/с** |

<!-- #### Суммарный суточный (Гбайт/сутки)

| **Тип трафика**           | **Размер 1 запроса (МБ)** | **Число действий в день** | **Суточный трафик (ПБ)** | **Суточный трафик (ГБ/сутки)** |
| ------------------------- | ------------------------- | ------------------------- | ------------------------ | ------------------------------ |
| **Загрузка карт (тайлы)** | 0,2 МБ                    | 33.3M \* 2,5               | **0,25 ПБ**              | **250 000 ГБ**                 |
| **Информация о местах**   | 0,05 МБ                   | 33.3M \* 2,5               | **0,062 ПБ**             | **62 500 ГБ**                  |
| **Поиск мест**            | 0,1 МБ                    | 33.3M \* 1,5               | **0,075 ПБ**             | **75 000 ГБ**                  |
| **Построение маршрутов**  | 0,15 МБ                   | 33.3M \* 1,2               | **0,09 ПБ**              | **90 000 ГБ**                  | -->

<!-- ### RPS

| **Тип запроса**                | **Число действий в день** | **Среднее RPS (сутки)**    | **Пиковый RPS (10% активных пользователей)** |
| ------------------------------ | ------------------------- | -------------------------- | -------------------------------------------- |
| **Отображение карты**          | 33.3M \* 2,5 = 1,25 млрд  | 1,25B / 86400 ≈ **14 470** | (33.3M _ 2,5 _ 10%) / 180 ≈ **7 млн**        |
| **Запрос информации о местах** | 33.3M \* 2,5 = 1,25 млрд  | 1,25B / 86400 ≈ **14 470** | (33.3M _ 2,5 _ 10%) / 180 ≈ **7 млн**        |
| **Поиск мест**                 | 33.3M \* 1,5 = 750 млн    | 750M / 86400 ≈ **8 680**   | (33.3M _ 1,5 _ 10%) / 180 ≈ **4,2 млн**      |
| **Построение маршрутов**       | 33.3M \* 1,2 = 600 млн    | 600M / 86400 ≈ **6 940**   | (33.3M _ 1,2 _ 10%) / 180 ≈ **3,4 млн**      | -->

# Список источников

[^1]: [Анализ веб-трафика maps.google.com](https://www.similarweb.com/website/maps.google.com/#geography)
[^2]: [Почему Google? Сайт Google с описанием преимуществ Google Maps](https://mapsplatform.google.com/why-google/)
[^3]: [Leading US Map and Navigation Smartphone Apps, Ranked by Monthly Unique Users, Aug 2019](https://www.emarketer.com/chart/234831/leading-us-map-navigation-smartphone-apps-ranked-by-monthly-unique-users-aug-2019)
[^4]:
    [20 things you didn’t know you could do with Google Maps
    ](https://blog.google/products/maps/20-years-google-maps-20-features/)

[^5]:
    [Google Maps Statistics 2024 By Usage, Revenue, Accuracy, Traffic Data, Trends, Web Usage and API Usage
    ](https://www.enterpriseappstoday.com/stats/google-maps-statistics.html#:~:text=Google%20Maps%20website%20had%20a%20record%20number%20of%20visits.%20Google%20Maps%20website%20had%20around%20196.8%20million%20visits%20total%20in%20May%202023.)

[^6]: [Google Maps on AppBrain](https://www.appbrain.com/app/google-maps/com.google.android.apps.maps)
[^7]:
    [Сколько данных используют Google Maps?
    ](https://truely.com/blog/how-much-data-does-google-maps-use#:~:text=with%20Standard%20Mode-,5%2D10%20MB,-Navigation%20with%20Traffic)

[^8]: [Open steet maps: Tile disk usage](https://wiki.openstreetmap.org/wiki/Tile_disk_usage)
