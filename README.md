
# Styx - ИНСТРУКЦИЯ ПО ИНТЕГРАЦИИ

# 1. Styx

## 1.1. Создание документа PKCS#7

Для создание документа [PKCS#7](https://www.rfc-editor.org/rfc/rfc2315) применяется функция [`signMSG`](http://127.0.0.1:6210/crypto/signMSG)

    function sign() {
      var params = {
        //Данные для подписи
        "obj": "Quick fox jumps over the lazy dog",
        //Boolean True или False, если True то это до подпись, если False это обычный первый подпись
        "cms": false,
        //Возможные значения: 'True' - будет создан PKCS#7/CMS документ без вложения исходных данных, 'False' - будет создан PKCS#7/CMS документ с вложением исходных данных
        "detached": false,

        //Идентификатор ключа подписывающего лица (Серийный номер или отпечаток полученный из фукнции [`getCertInfo`])
        "cert_sn": "",
        "cert_thumb": ""
      };
      var msg = JSON.stringify(params);

      $.ajax({
        type: "POST",
        url: "http://localhost:6210/crypto/signMSG",
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

## 1.2. Получении списка сертификатов

Для получении списка сертификатов применяется функция [`getCertInfo`](http://127.0.0.1:6210/crypto/getCertInfo)

    function getInfo() {    
      $.ajax({
        type: "POST",
        url: "http://localhost:6210/crypto/getCertInfo",
        success: function (data) {
          console.log(data);
        }
      });
    }

Ответ
```
    {
      "result":0,
      "status":"success",
      "errorCode":0,
      "comments":null,
      "version":null,
      "signedMsg":null,
      "signedMsgArray":null,
      "tokenStatus":null,
      "macaddresses":["005056C00001","005056C00008","58FB84519FDC"],
      "certInfos":[{
        "issuer":"CN=TEST-CA-CBRU",
        "notafter":"2024-10-10T06:57:32+05:00",
        "notbefore":"2023-10-10T05:00:00+05:00",
        "serialnumber":"1BEEF4155ECD0D02",
        "thumbprint":"F6AE6639F06675DB9622DAA0B62922E5C07CA0EA",
        "subject":"uzINN=123456789, uzPINFL=12345678901234, STREET=Amir Temur 1, SN=test, L=Tashkent, E=info@bank.uz, OU=UZB001, C=UZ, O=Ellips, CN=TEST",
        "pubCert":"MIIDPzCCAiegAw...dHEqWIYWLKzWIg=="
        }],
      "var1":null,
      "var2":null,
      "var3":null
    }
```

HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (0 - Успешно, иначе ошибка)

`message` - если `status` не равно 0, то сообщения об ошибки.

`certInfos` - информация о сертификате клиента.

Смотрите пример https://azizparpiyev.github.io/styx-example/

## 1.3. Запрос на новый сертификат (обновление старого сертификата)

Для получения нового сертификата воспользуйтесь функцией [`getCertificate`](http://127.0.0.1:6210/crypto/getCertificate)

Для получения сертификата нужно выполнить два запроса. Первый содержит номер телефона владельца сертификата в международном формате со знаком “+”, ИНН организации и тип токена. Тип токена “virtual” – для записи сертификата на виртуальный токен, “hard” –для записи на физический носитель (ePass или iKey).
Если запрос возвращает “result”:1, то клиент получит СМС с паролем для второго запроса. Пример СМС сообщения: Пароль для получения сертификата 12345. Никому не сообщайте. (ID организации 123456789)

    --Первый запрос:
    function sign() {
      var params = {
        //Телефон номер клиента
        "phone": "+998XXXXXXXXX",
        //ИНН или ПИНФЛ клиента 
        "inn": "XXXXXXXXX",
        //Пароль OTP для получения сертификата
        "password": ""
        //PIN-код сертификата, который будет использоваться для подписи документов
        "pin": "",
        //Тип токена
        "token_type":"ePass"
      };
      var msg = JSON.stringify(params);

      $.ajax({
        type: "POST",
        url: "http://localhost:6210/crypto/signMSG",
        data: msg,
        success: function (data) {
          console.log(data);
        }
      });
    }

Во втором запросе добавляются ещё два значения: “pin” – пин-код для чтения сертификата, “password” –код, который поступил посредством СМС.
Если запрос возвращает “status”:”success”, то сертификат запишется на токен.

    --Второй запрос:
    function sign() {
      var params = {
        //Телефон номер клиента
        "phone": "+998XXXXXXXXX",
        //ИНН или ПИНФЛ клиента 
        "inn": "XXXXXXXXX",
        //Пароль OTP для получения сертификата
        "password": "XXXXX"
        //PIN-код сертификата, который будет использоваться для подписи документов
        "pin": "",
        //Тип токена
        "token_type":"ePass"
      };
      var msg = JSON.stringify(params);

      $.ajax({
        type: "POST",
        url: "http://localhost:6210/crypto/signMSG",
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

## 2. STYX-SERVER

STYX-SERVER - ПО предназначена для проверки подписи PKCS#7 документа. 

## 2.1. Запуск и настройка

Для запуска требуется:
 - Локальное соединение до сервера `KMS`
 - MIN DOTNET v6.0
 - NGINX

Запуск выполняется командой:

    systemctl start metin

Проверка статуса сервера выполняется командой:

    systemctl status metin

Остановка сервиса выполняется командой:

    systemctl stop metin

После запуска команды в консоле напечатается лог примерно следующего содержания:
```
metin.service - AS server .NET Web API App
  Loaded: loaded (/etc/systemd/system/metin.service; enabled; vendor preset: disabled)
  Active: active (running) since Mon 2024-01-01 12:12:12 +05; 42s ago
Main PID: 817 (dotnet)
  Tasks: 14 (limit 11051)
  Memory: 67.0M
  CGroup: /system.slice/metin.service
          817 /usr/bin/dotnet /home/metin/StyxMetinServer.dll

Jan 01 12:12:12 localhost.localdomain systemd[1]: Started AS server .NET Web API App.
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]: info: Microsoft.Hosting.Lifetime[14]
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]:       Now listening on: http://localhost:5000
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]: info: Microsoft.Hosting.Lifetime[0]
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]:       Application started. Press Ctrl+C to shut down.
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]: info: Microsoft.Hosting.Lifetime[0]
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]:       Hosting environment: Production
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]: info: Microsoft.Hosting.Lifetime[0]
Jan 01 12:12:16 localhost.localdomain dotnet-AS[817]:       Content root path: /home/metin
```
Содержание файла конфигурации `StyxMetinServer.exe.config`:
```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <!-- Connectionstring для подключений в БД KMS -->
    <add key="Conn" value="Server=localhost;Port=3306;charset=utf8;Database=kms_db;User=root;Password=;"/>
  </appSettings>
  <system.webServer>
    <security>
      <requestFiltering>
        <!-- This will handle requests up to 50MB -->
        <requestLimits maxAllowedContentLength="52428800" />
      </requestFiltering>
    </security>
  </system.webServer>
</configuration>
```
*Тестовая конфигурация может отличаться от приведенной выше конфигурации*

Для проверки соединений в БД выполните следующую команду:
```
mysql -u {username} -p'{password}' \
      -h {remote server ip or name} -P {port} \
      -D {DB name}
```
Ответ
```
{
  "serverDateTime": "2022-10-06 16:47:29",
  "yourIP": "127.0.0.1",
  "vpnKeyInfo": {
    "serialNumber": "3",
    "X500Name": "CN=Client",
    "validFrom": "2022-09-24 12:17:24",
    "validTo": "2022-10-24 12:17:24"
  }
}
```

## 2.2. Описание методов STYX-SERVER

STYX-SERVER предоставляет REST-API методы к которым может обращаться Backend приложение или HTML/JavaScript приложение на прямую.

### 2.2.1. `/TransCryptAS/GetVersion`

Метод возвращает версию сервера.

Пример вызова CURL командой:
```
curl -v http://localhost:5000/TransCryptAS/getVersion
```
Ответ
```
{
  0.6.3
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

### 2.2.2. `/TransCryptAS/SignString`

Метод нужен для подписания документов (формирование PKCS#7 контейнера)

Пример вызова CURL командой:

```
curl -X POST -H "Content-Type: application/json" -d '{"msg":"quick fox jumps over the lazy dog", "id":"SAP"}' http://localhost:5000/TransCryptAS/SignString
```
В теле запроса поля `msg` должно содержать строку для подписи, поля это `id` идентификатор сертификата.

Ответ:
```
{
  "status":"00",
  "message":Success",
  "cms":"MIIIHgIBAzCCB9oG...vWMCAgQA"
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | См. сообщение о других ошибках. |
| 02 | Запрошенный сертификат не найден. |
| 03 | Ошибка сертификат недействителен. |

`message` - сообщения об ошибках.

`cms` - PKCS#7 контейнер (подписанный документ).


### 2.2.3. `/TransCryptAS/GetCmsContent`

Метод возвращает исходный документ.

Пример вызова CURL командой:
```
curl -X POST -H "Content-Type: application/json" -d '{"cms":"MIIEiQYJKoZIh...5sZMuVZ6d1IjYg==","codepage":1251}' http://localhost:5000/TransCryptAS/GetCmsContent
```
Тело запроса поля `cms` должно содержать Base64-закодированный PKCS#7 документ. Поля codepage - кодировка исходного документа, если не UTF8. 

Ответ:
```
{
  "cms":null,
  "status":"00",
  "message":"Success",
  "content":"test content",
  "serialnumber":null,
  "certInfos":null,
  "identifier":"12345678901234",
  "orgunit":"test OU field",
  "signTime":null,
  "revokeStatus":false,
  "verify":false
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | incorrect base64 string. |

`message` - сообщения об ошибках.

`content` - исходный документ.

`identifier` - ПИНФЛ.

`orgunit` - Поля OU.

### 2.2.4. `/TransCryptAS/GetCmsUserId`

Метод возвращает список идентификаторов всех пользователей документа(все кто подписывал документ) PKCS#7.

Пример вызова CURL командой:
```
curl -X POST -H "Content-Type: application/json" -d '{"cms":"MIIEiQYJKoZIh...5sZMuVZ6d1IjYg=="}' http://localhost:5000/TransCryptAS/GetCmsUserId
```
Тело запроса должно содержать Base64-закодированный PKCS#7 документ.

Ответ:
```
{
  "cms":null,
  "status":"00",
  "message":"Success",
  "content":null,
  "serialnumber":["1B1ACF4D9FC5C13A"],
  "certInfos":null,
  "identifier":"12345678901234",
  "orgunit":"test OU field",
  "signTime":null,
  "revokeStatus":false,
  "verify":false
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | incorrect base64 string. |

`message` - сообщения об ошибках.

`serialnumber` - Серийный номер сертификата с которым был подписан документ.

`identifier` - ПИНФЛ.

`orgunit` - Поля OU.

### 2.2.5. `/TransCryptAS/GetCmsUserIdAndTime`

Метод возвращает список идентификаторов всех пользователей документа(все кто подписывал документ) и время подписи PKCS#7.

Пример вызова CURL командой:
```
curl -X POST -H "Content-Type: application/json" -d '{"cms":"MIIEiQYJKoZIh...5sZMuVZ6d1IjYg=="}' http://localhost:5000/TransCryptAS/GetCmsUserId
```
Тело запроса должно содержать Base64-закодированный PKCS#7 документ.

Ответ:
```
{
  "cms":null,
  "status":"00",
  "message":"Success",
  "content":null,
  "serialnumber":["1B1ACF4D9FC5C13A;6/7/2023 7:15:02 AM"],
  "certInfos":null,
  "identifier":"12345678901234",
  "orgunit":"test OU field",
  "signTime":null,
  "revokeStatus":false,
  "verify":false
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | incorrect base64 string. |

`message` - сообщения об ошибках.

`serialnumber` - Серийный номер сертификата с которым был подписан документ.

`identifier` - ПИНФЛ.

`orgunit` - Поля OU.

### 2.2.6. `/TransCryptAS/VerifyAndGetCmsContent`

Метод проверяет подпись, сертификат и возвращает исходный документ.

Пример вызова CURL командой:
```
curl -X POST -H "Content-Type: application/json" -d '{"cms":"MIIEiQYJKoZIh...5sZMuVZ6d1IjYg==","codepage":1251}' http://localhost:5000/TransCryptAS/VerifyAndGetCmsContent
```
Тело запроса поля `cms` должно содержать Base64-закодированный PKCS#7 документ. Поля codepage - кодировка исходного документа, если не UTF8. 

Ответ:
```
{
  "cms":null,
  "status":"00",
  "message":"Success",
  "content":"test content",
  "serialnumber":["1B1ACF4D9FC5C13A"],
  "certInfos":[{
    "status":"00",
    "message":"Success",
    "infos":[{
      "fio":"Test FIO",
      "pinfl":"12345678901234",
      "company":"Test company",
      "thumbprint":"B89BE9FE4CEA873C73ADCFDB3C13E48BC61CF088",
      "serialnumber":"1B1ACF4D9FC5C13A"
      }],
      "thumbprintCurrent":"B89BE9FE4CEA873C73ADCFDB3C13E48BC61CF088",
      "fulldata":"notafter:3/2/2025 12:50:41 PM notbefore:5/30/2023 5:00:00 AM serialnumber:1B1ACF4D9FC5C13A thumbprint:B89BE9FE4CEA873C73ADCFDB3C13E48BC61CF088 subject:CN=Test FIO, L=Tashkent, S=Tashkent, C=UZ, STREET=Amir Temur 1, E=info@bank.uz, O=Test Org, OU=Test OrgUnit, uzINN=123456789, uzPINFL=12345678901234"
    }],
    "identifier":"12345678901234",
    "orgunit":"Test OrgUnit",
    "signTime":"6/7/2023 7:15:02 AM",
    "revokeStatus":false,
    "verify":true
  }
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | incorrect base64 string. |

`message` - сообщения об ошибках.

`content` - исходный документ.

`serialnumber` - серийный номер сертификата которым был подписан документ.

`certInfos` - более подробная информация о сертификате.

`identifier` - ПИНФЛ.

`orgunit` - поля OU.

`signTime` - время подписи.

`revokeStatus` - статус отзыва. true если сертификат отозван, false если сертификат действителен.

`verify` - статус проверки подписи, true если подпись действителен, false если не действителен.

### 2.2.7. `/TransCryptAS/VerifyCMS`

Метод проверяет подпись.

Пример вызова CURL командой:
```
curl -X POST -H "Content-Type: application/json" -d '{"cms":"MIIEiQYJKoZIh...5sZMuVZ6d1IjYg=="}' http://localhost:5000/TransCryptAS/VerifyCMS
```
Тело запроса поля `cms` должно содержать Base64-закодированный PKCS#7 документ.

Ответ:
```
{
  "cms":null,
  "status":"00",
  "message":"Success",
  "content":null,
  "serialnumber":null,
  "certInfos":null,
  "identifier":null,
  "orgunit":null,
  "signTime":null,
  "revokeStatus":false,
  "verify":true
}
```
HTTP 503 - Посмотрите лог STYX-SERVER.

HTTP 400 - означает что есть ошибка в параметрах запроса. Посмотрите лог STYX-SERVER.

HTTP 200 - означает успешное выполнение HTTP запроса

`status` - код состояния (00 - Успешно, иначе ошибка)

| status | Описание |
|--|--|
| 00 | Успешно |
| 01 | incorrect base64 string. |

`message` - сообщения об ошибках.

`verify` - статус проверки подписи, true если подпись действителен, false если не действителен.

# 3. Styx Metin SDK

## Описание взаимодействия с библиотека Metin Android SDK.
