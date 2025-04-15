# Мобильный клиент lsFusion

## Дополнительный функционал клиента

Начиная с версии 2.0.0006 в тестовом режиме добавлена возможность использования дополнительного функционала клиента из
кода lsFusion. В качестве теста добавлена возможности:

- показ Toast-уведомленй на устройстве клиента
- использование камеры устройства для одиночных снимков (даже при подключении к серверу по протоколу http)

## Общие сведения

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

Модуль _Main.lsf_

```Lsf
MODULE Main;

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

onWebClientStarted() + {
    INTERNAL CLIENT 'mdt.js';
    SHOW mdtDemo;
}
```

## Toast

Через интерфейс `MobileDataTerminal` можно вызвать стандартный Android-Toast с сообщением:

Файл _mdt.js_

```javascript
function showToast(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.showToast(text)
    return true
}
```

Модуль _Main.lsf_

```Lsf
MODULE Main;

REQUIRE SystemEvents;

showToast 'Показать Toast' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'showToast' PARAMS 'Hello world!!!' TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

FORM mdtDemo 'Demo'
    PROPERTIES showToast()
;

onWebClientStarted() + {
    INTERNAL CLIENT 'mdt.js';
    SHOW mdtDemo;
}
```

Аналогичным образом можно вызвать Toast в другом соединении. Пример ниже показывает Toast на всех активных устройствах,
подключенных с помощью приложения:

```Lsf
MODULE Main;

REQUIRE SystemEvents;

showToastForAll 'Показать Toast всем' () {
    FOR connectionStatus(Connection c) = ConnectionStatus.connectedConnection AND NOT c = currentConnection() DO
        NEWTHREAD
            INTERNAL CLIENT 'showToast' PARAMS 'Hello world!!!';
            CONNECTION c;
}

FORM mdtDemo 'Demo'
    PROPERTIES showToastForAll()
;

onWebClientStarted() + {
    INTERNAL CLIENT 'mdt.js';
    SHOW mdtDemo;
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
приложении сделал снимок камерой. Функция должна принимать на вход два обязательных параметра `tag` и `encodedImage`. В
при вызове callback-функции параметре `tag` будет передаваться значение, которое было передано в параметре `tag`
метода `MobileDataTerminal.captureImage`, а в параметре `encodedImage` - base64-строка с jpeg-изображением.

Если параметр `callback` опущен, то вместо обратного вызова js-функции приложение отправит запрос серверу приложений:

`POST /exec/postImage?p=<tag>`

где в параметре `p` адресной строки будет передан `tag` метода `MobileDataTerminal.captureImage`, а в теле запроса будет
передан файл изображения. При этом в заголовке запроса будут переданы параметры Basic-авторизации с логином и паролем
пользователя, авторизированного в приложении.

Обращение к камере с callback можно использовать, например, в custom-представлении lsFusion.

Файл _mdt.js_

```javascript
function captureCameraImage() {
    return {
        render: element => {
            if (typeof MobileDataTerminal === 'undefined') return
            element.id = Date.now().toString()
            const button = document.createElement('button')
            button.addEventListener(
                'click',
                (_) =>
                    MobileDataTerminal.captureImage(element.id, 'captureCameraImage().callback')
            )
            button.innerHTML = 'captureImage'
            element.append(button)
        },
        update: (element, controller, value) => {
            element.controller = controller
        },
        callback: (id, image) => {
            const element = document.getElementById(id)
            if (element) {
                element.controller.changeValue(image)
            }
        }
    }
}
```

Модуль _Main.lsf_

```Lsf
MODULE Main;

REQUIRE SystemEvents, Utils;

image = DATA LOCAL IMAGEFILE ();

FORM mdtDemo 'Demo'
    PROPERTIES image '' = image() READONLY, button '' = image() CUSTOM 'captureCameraImage'
    ON CHANGE {
        INPUT image = TEXT DO {
            IF image THEN image() <- decode(image, 'base64');
        }
    }
;

DESIGN mdtDemo {
    PROPERTY (image) { size = (240, 320); }
}

onWebClientStarted() + {
    INTERNAL CLIENT WAIT 'mdt.js';
    SHOW mdtDemo;
}
```

Обращение к камере без callback можно использовать, например, если изображение не требуется отображать на форме:
Пример ниже сохраняет изображение на сервере приложений, не отображая его в интефейсе:

Файл _mdt.js_

```javascript
function showToast(text) {
    MobileDataTerminal.showToast(text)
}

function captureImage(tag) {
    if (typeof MobileDataTerminal === 'undefined') return
    MobileDataTerminal.captureImage(tag)
    return true
}
```

Модуль _Main.lsf_

```Lsf
MODULE Main;

REQUIRE SystemEvents;

image = DATA LOCAL IMAGEFILE ();

captureImage 'Сделать снимок' () {
    LOCAL flag = BOOLEAN ();
    INTERNAL CLIENT 'captureImage' PARAMS currentConnection() TO flag;
    IF NOT flag() THEN MESSAGE 'Это не мобильный клиент';
}

showToast(LONG tag, STRING text) {
    NEWTHREAD INTERNAL CLIENT 'showToast' PARAMS 'Файл успешно сохранен';
        CONNECTION [GROUP MAX Connection c AS Connection BY LONG(c)](tag);
}

postImage(LONG tag, IMAGEFILE image) {
    TRY {
        WRITE image TO '/tmp/image';
        showToast(tag, 'Изображение успешно сохранено');
    } CATCH {
        showToast(tag, 'Ошибка сохранения изображения');
    }
}@@api;

FORM mdtDemo 'Demo'
    PROPERTIES captureImage()
;

onWebClientStarted() + {
    INTERNAL CLIENT 'mdt.js';
    SHOW mdtDemo;
}
```

## Получение сведений об устройстве и приложении

Начиная 2.0.0010 через интерфейс `MobileDataTerminal` можно получить сведения об устройстве, на котором запущен 
мобильный клиент и о настройках самого мобильного клиента.

Файл _mdt.js_

```javascript
function getAppInfo(text) {
    if (typeof MobileDataTerminal === 'undefined') return
    return JSON.parse(MobileDataTerminal.getAppInfo())
}
```

Модуль _Main.lsf_

```Lsf
MODULE Main;

REQUIRE SystemEvents;

appInfo 'Информация приложения' = DATA LOCAL JSON ();

getInfo 'Получить инфо' () {
    INTERNAL CLIENT 'getAppInfo' PARAMS currentConnection() TO appInfo;
    IF NOT appInfo() THEN MESSAGE 'Это не мобильный клиент';
}

FORM mdtDemo 'Demo'
    PROPERTIES () appInfo, getInfo
;

onWebClientStarted() + {
    INTERNAL CLIENT 'mdt.js';
    SHOW mdtDemo;
}
```

В результате выполнения действия `getInfo()` в свойство `appInfo()` будет записаны сведения об устройстве, на 
котором запущен мобильный клиент, а также о настройках и версии мобильного клиента:

```json
{
  "applicationSettings": {
    "hideKeyboardOnLogin": false,
    "hideLogo": false,
    "instanceId": "901da373-6400-42fe-ac70-1f572fc4daf9",
    "lockOrientation": false,
    "loginOnEnter": false
  },
  "build": {
    "board": "goldfish_x86_64",
    "brand": "google",
    "device": "emu64xa",
    "display": "sdk_gphone64_x86_64-userdebug 16 BP22.250221.010 13193326 dev-keys",
    "host": "r-456ae1c9fa6a8c5c-gd0k",
    "id": "BP22.250221.010",
    "manufacturer": "Google",
    "model": "sdk_gphone64_x86_64",
    "version": {
      "release": "16",
      "sdkInt": 36,
      "securityPatch": "2025-03-05"
    }
  },
  "config": {
    "applicationId": "org.lsfusion.mobileclient",
    "versionCode": 34,
    "versionName": "2.0.0012"
  },
  "environment": {
    "webView": {
      "lastUpdateTime": 1744316380215,
      "versionCode": 661308838,
      "versionName": "128.0.6613.88"
    }
  }
}
```

Следует отметить, что в `applicationSettings.instanceId` возвращается uuid, который генерируется при первом запуске 
приложения, и хранится во внутренней памяти устройства. При полном удалении данных приложения uuid будет 
сгенерирован снова.
