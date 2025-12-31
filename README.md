# Перевод anycubic ace pro на happy hare(Happy ace)
Данная статья посвящена переводу anycubic ace pro на happy hare

0. [Пути реализации](#Пути-реализации)
2. [Преимущества](#Преимущества-над-вышеперечисленными-проектами)
3. [Обозначения](#Обозначения)
4. [Планы развития](#Планы-развития)
5. [Пути реализации](#Пути-реализации)
7. [Прошивка стоковой платы через SWD](#Прошивка-стоковой-платы-через-SWD)
8. [Прошивка стоковой платы через DFU](#Прошивка-стоковой-платы-через-DFU)
9. [Замена материнской платы](#Замена-материнской-платы)
10. [Установка и настройка happy hare](#Установка-и-настройка-Happy-Hare)
11. [Мое наработки и мысли](#Мои-наработки-и-мысли)
12. [Благодарность, ссылки, автор](#Благодарность-ссылки-автор)


## Требования 
1) Klipper + Moonraker(web ui)
2) Доступ к терминалу устройства с Klipper(ssh)
   
## Пути реализации
| Путь | Минусы | Плюсы |
|:----:|:------|:------|
| Замена материнской платы | - Нужны навыки пайки или обжимания клемм, совместимых с платой, на которую вы будете менять<br>- Придется докупать бп на 24в, твердотельные реле для работы сушки<br>- В ace достаточно мало места для этого всего, придется выносить на корпус | - Есть возможность вернуться обратно, если вам не понравится hh |
| Прошивка стоковой платы swd или dfu | - Теряется возможность вернуться на стоковую прошивку (даже если сняли дамп с процессора, по крайней мере у меня не получилось)<br> | - Нужны вложения только на st link(2$) |

## Преимущества над существующими драйверами:
- Гибкость настройки 
- Стабильность работы 
- Удобный web ui
- KlipperScreen под MMU
- Возможность подключения к самой Ace дополнительных датчиков (если не нужен NFC освобождается 12 пинов), например таких как bme280 (для отображения реальной температуры и влажности в камере сушилки, а не погоду в блоке грелок), дисплеев и тд.

## Обозначения:
| Обозначение | Сокращение |
|:----:|:----:|
| Happy hare | HH |
| Anycubic ace pro | Ace |
| Блок питания | Бп |

## Планы развития
- [ ] Настройка встроенного буфера Ace pro 
- [ ] Информация по замене родной материнской платы
- [ ] Итоговая настройка конфигурации

## Прошивка стоковой платы через SWD
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

1) Подключаемся по SSH

2) Сборка bootloader katapult
   
   ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/e4dd7303670d8632b2bcb5b6147ad77607ada709/photos/katapult%20menuconfig.png)
   
   ```
    git clone https://github.com/Arksine/katapult
    cd katapult
    make menuconfig
    make
   ```
3) Собираем прошивку
    Без резистора (оптимально)
   
    ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/e4dd7303670d8632b2bcb5b6147ad77607ada709/photos/menuconfig%20without%20resistor.png)
   

    ~~С резистором~~
   
    ~~Паяем резистор к D+ и 3.3В, а также делаем технологический вырез в корпусе (если припаяли под платой).~~
   
    ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/menuconfig.png)
   
    ```
    cd klipper
    make clean 
    make menuconfig
    make
    ```
    
4) Скачиваем с хоста (с помощью WinSCP или другим файловым менеджером) katapult.bin.
   
5) Устанавливаем драйвера и программу STM32 ST-LINK Utility.

6) Подключаем ST-Link к плате. При питании Ace pro от 220 подключаем GND, SWDIO, SWCLK, если нет, то подключаем еще 3.3В.
#### Pinout
![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/3ed1990230bfc7427506d5a0d0f0fb49e8a9e4b6/photos/pinout.png)

__!!! ОБРАТНОГО ПУТИ УЖЕ НЕ БУДЕТ !!!__

7) Открываем katapult.bin, отключаем питание ace, включаем и подключаемся в STM32 ST-LINK Utility (обязательно в такой последовательности, иначе будет выпадать ошибка, что невозможно подключиться).

8) Дальше есть несколько путей подключения:
    1) Использовать стоковый хаб (понадобится провод MicroFit — USB)
       
       ![Alt-текст](https://github.com/BlackFrogKok/BunnyACE/blob/d6ccf88f94635bab0808e6b7cbd1ee6fd0c649b3/.github/img/pinout.png)
       
       Важно: VCC (24 В) для логики не обязателен — ACE питает себя сам. Подключение по USB к обычному порту.
       
    3) Припаяться к пинам и вывести удобный для вас разъем(см [pinout](#pinout))

9)  Узнаем serial: путь к ACE (чаще всего, /dev/serial/by-id/usb-Klipper_stm32f103xe_3CFC3B841812300147323935-if00)

   ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/ls%20dev.png)

   ![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/device%20mainsail.png)

10) Прошиваем klipper.bin через ранее установленный katapult

    Проверяем работает ли bootloader.
    __Вместо /dev/ttyACM2 указываем ваш путь из пункта 10__
        
        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -s
    
    Вывод:
    
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
    
    __Вместо /dev/ttyACM2 указываем ваш путь из пункта 10__
    
          python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -f ~/klipper/out/klipper.bin
        
    Вывод:

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
    
    __!!! Для повторной прошивки !!!__

        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -r
    
      Далее:
    
        python3 ~/katapult/scripts/flashtool.py -d /dev/ttyACM2 -b 230400 -f ~/klipper/out/klipper.bin
        
        
11) Проверка работоспособности:
    Добавить в printer.cfg
    ```
    [mcu mmu]
    serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_3CFC3B841812300147323935-if00
    ```
# Прошивка стоковой платы через DFU
Anycubic вывели пин boot0 к gnd через резистор, а рядом расположили пятку vcc. Узнал я это уже после того как прошил через swd, так что не могу утверждать, что способ сработает, ведь неизвестно, что производитель сделал с загрузчиком, а также по какой-то неведомой мне причине в DFU устройство не видится.

![Alt-текст](https://github.com/saturnechek/conversion-of-anycubic-ace-pro-to-happy-hare/blob/1b01261e31c6ff0b7a432ea29c4232ead8a769b6/photos/pinout%20dfu%20mode.png)

Если захотите попробовать:
1) Отпаиваем резистор BOOT0-GND
2) Паяемся BOOT0-VCC
3) Насчёт прошивки никак не подскажу, сам не проходил этот этап
4) Убираем перемычку и ставим обратно (BOOT0-GND)

# Замена материнской платы

   __COMING SOON__
   
   
   Ждем пока [@niknikdm](https://web.telegram.org/k/#@niknikdm) найдет время для описания процесса


# Установка и настройка Happy Hare

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

После установки советую взять только mmu.cfg, в данный момент только он достаточно хорошо настроен 

Сейчас будут параметры без настройки которых ничего работать не будет, указываю я их для того чтобы указать вам примерный путь для ace pro и не брать мои кривые cfg :)
1) В mmu_vars.cfg
   ```
   mmu_gear_rotation_distances = [16, 16, 16, 16] # ОБЯЗАТЕЛЬНО ОТКАЛИБРОВАТЬ
   mmu_selector_offsets = [40, 30, 20, 10] # Отвечает за положения селектора относительно дома. ОБЯЗАТЕЛЬНО ОТКАЛИБРОВАТЬ
   ```
2) В mmu_hardware.cfg
   ```
   selector_type: RotarySelector 
   ```
3) В mmu_parameters.cfg
   ```
   cad_gate_directions: 1, 1, 1, 1	
   ```
Ну и в целом все, остальное настраивается под ваш принтер / хотелки, в этом поможет офицальная страница HH
P.S Советую откалиюровать mmu_selector_offsets и mmu_gear_rotation_distances, параметры взяты примерно  
        
# Мои наработки и мысли
__не обязательно к прочтению__
Ace имеет не стандартные драйвера о которых я информации не нашел, но распиновка сходится по даташиту tmc2208. Драйвера имеют общий пин enable, а также обязательно нужно указывать секцию [tmc2208 stepper_mmu_gear], иначе будут спонтанные lost comunication при загрузке / выгрузке филамента.

Мое мнение! В данный момент покупать ace pro и заморачиваться с не стабильными драйверами или переходом на happy hare не имеет никакого смысла 

## Сушилка
На борту у ace pro имеется по 1 нагревателю, термистору, вентилятору на каждую секцию. Один транзистор управляет питанием сразу двух секций, и приходится осуществлять контроль температуры только по одному термистору, второй просто выводит температуру. А также если отключится вентилятор, охлаждающий нагреватель, он просто перегреется, и Klipper не упадет в ошибку, но хорошо, что anycubic подумали и поставили термопредохранитель под нагревателем (на 100 градусов). Эти проблемы можно решить с помощью дополнительного твердотельного реле, но его придется выносить за корпус, т.к. в нем просто нет места. А также неизвестно, какая максимальная температура сушки, при которой не поплывет пластик, максимальная, которую пробовал — 65 градусов (в камере).

UPD

Как выяснилось позже нагреватели ace pro расчитаны на 110v. На сколько безопасно можно использовать их на 220v мне не известно. Для решения этой проблемы можно соеденить нагреватели последовательно или заменить на подходящие 

# Благодарность, ссылки, автор
   [Happy hare](https://github.com/moggieuk/Happy-Hare)
   
   Фотографии ace pro взяты отсюда:
   
   - [ACEResearch](https://github.com/printers-for-people/ACEResearch) 
        
   Реализует способ с заменой платы
   
   - [@niknikdm](https://web.telegram.org/k/#@niknikdm)
     
   #### Драйвера
   Если вы наткнулись на эту статью и не готовы так заморачиваться, можете воспользоваться данными проектами:
   1) [BunnyACE](https://github.com/BlackFrogKok/BunnyACE/blob/development/README.md) (отсюда были взяты несколько фотографий)
   2) [ValgACE](https://github.com/agrloki/ValgACE)
        
   ##### Автор 
   Пишите по всем вопросам, чем смогу тем помогу :)
   - [@saturnechek](https://web.telegram.org/k/#@saturnechek)

В данный момент работа по допиливанию cfg стоит из за того что принтер занят, а другого подопытного у меня нету, так что все в ваших руках. А также хотел сказать что это моя первая статьтя в моей жизни, прошу не судить строго 
