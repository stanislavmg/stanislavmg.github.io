---
layout: post
title: "Заметки об администрировании"
category: admin
---
# Заметки об администрировании

# Командая строка windows

## Команды диагностики сети:

1. ping
Проверка доступности хоста, измерение времени отклика от хоста, проверить разрешается ли доменное имя.
2. netstat
Просмотр активных сетевых соединений и состояния портов по каждому процессу, просмотр статистики по UDP и TCP(сколько пакетов получено, отправлено, сколько открыто соединений), просмотр таблицы маршрутизации
Основные статусы портов:
- LISTEN - в режиме прослушивания
- ESTABLISHED - соединение по порту установлено
- CLOSED - порт закрыт 
3. telnet
Установка соединения по протоколу telnet. Устанавливает TCP сессию к хосту на указанный порт, если такое соединение возможно. Используется для проверки доступности порта. 
4. tracert/tracerout 
Отслеживание всех хопов пакета к хосту назначения. Позволяет понять в каком узле на пути пакета возникает проблема.
Утилита работает так:
отправляются icmp/udp пакеты с TTL 1 к хосту назначения.
от каждого промежуточного узла приходит ответ TIME_EXCEEDED.
после чего tracept увеличиват TTL на 1 и отправляет снова.
так продолжается, пока не будет достигнут хост назначение
В выводе команды *** означают, что узел не предоставил ответ (например стоит фильтрация пакетов в фаерволе) или время отклика превысило 5 секунд. По умолчанию максимальный TTL = 30. 
5. nslookup/dig/host
Проверка разрешения имен на конкретном DNS сервере. Можно также проверить определенную запись(A, AAAA, PTR, NS, SOA) на DNS сервере. Не авторитетный ответ означает, что днс сервер не имеет доступа к именам домена, запрашиваемого хоста и запрос был перенаправлен владельцу домена.
6. ipconfig
Просмотр сетевый интерфесов и их настроек. С помощью команды можно запросить новый адрес у DHCP сервера, очистить кеш DNS службы.
7. hostname
Отображения имени хоста.
8. getmac
Отобразить MAC адреса интерфейсов на определенном хосте.
9. arp
Отображение, добавление и удаление mac адресов в таблицу.
10. nbtstat
11. route

## Команды Windows общего назначения:

1. tasklist | taskkil /PID 9900
Отображает запущенные процессы
2. systeminfo
Отображение общей системной информации

# Траблшутинг

Любой поиск неисправности стоит начать с этих шагов:
- уточнить у пользователя имеет ли проблемы массовый характер.
- перезапустить машину
- уточнить с чем конкретно возникает проблема(локально или когда пытается запросить/отправить данные)

## Траблшутинг сети

Можно разделить условно проблемы на 2 категории: Проблемы с соединением и Проблемы и доступом

Проблемы с соединением.
Отстутствие соединиения. Проблемы L1 уровня OSI. Может быть связано с плохо обжатым кабелем, неисправность сетевой карты или драйвера сетевой карты. Проблема на стороне провайдера, обычно массовый характер.

Ограниченное соединение
Проблемы уровня L3 и выше. Самы распространенные:
- не получает адрес от DHCP сервера, адрес 169.254.Х.Х APIPA
- получает адрес не из нужного диапазона, соответственно не проходит через ACL на маршрутизаторе
- конфликт адресов случается, когда на устройсте вручную прописан адрес.

Проблемы доступа
Не может подключиться к определенному сайту - запрещено на прокси сервере или на маршрутизаторе в ACL
- проверяем доступность прокси через telnet. Адрес прокси и порт можно узнать в Interne options. Если порт открывается - проблема на прокси сервере, если нет, что проблема в сети

Ошибка DNS 105
Проверяем с помощью nslookup доменное имя на сервере по умолчанию и используя сторонний сервер. Если на сервере по умолчанию ошибка, а на стороннем разрешается, то проблема в нашем DNS сервере. Одна из проблем может быть в кешировании устаревшего адреса, что решается через команду ipconfig /flushdns, посмотреть кеш - ipconfig /dispaydns 

## Траблшутинг приложений

Нужно определить с какой частью приложения возникает ошибка. 
- Ошибка в программном коде - не может быть решена
- Если приложение является клиентом и отправляет запросы на сервер, то нужно убедиться что: с сервером все в порядке и проблема не массовая
- у клиента все в порядке с доступами на сервер
- если проблема возникает с сетью, то траблшутим сеть
- если с сетью норм и сервером все окей, проблема локальна, ищем на хосте.

Ошибка доступа может быть решена с помощью утилиты procmon.exe из пакета sysinternals utilities. Программа позволяет просмотреть запросы приложений к файловой системе и ключам реестра. 
Одно из основных применений **Process Monitor** - определение причин аварийного завершения приложений в случае отсутствия или неверного  
расположения файлов, каталогов,  разделов и ключей реестра. 

## Система

BOSD - ошибки, связанные с ядром системы. Могу возникать при загрузке ядра, инициализации и в процессе. 
Наиболее распространенной причиной данной ошибки является некорректная загрузка драйвера. 

Синий экран смерти (BSOD) с разными кодами на разных драйверах с большой
 вероятностью говорит о неполадках в оборудовании, обычно это:

- оперативная память
- материнская плата
- несовместимость памяти и материнской платы
- перегрев микросхем чипсета материнской платы
- вздувшиеся электролитические конденсаторы на материнской плате
- блок питания.

Одним из способов траблшутинга - просмотра дампа памяти после ошибки с помощью средств Windows (Windbg, Dumpchk, DaRT) и сторонних программ BlueScreenView. Это можно сделать из безопасного режима запуска. Если запустить безопасный режим нет возможности, можно прибегнуть к загрузке с liveCD и восстановлению через DaRT.

### Компьютер виснет

Для диагностики можно воспользоваться утилитой из набора sysinternals **ProcessExplorer**. Программа позволяет посмотреть, какое приложение жрет ресурс CPU и предпринять меры. Например, терминировать процесс или понизить его приоритет. 
Еще один из показателей неисправности - процент загруженности процессора операциями прерывания. В норме он должен быть 1-5%. Если этот показател превышен, это может говорить о некоректной работе драйвера устройства.  Провери использование драйверами процессорного времени можно с помощью прокраммы kernrate.

### Удаление вируса

Через **ProcessExplorer** также позволяет обнаружить вирусное ПО. Процессы таких программ подсвечиваются фиолетовым цветом и могут иметь ряд признаков:
- нет цифровой подписи, можно проверить с интернетом. 
- не указан производитель ПО
- имя процесса маскируется под системный процесс

Утилита **SearchMyFiles** позволяет просмотреть все файли которые создавались/открывались/редактировались за определенный промежуток времени. Так можно определить сторонее ПО, отфильтровав созданные файлы по дате поломки. Программа также может использоваться на LiveCD в средства восстановления Windows.

## Железо

### Не начинается загрузка

Не проходит первый этап загрузки (POST), когда система тестирует на работоспособность компонента, необходимые для запуска OS. Обычно производителем материнских плат предусмотрен набор световых или звуковых сигналов ошибок, по которым можно понять, с каким компонентом возникла проблема. Иногда система может не подавать никаких сигналов, что может свидетельствовать о более серьезных проблемах при которых не начинается даже первый этап загрузки OS.

Решение:
- отключить все возможные перефирийные устройства, оставить 1 модуль памяти. 
- на этапе POST система опрашивает все доступные контроллеры и заносит их в специальную таблицу DMI. При смене какого-либо оборудования такая таблица становится неактуальной и вызывает ошибку загрузки. При такой ошибке система оповещает сообщением “Building DMI pool” или “Verifying DMI pool data” 

### Произвольное выключение

Может быть связано с перегревом. Проверяется через системы мониторинга таких как AIDA64

Еще одна из причин - срабатывание защиты БП. Может срабатывать при недостаточной мощности БП или кратковременном замыкании на плате.

# Машрутизация в сети

MAC адрес является физическим адресом сетевого интерфейса и служит для взаимодействия между устройствами в локальной сети на канальном уровне. Назначается производителем устройства или администратором. 

IP адрес нужен для взаимодействия на сетевом уровне в интернете.

Как происходит отправка пакетов. 
Устройство формирует ip-пакет, указывая адрес отправителя и адрес получателя. Инкапсулирует пакет в фрейм и указывает MAC адрес отправителя и получателя. Если MAC адрес неизвестен, рассылается ARP запрос. 

ARP (Adress Resolution Protokol) - протокол разрешения MAC адресов. Состоит из этапов:
- устройством-отправителем формируется широковещателный запрос принадлежности IP адреса
- устройства, которым не принадлежит запрашиваемый IP, отбрасывают пакет
- устройство-получатель отправляет ARP ответ отправителю, указывая свой MAC адрес
- MAC адрес заносится в ARP таблицу
В таблице есть динамические записи, которые заносятся протоколом ARP и статические, которые прописываются вручную. Динамически обновляются через какое-то время, статические остаются в таблице навсегда.

# Загрузка OS

Загрузка системы начинается с подачи питания и выполнения следующих шагов:

1. Запуск BIOS/UEFI на ПЗУ. Этап тестирования контроллеров подпрограммой POST (Power On Self Test), которая проверяет необходимые утройства для запуска OS (процессор, память, клавиатура, видеоадаптер, диск). Если какое-то утсройство неисправно, выдается звуковой/световой сигнал ошибки.
2. Руководствуясь очередностью загрузки, BIOS ищет MBR на диске и помещает находящийся в ней загрузчик в оперативную память, после чего передает ему управление. Определяет главную загрузочную запись по сигнатуре 55AA в последних двух байтах первого сектора диска. Если MBR не найдена, считывается следующее устройство в таблице загрузки. 
3. Загрузчик просматривает таблицу разделов на диске, находит активный раздел (PBR) с OS и помещает в память ядро, после чего передает управление системе инициализации ядра.
4. Инициализация модулей ядра и запуск необходимых системных служб.