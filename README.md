  ## Система удаленного запуска двигателя автомобиля (SIM800L + Arduino), c управлением по DTMF, и отчетами по SMS.

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/V1.7.2.JPG)

Без корпуса

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/1.7.2.0.JPG)


![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/1.72-01.JPG)

# Прошивка, скетч

[Актуальный скетч с отправкой данный на сервис narodmon.ru](https://raw.githubusercontent.com/martinhol221/SIM800L_DTMF_control/master/Autostart_Sim800L_narodmon.ino) в Arduino Pro Mini (8Mhz/3.3v) для [Arduino IDE](https://www.arduino.cc/en/main/software)

[Актуальный скетч с управлением по MQTT протоколу (из приложения)](https://github.com/martinhol221/SIM800L_MQTT) в Arduino Pro Mini (8Mhz/3.3v) для [Arduino IDE](https://www.arduino.cc/en/main/software)

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/loading.JPG)

Последние изменения в прошивке:

* добавлен аглоритм активациии и деактивации автопрогрева

* добавлен аглоритм активациии и деактивации отправки данных на сервер

* добавлена функция перезагрузки модема если оператор блокирует трафик (бывает при отрицательном балансе), при новой регистрации в сети передача данных возобновляется

# Конфигурация скетча :

* номер телефона хозяина для входящих вызовов `call_phone= "+375290000000";` 

* номер телефона куда отправляем СМС отчет `SMS_phone= "+375290000000";`

* адрес устройства на сервере `MAC = "80-01-AA-00-00-00";` - нули заменить на свои придуманные цифры 

* имя устройства на сервере народмон `SENS = "VasjaPupkin";` - аналогично

* точка доступа для выхода в интернет `APN = "internet.mts.by";` вашего сотового оператора

* имя ` USER = "mts"; ` и пароль `PASS = "mts";` для выхода в интернет вашего сотового оператора

* `n_send = true;` если вы хотите, или `n_send = false;` если не хотите отправлять данные на сервер

* `sms_report = true;` - разрешить отправку SMS отчета, или `sms_report = false;` если жалко денег на SMS

* `Vstart = 13.50` - порог детектирования по которому будем считать что авто зарежает АКБ

* `m = 69.91;` - делитель, для точной калебровки напряжения АКБ 

# Подключение:

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/Sin800LV1.5.jpg)

Для подключения к авто c класическим замком на 4 провода, если у вас япошка с замком на 6 проводов, то там все веселее... 

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/conecting-001.JPG)

* **выход на реле иммобилайзера и первого положения замка зажигания** `FIRST_P_Pin 8`, на плате `OUT1`

* **выход на реле зажигания** `ON_Pin 9`, на плате `OUT2` 

* **выход на реле стартера** `STARTER_Pin 12`, на плате `OUT3` 

* выход на включение обогрева сидений или вебасто `WEBASTO_pin 11`, на плате `OUT4` (опция)

* выход на реле управления подогревом сидений, на плате `OUT5` (опция)

* выход на сигнальный светодиод `ACTIV_Pin 13` на плате `OUT6`(опция)

* **вход `Feedback_Pin A1` - для проверки на момент включенного зажигания с ключа**, на плате `FB`

* **вход `STOP_Pin A2` - на концевик педали тормоза (АКПП) или на датчик нейтрали в МКПП**, на плате `IN2`

* вход `PSO_Pin A3`  - на датчик давления масла, если кому горит (опция), на плате `IN3`

* вход `D3`- для датчиков объема или вибрации (аппаратное прерываение), на плате `IN1` (опция)

* вход `D2` - для подключения к датчику распредвала через оптопару, если кому горит `IN0` (опция)

* линия `L` - на пин 15 K-line шины в OBDII разъёме, если такова имеется (опция)

* линия `K` - на пин 7 K-line шины в OBDII разъёме, если такова имеется (опция)

* **масса `GND`  - она же минус, для шины датчиков температуры DS18B20**

* **провод `DS18` - на линию опроса вышеупомянутых датчиков**, приходит на 4й пин ардуино с подтяжкой к 3.3V

* **клемма `3.3V` - напряжение питания датчиков температуры**

* **клемма `12V` - питание платы через предохранитель на 2А от "постоянного плюса"**

* клеммы `REL`, `NO` и ` NC` - входы и выходы  реле для коммутации антенны обходчика иммбилайзера


## Алгоритм запуска: 
После получения команды на запуск, ардуино;

1 Обнуляет счётчик попыток запуска, в зависимости от температуры двигателя на датчике `Temp0` автоматически подбирается:
 
 * Время работы стартера `StTime` от 1 до 6 сек 
 
 * Таймер обратного отсчета `Timer` от 5 до 30 минут

 * Число повторов прогрева свечей накала (для дизелистов) о 0 до 5
 
 в соответствии с [таблицей](https://raw.githubusercontent.com/martinhol221/M590_autostart_car_engine/master/other/calibr.log)

3 Проверяем что бы напряжение АКБ было больше 10 вольт, зажигание с ключа не включено (гарантия что двигатель не работает), температура  `Temp[0]` выше -25, и число попыток запуска не достигло максимальных (5-ти попыток).

4 Если предыдущие условие выполненной то включаем реле первого положения замка зажигания , ожидаем 1 сек.

5 Включаем реле зажигания, ожидаем 4 сек., проверяем не было ли предыдущих неудачных попыток запуска
  
  5.1 Eсли их было 2 и более то дополнительно выключаем/включаем зажигание на 2/8сек

  5.2 Если предыдущих неудачных попыток запуска было 4 и более то дополнительно выключаем/включаем зажигание на 10/8сек

6 Проверяем не нажата ли педаль тормоза (датчик нейтрали), включаем реле стартера установленное время ` StTime ` и выключаем его.

7 Выжидаем 6 сек. на набор аккумулятором напряжения заряда от генератора.

8 Заменяем напряжение АКБ, и если измеренное напряжение выше установленного порога в 13.5 то считаем старт успешным;
 
  * включаем реле подогрева сидений подключенное к `OUT5`, но только при успешном старте
  
  * отправляем смс если попыток зпуска было 2 и более
  
иначе возвращаемся к пункту 4, и так оставшихся 4 раза.

## Обходчик иммобилайзера:
 
  Обходчик представляет собой две катушки с равным количесвом витков, намотанные одним и тем же проводом, поверх антенны на замке зажигания и на ключ (чип от ключа). Катушки соеденяются последовательно, свободные концы катушек соеденяютсяc клеммами `REL` и `NO`
на плате, тем самым реле при включении замыкает контур ретранслируя сигнал от чипа на штатную антенну замка зажигания.
![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/immo.JPG)

# Какие функции поддерживает прошивка

## 1. Входящий звонок.
  
При входящем звонке с номера `call_phone` "снимает трубку" и проигрывает DTMF-гудок, ожидая ввода команды с клавиатуры телефона;

* ввод `123` включает запуск двигателя с 3-ю попытками 
* ввод `456` включает таймер автопрогрева
* ввод `789` останавливает таймер прогрева и автопрогрева
* ввод `741` отключает отправку данных на сервер
* ввод `852` включает отправку данных на сервер
* ввод `*`   затирает ошибочно введенные цифры
* ввод `#`   разрывает соединение и отправляет смс на номер указанный как `SMS_phone`

## 2. Исходящий звонок.

Звоним на номер на номер хозяина `call_phone` при смене потенциала  0V на +12V на клемме `IN1`, к которому подключен какой нибудь тревожный датчик объема или др., жду по этому пункту идей.

## 3. Входящие SMS команды

* СМС с текстом `#123start` запустит двигатель на пргрев с автоматическим определением времени прогрева

* СМС c текстом `#autoh` включит автопрогрев (проверка температуры каждых 3 часа)

* СМС c текстом `#stop` остановит прогрев и автопрогрев

 `123` можно заменить на свой секретный трёхзначные пароль в скетче


## 4. Исходящее SMS сообщение
 
 * каждый раз когда авто завелось не с первой попытки, или вобще не завелось уходит СМС на номер `SMS_phone`
 
 * за 2 минуты до окончания прогрева, если до истечении времени не была нажата педаль СТОП, отправляется СМС
 
 ![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/SMS.jpg)
 
Текст СМС

`Privet Vasja Pupkin` - имея сенсора задаваемого в шапке скетча

`Temp0:  42.05`       - температура датчика DS18B20 расположенного на трубках отопителя салона

`Temp1: 24.01`        - температура датчика DS18B20 расположенного в ногах водителя

`Temp2: 15.03`        - температура датчика DS18B20 расположенного снаружи автомобиля

`Voltage Now: 14.23V` - напряжение АКБ автомобиля в этот момент времени (заряжается)

`Voltage Min: 7.81V`  - напряжение АКБ автомобиля в этот момент времени

`Voltage for Start: 12.75V`   - напряжение АКБ автомобиля перед включением стартера

`Timer 1`            - состояние таймера обратного отсчета в минутах

`Attempts 1`         - Число включения стартера с последнего удачного или неудачного запуска

`Uptime: 10H`        - время непрерывной работы ардуино в часах

И ссылка на расположение автомобиля на картах гугл если разкоментировать соответствующие строки в скетче

## 5. Отправка показаний датчиков на сервер narodmon.ru

 ![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/narod.jpg)

Каждые 5 минут открывает GPRS соединение с сервером `narodmon.ru:8283` и отправляет пакет вида:
 
`#80-00-00-XX-XX-XX#SensorName`  - из шапки скетча         

`#Temp1#42.05` - температрура с датчика №1, DS18B20 подключенного на 4 й пин ардуино  

`#Temp2#24.01` - температрура с датчика №2, номер присваивается случайно исходя из серийного номера датчика

`#Temp2#15.03` - температрура с датчика №3, номер присваивается случайно исходя из серийного номера датчика

`#Vbat#13.01`  - Напряжение АКБ, пересчитанное через делитель `m = 66.91;`, 876 значение АЦП 66.91

`#Uptime#7996` - Время непрерывной работы ардуино без перезагрузок, для статистики бесперебойной работы. 

`#Time2`       - таймер автопрогрева в минутах. 

`##`           - Окончание пакета данных.            

Расход трафика до 20 Мб в месяц c ПОБАЙТНЫМ округлением сессии, которая к слову длится 20 сек, и открывается каждых 5 минут.

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/GPRSTrafic.JPG)


## 6. Прием команд из приложения Народмон 2017

Команды такие же как и при входящем СМС, отличие в том что команда доходит только в момент связи с сервером от 0 до 5 минут, как повезет.

В приложении [Народный мониторинг](https://play.google.com/store/apps/details?id=com.axbxcx.narodmon&hl=ru), залогинившись, перейти в УПРАВЛЕНИЕ > + > ПРОИЗВОЛЬНАЯ КОМАНДА > выбрать устройство, заполнить КОМАНДА: `123start`, `123stop`, или `autoH`.

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/narodmon2017apk.jpg)

## 7. Автопрогрев

Каждых 3 часа происходит проверка на низкую температуру:

Если температура упала ниже -18 градусов выполняем запуск двигателя на 20 минут тремя попытками.

Активация `456` и дезактивация `789`, либо нажанием педали STOP


## 8. Отключение зажигания по таймеру, при низком напряжении и превышении температуры выше 86 градусов

Отключение зажигания при просадке напряжения АКБ ниже 11.0V, возникает при внезапно заглохшем двигателе, за это отвечает строка

`if (heating == true && Vbat < 11.0 ) heatingstop();     // остановка прогрева если напряжение просело ниже 11 вольт`

За отключение при достижении температуры в 86 градусов строка 

`if (heating == true && TempDS[0] > 86) heatingstop();     // остановка прогрева если температура достигла 70 град ` 

За отключение прогрева при оконсчании осчета таймера

`if (heating == true && Timer <1)    heatingstop();      // остановка прогрева если закончился отсчет таймера `

## 9. Моргалка светодиодом

Каждых 10 секунд на 50 милисекунд вспихивает светодиод подключенный между `out6` и `+12`с последовательно подключенным резистором в 1кОм

`if (heating == false) digitalWrite(ACTIV_Pin, HIGH), delay (50), digitalWrite(ACTIV_Pin, LOW);  // моргнем светодиодом`
в режиме прогрева светодиод горит постоянно

## 10. Голосовое информирование о событиях в "трубку"

* "Привет, жду команду" - сразу после "снятия трубки"

* "Все поняла, завожу"  - после ввода `123` в DTMF формате

* "Включаю зажигание"   - если если зажигание выключено и напряжение выше 11 вольт

* "Прогреваю свечи"     - в случае дополнительного прогрева свечей

* "Кручу стартером"     - в момент включения стартера

* "Подожди"             - после выключения стартера

* "Двигатель заведен"   - в случае успешного старта

* "Упс, повторный запуск" - в случае не запуска уходя на следующую попытку

* "Я на передаче"        - если нажата педаль тормоза или МКПП на передаче

* "Стоп"                 - в случае неудачного запуска при выходе из цикла
 
[Загрузка аудиофайлов в память SIM800](https://github.com/martinhol221/SIM800L_DTMF_control/wiki/%D0%97%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B0-%D0%B0%D1%83%D0%B4%D0%B8%D0%BE%D1%84%D0%B0%D0%B9%D0%BB%D0%BE%D0%B2-%D0%B2-SIM800L,-AT-CREC)



## 11. Геолокация и микрофон

На основании УК РФ Статья 138.1. **"Незаконный оборот специальных технических средств, предназначенных для негласного получения информации"** и **ч.1 ст.376 УК Беларуси "Незаконное изготовление, приобретение либо сбыт средств для негласного получения информации"** 
запрещается вносить конструктивные изменения в устройство, а именно подпаивать микрофон и вносить изменения в прошивку, что может  превратить ваше устройство в спейц средство и у вас будут проблемы с законом. 

Запрещается заливать скетч с раскоментированной строками: 

`String LAT = "";  `    

`String LNG = ""; `                

`SIM800.print("\n https://www.google.com/maps/place/"), SIM800.print(LAT), SIM800.print(","), SIM800.print(LNG);`

`SIM800.print("\n#LAT#"),          SIM800.print(LAT); `

`SIM800.print("\n#LNG#"),          SIM800.print(LNG); `

`
SIM800.println("AT+CIPGSMLOC=1,1"),    delay (3000);    
      } else if (at.indexOf("+CIPGSMLOC: 0,") > -1   )      {LAT = at.substring(at.indexOf("+CIPGSMLOC: 0,")+24, at.indexOf("+CIPGSMLOC: 0,")+33);
                                                             LNG = at.substring(at.indexOf("+CIPGSMLOC: 0,")+14, at.indexOf("+CIPGSMLOC: 0,")+23); 
                                              delay (200),
`
Код только для ознакомления.

Хотя это не GPS треккер, но в теории модем может определять свое расположение по информации базовых станциий сотового оператора, аналогично как и в смартфонах без GPS, точность при этом составляет от 100 до 800 м, в зависимосте от местности, в городе обычно 100-200 м.

Работа прошивки с гелокацией это только теория, и ни в коем образе ниразу не опробывалось на практике, все скриншоты это плод работы в фотошоп, координаты придуманные.


# Возможные проблемы и их устраниение:

* Модем постоянно отваливается от сети - подать стабильное питание 3.5-4.4V c пиковым током в 3A !

* После подачи питания модем не возвращает `+CPIN: READY`, `Call Ready`и `SMS Ready`, модем не определил скрость, решение - швырнуть в модем команду `AT+IPR=9600;E1+DDET=1;+CMGF=1;+CSCS="gsm";+CNMI=2,1,0,0,0;+VTD=1;+CMEE=1;&W ` которая настроит в модеме скорость порта 9600, режим ЭХО, детектирование DTMF сигналов, тип кодировки СМС, автоизвещение о входящем смс, длительность тоновых сигналов, отображение ошибок и сохранит все настройки в энергонезависимую память.

* если ардуино постоянно перезагружется (не снимает трубку), то навешиваем дополнительных керамических конденсаторов на 0,1мкф на шину питания 3.3V Ардуино как можно ближе к микросхеме, и заменяем спиральную антенну на выносную, вся проблема из-за ВЧ наводок от переотражения в машине

![](https://github.com/martinhol221/SIM800L_DTMF_control/blob/master/img/SMA.jpg)

* если устройство включает стартер на рабртающем двигателе то не подключен провод обратной связи `FB` - подключите его

* если машина заводится и потом сама себе глошнет, то устройство не корректно замеряет напряжение заряда, необходима калибровка. Если напряжение в мониторе порта не соответствует действительности, то необходимо экспериментально подобрать `m = 65.....72;`, пока напряжение на мультиметре и в мониторе порта не окажутся приблизительно одинаковыми.

* если зажигание включается , стартер крутит, но двигатель не заводится, то подберите другое количество витков на катушке импровизированного обходчика иммобилайзера 

* если температура с датчиков не отображется в СМС отчете, то они физически не подключены

* если модуль ревизии ниже Revision:1418B04SIM800L24 то скорее всего AT+CREC работать не будет


# Ссылки на мои предыдущие проекты на эту тему:

[Видео в работе на Youtube](https://www.youtube.com/watch?v=WsydZUWQ4po&t=)

[Анатомия автозапуска 5.0 (DRIVE2.RU)](https://www.drive2.ru/c/492206551730225564)

[Анатомия автозапуска 3.0 (DRIVE2.RU)](https://www.drive2.ru/l/474891992371823652/) в пластиколвом корпусе

[Анатомия автозапуска (DRIVE2.RU)](https://www.drive2.ru/l/471004119256006698/) смый первый опыт , еще литием на борту

[Новые платки и новый скетч для автозапуска (DRIVE2.RU)](https://www.drive2.ru/c/485387655492665696/) 

[Подделка на подделку ELM327, или как еще читать температуру ДВС]( https://www.drive2.ru/c/479335393737572613/) опыт работы с K-line шиной по протоколу ISO 14230-4 kwp связкой Arduino + L9637D

martinhool@yandex.by
