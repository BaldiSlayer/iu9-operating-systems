# Гайд на лаобраторную работу номер 1

## ReactOS

Как билдить образ - разбирайтесь сами, для этого есть [дока](https://reactos.org/wiki/Building_ReactOS)
Могу лишь сказать, что под Intel + Windows все получилось, причем с первого раза, единственное, что могу сказать, что вам нахуй не нужен livecd образ, так что можете сбилдить только один 

VirtualBox думаю тоже сможете сами установить

Начну сразу с создания виртуальной машины

Создание виртуальной машины
Для этого нам нужен ISO образ (bootcd) и VirtualBox 
Образ я не дам, его надо сбилдить самому)

1) Машина
2) Создать
или просто Ctrl+N
3) Имя как-то сами
папку тоже
в образ ISO надо поставить собранный нами bootcd.iso файл
Тип: Other
4) оперативы я дал 8 гб
5) выбираем создать новый виртуальный жесткий диск 
размер диска - ваще хз, наугад 20 гб, все равно я потом ее удалю 😁
но думаю много не надо)
можно почитать в доке инфу про это
6) готово
7) запускаем виртуальную машину 
8) выбираем язык нашей операционной системы
9) дальше мне было лень читать и я тупо везде кликал Enter
10) у вас че-то установится и машина перезапустится и начнется уже уставовка с окошками, а не тупо в консоли, там опять можно прокликать enter и нажать установить какой-то пакет
11) после этого машина опять перезагрузится, надо ее выключить, причем без сохранения состояния оно нам нахуй не надо (скорее всего клавиатура захвачена и мышь тоже, нажмите правый Ctrl)
12) зайдите в настройки виртуалки (как это сделать написано на абзац ниже)
13) носители
14) берем и отключаем iso образ (правая кнопка мыши -> удалить устройство)

Как сделать вывод логов в файл?
Написано вот тут (https://stackoverflow.com/questions/38915802/how-can-i-collect-logs-from-linux-running-in-virtualbox-in-an-external-file)
Если просто и на русском:
1) кликаем правой кнопкой мыши по созданной виртуалке
2) настройки
3) COM - порты (у меня это 4 снизу пункт меню слева)
4) порт 1, ставим галочку включить
5) выбираем номер порта COM1
Режим: перенаправлен в файл
и дальше пишете путь
например у меня это C:\ReactOSLab1\serial.log

А как вывести фамилию в логи?
Тут все просто
1) Заходим в файл `ntoskrnl/kd64/kdinit.c`
2) переходим на строку 76 (примерно) и туда херачим что-то в духе
```C
DPRINT1("Imya Familia\n");
```

## NetBSD

1. Установка виртуальной машины проводится абсолютно также, как и для `ReactOS`, просто берем нужный нам iso файл например [отсюда](https://ftp.netbsd.org/pub/NetBSD/iso/)

2. логи ставим также (я про COM1 и файл), но будет прикол, что работать они не будут, пока мы не настроим их (ахуенно)

3. первый запуск - я лично все быстро прокликал и все, можете посмотреть хз у индусов бля гайды как ставить эту залупку

4. убираем iso и запускаем, нас попросит войти, нам похуй, вводим логин root и все

5. дальше начинается настройка, настроим логи
нам нужно открыть файл /etc/syslog.conf и написать туда строку
kern.* /dev/tty00 (перенаправляем логи на COM1)
для этого можно юзать vim, да, это жестко
```bash
vi /etc/syslog.conf
```
для перехода в интерактивный режим жмем I
пишем строку
жмем esc
пишем :wq
enter

6. теперь нам надо врубить dhcp
![alt text](image.png)
и еще я для этого поставил второй параметр в сети в virtual box
там будет что-то в духе сетевой мост

Типа
Открываем настройки машины
Сеть
Адаптер 1
Тип подключения у меня стоит сетевой мост

7. 
нам надо выкачать всю хуйню (исходники netbsd)

https://www.netbsd.org/docs/guide/en/chap-fetch.html#chap-fetch-cvs
Я сделал тупо 32.4.1

Если что, делать ток релиз можно, без stable
И только src, без xsrc

8. 
надо поменять сам код 
(да, опять vim)
файл
/usr/src/sys/kern/init_main.c

вставляем в самый конец (Esc + shift + G) на какое-то место, куда поставил я покажу на фотке
(*pr)("\n%s\n", "dhdh") ;

9. Билдим..
cd /usr/src/sys/arch/amd64/conf 
cp GENERIC MYKERN && config MYKERN
cd ../compile/MYKERN && make depend && make 
mv /netbsd /netbsd.old && mv netbsd /

10. 