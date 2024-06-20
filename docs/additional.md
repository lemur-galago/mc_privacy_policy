# Мобильный клиент lsFusion

## Дополнительный фунционал клиента

Начиная с версии 2.0.0006 в тестовом режиме добавлена возможность использования дополнительного
функционала клиента из кода lsFusion. В качестве теста добавлена возможности:
- показ Toast-уведомленй на устройстве клиента
- использование камеры устройства для одиночных снимков (даже при подключении к серверу по протоколу
http)

## Общие сведения

Для использования дополнительного функционала при открытии web-клиента lsFusion в приложении
"Мобильный клиент lsFusion" добавляется js-интерфейс `MobileDataTerminal`, позволяющий web-клиенту
взаимодействовать с приложением. В частности, убедиться в том, что web-клиент запущен в приложении
можно по существованию MobileDataTerminal:

Файл _mdt.js_ разместить в файле ресурсов проекта 
```javascript
function checkMobileDataTerminal() {
    return typeof MobileDataTerminal !== 'undefined'
}
```

Модуль _Main.lsf_
```
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
```
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

Аналогичным образом можно вызвать Toast в другом соединении. Пример ниже показывает Toast на всех
активных устройствах, подключенных с помощью приложения:

```
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

Через интерфейс `MobileDataTerminal` на Android-устройстве можно открыть фрагмент "Камера",
позволяющий сделать снимок, после чего вызвать или эндпоинт сервера lsFusion, или js-callback.

Для вызова фрагмента камеры используется метод:

`MobileDataTerminal.captureImage(<tag>[, <callback>])`

В параметре `tag` передается зачение, которое будет возвращено при обратном вызове вместе с
результатом. В частности в параметр `tag` можно передавать значение, позволяющее определить, что
возвращаемое в обратном вызове значение относится именно к этому вызову метода (это может быть
идентификатор соединения, имя или уникальный идентификатор пользователя, генерируемое уникальное
значение и т.д.)

В параметре `callback` можно указать имя callback-функции js, которая будет вызвана после того, как
в пользователь в приложении сделал снимок камерой. Функция должна принимать на вход два обязательных
параметра `tag` и `encodedImage`. В при вызове callback-функции параметре `tag` будет передаваться
значение, которое было передано в параметре `tag` метода `MobileDataTerminal.captureImage`, а в
параметре `encodedImage` - base64-строка с jpeg-изображением.

Если параметр `callback` опущен, то вместо обратного вызова js-функции приложение отправит запрос
серверу приложений:

`POST /exec/postImage?p=<tag>`

где в параметре `p` адресной строки будет передан `tag` метода `MobileDataTerminal.captureImage`, а
в теле запроса будет передан файл изображения. При этом в заголовке запроса будут переданы параметры
Basic-авторизации с логином и паролем пользователя, авторизированного в приложении.

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
```
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

Обращение к камере без callback можно использовать, например, если изображение не требуется
отображать на форме:
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
```
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
