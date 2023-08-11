# TLGRM - комбинированный скрипт оповещения в Телеграм и удалённого запуска функций, скриптов и команд RouterOS.

В скрипте использованы идеи и части кода разных авторов.

Для работы скрипта должны быть заранее известны и указаны BotID и ChatID (https://1spla.ru/blog/telegram_bot_for_mikrotik/).
Скрипт необходимо добавить в System/Scripts или System/Sheduler и установить запуск с нужной периодичностью (типовое значение: 1 минута)

Скрипт умеет оповещать о различных событиях на роутере, слушать Телеграм-чат и может реагировать на команды в чате.
Для отправки команды всем или одному конкретному роутеру, находящемуся в ТГ-чате, необходимо отправить соответствующее сообщение.
При этом скрипт должен быть запущен на всех интересующих роутерах.

Настройки прослушки и реакции на команды ТГ-чата хранятся в начале кода скрипта:

 - /bbroadCast/b, где false = реакция только на команды указанному роутеру; true = реакция на все распознанные команды
 - launchScr, где true = разрешение исполнения скриптов; false = запрет исполнения скриптов
 - launchFnc, где true = разрешение использования глобальных функций; false = запрет использования функций
 - launchCmd, где true = разрешение выполнения команд ROS; false = запрет выполнения команд ROS

Настройки оповещения о событиях:

 - sysInfo, где true = разрешение трансляции в ТГ-чат подозрительных событий журнала устройства; false = запрет трансляции событий журнала устройства
 - userInfo, где true = разрешение трансляции в ТГ-чат сообщений, сформированных по пользовательским условиям; false = запрет трансляции пользовательских сообщений

Работа скрипта состоит из двух этапов:

1. Слушатель ТГ-группы + парсер и исполнитель полученных команд

2. Анализатор журнала Mikrotik и транслятор подозрительных сообщений в ТГ-группу

Для отправки команды конкретному роутеру в Телеграм-группе, необходимо сформировать текстовое сообщение. Начало сообщения обозначается символом "/" с последующим указанием имени роутера (должно соответствовать записи в /system identity и не должно содержать пробелов), далее через пробел или подчеркивание пишется команда. Для отправки команды всем роутерам в Телеграм-группе, вместо имени указывается 'forall'.
В качестве команды могут выступать: имя глобальной функции, имя скрипта или команда RouterOS, например:

    /forall log warning [/system identity get name]
    /Mikrotik1 wol
    /MikroTik system reboot

Особенности работы скрипта:
 - поддерживается отправка сообщений в Телеграм-группу с кириллицей (CP1251)
 - отправляемое в Телеграм-группу сообщение обрезается до 4096 байтов
 - скрипт "слушает" только ПОСЛЕДНЕЕ сообщение в ТГ-группе, по этой причине не имеет смысла накидывать много команд, в любом случае будет выполнена только последняя.
 - поддерживается индивидуальная и групповая адресация команд
 - в терминале можно наблюдать поэтапно ход исполнения скрипта

---------------------------------------------------------------------------------------

Последовательность действий для формирования списка команд в Телеграм-группе через бота BotFather.

Дальнейшее взаимодействие производим в ТГ с BotFather:
вводим и отправляем команду: /setcommands
выбираем нужного бота
выскочит подсказка:

CODE: SELECT ALL

        command1 - Description

        command2 - Another description

По сути это шаблон с указанием формата ввода команд, в котором каждая команда должна быть оформлена отдельной строкой, в каждой строке слева от дефиса должен находиться текст с командой.
Этот текст в дальнейшем и будет отправляться при выборе команды. В каждой строке справа от дефиса пишется краткое описание команды. Оно нужно для того, чтобы не держать в голове все команды.
Описание поможет в дальнейшем вспомнить, что это за команда и что она делает.
Следует обратить внимание на важные детали при формировании текстовых строк с командами:
 - текст команды (слева от дефиса) может состоять ТОЛЬКО из цифр, маленьких латинских букв и знака подчёркивания (заглавные буквы, пробелы, спецсимволы и кириллица недопустимы).
 - текст описания (справа от дефиса) может содержать любые символы. Текст обязательно должен быть.
 - при добавлении новых команд, старые команды нужно вбивать вновь... Другими словами: все команды нужно вводить одним списком.

Вводим и отправляем команду по шаблону: имямикротик_имяскрипта - текст с описанием.

Если всё сделано правильно, получим ответ: 'Success! Command list updated.'

Обратите внимание! В "команду" мы на самом деле запихиваем 'имямикротик' и 'имяскрипта' через символ подчёркивания. Это делается чтобы BotFather не ругался и проглотил эту "команду", в которой на самом деле зашифровано имя роутера с названием скрипта, которые в свою очередь должны быть обработаны TLGRM. А т.к. для наших целей пробелы нужны как минимум для отделения ID маршрутизатора от команды, пришлось заниматься самообманом: отныне скрипт считает пробелы (' ') и знаки подчёркивания ('_') тождественными. Таким образом BotFather тянет за собой зависимость -> ID роутеров и названия скриптов могут состоять только из цифр и маленьких латинских букв (заглавные буквы, пробелы, знаки подчёркивания, спецсимволы и кириллица недопустимы!!!).
теперь, нажав на кнопку "/" в своём чате с ботом, можно посмотреть, какие команды (скрипты) и на каких устройствах нам доступны, а при желании можно выбрать нужную команду и отправить её на исполнение.

---------------------------------------------------------------------------------------

Для корректного отображения скриптом 'username' отправителя, в настройках Телеграм должно быть заполнено поле "Имя пользователя", бот должен быть подключен к ГРУППЕ (групповому чату), а не к КАНАЛУ.

Опытным путём выяснено, что предпочтителен групповой чат Телеграм с id БЕЗ префикса '-100', в таком чате сообщения от ГРУППЫ роутеров не теряются.

https://forummikrotik.ru/viewtopic.php?p=89956#p89956

https://forum.mikrotik.com/viewtopic.php?p=1012951#p1012951

https://habr.com/ru/post/650563/
