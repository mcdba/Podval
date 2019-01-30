# Podval
http://arduino.ru/forum/proekty/kontrol-vlazhnosti-podvala-arduino-pro-mini
Есть в загородном доме два подвала которые гидроизолированы от земли (нет поступления воды из грунта). Зимой на стенах появляется конденсат, предметы покрываются влагой. Температура в подвалах не опускается ниже 0...+2 градусов, т.е. вода не замерзает. Это достигнуто утеплением стен подвала снаружи ЭППС и нагревом пола подвала (он тепло не изолирован) землей.
Задача убрать выпадение конденсата в подвалах.

1. Принцип работы
Идея контроля влажности подвала была подсмотрена на http://geektimes.ru (http://geektimes.ru/post/255298/)
Вся идея состоит в том чтобы измерить температуру и относительную влажность в подвале и на улице, на основании температуры и относительной влажности рассчитать абсолютную влажность и принять решение о включении вытяжного вентилятора в подвале. Теория для расчета изложена здесь - https://carnotcycle.wordpress.com/2012/08/04/how-to-convert-relative-humidity-to-absolute-humidity/
2. Выбор элементной базы.
В качестве контроллера было выбрано Arduino pro mini.  Это мой второй проект (первый - контроль работы теплового насоса был сделан на uno). Изучив предлагаемые контроллеры Arduino  пришел к выводу что существует  всего три варианта - DUE (отдельная песня и мне кажется тупик),  MEGA и все остальное (сильно схожие контроллеры на одном чипе 328).  Задача достаточно простая, по этому выбор пал на pro mini (не на мегу).
В pro mini был прошит загрузчик optiboot 5 версии, это дало возможность использования сторожевого таймер и увеличением максимального размера кода.
Устройство индикации - до этого были испытаны монохромные дисплеи 4х20 символов и графический 128х64 точки на 9372. Появилось желание подключить  дисплей цветной ЖК. В виду малого количества ножек у МК было использовано подключение по SPI .  Ebay  - имеет большое количество предложений на контроллере ILI9341 с размерами диагонали 1.8-2.8 дюйма и разрешением 320х240 точек. В эту конструкцию был куплен дисплей 2.4 дюйма.
Т.к. устройство будет устанавливаться в подвале то для наблюдения я хочу сделать выносной пульт со связью по радиоканалу. Для организации радиоканала были использованы модули на основе nrf24. Планируется система типа "умного дома" каждый блок (а сейчас есть уже два блока в подвалах) посылает пакет о своем состоянии в головное устройство.
Исполнительное устройство. В начале был использован классический одно канальный релейный шилд для ардуино. Корпус проектировался под него (есть даже выступы для крепления). После установки устройства  в подвале был выявлен косяк - зависание контроллера дисплея при выключении вентилятора (25 вт). Сразу были увеличены конденсаторы по питанию, поставлен LC фильтр по 5 вольтам, дисплей был переведен на питание 3.3 вольта от внешнего стабилизатора (он уже был и использовался для питания nrf24). Все эти меры ничего не дали.
Затем возникла идея сброса контроллера дисплея при зависании. Программный сброс обеспечил. А вот как читать регистры дисплея так и не разобрался (типа прочел регистр все ок не прочел надо перегружать). Был организован периодический сброс - раз в час. Решение кривое. Все это делалось за городом из того что было. Решение меня не удовлетворило. Пришлось везти блок в Москву, там было установлено твердотельное реле шарп S202S02 ( с зеро кросс блоком) и помехогасящий конденсатор на 0.1 мкф. После этого устройсво заработало без зависаний.
Используется датчик темперуры и влажности DHT-22, штатное включение, подтягивающий резистор 5 кОм, длина провода до 6 метров (что испробовал).
Для согласования уровней 5 вольт ардуино и дисплея используются делители из резисторов 2к+4к. Согласвоания уровне й для nrf24 не требуется.

2. Программа.
У меня есть достаточно богатый опыт программирования (с 89 года) но все это касается полноценных компьютеров, последнее время занимаюсь разработкой управляющих систем на arm под самосборной допиленной Linux,  пишу либо на голом с или qt. Опыта программирования МК не имел, но старался перенести свой опыт на них.
Использую шедуллер задач leos ver 1 (первая версия использует таймер вторая использует сторожевой таймер), это "обертка" таймера для организации периодического вызова функций (у меня вызов опроса датчика).
Сторожевой таймер используется для борьбы с зависанием МК (я их не наблюдал но пусть будет).
Для работы с дисплеем используется библиотека ucglib версии не позднее 1.02. D более поздних версиях переработан формат шрифтов и есть проблемы с русификацией в кодировке UTF-8, допиливать было лень.
Для nrf24 используется библиотека от маньяка.
Для чтения датчиков DHT используется самая свежая библиотека с официального сайта. Там кодов ошибок больше.
Реализован подсчет мото часов блока и вентилятора.Кнопка повешена на прерывание и позволяет менять настройки включения вентилятора, смотреть информацию об устройстве (длительное нажатие) и сбрасывать счетчики (нажатие при включении).
Реализован вывод ошибок чтения DHT-22 (верхний правый угол цифры), с чтением были проблемы, после оптимизации кода они ушли.
На графике выводится не весь диапазон темперур (вывод -20...+20) для увеличения разрешающей способности графика в области допустимых темератру в подвале (0...+20) .
3. Конструктив.

Корпус напечатан на 3Д принтере (нижняя часть склеена из двух частей). Очень удобно, все отверстия сразу сделаны, только заусецы счистить и можно использовать. Получилось достаточно компактно. Вся электроника кроме входных цепей распаена на макетке, проводом (по времени один вечер).

Замечанные косяки -

1. На дисплее появляются "не штатные"  точки на графике (скоре всего помехи по SPI после перерисовке экрана они проподают),

2. Кнопка иногода "не четко" срабатывает проблемы с длительным нажатием.

3. nrf24 в блоке ожидает "квитанции" о приеме пакета иногда "квиатнция" не приходит (30-40%) но пакет доставляется до приемного устройства.

4. Уличный датчик DHT-22  показания влажности прыгают утром (ветер, солнце?). Датчик менял.