# Перевод anycubic ace pro на happy hare
Данная статья посвящена переводу anycubic ace pro на happy hare. Статья подразумевает, что у вас уже установлен klipper, веб-интерфейс, а также умеете подключаться к хосту по ssh, и вы имеете представление о stm32, klipper.

0. [Драйвера](#Драйвера)
1. [Благодарность](#Благодарность)
2. [Автор](#Автор)
3. [Преимущества](#Преимущества-над-вышеперечисленными-проектами)
4. [Обозначения](#Обозначения)
5. [Пути реализации](#Пути-реализации)
6. [Сушилка](#Сушилка)
7. [Прошивка стоковой платы через swd](#Прошивка-стоковой-платы-через-swd)
8. [Прошивка стоковой платы через dfu](#Прошивка-стоковой-платы-через-dfu)

# Драйвера
Если вы наткнулись на эту статью и не готовы так заморачиваться, можете воспользоваться данными проектами:
- https://github.com/BlackFrogKok/BunnyACE/blob/development/README.ru.md
- https://github.com/agrloki/ValgACE

# Благодарность:
- https://github.com/printers-for-people/ACEResearch
- [@niknikdm](https://web.telegram.org/k/#@niknikdm) (реализует способ с заменой платы)

# Автор 
Пишите по всем вопросам, чем смогу тем помогу :)
- [@saturnechek](https://web.telegram.org/k/#@saturnechek)

# Преимущества над вышеперечисленными проектами:
- Гибкость настройки 
- Стабильность работы 
- Удобный веб интерфейс
- KlipperScreen под MMU
- Возможность подключения к самой асе дополнительных датчиков (если не нужен nfc освобождается 12 пинов), например таких как bme280 (для отображения реальной температуры и влажности в камере сушилки, а не погоду в блоке грелок), дисплеев и тд.

# Обозначения:
| Обозначение | Сокращение |
|:----:|:----:|
| Happy hare | hh |
| Anycubic ace pro | ace |
| Блок питания | бп |

# Пути реализации
| Путь | Минусы | Плюсы |
|:----:|:------|:------|
| Замена родной материнской платы | - Нужны навыки пайки или обжимания клемм, совместимых с платой, на которую вы будете менять<br>- Придется докупать бп на 24в, твердотельные реле для работы сушки<br>- В ace достаточно мало места для этого всего, придется выносить на корпус | - Есть возможность вернуться обратно, если вам не понравится hh |
| Прошивка стоковой платы swd или dfu | - Теряется возможность вернуться на стоковую прошивку (даже если сняли дамп с процессора, по крайней мере у меня не получилось)<br> | -Нужны вложения только на st link(2$) |

# Сушилка
На борту у ace pro имеется по 1 нагревателю, термистору, вентилятору на каждую секцию. Один транзистор управляет питанием сразу двух секций, и приходится осуществлять контроль температуры только по одному термистору, второй просто выводит температуру. А также если отключится вентилятор, охлаждающий нагреватель, он просто перегреется, и Klipper не упадет в ошибку, но хорошо, что anycubic подумали и поставили термопредохранитель под нагревателем (на 100 градусов). Эти проблемы можно решить с помощью дополнительного твердотельного реле, но его придется выносить за корпус, т.к. в нем просто нет места. А также неизвестно, какая максимальная температура сушки, при которой не поплывет пластик, максимальная, которую пробовал — 65 градусов (в камере).

UPD

Как выяснилось позже нагреватели ace pro расчитаны на 110v. На сколько безопасно можно использовать их на 220v мне не известно. Для решения этой проблемы можно соеденить нагреватели последовательно или заменить на подходящие 

# Прошивка стоковой платы через swd
У разных ревизий плат отличается внутреннее строение.

Первая ревизия (0.0.8) — закрытый бп, твердотельное реле на нагреватели (используется watermark).

Вторая ревизия (0.0.9) — открытый бп, транзистор на нагреватели.

![Alt-текст](https://github.com/printers-for-people/ACEResearch/blob/d7145cf57ede717a8d92506ea8acc05371e99e26/photos/IMG_1301.JPG)

__!!! В зависимости от вашей ревизии платы советую проверять распиновку и конфиги !!!__

Дальше в статье будут конфиги именно под 0.0.9.

Итак, BOM:
- ST-Link v2
- ~~резистор 1.5 кОм для подтяжки 3.3В к D+~~ без него понадобиться указать пины при сборке проишивки 
- Колодки / DuPont (скорее опционально, можно и припаяться)

## Процесс прошивки:
Прежде хотел сказать, что в интернете достаточно много информации по прошивке stm32 с помощью ST-Link, для gd32 процесс аналогичен.

1) Подключаем ST-Link к плате. При питании ace pro от 220 подключаем GND, SWDIO, SWCLK, если нет, то подключаем еще 3.3В.

![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/3ed1990230bfc7427506d5a0d0f0fb49e8a9e4b6/photos/pinout.png)

2) Подключаемся по ssh

4) Сборка bootloader katapult
   
   ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/e4dd7303670d8632b2bcb5b6147ad77607ada709/photos/katapult%20menuconfig.png)
   
   ```
    git clone https://github.com/Arksine/katapult
    cd katapult
    make menuconfig
    make
   ```
5) Собираем прошивку
    Если не поставилил резистор(оптимально)
   
    ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/e4dd7303670d8632b2bcb5b6147ad77607ada709/photos/menuconfig%20without%20resistor.png)       

    ~~Если поставилил резистор~~
   
    ~~Паяем резистор к D+ и 3.3В, а также делаем технологический вырез в корпусе (если припаяли под платой).~~
   
    ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/menuconfig.png)
   
    ```
    cd klipper
    make clean 
    make menuconfig
    make
    ```
    
6) Достаем с хоста (с помощью WinSCP или другим файловым менеджером) katapult.bin.
   
8) Устанавливаем драйвера и программу STM32 ST-LINK Utility.

__!!! ОБРАТНОГО ПУТИ УЖЕ НЕ БУДЕТ !!!__

8) Открываем katapult.bin, отключаем питание ace, включаем и подключаемся в STM32 ST-LINK Utility (обязательно в такой последовательности, иначе будет выпадать ошибка, что невозможно подключиться).

9) Дальше есть несколько путей подключения:
    1) Использовать стоковый хаб (понадобится провод MicroFit — USB)
    2) Припаяться к пинам и вывести удобный для вас разъем

10) Узнаем видится ли устрйойсвто. В Веб интерфейсе отобразится устройство или вводим в терминале ls /dev/ttyACM*.

![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/ls%20dev.png)

![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/device%20mainsail.png)

11) Прошивка klipper через ранее установленный katapult

    Проверяем работает ли bootloader.
    Вместо /dev/ttyACM2 указываем ваш путь из пункта 10
        
        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s
    
    Вывод должен быть:
    
    ```
    Connecting to Serial Device /dev/ttyACM2, baud 230400
    Attempting to connect to bootloader
    Katapult Connected
    Software Version: v0.0.1-91-fdl3
    Protocol Version: 1.1.0
    Block Size: 64 bytes
    Application Start: 0x8002000
    MCU type: stm32f103xe
    Status Request Complete
    ```
    
    Вместо /dev/ttyACM2 указываем ваш путь из пункта 10
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -f ~/klipper/out/klipper.bin
    
    
        
    Вывод должен быть:

        Connecting to Serial Device /dev/ttyACM2, baud 230400
        Detected Klipper binary version v0.12.0-458-gd886c176, MCU: stm32f103xe
        Attempting to connect to bootloader
        Katapult Connected
        Software Version: v0.0.1-91-fdl3
        Protocol Version: 1.1.0
        Block Size: 64 bytes
        Application Start: 0x8002000
        MCU type: stm32f103xe
        Flashing '/home/orangepi/klipper/out/klipper.bin'...

        [##################################################]

        Write complete: 34 pages
        Verifying (block count = 531)...

        [##################################################]

        Verification Complete: SHA = 9A0A75F1987338EE8D4E1A5736E793B1E63E8914
        Programming Complete
    
    __!!!Для повторной прошивки понадобиться прописать!!!__

        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -r
    
    И также запустить процесс прошивки с указаниеим файла
        
13) Для минимальной проверки можно добавить в ваш printer.cfg
    ```
    [mcu mmu]
    serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_3CFC3B841812300147323935-if00
    ```
# Прошивка стоковой материнской платы через DFU
Anycubic вывели пин boot0 к gnd через резистор, а рядом расположили пятку vcc. Узнал я это уже после того как прошил через swd, так что не могу утверждать, что способ сработает, ведь неизвестно, что производитель сделал с загрузчиком, а также по какой-то неведомой мне причине в DFU устройство не видится.

![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/pinout%20dfu%20mode.png)

Если захотите попробовать:
1) Отпаиваем резистор BOOT0-GND
2) Паяемся BOOT0-VCC
3) Насчёт прошивки никак не подскажу, сам не проходил этот этап
4) Убираем перемычку и ставим обратно (BOOT0-GND)

# Переходим к установке и настройке happy hare

Официальная страница: https://github.com/moggieuk/Happy-Hare

### Преступим к установке:

      cd ~
      git clone https://github.com/moggieuk/Happy-Hare.git
      ./install -i
```
----------------
What type of MMU are you running?
1) Enraged Rabbit Carrot Feeder v1.1
2) ERCF v2.0
3) ERCF v3.0
4) Tradrack v1.0
5) Angry Beaver v1.0
6) Box Turtle v1.0
7) Night Owl v1.0
8) 3MS (Modular Multi Material System) v1.0
9) 3D Chameleon
10) PicoMMU
11) QuattroBox v1.0
12) QuattroBox v1.1
13) MMX
14) BigTreeTech ViViD (BETA)
15) KMS
16) Other / Custom (or just want starter config files)
MMU Type (1-16)? 16
----------------
Which of these most closely resembles your MMU design (this allows for some tuning of config files)?{}
1) Type-A (selector) with Encoder
2) Type-A (selector), No Encoder
3) Type-A (selector), No Encoder, No Servo, No ESpooler
4) Type-B (mutliple filament drive steppers) with Encoder
5) Type-B (multiple filament drive steppers) with shared Gate sensor and Encoder
6) Type-B (multiple filament drive steppers) with shared Gate sensor, No Encoder
7) Type-B (multiple filament drive steppers) with individual post-gear sensors and Encoder
8) Type-B (multiple filament drive steppers) with individual post-gear sensors, No Encoder
9) Just turn on all options and let me configure
Type (1-9)? 3

    IMPORTANT: Since you have a custom MMU with selector you will need to setup some CAD dimensions in mmu_parameters.cfg... See doc
----------------
How many gates (lanes) do you have?
Number of gates? 4
----------------
Select mcu board type used to control MMU
1) BTT MMB v1.0 (with CANbus)
2) BTT MMB v1.1 (with CANbus)
3) BTT MMB v2.0 (with CANbus)
4) Fysetc Burrows ERB v1
5) Fysetc Burrows ERB v2
6) Standard EASY-BRD (with SAMD21)
7) EASY-BRD with RP2040
8) Mellow EASY-BRD v1.x (with CANbus)
9) Mellow EASY-BRD v2.x (with CANbus)
10) TZB v1.0
11) AFC Lite v1.0
12) WGB v3.0
13) BTT SKR Pico v1.0
14) BTT EBB 42 CANbus v1.2 (for MMX or Pico)
15) Not in list / Unknown
MCU Type (1-15)? 15
----------------
Would you like to have neopixel LEDs setup now for your MMU?
Enable LED support? (y/n)? n
----------------
Selector homing using TMC Stallguard? This prevents the need for hard endstop homing but must be tuned
MCU must have DIAG output for selector stepper. Can configure later
Enable selector stallguard homing (y/n)? n
----------------
EndlessSpool? This uses filament runout detection to automate switching to new spool without interruption
Enable EndlessSpool (y/n)? y
----------------
Finally, would you like me to include all the MMU config files into your printer.cfg file
Add include? (y/n)? y
    Would you like to include Mini 12864 screen menu configuration extension for MMU (only if you have one!)
    Include menu (y/n)? y
    Recommended: Would you like to include the default pause/resume macros supplied with Happy Hare
    Include client_macros.cfg (y/n)? y
    Addons: Would you like to include the EREC filament cutter macro (requires EREC servo installation)
    Include mmu_erec_cutter.cfg (y/n)? n
    Addons: Would you like to include the Blobifier purge system (requires Blobifier servo installation)
    Include blobifier.cfg (y/n)? n
```

После установки берем конфиги из этого гита и заменяем папку mmu. 
Сейчас будут параметры без настройки которых ничего работать не будет, указываю я их для того чтобы указать вам примерный путь для ace pro и не брать мои кривые конфиги :)
1) В mmu_vars.cfg
   ```
   mmu_gear_rotation_distances = [16, 16, 16, 16] 
   mmu_selector_offsets = [40, 30, 20, 10] # Отвечает за положения селектора относительно дома
   ```
2) В mmu_hardware.cfg
   ```
   selector_type: RotarySelector	# обязательно 
   ```
3) В mmu_parameters.cfg
   ```
   cad_gate_directions: 1, 1, 1, 1	
   ```
Ну и в целом все, остальное настраивается под ваш принтер / хотелки, в этом поможет офицальный гит hh

P.S Советую откалиюровать mmu_selector_offsets и mmu_gear_rotation_distances, параметры взяты примерно  
