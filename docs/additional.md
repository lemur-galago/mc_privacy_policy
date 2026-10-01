# Мобильный клиент lsFusion

<!-- TOC -->
* [Мобильный клиент lsFusion](#мобильный-клиент-lsfusion)
  * [Дополнительный функционал клиента. Общие сведения](#дополнительный-функционал-клиента-общие-сведения)
    * [Методы MobileDataTerminal](#методы-mobiledataterminal)
  * [Сообщения пользователю](#сообщения-пользователю)
    * [Toast](#toast)
    * [Message](#message)
  * [Звуковые уведомления](#звуковые-уведомления)
  * [Возврат результата](#возврат-результата)
    * [Обратный вызов](#обратный-вызов)
    * [Запрос к серверу](#запрос-к-серверу)
    * [Отмена](#отмена)
  * [Снимок камерой Android-устройства](#снимок-камерой-android-устройства)
  * [Сканирование штрихкодов](#сканирование-штрихкодов)
    * [Сканирование штрихкодов камерой Android-устройства](#сканирование-штрихкодов-камерой-android-устройства)
    * [Сканирование штрихкодов broadcast-сканером](#сканирование-штрихкодов-broadcast-сканером)
  * [Получение сведений об устройстве и приложении](#получение-сведений-об-устройстве-и-приложении)
<!-- TOC -->

## Дополнительный функционал клиента. Общие сведения

"Мобильный клиент lsFusion" - Android-приложение, которое открывает web-клиент lsFusion во встроенном браузере
(WebView). Начиная с версии 2.0.0006 приложения из кода lsFusion можно использовать его дополнительные возможности:
показывать сообщения, воспроизводить звуки, делать снимки камерой, сканировать штрихкоды и получать сведения об
устройстве. Полный список методов приведен в разделе [Методы MobileDataTerminal](#методы-mobiledataterminal).

> [!IMPORTANT]
> Все примеры ниже рассчитаны на платформу lsFusion версии 6.1 и выше. Для более ранних версий платформы вместо
> реализации абстрактного действия `onWebClientInit[]` добавьте в реализацию абстрактного действия
> `onWebClientStarted[]` строку:
>
> `INTERNAL CLIENT 'mdt.js';`

Когда web-клиент lsFusion открыт в приложении "Мобильный клиент lsFusion", на его страницу добавляется JS-интерфейс
`MobileDataTerminal`, через который web-клиент взаимодействует с приложением. Поэтому убедиться, что web-клиент запущен
именно в приложении, можно по наличию объекта `MobileDataTerminal`:

Файл _mdt.js_ разместите в папке ресурсов проекта:

```javascript
function checkMobileDataTerminal() {
    return typeof MobileDataTerminal !== 'undefined'
}
```

Модуль _Check.lsf_

```Lsf
MODULE Check;

REQUIRE SystemEvents;

mdtCheck 'Проверить' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'checkMobileDataTerminal' TO flag;
    IF flag() THEN
        MESSAGE 'Это мобильный клиент';
    ELSE
        MESSAGE 'Это не мобильный клиент';
}

FORM mdtDemo 'Demo'
    PROPERTIES mdtCheck()
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

### Методы MobileDataTerminal

| Метод | Назначение | С версии | Раздел |
|-------|------------|----------|--------|
| `showToast(text)` | Всплывающее сообщение (Toast) | 2.0.0006 | [Toast](#toast) |
| `showMessage(text)` | Диалог с сообщением | 2.0.0014 | [Message](#message) |
| `playNotify()`, `playAlarm()`, `playRingtone()` | Воспроизведение стандартных звуков Android | 2.0.0018 | [Звуковые уведомления](#звуковые-уведомления) |
| `stopRingtone()` | Остановка воспроизведения звука | 2.0.0018 | [Звуковые уведомления](#звуковые-уведомления) |
| `captureImage(tag[, callback])` | Снимок камерой | 2.0.0006 | [Снимок камерой Android-устройства](#снимок-камерой-android-устройства) |
| `readBarcode(tag[, callback])` | Сканирование штрихкодов камерой | 2.0.0015 (`callback` - с 2.0.0018) | [Сканирование штрихкодов камерой Android-устройства](#сканирование-штрихкодов-камерой-android-устройства) |
| `waitBarcode(tag[, callback])` | Ожидание штрихкода от broadcast-сканера | 2.0.0018 | [Сканирование штрихкодов broadcast-сканером](#сканирование-штрихкодов-broadcast-сканером) |
| `stopWaitingBarcode()` | Отмена ожидания штрихкода | 2.0.0018 | [Сканирование штрихкодов broadcast-сканером](#сканирование-штрихкодов-broadcast-сканером) |
| `getAppInfo()` | Сведения об устройстве и приложении | 2.0.0010 | [Получение сведений об устройстве и приложении](#получение-сведений-об-устройстве-и-приложении) |

## Сообщения пользователю

### Toast

Через интерфейс `MobileDataTerminal` можно вызвать стандартный Android-Toast с сообщением:

Файл _mdt.js_

```javascript
function showToast(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.showToast(text)
    return true
}
```

Модуль _Toast.lsf_

```Lsf
MODULE Toast;

REQUIRE SystemEvents;

showToast 'Показать Toast' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'showToast' PARAMS 'Hello world!!!' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

FORM mdtDemo 'Demo'
    PROPERTIES showToast()
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

Аналогичным образом можно вызвать Toast в другом соединении. Пример ниже показывает Toast во всех активных соединениях,
кроме текущего (web-клиенты, запущенные не в приложении, вызов проигнорируют):

```Lsf
MODULE ToastEverybody;

REQUIRE SystemEvents;

showToastForEverybody 'Показать Toast всем' () {
    FOR connectionStatus(Connection c) = ConnectionStatus.connectedConnection AND NOT c = currentConnection() DO
        NEWTHREAD
            INTERNAL CLIENT 'showToast' PARAMS 'Toast for everybody!!!';
            CONNECTION c;
}

FORM mdtDemo 'Demo'
    PROPERTIES showToastForEverybody()
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

### Message

Начиная с версии 2.0.0014 через интерфейс `MobileDataTerminal` можно показать диалог с сообщением и одной кнопкой "OK".
Вызов асинхронный: метод возвращает управление сразу, не дожидаясь закрытия диалога.

Файл _mdt.js_

```javascript
function showMessage(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.showMessage(text)
    return true
}
```

Модуль _Message.lsf_

```Lsf
MODULE Message;

REQUIRE SystemEvents;

showMessage 'Показать сообщение' () {
    INTERNAL CLIENT 'showMessage' PARAMS 'Hello world!';
}

showMessageForEverybody 'Показать сообщение всем' () {
    FOR connectionStatus(Connection c) = ConnectionStatus.connectedConnection AND NOT c = currentConnection() DO
        NEWTHREAD
            INTERNAL CLIENT 'showMessage' PARAMS 'Hello ${userLogin(c)}';
            CONNECTION c;
}

FORM mdtDemo 'Demo'
    PROPERTIES () showMessage, showMessageForEverybody
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

Если в момент вызова на экране уже отображается диалог с сообщением, новое сообщение будет добавлено в него с новой
строки.

Модуль _MultiMessage.lsf_

```Lsf
MODULE MultiMessage;

REQUIRE SystemEvents, Utils;

showMessage 'Показать сообщение' () {
    FOR count(INTEGER i, 9) DO INTERNAL CLIENT NOWAIT 'showMessage' PARAMS 'Message ${i}';
}

FORM mdtDemo 'Demo'
    PROPERTIES () showMessage
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

## Звуковые уведомления

Начиная с версии 2.0.0018 через интерфейс `MobileDataTerminal` можно воспроизводить стандартные звуки Android:
уведомления, будильника и звонка.

Для воспроизведения используются соответствующие методы:

`MobileDataTerminal.playNotify()`

`MobileDataTerminal.playAlarm()`

`MobileDataTerminal.playRingtone()`

Остановить воспроизведение можно методом:

`MobileDataTerminal.stopRingtone()`

Метод останавливает любой из трех звуков. Запуск нового звука прерывает воспроизведение предыдущего. Если в
настройках приложения (раздел "Уведомления") включено "Регулировать громкость", звук воспроизводится с громкостью из
настройки "Уровень громкости, %" (на Android 9 и выше).

Пример:

Файл _mdt.js_

```javascript
function playNotify() {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.playNotify()
    return true
}

function playAlarm() {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.playAlarm()
    return true
}

function playRingtone() {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.playRingtone()
    return true
}

function stopRingtone() {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.stopRingtone()
    return true
}
```

Модуль _Ringtone.lsf_

```Lsf
MODULE Ringtone;

REQUIRE SystemEvents;

notify 'Уведомление' () {
    LOCAL flag = BOOLEAN();
    INTERNAL CLIENT 'playNotify' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

alarm 'Будильник' () {
    LOCAL flag = BOOLEAN();
    INTERNAL CLIENT 'playAlarm' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

ringtone 'Звонок' () {
    LOCAL flag = BOOLEAN();
    INTERNAL CLIENT 'playRingtone' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

stop 'Остановить' () {
    LOCAL flag = BOOLEAN();
    INTERNAL CLIENT 'stopRingtone' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

FORM mdtDemo 'Demo'
    PROPERTIES () notify, alarm, ringtone, stop
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo DOCKED;
}
```

## Возврат результата

Методы `captureImage`, `readBarcode` и `waitBarcode` возвращают результат асинхронно: приложение вызывает JS-функцию
обратного вызова или, если она не указана, отправляет результат на сервер lsFusion.

В параметре `tag` передается произвольное значение, которое приложение вернет вместе с результатом. По нему можно
определить, к какому вызову относится результат. Подойдет, например, идентификатор соединения, имя или идентификатор
пользователя либо сгенерированное уникальное значение.

### Обратный вызов

В параметре `callback` можно указать JS-выражение, задающее callback-функцию: имя глобальной функции или, как в примерах
ниже, `customCameraImage().callback`. Приложение вызовет функцию с двумя строковыми аргументами: первый - значение
параметра `tag`, второй - результат. Что передается во втором аргументе, описано в разделе каждого метода.

Значение `tag` подставляется в вызов callback-функции без экранирования, поэтому при вызове с `callback` оно не должно
содержать двойных кавычек, обратной косой черты и переводов строк.

### Запрос к серверу

Если параметр `callback` опущен, приложение отправит результат на сервер, к которому оно подключено:

`POST /exec/mobileApiResult?p=<tag>`

В параметре `p` адресной строки передается значение `tag`, а в теле запроса - результат: изображение в формате JPEG
(`image/jpeg`) или JSON-массив штрихкодов (`application/json`). В заголовке запроса передаются параметры
Basic-авторизации с логином и паролем пользователя, авторизованного в приложении.

Для приема результата на сервере должно быть объявлено действие `mobileApiResult` с аннотацией `@@api` и двумя
параметрами: значением `tag` и файлом с результатом. Тип первого параметра должен соответствовать передаваемому значению
`tag`. Примеры такого действия приведены в модулях _ApiCameraImage.lsf_, _ApiReadBarcode.lsf_ и _ApiWaitBarcode.lsf_.

О результате отправки приложение сообщает всплывающим сообщением: "Снимок успешно передан" или "Штрихкод успешно
передан". Если сервер недоступен или ответил кодом не из диапазона 2xx, приложение покажет сообщение об ошибке.

> [!WARNING]
> До версии 2.0.0015 серверу приложений отправлялся запрос `POST /exec/postImage?p=<tag>`.
>
> В версии 2.0.0015 добавлена настройка "Использовать метод postImage" (раздел "Обратная совместимость" настроек
> приложения). При включенной настройке вместо `mobileApiResult` отправляется запрос `POST /exec/postImage?p=<tag>`
> с тем же содержимым.
>
> Данная настройка добавлена для временного сохранения обратной совместимости и будет удалена в следующих версиях.

### Отмена

Если пользователь закроет фрагмент камеры или сканера штрихкодов без результата, не будет выполнен ни обратный вызов,
ни запрос к серверу: сервер об отмене не узнает. Подписку на штрихкоды от broadcast-сканера отменяет метод
`MobileDataTerminal.stopWaitingBarcode()` (см. раздел
[Сканирование штрихкодов broadcast-сканером](#сканирование-штрихкодов-broadcast-сканером)).

## Снимок камерой Android-устройства

Через интерфейс `MobileDataTerminal` на Android-устройстве можно открыть фрагмент "Камера" и сделать снимок. Приложение
передаст снимок в JS-функцию обратного вызова или, если она не указана, на эндпоинт сервера lsFusion.

Для вызова фрагмента камеры используется метод:

`MobileDataTerminal.captureImage(<tag>[, <callback>])`

Параметры `tag` и `callback` и способы возврата результата описаны в разделе [Возврат результата](#возврат-результата).

Callback-функция будет вызвана после того, как пользователь сделает снимок камерой. Вторым аргументом (`encodedImage`)
ей передается Base64-строка с JPEG-изображением. Если параметр `callback` опущен, в теле запроса
`POST /exec/mobileApiResult?p=<tag>` передается файл изображения в формате JPEG.

Обращение к камере с callback можно использовать, например, в custom-представлении lsFusion:

Файл _mdt.js_

```javascript
function customCameraImage() {
    return {
        render: element => {
            if (typeof MobileDataTerminal === 'undefined') return
            element.id = Date.now().toString()
            const button = document.createElement('button')
            button.addEventListener(
                'click',
                (_) =>
                    MobileDataTerminal.captureImage(element.id, 'customCameraImage().callback')
            )
            button.innerHTML = 'captureImage'
            element.append(button)
        },
        update: (element, controller, value) => {
            element.controller = controller
        },
        callback: (tag, encodedImage) => {
            const element = document.getElementById(tag)
            if (element) {
                element.controller.changeValue(encodedImage)
            }
        }
    }
}
```

Модуль _CustomCameraImage.lsf_

```Lsf
MODULE CustomCameraImage;

REQUIRE SystemEvents, Utils;

image = DATA LOCAL IMAGEFILE ();

FORM mdtDemo 'Demo'
    PROPERTIES image '' = image() READONLY, button '' = image() CUSTOM 'customCameraImage'
    ON CHANGE {
        INPUT encoded = TEXT DO {
            IF encoded THEN image() <- decode(encoded, 'base64');
        }
    }
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

Обращение к камере без callback можно использовать, например, если изображение не требуется отображать на форме.
Пример ниже сохраняет изображение на сервере приложений, не отображая его в интерфейсе:

Файл _mdt.js_

```javascript
function apiCameraImage(tag) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.captureImage(tag)
    return true
}
```

Модуль _ApiCameraImage.lsf_

```Lsf
MODULE ApiCameraImage;

REQUIRE SystemEvents;

captureImage 'Сделать снимок' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'apiCameraImage' PARAMS currentConnection() TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

mobileApiResult(LONG tag, FILE file) {
    LOCAL connection = Connection();
    connection() <- [GROUP MAX Connection c AS Connection BY LONG(c)](tag);
    TRY {
        WRITE file TO '/tmp/file';
        NEWTHREAD MESSAGE 'Файл успешно сохранен'; CONNECTION connection();
    } CATCH {
        NEWTHREAD MESSAGE 'Ошибка сохранения файла'; CONNECTION connection();
    }
}@@api;

FORM mdtDemo 'Demo'
    PROPERTIES captureImage()
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

## Сканирование штрихкодов

### Сканирование штрихкодов камерой Android-устройства

Начиная с версии 2.0.0015 через интерфейс `MobileDataTerminal` на Android-устройстве можно открыть фрагмент "Сканер
штрихкодов", который считывает штрихкоды задней камерой устройства. Результат передается на эндпоинт сервера lsFusion
в виде JSON-массива или, начиная с версии 2.0.0018, в JS-функцию обратного вызова.

> [!NOTE]
> В настоящее время поддерживаются следующие виды штрихкодов:
> - Линейные форматы: Codabar, Code 39, Code 93, Code 128, EAN-8, EAN-13, ITF, UPC-A, UPC-E.
> - 2D-форматы: Aztec, Data Matrix, PDF417, QR-код.

> [!NOTE]
> Обратите внимание, что приложение не распознает следующие штрихкоды:
> - 1D-штрихкоды, состоящие только из одного символа;
> - штрихкоды в формате ITF с количеством символов менее шести.

Для вызова фрагмента сканера штрихкодов используется метод:

`MobileDataTerminal.readBarcode(<tag>[, <callback>])`

Параметры `tag` и `callback` и способы возврата результата описаны в разделе [Возврат результата](#возврат-результата).

Callback-функция будет вызвана после того, как приложение успешно распознает хотя бы один штрихкод. Вторым аргументом
(`barcode`) ей передается распознанный штрихкод. Если в кадре распознано несколько штрихкодов, передается ближайший к
центру кадра (до версии 2.0.0020 - произвольный из них). Остальные штрихкоды в callback-функцию не передаются: полный
список можно получить только при отправке результата на сервер.

Если параметр `callback` опущен, то после успешного распознавания хотя бы одного штрихкода в теле запроса
`POST /exec/mobileApiResult?p=<tag>` передается JSON-файл с массивом распознанных штрихкодов. Начиная с версии 2.0.0020
штрихкоды в массиве упорядочены по удаленности от центра кадра: первым идет ближайший к центру.

Пример файла, содержащего массив штрихкодов:

```json
[
  {
    "format":32,
    "rawValue":"6942103112508",
    "valueType":5
  }
]
```

Поле `format` - целое число, обозначающее формат штрихкода (значения совпадают с константами `Barcode.FORMAT_*`
библиотеки Google ML Kit):

| Возвращаемое<br/>значение |           Формат штрихкода           |
|:-------------------------:|:------------------------------------:|
|             1             |               Code 128               |
|             2             |               Code 39                |
|             4             |               Code 93                |
|             8             |               Codabar                |
|            16             |             Data Matrix              |
|            32             |                EAN-13                |
|            64             |                EAN-8                 |
|            128            |                 ITF                  |
|            256            |                QR-код                |
|            512            |                UPC-A                 |
|           1024            |                UPC-E                 |
|           2048            |                PDF417                |
|           4096            |                Aztec                 |
|            -1             |          Неизвестный формат          |

Поле `rawValue` содержит строковое представление распознанного штрихкода.

Поле `valueType` содержит тип информации, закодированной штрихкодом (значения совпадают с константами `Barcode.TYPE_*`
библиотеки Google ML Kit):

| Возвращаемое<br/>значение | Тип информации                        |
|:-------------------------:|:--------------------------------------|
|             1             | Контактная информация                 |
|             2             | Адрес электронной почты               |
|             3             | Код ISBN                              |
|             4             | Номер телефона                        |
|             5             | Код товара                            |
|             6             | SMS-сообщение                         |
|             7             | Произвольный текст                    |
|             8             | URL                                   |
|             9             | Данные для подключения к Wi-Fi        |
|            10             | Геопозиция                            |
|            11             | Событие календаря                     |
|            12             | Сведения о водительском удостоверении |
|             0             | Неизвестный тип информации            |

Чтение штрихкода камерой устройства с callback можно использовать, например, в custom-представлении lsFusion:

Файл _mdt.js_

```javascript
function customReadBarcode() {
    return {
        render: element => {
            element.id = Date.now().toString()
            const input = document.createElement('input')
            element.append(input)
            element.input = input
            if (typeof MobileDataTerminal === 'undefined') return
            const button = document.createElement('button')
            button.addEventListener(
                'click',
                (_) =>
                    MobileDataTerminal.readBarcode(element.id, 'customReadBarcode().callback')
            )
            button.innerHTML = 'readBarcode'
            element.append(button)
        },
        update: (element, controller, value) => {
            element.controller = controller
            element.input.value = value
        },
        callback: (tag, barcode) => {
            const element = document.getElementById(tag)
            if (element) {
                element.controller.changeValue(barcode)
            }
        }
    }
}
```

Модуль _CustomReadBarcode.lsf_

```Lsf
MODULE CustomReadBarcode;

REQUIRE SystemEvents;

barcode = DATA LOCAL STRING();

FORM mdtDemo 'Demo'
    PROPERTIES barcode() CUSTOM 'customReadBarcode'
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

Чтение штрихкода камерой без callback можно использовать, например, если штрихкод не требуется отображать
на форме или необходимо получить дополнительные сведения о штрихкоде (формат штрихкода и тип информации).
Пример ниже сохраняет на сервере приложений JSON-файл, содержащий массив распознанных штрихкодов с дополнительными
сведениями о них, не отображая его в интерфейсе:

Файл _mdt.js_

```javascript
function apiReadBarcode(tag) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.readBarcode(tag)
    return true
}
```

Модуль _ApiReadBarcode.lsf_

```Lsf
MODULE ApiReadBarcode;

REQUIRE SystemEvents;

readBarcode 'Получить штрихкод' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'apiReadBarcode' PARAMS currentConnection() TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

mobileApiResult(LONG tag, FILE file) {
    LOCAL connection = Connection();
    connection() <- [GROUP MAX Connection c AS Connection BY LONG(c)](tag);
    TRY {
        WRITE file TO '/tmp/file';
        NEWTHREAD MESSAGE 'Файл успешно сохранен'; CONNECTION connection();
    } CATCH {
        NEWTHREAD MESSAGE 'Ошибка сохранения файла'; CONNECTION connection();
    }
}@@api;

FORM mdtDemo 'Demo'
    PROPERTIES readBarcode()
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

### Сканирование штрихкодов broadcast-сканером

Начиная с версии 2.0.0018 через интерфейс `MobileDataTerminal` можно подписаться на широковещательное сообщение
(broadcast), передаваемое сканером штрихкодов при успешном сканировании. Работу в broadcast-режиме поддерживает
большинство моделей терминалов сбора данных, а также некоторые модели внешних сканеров штрихкодов.

> [!NOTE]
> Для работы в режиме broadcast:
> - в настройках сканера устройства включите передачу результата сканирования через broadcast;
> - в настройках сканера устройства и в разделе "Сканер" настроек приложения (поля "Intent" и "Resource") укажите
>   одинаковые имя Intent и имя ресурса (extra), в котором передается штрихкод. По умолчанию в приложении заданы
>   `org.lsfusion.mobileclient.scanner.broadcast` и `data`.

Для подписки на широковещательное сообщение сканирования штрихкода используется метод:

`MobileDataTerminal.waitBarcode(<tag>[, <callback>])`

Параметры `tag` и `callback` и способы возврата результата описаны в разделе [Возврат результата](#возврат-результата).

Callback-функция будет вызвана после того, как сканер пришлет широковещательное сообщение о прочитанном штрихкоде.
Вторым аргументом (`barcode`) ей передается прочитанный штрихкод. Если параметр `callback` опущен, в теле запроса
`POST /exec/mobileApiResult?p=<tag>` передается JSON-файл с массивом из одного элемента - прочитанного штрихкода.

Пример файла, передаваемого в теле запроса:

```json
[
  {
    "format": -1,
    "rawValue":"6942103112508",
    "valueType": 0
  }
]
```

Поле `rawValue` содержит строковое представление прочитанного штрихкода. Поле `format` всегда равно -1 (формат
неизвестен), а `valueType` - 0 (тип информации неизвестен).

Одновременно может быть активна только одна подписка: повторный вызов `MobileDataTerminal.waitBarcode` отменяет
предыдущую. Подписка одноразовая - после получения штрихкода она отменяется автоматически, поэтому, чтобы получить
следующий штрихкод, метод нужно вызвать снова. Подписка отменяется и при закрытии web-клиента в приложении. Отменить
подписку досрочно, например на время, когда сканирование не ожидается, можно методом:

`MobileDataTerminal.stopWaitingBarcode()`

Получение штрихкода от broadcast-сканера с callback можно использовать, например, в custom-представлении lsFusion:

Файл _mdt.js_

```javascript
function customWaitBarcode() {
    return {
        render: element => {
            element.id = Date.now().toString()
            const input = document.createElement('input')
            element.append(input)
            element.input = input
        },
        update: (element, controller, value) => {
            element.controller = controller
            element.input.value = value
            if (typeof MobileDataTerminal === 'undefined') return
            MobileDataTerminal.waitBarcode(element.id, 'customWaitBarcode().callback')
        },
        clear: (element) => {
            if (typeof MobileDataTerminal === 'undefined') return
            MobileDataTerminal.stopWaitingBarcode()
        },
        callback: (tag, barcode) => {
            const element = document.getElementById(tag)
            if (element) {
                element.controller.changeValue(barcode)
                MobileDataTerminal.waitBarcode(element.id, 'customWaitBarcode().callback')
            }
        }
    }
}
```

Модуль _CustomWaitBarcode.lsf_

```Lsf
MODULE CustomWaitBarcode;

REQUIRE SystemEvents;

barcode = DATA LOCAL STRING();

FORM mdtDemo 'Demo'
    PROPERTIES barcode() CUSTOM 'customWaitBarcode'
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

Получение штрихкода от broadcast-сканера без callback можно использовать, например, если штрихкод не
требуется отображать на форме.
Пример ниже сохраняет на сервере приложений JSON-файл с прочитанным штрихкодом, не отображая штрихкод в интерфейсе,
и после успешного сохранения снова подписывается на сканирование:

Файл _mdt.js_

```javascript
function stopWaitingBarcode() {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.stopWaitingBarcode()
    return true
}

function apiWaitBarcode(tag) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.waitBarcode(tag)
    return true
}
```

Модуль _ApiWaitBarcode.lsf_

```Lsf
MODULE ApiWaitBarcode;

REQUIRE SystemEvents;

mobileApiResult(LONG tag, FILE file) {
    LOCAL connection = Connection();
    connection() <- [GROUP MAX Connection c AS Connection BY LONG(c)](tag);
    TRY {
        WRITE file TO '/tmp/file';
        NEWTHREAD {
            MESSAGE 'Файл успешно сохранен';
            INTERNAL CLIENT 'apiWaitBarcode' PARAMS currentConnection();
        }  CONNECTION connection();
    } CATCH {
        NEWTHREAD MESSAGE 'Ошибка сохранения файла'; CONNECTION connection();
    }
}@@api;

FORM mdtDemo 'Demo'
    EVENTS
        ON INIT {
            LOCAL flag = BOOLEAN ();
            INTERNAL CLIENT 'apiWaitBarcode' PARAMS currentConnection() TO flag;
            IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
        },
        ON CLOSE {
            INTERNAL CLIENT 'stopWaitingBarcode';
        }
;

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo;
}
```

## Получение сведений об устройстве и приложении

Начиная с версии 2.0.0010 через интерфейс `MobileDataTerminal` можно получить сведения об устройстве, на котором
запущен мобильный клиент, и о настройках самого мобильного клиента.

Файл _mdt.js_

```javascript
function getAppInfo() {
    if (typeof MobileDataTerminal === 'undefined') return
    return JSON.parse(MobileDataTerminal.getAppInfo())
}
```

Модуль _AppInfo.lsf_

```Lsf
MODULE AppInfo;

REQUIRE SystemEvents;

appInfo 'Сведения о приложении' = DATA LOCAL JSON ();

getInfo 'Получить сведения' () {
    INTERNAL CLIENT 'getAppInfo' TO appInfo;
    IF NOT appInfo() THEN MESSAGE 'Это не мобильный клиент';
} TOOLBAR;

FORM mdtDemo 'Demo'
    PROPERTIES () appInfo, getInfo
;

DESIGN mdtDemo {
    GROUP() { fill = 1; }
    PROPERTY (appInfo()) { fill = 1; captionVertical = TRUE; }
}

onWebClientInit() + {
    onWebClientInit('mdt.js') <- 10;
}

onWebClientStarted() + {
    SHOW mdtDemo DOCKED;
}
```

В результате выполнения действия `getInfo()` в свойство `appInfo()` будут записаны сведения об устройстве, на
котором запущен мобильный клиент, а также о настройках и версии мобильного клиента:

```json
{
  "applicationSettings": {
    "barcodeScanner": {
      "broadcast": false,
      "intent": "org.lsfusion.mobileclient.scanner.broadcast",
      "resource": "data"
    },
    "instanceId": "001a36bd-1f61-467b-bcbb-949a6712cc28",
    "lockOrientation": false,
    "login": {
      "hideKeyboardOnLogin": false,
      "hideLogo": false,
      "loginOnEnter": false
    }
  },
  "build": {
    "board": "goldfish_x86_64",
    "brand": "google",
    "device": "emu64xa16k",
    "display": "CP21.260306.017.A1 dev-keys",
    "host": "970e3cfe68aa",
    "id": "CP21.260306.017.A1",
    "manufacturer": "Google",
    "model": "sdk_gphone16k_x86_64",
    "version": {
      "release": "17",
      "sdkInt": 37,
      "securityPatch": "2026-03-05"
    }
  },
  "config": {
    "applicationId": "org.lsfusion.mobileclient",
    "versionCode": 40,
    "versionName": "2.0.0018"
  },
  "environment": {
    "webView": {
      "lastUpdateTime": 1774982604197,
      "versionCode": 763204538,
      "versionName": "145.0.7632.45"
    }
  }
}
```

Поля `applicationSettings`, кроме `instanceId`, соответствуют настройкам приложения:

| Поле | Настройка | Раздел настроек |
|------|-----------|-----------------|
| `lockOrientation` | "Фиксировать ориентацию экрана" | "Приложение" |
| `login.hideKeyboardOnLogin` | "Скрывать клавиатуру при авторизации" | "Приложение" |
| `login.loginOnEnter` | "Авторизоваться по Enter" | "Приложение" |
| `login.hideLogo` | "Скрывать логотип" | "Приложение" |
| `notifyToken` | "Разрешить уведомления" (см. ниже) | "Уведомления" |
| `barcodeScanner.broadcast` | "Broadcast-сканер" | "Сканер" |
| `barcodeScanner.intent` | "Intent" | "Сканер" |
| `barcodeScanner.resource` | "Resource" | "Сканер" |

Поле `applicationSettings.instanceId` содержит UUID, который генерируется при первом запуске приложения и хранится в его
данных. После очистки данных приложения будет сгенерирован новый UUID.

Начиная с версии 2.0.0016, если в настройках приложения (раздел "Уведомления") включено "Разрешить уведомления",
в `applicationSettings` также возвращается поле `notifyToken` - токен push-уведомлений Firebase Cloud Messaging,
выданный приложению на устройстве. Если настройка выключена или токен еще не получен, поле отсутствует.
