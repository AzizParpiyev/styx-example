# 1. Styx

## 1.1. Создание документа PKCS#7 HashKey

Для создание документа [PKCS#7](https://www.rfc-editor.org/rfc/rfc2315) применяется функция [`signMSG`](http://127.0.0.1:6210/crypto/HashKey)

    function HashKey() {
      var params = {
        //Данные для подписи
        "obj": "Quick fox jumps over the lazy dog",
        //string ИНН клиента (юр лицо). Не обязательно если передать ПИНФЛ.
        "inn": "123456789",
        //ПИНФЛ клиента. Не обязательно если передать ИНН.
        "pinfl": "12345678901234",
      };
      var msg = JSON.stringify(params);

      $.ajax({
        type: "POST",
        url: "http://localhost:6210/crypto/HashKey",
        data: msg,
        success: function (data) {
          console.log(data);
        }
      });
    }

HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (0 - Успешно, иначе ошибка)

`message` - если `status` не равно 0, то сообщения об ошибки.

Смотрите пример https://azizparpiyev.github.io/styx-example/
