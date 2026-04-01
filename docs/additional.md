# Мобильный клиент lsFusion

<!-- TOC -->
* [Мобильный клиент lsFusion](#мобильный-клиент-lsfusion)
  * [Дополнительный функционал клиента. Общие сведения](#дополнительный-функционал-клиента-общие-сведения)
  * [Сообщения пользователю](#сообщения-пользователю)
    * [Toast](#toast)
    * [Message](#message)
  * [Звуковые уведомления](#звуковые-уведомления)
  * [Снимок камерой Android-устройства](#снимок-камерой-android-устройства)
  * [Сканирование штрихкодов](#сканирование-штрихкодов)
    * [Сканирование штрихкодов камерой мобильного устройства](#сканирование-штрихкодов-камерой-мобильного-устройства)
    * [Сканирование штрихкодов broadcast-сканером](#сканирование-штрихкодов-broadcast-сканером)
  * [Получение сведений об устройстве и приложении](#получение-сведений-об-устройстве-и-приложении)
<!-- TOC -->

## Дополнительный функционал клиента. Общие сведения

Начиная с версии 2.0.0006 добавлена возможность использования дополнительного функционала клиента из кода lsFusion.

> [!IMPORTANT]  
> Все примеры ниже рассчитаны на работу с платформой версии 6.1 и выше. При использовании мобильного клиента с более 
> старыми версиями платформы вместо реализации абстрактного действия `onWebClientInit[]` в реализацию абстрактного 
> действия `onWebClientStarted[]` следует добавить строку:
> 
> `INTERNAL CLIENT 'mdt.js';`

Для использования дополнительного функционала при открытии web-клиента lsFusion в приложении
"Мобильный клиент lsFusion" добавляется js-интерфейс `MobileDataTerminal`, позволяющий web-клиенту взаимодействовать с
приложением. В частности, убедиться в том, что web-клиент запущен в приложении можно по существованию
MobileDataTerminal:

Файл _mdt.js_ разместить в файле ресурсов проекта

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

Аналогичным образом можно вызвать Toast в другом соединении. Пример ниже показывает Toast на всех активных устройствах,
подключенных с помощью приложения:

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

Начиная с версии 2.0.0014, аналогично Toast, через интерфейс `MobileDataTerminal` можно вызвать асинхронный диалог с 
одной кнопкой позитивной реакции.

Файл _mdt.js_

```javascript
function showMessage(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.showMessage(text)
    return true;
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

Если во время вызова диалога на экране уже отображается другой диалог, его содержимое будет дополнено новым 
сообщением.

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

Начиная с версии 2.0.0018 добавлена возможность через интерфейс `MobileDataTerminal` воспроизведения стандартных 
звуков Android: уведомление, будильник и звонок.

Для воспроизведения используются соответствующие методы:

`MobileDataTerminal.playNotify()`

`MobileDataTerminal.playAlarm()`

`MobileDataTerminal.playRingtone()`

Остановить воспроизведение можно методом:

`MobileDataTerminal.stopRingtone()`

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

## Снимок камерой Android-устройства

Через интерфейс `MobileDataTerminal` на Android-устройстве можно открыть фрагмент "Камера", позволяющий сделать снимок,
после чего вызвать или эндпоинт сервера lsFusion, или js-callback.

Для вызова фрагмента камеры используется метод:

`MobileDataTerminal.captureImage(<tag>[, <callback>])`

В параметре `tag` передается значение, которое будет возвращено при обратном вызове вместе с результатом. В частности в
параметр `tag` можно передавать значение, позволяющее определить, что возвращаемое в обратном вызове значение относится
именно к этому вызову метода (это может быть идентификатор соединения, имя или уникальный идентификатор пользователя,
генерируемое уникальное значение и т.д.)

В параметре `callback` можно указать имя callback-функции js, которая будет вызвана после того, как пользователь в
приложении сделал снимок камерой. Функция должна принимать на вход два обязательных параметра `tag` и `encodedImage`.
При вызове callback-функции параметре `tag` будет передаваться значение, которое было передано в параметре `tag` 
метода `MobileDataTerminal.captureImage`, а в параметре `encodedImage` - base64-строка с jpeg-изображением.

Если параметр `callback` опущен, то вместо обратного вызова js-функции приложение отправит запрос серверу приложений:

`POST /exec/mobileApiResult?p=<tag>`

где в параметре `p` адресной строки будет передан `tag` метода `MobileDataTerminal.captureImage`, а в теле запроса будет
передан файл изображения. При этом в заголовке запроса будут переданы параметры Basic-авторизации с логином и паролем
пользователя, авторизованного в приложении.

> [!WARNING]
> До версии 2.0.0015 серверу приложений отправляется запрос `POST /exec/postImage?p=<tag>`
> 
> В версии 2.0.0015 добавлена настройка "Использовать метод postImage". При включённой настройке для передачи 
> данных серверу приложений будет отправляться старый запрос.
> 
> Данная настройка добавлена для временного сохранения обратной совместимости будет удалена в следующих версиях. 
  

Обращение к камере с callback можно использовать, например, в custom-представлении lsFusion.

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
        INPUT image = TEXT DO {
            IF image THEN image() <- decode(image, 'base64');
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

Обращение к камере без callback можно использовать, например, если изображение не требуется отображать на форме:
Пример ниже сохраняет изображение на сервере приложений, не отображая его в интерфейсе:

Файл _mdt.js_

```javascript
function apiCameraImage(tag) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.captureImage(tag)
    return true
}
```

Модуль _Main.lsf_

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
        NEWTHREAD MESSAGE 'Файл успешно сохранён'; CONNECTION connection();
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

### Сканирование штрихкодов камерой мобильного устройства

Начиная с версии 2.0.0015 Через интерфейс `MobileDataTerminal` на Android-устройстве можно открыть фрагмент "Сканер 
штрихкодов", позволяющий распознать и прочитать штрихкоды задней камерой мобильного устройства, после чего вызвать 
эндпоинт сервера lsFusion и передать json-массив прочитанных штрихкодов.

> [!NOTE]
> В настоящее время поддерживаются следующие виды штрихкодов:
> - Линейные форматы: Codabar, Code 39, Code 93, Code 128, EAN-8, EAN-13, ITF, UPC-A, UPC-E.
> - 2D-форматы: Aztec, Data Matrix, PDF417, QR-код.

> [!NOTE]
> Обратите внимание, что приложение не распознает следующие штрихкоды:
> - 1D штрих-коды, состоящие только из одного символа;
> - Штрих-коды в формате ITF с количеством символов менее шести.

Для вызова фрагмента сканера штрихкодов используется метод:

`MobileDataTerminal.readBarcode(<tag>[, <callback>])`

В параметре `tag` передается значение, которое будет возвращено при обратном вызове вместе с результатом. В частности в
параметр `tag` можно передавать значение, позволяющее определить, что возвращаемое в обратном вызове значение относится
именно к этому вызову метода (это может быть идентификатор соединения, имя или уникальный идентификатор пользователя,
генерируемое уникальное значение и т.д.)

Начиная с версии 2.0.0018 в параметре `callback` можно указать имя callback-функции js, которая будет вызвана после 
того, как в приложении был успешно распознан хотя бы один штрихкод. Функция должна принимать на вход два 
обязательных параметра `tag` и `barcode`. При вызове callback-функции в параметре `tag` будет передаваться значение, 
которое было передано в параметре `tag` метода `MobileDataTerminal.readBarcode`, а в параметре `barcode` - строка, 
содержащая распознанный штрихкод.

Если параметр `callback` опущен, то, если хотя бы один штрихкод прочитан и успешно распознан, то вместо обратного 
вызова js-функции приложение отправит запрос серверу приложений:

`POST /exec/mobileApiResult?p=<tag>`

где в параметре `p` адресной строки будет передан `tag` метода `MobileDataTerminal.readBarcode`, а в теле запроса будет
передан json-файл, содержащий массив распознанных штрихкодов. При этом в заголовке запроса будут переданы параметры 
Basic-авторизации с логином и паролем пользователя, авторизованного в приложении.

> [!WARNING]
> При включённой настройке "Использовать метод postImage" серверу приложений будет отправляться запрос
> `POST /exec/postImage?p=<tag>` содержащий в теле json-массив распознанных штрихкодов.
>
> Данная настройка добавлена для временного сохранения обратной совместимости будет удалена в следующих версиях.


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

Поле `format` элемента массива, содержащего штрихкод - целое число, определяющее формат распознанного штрихкода:

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
|             0             | Другой формат, не перечисленный выше |

Поле `rawValue` содержит строковое представление распознанного штрихкода.

Поле `valueType` содержит тип информации, закодированной штрихкодом:

| Возвращаемое<br/>значение | Тип информации                        |
|:-------------------------:|:--------------------------------------|
|             1             | Контактная информация                 |
|             2             | Адрес email                           |
|             3             | Код ISBN                              |
|             4             | Номер телефона                        |
|             5             | Код товара                            |
|             6             | SMS-сообщение                         |
|             7             | Произвольный текст                    |
|             8             | URL                                   |
|             9             | Данные для подключения к WiFi         |
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
            if (typeof MobileDataTerminal === 'undefined') return
            element.id = Date.now().toString()
            const input = document.createElement('input')
            const button = document.createElement('button')
            button.addEventListener(
                'click',
                (_) =>
                    MobileDataTerminal.readBarcode(element.id, 'customReadBarcode().callback')
            )
            button.innerHTML = 'readBarcode'
            element.append(input)
            element.append(button)
            element.input = input
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

Файл _CustomReadBarcode.lsf_

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

Чтение штрихкода камерой без callback можно использовать, например, если, например, штрихкод не требуется отображать 
на форме или необходимо получить дополнительные сведения о штрихкоде (формат штрихкода и тип информации).
Пример ниже сохраняет на сервере приложений json-файл, содержащий массив распознанных штрихкодов с дополнительными 
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
        NEWTHREAD MESSAGE 'Файл успешно сохранён'; CONNECTION connection();
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
> Для корректной работы в режиме broadcast у сканера мобильного устройства должен быть включен соответствующий режим 
> и в настройках сканера и приложения должны быть заданы совпадающие имя Intent и имя ресурса, содержащего 
> отсканированный штрихкод.  

Для подписки на широковещательное сообщение сканирования штрихкода используется метод:

`MobileDataTerminal.waitBarcode(<tag>[, <callback>])`

В параметре tag передается значение, которое будет возвращено при обратном вызове вместе с результатом. В частности в
параметр tag можно передавать значение, позволяющее определить, что возвращаемое в обратном вызове значение относится
именно к этому вызову метода (это может быть идентификатор соединения, имя или уникальный идентификатор пользователя,
генерируемое уникальное значение и т.д.)

В параметре `callback` можно указать имя callback-функции js, которая будет вызвана после того, как сканер передал 
широковещательное сообщение о прочитанном штрихкоде. Функция должна принимать на вход два обязательных параметра 
`tag` и `barcode`. При вызове callback-функции в параметре `tag` будет передаваться значение, которое было передано в 
параметре `tag` метода `MobileDataTerminal.waitBarcode`, а в параметре `barcode` - строка с прочитанным штрихкодом.

Если параметр `callback` опущен, то вместо обратного вызова js-функции приложение отправит запрос серверу приложений:

`POST /exec/mobileApiResult?p=<tag>`

где в параметре `p` адресной строки будет передан `tag` метода `MobileDataTerminal.waitBarcode`, а в теле запроса будет
передан json-файл, содержащий массив, состоящий из единственного элемента, содержащего прочитанный штрихкод. При этом в 
заголовке запроса будут переданы параметры Basic-авторизации с логином и паролем пользователя, авторизованного в 
приложении.

> [!WARNING]
> При включённой настройке "Использовать метод postImage" серверу приложений будет отправляться запрос
> `POST /exec/postImage?p=<tag>` содержащий в теле json-массив, состоящий из единственного элемента, содержащего 
> прочитанный штрихкод.
>
> Данная настройка добавлена для временного сохранения обратной совместимости будет удалена в следующих версиях.

Пример файла, передаваемого в теле запроса:

```json
[
  {
    "format": 0,
    "rawValue":"6942103112508",
    "valueType": 0
  }
]
```
Поле `rawValue` содержит строковое представление прочитанного штрихкода, а поля `format` и `valueType` всегда равны 
нулю.

Одномоментно может быть активна только одна подписка. Повторный вызов метода перед новой подпиской отменяет 
предыдущую. Для отмены подписки, на период, когда сканирование штрихкода не ожидается, используется метод:

`MobileDataTerminal.stopWaitingBarcode()`

Подписка на широковещательное сообщение отменяется автоматически после получения штрихкода от сканера.

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

Получение штрихкода от broadcast-сканера без callback можно использовать, например, если, например, штрихкод не 
требуется отображать на форме.
Пример ниже сохраняет на сервере приложений json-файл, содержащий массив распознанных штрихкодов с дополнительными
сведениями о них, не отображая его в интерфейсе:

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
            MESSAGE 'Файл успешно сохранён';
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
запущен мобильный клиент и о настройках самого мобильного клиента.

Файл _mdt.js_

```javascript
function getAppInfo(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    return JSON.parse(MobileDataTerminal.getAppInfo())
}
```

Модуль _AppInfo.lsf_

```Lsf
MODULE AppInfo;

REQUIRE SystemEvents;

appInfo 'Информация приложения' = DATA LOCAL JSON ();

getInfo 'Получить инфо' () {
    INTERNAL CLIENT 'getAppInfo' PARAMS currentConnection() TO appInfo;
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

В результате выполнения действия `getInfo()` в свойство `appInfo()` будет записаны сведения об устройстве, на 
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

Следует отметить, что в `applicationSettings.instanceId` возвращается uuid, который генерируется при первом запуске 
приложения и хранится во внутренней памяти устройства. При полном удалении данных приложения uuid будет сгенерирован 
снова.
