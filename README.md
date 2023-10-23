# Highload YouTube

## Содержание
* ### [1. Тема, целевая аудитория](#1)
* ### [2. Расчет нагрузки](#2)
* ### [3. Глобальная балансировка нагрузки](#3)
* ### [4. Локальная балансировка нагрузки](#4)
* ### [5. Логическая схема БД](#5)
* ### [6. Физическая схема БД](#6)
* ### [7. Алгоритмы](#7)

## 1. Тема и целевая аудитория<a name="1"></a>

### 1.1 Тема
Сервис **YouTube** - видеохостинг, предоставляющий пользователям услуги хранения, доставки и показа видео

### 1.2 MVP
Функциональность сервиса
1. **Просмотр** видео
2. **Загрузка** видео
3. **Комментирование, лайки, подписки** на канал
4. **Рекомендации** на главной странице и на странице видео
5. **Авторизация**
6. **Поиск**

### 1.3 Целевая аудитория
- Местоположение - **Россия**
- **Размер аудитории** на локальном рынке \[[1](https://mediascope.net/data/#internet)]
	- месячный охват 95 млн. человек
	- дневной охват 52 млн. человек
- Среднее время просмотра в день - 85 мин \[[2](https://mediascope.net/upload/iblock/ee9/b91rtnqh1jf0zhalydi9jf125voo5o8e/медиапотребление.pdf)], хотя по миру это значение намного меньше \[[3](https://www.oberlo.com/statistics/average-time-spent-on-youtube)]
- Некоторые факты об аудитории
	- 15% доля YouTube в интернет-потреблении на локальном рынке (18% - доля видео в целом) \[[4](https://mediascope.net/upload/iblock/ee9/b91rtnqh1jf0zhalydi9jf125voo5o8e/медиапотребление.pdf)]
	- Средний возраст 25-34 лет
	- 40% - доля женщин \[[5](https://bloggingwizard.com/youtube-statistics/#:~:text=60.59%25%20of%20YouTube%E2%80%99s%20audience%20is%20male)]

## 2. Расчет нагрузки<a name="2"></a>

Статистика для расчетов

|Метрика|Аудитория|Значение|Обозначение|
| ------------- | --- |-------------|--|
|MAU (чел.)\[[6](https://mediascope.net/data/#internet)]|Мир|2500 млн.|MAU_WORLD|
|MAU (чел.)\[[7](https://mediascope.net/data/#internet)]|Россия|95 млн.|MAU_RUS|
|DAU (чел.)\[[8](https://mediascope.net/data/#internet)]|Россия|52 млн.|MAU_RUS|
|Количество просмотров / месяц \[[9](https://www.globalmediainsight.com/blog/youtube-users-statistics/#:~:text=YouTube%20Views%20by%20Country)]|Россия|207 млрд.|VIEWS_MONTH|
|Среднее время просмотра / день (мин)\[[10](https://inclient.ru/youtube-stats/#:~:text=%D0%92%20%D0%A0%D0%BE%D1%81%D1%81%D0%B8%D0%B8%20YouTube%20%D1%81%D0%BC%D0%BE%D1%82%D1%80%D1%8F%D1%82%20%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%BD%D0%BE%2086%20%D0%BC%D0%B8%D0%BD%D1%83%D1%82%20%D0%B2%20%D0%B4%D0%B5%D0%BD%D1%8C)]|Россия|86|VIEW_DURATION_DAY|
|Загрузка видео / минута (час)\[[11](https://bloggingwizard.com/youtube-statistics/#:~:text=That%E2%80%99s%2030%2C000%20hours%20of%20video%20uploaded%20every%20hour%2C%20720%2C000%20uploaded%20every%20day%2C%205.04%20million%20uploaded%20every%20week%2C%2021.9%20million%20uploaded%20every%20month%20and%20262.8%20million%20uploaded%20every%20year.)] |Мир|500|UPLOAD_MINUTE|
|Загрузка видео / час (час)\[[12](https://bloggingwizard.com/youtube-statistics/#:~:text=That%E2%80%99s%2030%2C000%20hours%20of%20video%20uploaded%20every%20hour%2C%20720%2C000%20uploaded%20every%20day%2C%205.04%20million%20uploaded%20every%20week%2C%2021.9%20million%20uploaded%20every%20month%20and%20262.8%20million%20uploaded%20every%20year.)] |Мир|30 тыс.|UPLOAD_HOUR|
|Загрузка видео / день (час)\[[13](https://bloggingwizard.com/youtube-statistics/#:~:text=That%E2%80%99s%2030%2C000%20hours%20of%20video%20uploaded%20every%20hour%2C%20720%2C000%20uploaded%20every%20day%2C%205.04%20million%20uploaded%20every%20week%2C%2021.9%20million%20uploaded%20every%20month%20and%20262.8%20million%20uploaded%20every%20year.)] |Мир|720 тыс.|UPLOAD_DAY|
|Загрузка видео / месяц (час)\[[14](https://bloggingwizard.com/youtube-statistics/#:~:text=That%E2%80%99s%2030%2C000%20hours%20of%20video%20uploaded%20every%20hour%2C%20720%2C000%20uploaded%20every%20day%2C%205.04%20million%20uploaded%20every%20week%2C%2021.9%20million%20uploaded%20every%20month%20and%20262.8%20million%20uploaded%20every%20year.)] |Мир|21.9 млн.|UPLOAD_MONTH|
|Загрузка видео / год (час)\[[15](https://bloggingwizard.com/youtube-statistics/#:~:text=That%E2%80%99s%2030%2C000%20hours%20of%20video%20uploaded%20every%20hour%2C%20720%2C000%20uploaded%20every%20day%2C%205.04%20million%20uploaded%20every%20week%2C%2021.9%20million%20uploaded%20every%20month%20and%20262.8%20million%20uploaded%20every%20year.)] |Мир|263 млн.|UPLOAD_YEAR|
|Передача видео / минуту (час)\[[16](https://www.wyzowl.com/youtube-stats/#:~:text=An%20average%20of%20694%2C000%20hours%20of%20video%20are%20streamed%20by%20YouTuber%20users%20each%20and%20every%20minute.%20This%20is%20even%20higher%20than%20Netflix%2C%20whose%20users%20stream%20452%2C000%20hours%20per%20minute.)] |Мир|694 тыс.|DOWNLOAD_MIN|
|Количество комментариев / 1000 просмотров\[[17](https://tubularlabs.com/blog/3-metrics-youtube-success/#:~:text=Comments%20to%20Views%3A%20How%20High%20is%20Engagement%3F)]|Мир|5|COMMENTS|
|Количество лайков / 100 просмотров\[[18](https://tubularlabs.com/blog/3-metrics-youtube-success/#:~:text=Likes%20to%20Views%3A%20How%20Popular%20is%20Your%20Video%3F)].|Мир|4|LIKES|
|Количество новых подписок / день / канал \[[19](https://medium.com/@jasonrbodie/average-youtube-channel-growth-rate-f6837584c9ac)]|Мир|1000|SUBS|
|Количество поисковых запросов / день \[[20](https://www.globalmediainsight.com/blog/youtube-users-statistics/#searchstat:~:text=Every%20day%2C%20more%20than%203.5%20billion%20searches%20are%20made%20on%20YouTube.)]|Мир|3.5 млрд.|SEARCH_DAY|
|Средняя продолжительность видео (мин)\[[20](https://bloggingwizard.com/youtube-statistics/#:~:text=7.-,The%20average%20length%20of%20a%20YouTube%20video%20is%2011.7%20minutes,-According%20to%20Statista)]|Мир|11.7|DURATION|
|Количество посещений страниц / день / пользователь \[[21](https://www.globalmediainsight.com/blog/youtube-users-statistics/#:~:text=An%20average%20YouTube%20visitor%20checks%20nearly%20nine%20pages%20per%20day.%C2%A0)]|Мир|9|VISITS_DAY_USER|
|Средняя продолжительность посещения сайта (мин)\[[22](https://inclient.ru/youtube-stats/#:~:text=%D0%9F%D0%BE%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC%20Similarweb%20%D1%81%D1%80%D0%B5%D0%B4%D0%BD%D1%8F%D1%8F%20%D0%BF%D1%80%D0%BE%D0%B4%D0%BE%D0%BB%D0%B6%D0%B8%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D1%81%D1%82%D1%8C%20%D0%BF%D0%BE%D1%81%D0%B5%D1%89%D0%B5%D0%BD%D0%B8%D1%8F%20%D1%81%D0%B0%D0%B9%D1%82%D0%B0%20youtube.com%20%D1%81%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D1%8F%D0%B5%D1%82%2020%20%D0%BC%D0%B8%D0%BD%D1%83%D1%82%2016%20%D1%81%D0%B5%D0%BA%D1%83%D0%BD%D0%B4)] \[[23](https://www.statista.com/statistics/1257254/youtubecom-time-spent-per-visit/#:~:text=In%20March%202021%2C%20users%20worldwide%20spent%20an%20average%20of%201777%20seconds%20(29%20minutes%20and%2037%20seconds))]|Мир|20|VISIT_TIME|

Некоторую метрику на российском рынке приблизительно можно рассчитать относительно мировой метрики
- METRICS (rus) = METRICS (world) * (MAU / MAU)
- METRICS (rus) = METRICS (world) * K
- K = 95 / 2500 = 0.038

### 2.1 Продуктовые метрики

### 2.1.1 Сводная таблица продуктовых метрик

|Действие пользователя |Количество / месяц|
| ------------- |-------------|
|Просмотр видео|207 млрд.|
|Показ рекомендаций|70 млрд.|
|Добавление лайка|8.3 млрд.|
|Посещение сервиса (проверка авторизации)|6.2 млрд.|
|Поиск|3.9 млрд.|
|Подписка на канал|3.6 млрд.|
|Лента подписок|2 млрд.|
|Добавление комментария|1 млрд.|
|Загрузка видеоролика на канал|4.3 млн.|

### 2.1.2 Расчет продуктовых метрик

- **Просмотров видео** - VIEWS_MONTH.
- **Комментарии** - 1 млрд. / мес.
	- На каждые 1000 просмотров приходится COMMENTS комментариев.
	- Значит, в месяц происходит ```VIEWS_MONTH / 1000 * COMMENTS = 207 млрд. / 1000 * 5 = 1 млрд``` запросов на добавление комментария.
- **Лайки** - 8.28 млрд. / мес.
	- На каждые 100 просмотров приходится LIKES лайков.
	- Значит, в месяц происходит ```VIEWS_MONTH / 100 * LIKES = 207 млрд. / 100 * 4 = 8.28 млрд``` запросов на установку лайка.
- **Подписки** - 3.6 млрд. / мес.
	- В среднем, каждый канал в день набирает SUBS подписчиков при MAU_WORLD активных пользователей в месяц.
	- Приблизительно, каждый канал на локальном рынке в день набирает ```SUBS * K = 1000 * 0.038 = 38```.
	- Значит, ```MAU_RUS * 38 = 95 млн * 38 = 3.6 млрд ```подписок осуществляется в месяц. 
- **Загрузка видеоролика на канал** - 4.3 млн. / мес.
	- В России загружается UPLOAD_MONTH часов контента в месяц.
	- Каждое видео в среднем длится DURATION минут.
	- Следовательно, 1.6 млн. видеороликов в месяц загружаются на платформу.  
	- `(UPLOAD_MONTH * K * 60) мин / DURATION = (21_900_000 час * 0.038 * 60) мин / 11.7 мин = 4.3 млн видеороликов`
- **Показ рекомендаций на главной странице** - 70 млрд. / мес.
	- В среднем, каждый пользователь в день посещает VISITS_DAY_USER страниц сервиса.
	- Допустим, на каждые 3 посещения страницы видео приходится одно посещение главной страницы
	- Тогда, общее количество посещений этих страниц, а, следовательно, и предложенных рекомендаций 276 млрд. 
	- ```VIEWS_MONTH * 1/3 = 207 * 1/3 = 70 млрд```
- **Лента подписок** -  2.07 млрд. / мес.
	- Допустим, что на каждые 100 просмотров видео приходится 1 просмотр ленты подписок. 
	- `VIEWS_MONTH * 1/100 = 207 * 1/100 = 2.07 млрд`
- **Авторизация** - 6.24 млрд. / мес
	- При каждом пользовательском визите сервиса (при каждой новой вкладке браузера) клиентской стороне необходимо проверить авторизацию пользователя по id сессии.
	- Допустим, что в день пользователь заходит на сервис 4 раза (`VIEW_DURATION_DAY / VISIT_TIME = 86 / 20 = 4`)
	- Тогда `DAU_RUS * 4 = 52 млн * 4 = 208 млн.` запросов на проверку авторизации в день. Значит, `208 млн. * 30 = 6.24 млрд.` запросов на авторизацию в месяц.
- **Поиск** - 3.9 млрд. / мес.
	- В день происходит SEARCH_DAY поисковых запросов
	- `SEARCH_DAY * K * 30 = 3.5 млрд. 0.038 * 30 = 3.9 млрд`

## 2.2 Технические метрики

### 2.2.1 Сводные таблицы технических метрик

|Хранилище |Стартовый размер (Пб)|Увеличение (Пб/год)|
| ------------- |-------------|-------------|
|Видео |540|540\[[24](https://youtubeprofi.info/raspolozhenie-i-kolichestvo-serverov-youtube/#:~:text=%D0%92%202016%20%D0%B3.%20%D1%83%20YouTube%20%D0%B1%D1%8B%D0%BB%D0%BE%201%20%D0%9F%D0%91%20(1%20000%20%D0%A2%D0%91)%20%D0%B2%20%D0%B4%D0%B5%D0%BD%D1%8C%20%D0%BD%D0%BE%D0%B2%D0%BE%D0%B3%D0%BE%20%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%BD%D1%82%D0%B0%2C%20%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B6%D0%B5%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE%20%D0%BD%D0%B0%20%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D1%8B.)]|
|Данные пользователя |4|2|

|Сетевой трафик |Потребление|
| ------------- |-------------|
|Загрузка видео (Пб / сутки) |2.4|
|Отдача видео (Пб / сутки) |133|
|Отдача статики (Пб / сутки)|25|
|Средний исходящий трафик (Пб / сутки) |158 (133 + 25)|
|Пиковый трафик отдачи видео (Тбит / с)|44|
|Пиковый исходящий трафик (Тбит / с)|54|
|Пиковый входящий трафик (Тбит / с)|0.8|

|Тип запроса |RPS (Статика)|
| ------------- |-------------|
|Станица видео|2.5 млн|
|Главная страница|1351 тыс|
|Главная страница|75 тыс|
|Лента подписок|38 тыс.|
|Загрузка видеоролика на канал|17|

|Тип запроса |RPS (API)|
| ------------- |-------------|
|Станица видео|307 тыс|
|Главная страница|54 тыс|
|Добавление лайка|6.4 тыс|
|Проверка авторизации|4.8 тыс|
|Поиск|3 тыс|
|Подписка на канал|2.8 тыс|
|Лента подписок|1.5 тыс|
|Добавление комментария|800|
|Загрузка видеоролика на канал|4|

### 2.2.2 Расчет технических метрик
#### 2.2.2.1 Хранилище
- **Видео**
	- Во всем мире UPLOAD_DAY часов видео загружается на платформу каждый день. Значит, в России будет загружаться ```UPLOAD_DAY * K = 720 тыс. * 0.038 = 27_360 часов``` контента в день.
	- Допустим, все видео имеют разрешения 1080p \[[25](https://osgamers.com/frequently-asked-questions/what-is-the-best-quality-for-youtube#:~:text=Statistics%20and%20research%20has%20shown%20that%20the%20amount%20people%20watching%20care%20about%20quality%20peaks%20at%201080p.%20There%20is%20no%20advantage%20as%20far%20as%20your%20viewers%20are%20concerned.)] \[[26](https://osgamers.com/frequently-asked-questions/what-is-the-best-quality-for-youtube#:~:text=The%20most%20common%20resolutions%20for%20Youtube%20videos%20are%201280%20x%20720p%20and%201920%20x%201080p)] с частотой кадров 30 fps в формате SDR. В хранилище хранятся все варианты разрешений для обеспечения адаптивного битрейта.
		- 2160p (4K) - 40 Mbps \[[27](https://support.google.com/youtube/answer/1722171?hl=en#zippy=%2Cframe-rate%2Cbitrate%2Cvideo-codec-h%2Cvideo-resolution-and-aspect-ratio%2Ccolor-space:~:text=Recommended%20video%20bitrates%20for%20SDR%20uploads)]
		- 1440p (2K) - 16 Mbps
		- **1080p - 8 Mbps** 
		- 720p - 5 Mbps                                                 
		- 480p - 2.5 Mbps
		- 360p - 1 Mbps
	- ```(27_360 час * 60 * 60) сек * (8 + 5 + 2.5 + 1) Mpbs / 8 / 1024 = 198.4 тыс. Гб```
	- В день 95 млн. человек загружают 198.4 тыс. Гб видео. Значит, каждый год требуется `198_400 * 365 = 72 Пб` памяти для хранения видео (не включая резервные копии)
	- Для обеспечения надежности хранения данных в S3 хранилище допустим, что каждое видео имеет **2 резервных копии** (хранение разных версий файлов не учитываем). Таким образом, в год необходимо резервировать хранилище размером 216 Пб. 
	- Так как в современном мире все больше выпускается **4K камер**, данные разрешение становится все более и более доступным для пользователя. Поэтому увеличим размер хранилища в 2.5 раза (40 Mbps / 8 Mbps / 2 = 2.5). Выходит, **540 Пб**. 
	- Пусть стартовый объем хранилища на первый год будет таким же.
- **Пользовательские данные**
	- Рассчитаем размер данных в объектом хранилище на пользователя. Размер данных, хранящихся в СУБД, приведен в разделе Физическая схема БД.
	 - Допустим, остальные пользовательские данные имеют следующий размер
		 - Аватар пользователя - 2 Мб
		 - Превью видео - 10 Мб
	- 36 Мб с учетом **двух резервных копий** необходимо на каждого пользователя. 
	- MAU_RUS - 95 млн, пользователей интернета в России - 130 млн.
	- Допустим, что общее количество аккаунтов 110 млн, значит, **стартовый размер хранилища** `45 Мб * 110 млн =` **4 Пб** 
	- Пусть увеличение хранилища в год требует двое меньше объема, т.к. многие данные фиксируются на момент регистрации пользователя.  

#### 2.2.2.2 Потребление трафика
- **Среднее использование трафика (в сутки)**
	- **Отдача видео (download)** - 133 Пб / сутки
		- В среднем, DOWNLOAD_MIN часов видео транслируется каждую минуту. 
			- `(DOWNLOAD_MIN * K * 60 * 60) (сек/мин) / 60 (сек/сек) * 8 Mbps / 8 / 1024 = 1545 Гб / сек`
			- `1545 * 60 * 60 * 24 = 133 Пб / сутки`
	> Другой вариант расчета: `VIEW_DURATION_DAY * 60 * DAY_RUS * 8 Mbps / 8 / 1024 / 1024 / 1024 = 250 Пб / сутки`
	- **Страницы (стартовая, страница видео, лента подписок)** - 25 Пб / сутки
		- html, js, css - 2 Мб
		- Превью в формате WebP - 20 шт. * 50 Кб = 1 Мб
		- Выше нами было допущено, что в месяц
			- главная страница открывается 1/3 * VIEWS_MONTH раз
			- страница видео открывается VIEWS_MONTH раз
			- лента подписок открывается 1/100 VIEWS_MONTH раз
		- `(1 + 1/3 + 1/100) * VIEWS_MONTH * 3 Мб / 30 = 1.34 * 207 млрд. * 3 Мб = 25 Пб / сутки`
	- **Загрузка видео (upload)** - 2.4 Пб / сутки
		- В день загружается UPLOAD_DAY часов видео в день. 
			- `UPLOAD_DAY * 60 * 60 * 8 Mbps / 8 /  1024 / 1024 / 1024 = 720 тыс. * 60 * 60 * 8 Mbps / 8 / 1024 / 1024 / 1024 = 2.4 Пб / сутки`
- **Пиковое использование трафика (в секунду)**
	- **Отдача видео (download)** - 44 Тбит / c 
		- Допустим, что пиковое (дневное) потребление в 1.8 раза больше среднего \[[28](https://cloud.mail.ru/public/8Mau/tKmAN3kqb)] (слайд 14), получим `133 Пб/сутки * 1024 Тб * 1.8 / 86400 * 8 = 22 Тбит / c`
		- Также учтем некоторый **запас** по потреблению трафика - x2. Таким образом, потребление **44 Тбит/c**. 
	- **Общее пиковое потребление (видео и страницы)** - 54 Тбит / c 
		-  `(133 Пб/сутки + 25 Пб/сутки) * 1024 Тб * 1.8 / 86400 * 8 * 2 = 54 Тбит / c`
	- **Загрузка видео (upload)** - 0.8 Тбит / c
		- С учетом дневного пика и запаса (1.8 \* 2 = x3.6) - **0.8 Тбит / c**

#### 2.2.2.3 RPS по типам запросов

- Количество подзапросов для каждого действия пользователя
	- **Страница видео** (18 запросов)
		- Статика
			- Первый видеочанк - 1 запрос
			- Превью - 10 запросов 
			- html, js, css - 5 запросов
		- Динамика
			- Комментарии, кол-во лайков, просмотров, описание видео - 1 запрос
			- Рекомендации - 1 запрос
	- **Поиск / главная страница** (26 запросов)
		- Статика
			- Превью - 20 запросов 
			- html, js, css - 5 запросов
		- Динамика
			- Поиск / Рекомендации - 1 запрос
	- **Лента подписок** (26 запросов)
		- Статика
			- Превью - 20 запросов 
			- html, js, css - 5 запросов
		- Динамика
			- Список подписок - 1 запрос
	- **Добавление лайка** (Динамика - 1 запрос)
	- **Подписка на канал** (Динамика - 1 запрос)
	- **Добавление комментария** (Динамика - 1 запроса)
	- **Загрузка видеоролика на канал** (6 запросов)
		- Статика
			- html, js, css - 5 запросов
		- Динамика
			- Отправка параметров загрузки видео - 1 запрос
	- **Проверка авторизации** (Динамика - 1 запрос)

> Увеличим каждое значение RPS в 2 раза для отказоустойчивости в случае пиковых нагрузок

|Тип запроса |RPS (Статика)|
| ------------- |-------------|
|Станица видео|`207 млрд / 30 / 86400 * 16 * 2 = 2.5 млн`|
|Главная страница|`70 млрд / 30 / 86400 * 27 * 2 = 1351 тыс`|
|Поиск|`3.9 млрд / 30 / 86400 * 27 * 2 = 75 тыс`|
|Лента подписок|`2 млрд / 30 / 86400 * 25 * 2 = 38 тыс.`|
|Загрузка видеоролика на канал|`4.3 млн / 30 / 86400 * 5 * 2 = 17`|

|Тип запроса |RPS (API)|
| ------------- |-------------|
|Станица видео|`207 млрд / 30 / 86400 * 2 * 2 = 307 тыс`|
|Главная страница|`70 млрд / 30 / 86400 * 1 * 2 = 54 тыс`|
|Поиск|`70 млрд / 30 / 86400 * 1 * 2 = 3 тыс`|
|Лента подписок|`2 млрд / 30 / 86400 * 1 * 2 = 1.5 тыс.`|
|Добавление лайка|`8.3 млрд / 30 / 86400 * 1 * 2 = 6.4 тыс`|
|Проверка авторизации|`6.2 млрд / 30 / 86400 * 1 * 2 = 4.8 тыс`|
|Подписка на канал|`3.6 млрд / 30 / 86400 * 1 * 2 = 2.8 тыс`|
|Добавление комментария|`1 млрд / 30 / 86400 * 1 * 2 = 800`|
|Загрузка видеоролика на канал|`4.3 млн / 30 / 86400 * 1 * 2 = 4`|

## 3. Глобальная балансировка нагрузки <a name="3"></a>

### 3.1 Схема глобальной балансировки до датацентров

Схема глобальной балансировки включает следующее:
1. Кэши в локальных ISP.
2. CDN подключены к крупным IX.
3. GeoDNS выбирает ближайшую группу CDN.
4. BGP Anycast до ближайшего CDN.
5. CDN кэширует статику и ускоряет динамику с ЦОДов


*Пояснения:*
1. Предложение всем локальным **ISP** использовать специальные **Storage сервера сервиса YouTube**\[[29](https://www.youtube.com/watch?v=g5v2-H-sabM&t=537s)] (уменьшение трафика с внешних линков ISP, ускорение доставки контента пользователям).
2. Сеть **CDN** подключена к крупным точкам обмена трафика **Internet Exchange** (Cloud-IX объединяет в себе крупные Российские и не только IX \[[30](https://hww.ru/2021/03/tochki-obmena-trafikom-ili-kak-ustroen-internet-v-rossii/#:~:text=SDN%2C%20Xelent%2C%20Infobox.-,Cloud%2DIX,-%D0%92%202012%20%D0%B3)])
3. **GeoDNS** сервера определяют локацию пользователя и возвращают IP адрес, за которым находится группа CDN. CDN, в свою очередь, знают об адресах ЦОДов. 
4. Узлы сети в России распределены неравномерно \[[31](https://telecomlife.ru/magistralnaya-set/)], поэтому, если назначить всем CDN один IP адрес, **BGP Anycast** может выбрать оптимальный маршрут с точки зрения топологии (метрика - количество хопов), но географически неоптимальный \[[32](https://youtu.be/LPCbKzhvAGc?t=1265)]. Необходимо кластеризовать сервера CDN на локальные группы и назначить каждой группе один IP адрес. Маршрут до ближайшего к пользователю CDN в рамках группы будет выбран BGP роутером. Текущая конфигурация называется "Региональный Anycast" \[[33](https://www.youtube.com/watch?v=LPCbKzhvAGc&t=1370s)]. Преимущества и недостатки BGP Anycast \[[34](https://www.youtube.com/watch?v=9VVzu87lVbE&list=LL&index=12&t=1240s)].
5. CDN, близкий к пользователю сервер, держит с граничными маршрутизаторами ЦОДов постоянные "прогретые" соединения (увеличенное окно передачи) для ускорения доставки динамического контента и статики. Статика кэшируется на CDN.  

### 3.2 Физическое расположение датацентров

- Наибольшая плотность населения России приходится на западную и юго-западную часть \[[35](https://www.statdata.ru/karta/plotnost-naseleniya-rossii)]. 
- Расположим сервера в следующих городах: в Москве в двух зонах доступности и в Новосибирске (расположение ЦОДов в России \[[36](https://yandex.ru/maps/?display-text=Дата-центры&ll=42.072758%2C49.349507&mode=search&sctx=ZAAAAAgBEAAaKAoSCQiPNo5Y4FhAEUsgJXZt2U5AEhIJYJD0aZV0ZUAR9wX0wp1ZREAiBgABAgMEBSgKOABAkE5IAWIScG9pbnRfY29udGV4dF92Mj0xagJydZ0BzcxMPaABAKgBAL0B4DucO8IBMOO21I%2BKB8OQmYjwAfD6p5jyAvLMr6q2Aqi8qOj3BeP4xfGAAdPYgcuVBZG31ZugBeoBAPIBAPgBAIICGmNhdGVnb3J5X2lkOigxNzE2NDcwNDgzNDUpigIMMTcxNjQ3MDQ4MzQ1kgIAmgIMZGVza3RvcC1tYXBzqgIzMTkzNzMyNDc4MjQyLDIyMjI1ODM2NTUyMywyMDcyNDkzNzMyOTgsMjE2NjE4NTAzODAx&sll=42.072758%2C49.349507&sspn=18.673879%2C7.829843&text=category_id%3A%28171647048345%29&z=5.81)]\[[37](https://www.datacentermap.com/russia/)])
- Сеть CDN будет состоять из 100 серверов, расположенных по всей стране. 
- Сервера можно арендовать в ЦОДах существующих компаних, например, Selectel \[[1](https://docs.selectel.ru/control-panel-actions/selectel-infrastructure/#инфраструктура-selectel)] имеет 2 AZ в Москве и 1 AZ в Новосибирске, или построить свои ЦОДы. Этот вопрос необходимо согласовать с бизнесом.  
 ![](img/Pasted%20image%2020231002075842.png)
 > 1. VK имеет 3 ЦОДа в Москве, в Санкт-Петербурге и Екатеринбурге \[[38](https://uchet-jkh.ru/i/gde-naxodyatsya-servera-vkontakte/#:~:text=%D0%A1%D0%B5%D0%B9%D1%87%D0%B0%D1%81%20%D0%92%D0%9A%D0%BE%D0%BD%D1%82%D0%B0%D0%BA%D1%82%D0%B5%20%D0%B8%D0%BC%D0%B5%D0%B5%D1%82%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D0%BA%D1%80%D1%83%D0%BF%D0%BD%D1%8B%D1%85%20%D1%86%D0%B5%D0%BD%D1%82%D1%80%D0%BE%D0%B2%20%D0%BE%D0%B1%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%BA%D0%B8%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)], а также сеть CDN, состоящую из 100 серверов \[[39](https://habr.com/ru/news/731714/#:~:text=%D0%9F%D0%BE%20%D0%B8%D1%85%20%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC%2C%20VK%20%D1%83%D0%B6%D0%B5%20%D0%BD%D0%B5%D1%81%D0%BA%D0%BE%D0%BB%D1%8C%D0%BA%D0%BE%20%D0%BB%D0%B5%D1%82%20%D1%80%D0%B0%D0%B7%D0%B2%D0%B8%D0%B2%D0%B0%D0%B5%D1%82%20%D1%81%D0%B2%D0%BE%D1%8E%20CDN%2D%D1%81%D0%B5%D1%82%D1%8C%2C%20%D0%B0%20%D0%B2%20%D1%8D%D1%82%D0%BE%D0%BC%20%D0%B3%D0%BE%D0%B4%D1%83%20%D0%BA%D0%BE%D0%BC%D0%BF%D0%B0%D0%BD%D0%B8%D1%8F%20%D1%80%D0%B5%D1%88%D0%B8%D0%BB%D0%B0%20%D1%83%D0%B2%D0%B5%D0%BB%D0%B8%D1%87%D0%B8%D1%82%D1%8C%20%D0%B5%D1%91%20%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80%D0%BD%D0%BE%20%D0%B2%D0%B4%D0%B2%D0%BE%D0%B5%20%D0%B1%D0%BE%D0%BB%D0%B5%D0%B5%20%D1%87%D0%B5%D0%BC%20%D1%81%D0%BE%20100%20CDN%2D%D1%83%D0%B7%D0%BB%D0%BE%D0%B2%2C%20%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5%20%D1%81%D0%B5%D0%B9%D1%87%D0%B0%D1%81%20%D0%B5%D1%81%D1%82%D1%8C%20%D1%83%20VK%20%D0%BF%D0%BE%20%D0%B2%D1%81%D0%B5%D0%B9%20%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B5.)]. 
 > 2. Сервис YouTube имеет один российский ДЦ в Москве \[[40](https://youtubeprofi.info/raspolozhenie-i-kolichestvo-serverov-youtube/#:~:text=%D0%B2%20%D0%A0%D0%BE%D1%81%D1%81%D0%B8%D0%B8%20%D0%B8%20%D0%AE%D0%B6%D0%BD%D0%BE%D0%B9%20%D0%90%D0%BC%D0%B5%D1%80%D0%B8%D0%BA%D0%B5%20%E2%80%93%20%D0%BF%D0%BE%201.)] (основную нагрузку по потреблению видео берут на себя CDN)
 

## 4. Локальная балансировка нагрузки <a name="4"></a>

### 4.1 Схема локальной балансировки для входящих и межсервисных запросов

1. **CDN держит "прогретые" соединения с граничными маршрутизаторами** в ЦОДах. Для обеспечения доступности и отказоустойчивости зарезервируем второй маршрутизатор (резервирование N+1). Используем для этого протокол **VRRP**. 
2. Многоуровневая балансировка. После граничного маршрутизатора следует **слой коммутаторов**, далее - **слой L7 балансировщиков**. Симметричная топология позволяет балансировать трафик посредством стека **BGP, ECMP**.
	- на каждом балансере программный роутер (для BGP Anycast)
	- режим ECMP **per-flow** для "закрепления" одной tcp сессии за одним сервером
	- для избавления от поляризации в ECMP добавление соли в хэш каждым узлом.
	- ECMP балансирует на основе **5-tuple hashing**. Расширим реализацию добавлением **consistent hashing** для минимизации расщепления tcp сессий в случае добавления или удаление из балансировки L7 балансировщиков. 
4. **L7 балансировщики** - **ingress controller**-ы кластера **k8s**. Контроллеры расположены на отдельных нодах из соображений безопасности, доступности, отказоустойчивости.  На каждом узле k8s присутствует Service Discovery (kube-dns). 
	- реализация L7 balancer - **envoy**
	- динамическое конфигурирование upsteram-ов через Service Discovery (**?**).
	- **liveness** и **readiness** пробы
	- **функциональная балансировка** по субдоменам и префиксам. 
	- **резервирование балансировщиков через VRRP**
	- **rate limiting**
	- **ssl termination**. Оптимизация: **session cache, session tickets** 
5. Балансировка Service-ом запросов между подами в рамках одного replicaset. Т.к. сервис - это правила iptables, то балансировка примитивная - **случайное распределение**. 

## 5. Логическая схема БД <a name="5"></a>

### 5.1 ER диаграмма логического уровня

На рисунке представлена **нормализованная ER диаграмма логического уровня**, показывающая все сущности, связи между ними и атрибуты ([ER-диаграмма](https://drawsql.app/teams/bmstu-7/diagrams/youtube))

![](files/drawSQL-youtube-export-2023-10-19.png)

Видеофайлы, превью видео, аватарки будут храниться в **объектом хранилище**. В соответствующих таблицах присутствуют ссылки на хранилище.

### 5.2 Характер нагрузки, требования к системе
- **Много чтений, мало записей** (см. сводную таблицу продуктовых метрик)
- **Чтение может отставать** от записи (**not consistency**)
	- Только что вышедший ролик может не сразу появляться на странице
	- Лайк одного пользователя может не сразу инкрементировать счетчик лайков у другого
- Есть блогеры, у которых много контента (**hot spot**)
- **Не толерантны к задержке**
	- Главная страница с рекомендациями должна загружаться быстро
	- Видео на странице видео должно сразу воспроизводиться
- **Большинство запросов селективные по ключу** 

|Тип запроса |RPS (API)|Характер запроса|
| ------------- |-------------|--------|
|Станица видео|307 тыс|**Селективный** запрос по video_id|
|Главная страница|54 тыс|**Селективные** запросы по video_ids (video_ids от рекомендательной системы)|
|Добавление лайка|6.4 тыс|**Селективный** запрос по video_id и инкремент поля|
|Проверка авторизации|4.8 тыс|**Селективный** запрос по user_id|
|Поиск|3 тыс|**Full scan** запрос|
|Подписка на канал|2.8 тыс|**Селективный** запрос по user_id и обновление поле списка подписчиков|
|Лента подписок|1.5 тыс|**Селективный** запрос по user_id, **селективные** запросы по video_ids|
|Добавление комментария|800|Добавление записи по новому comment_id|
|Загрузка видеоролика на канал|4|Добавление записи по новому video_id, загрузка исходного видео в хранилище, асинхронная обработка видео|
- **Онлайн выдача рекомендаций** по user_id
- Быстрые атомарные **инкременты** (количество лайков, подписчиков, комментариев, просмотров)
- **Асинхронная** многостадийная **обработка загружаемых видео**

Исходя из характера запросов и особенностей системы схему данных нужно **денормализировать** (см. физическую схему БД). 
> Денормализованная БД позволит не обращаться к разным сущностям для сбора всей информации, необходимой для рендера страницы, что уменьшит задержку. Однако вследствие дублирования данных в нескольких таблицах необходимо организовать **асинхронные процессы обновления дублируемых полей** в одних таблицах при обновлении этих полей в других. 

## 6. Физическая схема БД <a name="6"></a>

> При расчете размера хранения id всех сущностей имеет тип uuid4 (16 байт)

### 6.1 Метаданные видео
![](files/Pasted%20image%2020231020011417.png)


- **СУБД**
	- Cassandra (key - wide columns)
- **Индексы**
	- Partition key - video_id
- **Запросы**
	- Запросы по video_id
		- Чтение данных column set-а
		- Обновление полей
	- Добавление строки
- **Подходы к масштабированию**
	- Шардирование по video_id
- **Резервное копирование** 
	- Присутствует
- **Размер хранения** - 2.3 Тб / год
	- Средний размер строки 660 байт
	- Загрузки видео имеет 4 RPS
	- `660 * 4 * 86400 * 30 * 365 = 2.3 Тб / год`

*Замечания*
- Размерность ML фичей фиксирована, поэтому количество столбцов заранее известно

### 6.2 Комментарии под видео

![](files/Pasted%20image%2020231020011615.png)


-  **СУБД**
	- Mogno (document-oriented)
- **Индексы**
	- **btree index**: (video_id, count_likes)
	- **btree index**: (video_id, created_at) 
	- **btree index**: (video_id, comment_id) 
- **Запросы**
	- Топ комментариев по количеству лайков
	- Новые комментарии
	- Конкретный комментарий (для вложенных комментариев)
	- Добавление новых комментариев
	- Обновление полей
		- count_likes
		- text
		- likes_user_ids
		- username
- **Подходы к масштабированию**
	- Много реплик на чтение (т.к. страница видео имеет большой RPS, каждый раз отображаются топ комментариев)
- **Резервное копирование** 
	- Присутствует
- **Размер хранения** - 227 Тб / год
	- Средний размер строки 330 байт
		- Среднее количество лайков на видео 12 \[[1](https://charmbeautify.com/lifestyle/how-many-likes-does-the-average-person-get-on-youtube#:~:text=got%20your%20answer%3A-,The%20average%20person%20on%20YouTube%20gets%20about%2012%20likes%20per%20video,-.)]
		- Среднее количество символов в комментарии 58 \[[2](http://cba.scit.wlv.ac.uk/~cm1993/papers/CommentingYouTubePreprint.pdf)]
	- Добавление комментария имеет 800 RPS
	- `330 * 800 * 86400 * 30 * 365 = 227 Тб / год`



### 6.3 Реакции пользователя
![](files/Pasted%20image%2020231020011631.png)

- **СУБД**
	- Tarantool (in-memory)
- **Индексы**
	- **btree index**: (user_id, video_id)
	- **btree index**: (user_id, created_at) 
-  **Запросы**
	- Поиск реакции пользователя на видео (селективный запрос по ключу)
	- Понравившиеся и просмотренные видео пользователя (количество лимитировано)
	- Добавление строк
	- Обновление views.continue по (user_id, video_id)
	- Обновление likes.vote по (user_id, video_id)
- **Подходы к масштабированию**
	- Шардинг по user_id
	- Репликация master/slave
- **Резервное копирование** 
	- Присутствует
- **Размер хранения** - Лайки 240 Тб/год и просмотра 11 Пб / год
	- Средний размер строки 44 байта
	- Добавление лайка имеет 6.4 тыс. RPS
	- Страница видео (добавление просмотра) имеет 307 тыс. RPS
	- Лайки: `44 * 6400 * 86400 * 30 * 365 = 240 Тб / год`
	- Просмотры:  `44 * 307000 * 86400 * 30 * 365 = 11.2 Пб / год`

### 6.4 Данные пользователя 

![](files/Pasted%20image%2020231020011807.png)

-  **СУБД**
	- Cassandra (key - wide columns)
- **Инексы**
	- Partition key - user_id
 - **Запросы**
	- Запросы по user_id
		- Чтение данных column set-а
		- Обновление полей
	- Добавление строки
- **Подходы к масштабированию**
	- Шардирование по user_id
- **Резервное копирование** 
	- Присутствует
- **Размер хранения** - 57 Гб
	- Средний размер строки - 530 байт
		- В среднем на человека приходится 15 видео (`(UPLOAD_YEAR / MAU_RUS) * 60 / DURAION = 15`)
	- MAU в России 95 млн. Берем 110 млн, т.к. необходимо хранить данные не только активных пользователей.
	- `530 * 110_000_000 = 57 Гб`

### 6.5 Сессия пользователя

![](files/Pasted%20image%2020231020014838.png)

- **СУБД**
	- Redis (key - touple)
- **Индексы**
	- Ключ - session_id
- **Запросы**
	- Чтение по ключу session_id
	- Добавление новой сессии
- Подходы к масштабированию**
	- Шардирование по session_id
	- Репликация master/slave
- **Резервное копирование** 
	- Присутствует
- **Размер хранения** - 3.7 Гб
	- Средний размер строки - 40 байт
	- MAU в России 95 млн. Сессии хранятся только активных пользователей.
	- `40 * 95_000_000 = 3.7 Гб`

### 6.6 Другие хранилища и сервисы
- **Kafka**
	- Прием логов, статистики, стриминг consumer-ам
	- Очередь на обновление данных в денормализованной схеме
	- Обеспечение работы pipeline-а обработки видео
	- Буфер запросов
- **S3** 
	- Видеочанки
	- Аватарки пользователей
	- Превью видео
	- ML модели
- **Feature store**
	- Специализированная БД для хранения фич для ML
- **Spark streaming**
	- Прием на потоке данных из kafka
	- MapReduce (e.g. определение топ тегов для каждого видео)
- **Hadoop**
	- Хранение данных для рекомендательной системы
- **ClickHouse**
	- Аналитика для аналитиков
- **Elastic Search**
	- Поиск видео по запросу
	- Поиск по названию, описанию, тегам, специфике канала
 
### [Физическая схема](https://drive.google.com/file/d/1pL2ZMze6rzH9rk132Jd2hL3U_TRu0PPp/view?usp=sharing)
![2 drawio](https://github.com/mmikhail2001/Highload_YouTube/assets/71098937/a6b82270-21d2-46da-b4d8-fcf3216bf799)



## 7. Алгоритмы <a name="7"></a>

### 7.1 Рекомендации 

Стандартный пайплайн включает следующие этапы.
- Подготовка рекомендаций (оффлайн в hadoop)
	- Расчет дневных статистик (совстречаемости товаров), фичей
	- Агрегирование, подбор кандидатов
	- Фильтрация/дополнительное ранжирование
	- В общем, **каждую ночь готовиться новый пакет рекомендаций в соответствие с моделью совстречаемости**
- Ответ на запрос, формирование выдачи (онлайн)
	- Разбор запроса
	- Поход за рекомендациями в рекомендательную систему (item2item рекомендации)
	- Сортировка (по какому-то полю), фильтрация.
	- Походы в БД за метаданными видео (превью, название, канал), проверка.

### 7.2 Пайплайн обработки видео

Пайплан обработки видео состоит из следующих стадий. Запускается во много потоков на многих нодах.
- file chunker
	- Разбиение исходных видеофайлов на чанки
- content filter
	- Фильтрация контента нейросетями (цензура).  
- content tagger
	- Определение ключевых тегов видео, категории видео нейросетями.
- transcoding
	- Транскодирование видеочанка в несколько форматов, чтобы обеспечить доступность для разных типов устройств (MP4, WebM и т.д.)
- quality converter
	- Конвертация видеочанка в различные разрешения (1080, 720, 480 и т.д.)
- upload to CDN, S3
	- Загрузка видео на главные сервера CDN и в кластер S3. 