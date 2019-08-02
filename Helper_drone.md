Коптер для измерения уровня радиации, углевдородных газов и углекислого газа

Для начала, пришлось найти подходящие датчики. Датчиков для Raspberry Pi не нашлось, выбрали датчики radiationD-v1.1, MQ-2 и MQ-135, которые совместимы с Arduino. Сам проект собран на основе Клевер 3. Также пришлось смоделировать кейс для крепления Arduino Uno и трех датчиков, что позволит легко устанавливать GR-часть на любой дрон.

##Установка

Вначале на Arduino надо установить библиотеку:

    #include <TroykaMQ.h>

Далее через Serial-соединение выводим показания:

    Serial.println(cpm);
    Serial.println(mq2.readMethane());
    Serial.println(mq2.readHydrogen());
    Serial.println(mq135.readCO2());
    Serial.println(mq2.readSmoke());

На Raspberry есть часть кода, которая принимает эти значения из Serial-соединения. Далее, программа Raspberry создает ссылку на сайт, где в автономном режиме выводятся эти данные:

    import serial

    ser = serial.Serial('/dev/ttyUSB0', 9600)

    strTable = "<html>\n<head>\n<meta http-equiv='Refresh' content='3' />\n<style>\ntable {\nbackground: #f5e8d0;"\
               "border-spacing: 0; \n"\
               "}\n"\
               "th \n{"\
               "background: #496791; \n"\
               "color: #fff;\n"\
               "}\n"\
               "td, th {\n"\
               "padding: 5px 10px; \n"\
               "}\n"\
               "</style>\n"\
               "</head>\n<body>\n<table>\n<tr>\n<th>Indicator</th><th>Value</th><th>Normal Value</th></tr>\n"

    s = []
    while True:
            data = ser.readline()
            if data:
                   s.append(data)
            if len(s) == 5:
                    strRW = "<tr><td>" + "Radiation" + "</td><td>" + str(s[0]) + "</td><td>50-250</td></tr>\n"
                    strTable += strRW
                    strRW = "<tr><td>" + "Methane" + "</td><td>" + str(s[1]) + "</td><td>5-40</td></tr>\n"
                    strTable += strRW
                    strRW = "<tr><td>" + "Hydrogen" + "</td><td>" + str(s[2]) + "</td><td>5-50</td></tr>\n"
                    strTable += strRW
                    strRW = "<tr><td>" + "Carbon dioxide" + "</td><td>" + str(s[3]) + "</td><td>150-500</td></tr>\n"
                    strTable += strRW
                    strRW = "<tr><td>" + "Smoke" + "</td><td>" + str(s[4]) + "</td><td>10-50</td></tr>\n"
                    strTable += strRW
                    strTable += "</table>\n</body>\n</html>"
                    hs = open("index.html", "w")
                    hs.write(strTable)
                    s = []
                    strTable = "<html>\n<head>\n<meta http-equiv='Refresh' content='3' />\n<style>\ntable " \
                               "{\nbackground: #f5e8d0;" \
                               "border-spacing: 0; \n" \
                               "}\n" \
                               "th \n{" \
                               "background: #496791; \n" \
                               "color: #fff;\n" \
                               "}\n" \
                               "td, th {\n" \
                               "padding: 5px 10px; \n" \
                               "}\n" \
                               "</style>\n" \
                               "</head>\n<body>\n<table>\n<tr>\n<th>Indicator</th><th>Value</th><th>Normal Value</th></tr>\n"
                    hs.close()

После в директории

    cd /lib/systemd/system

прописываем свой сервис:

    [Unit]

    Description=Helper_drone HTTP Server

    [Service]

    WorkingDirectory=/usr/share/monkey-static/Helper_drone/

    ExecStart=/usr/bin/python /usr/share/monkey-static/Helper_drone/prog.py

    Restart=always

    RestartSec=3

    [Install]

    WantedBy=multi-user.target

Благодаря этому, мы упросим запуск GR дрона, а именно, только подав питание на Raspberry, программа автоматически запустится.

##Практика

Подаем питание на Raspberry, подключаемся через Wi-Fi и заходим на сайт. Выбираем нижнюю ссылку «GRI», где раз в 3 секунды обновляются данные.

![web](../Helper_drone/a2.png)

![data](../Helper_drone/a1.png)
проект сделан на базе клевер-3, raspberry pi 3B+ с образом для коптера клевер, arduinio nano и датчиком MQ-2 MQ-135 и счетчика гейгера.
Проект выполнили Артём Сыров и Тимофей Карпеев.
