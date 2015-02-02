# Партнерский Програмный Интерфейс Adaperio.ru

## Для работы с Adaperio через партнерский API нужно:

     1) Обратиться на support@adaperio.ru с просьбой выдать логин/пароль для партерской программы.

     2) Обговорить модель покупки отчетов (предоплата или по факту).

     3) Запустить тесты и убедиться, что они завершились без ошибок.

     4) Встроить API в свою систему.

## Для тестирования нужно:

     1) Добавить полученный от нас логин/пароль в файл test/adaperio_api_tst.js

     2) Установить нужные для тестирования пакеты: 
     npm install https fs crypto assert

     3) Установить нужный для тестирования фреймворк: 
     npm install -g mocha

     4) Запустить все тесты: 
     mocha reporter --spec

В результате выполнения тестов, вы произведете тестовую покупку и получите ссылку на отчет об автомобиле **а999му99**

## Описание API
Для работы используются REST-подобное API с JSON в качестве передаваемых данных. 

### 1. Получить данные о наличии информации об автомобиле

```javascript
GET /v1/data_for_cars/:num

Выполняется следующее: 
    1. Проверяется наличие данных о данном автомобиле;
    2. Возвращается статус 200 + тело в формате JSON.

Возвращается: 200 + JSON если какие-либо данные были найдены; 
              404 если ничего не найдено.
```

Тело ответа:
```javascript
[{
     "num":"а001кк12",
     "vin":"XTA2*********1931",
     "carModel":"ВАЗ 21102 ЛЕГКОВОЙ",
     "year":"2000"

     // ДТП (у автомобилей Мск региона ДТП может быть найдено после покупки отчета) 
     "accidentFound":false,

     // Найдены ли данные с аукционов
     “auctionsFound”: true,

     // Найдены ли фотографии ДТП/повреждений
     “picsFound":false,

     // Использование в качестве такси
     "taxiFound":false,

     // Найден ли пробег
     "milleageFound":true,

     // Фотографии автомобиля
     "autoNomerPics":[],

     // Найдены ли сведения из таможни
     "customsFound":false,

     // В розыске ли автомобиль
     // (только при запросе с gibdd_check==1)
     "gibddWanted":false,

     // Есть ли ограничения
     // (только при запросе с gibdd_check==1)
     "gibddRestricted":false,

     // Есть ли информация о комплектации
     "equipInfoFound":false 

     ...
}
]
```

### 2. Создать новый заказ

```javascript
POST /v1/partner_orders

Выполняется следующее:
    1. Создается новый заказ с уникальным OrderId;
    2. Состояние заказа устанавливается в 'ожидает покупки'.
    3. Возвращается статус 200 + значение OrderID в теле ответа.

Возвращается: 200 + значение OrderID в теле ответа;
              404 если произошла ошибка.

HTTP заголовки запроса:
         Content-Type: application/json
         Content-Length: <длина тела в байтах>
```

Тело запроса должно иметь следующий вид: 
     {"num":"а001кк12”,”login”:”your_login_here”,”email”:”your_email_here@mail.com”}

     num - гос.номер автомобиля в кодировке utf8 (кириллица); 
     login - логин, который вы получили от нас;
     email - ваш e-mail (не клиента).

Ответ содержит идентификатор созданного отчета: 
```javascript
{id:”3269587”}
```

### 3. Завершить заказ и получить ссылку на отчет

```javascript
POST /v1/buy_order/:orderId

Выполняется следующее:
    1. Проверяется подпись.
    2. Находится заказ по его номеру.
    3. Состояние заказа изменяется: 'ожидает покупки' -> 'завершен'.
    4. Возвращается статус 200 + тело со ссылкой на отчет.

Возвращается: 200 + тело cо ссылкой на отчет;
              404 если произошла ошибка.

HTTP заголовки запроса:
         Content-Type: application/json
         Content-Length: <длина тела в байтах>
```

Тело запроса должно иметь следующий вид:
     {"OutSum":"100.000000”,”SignatureValue":"f2c60992ac7d90a906fcb36761da6294"}
     OutSum - сумма в рублях;
     SignatureValue - специальная подпись (см.далее).

В случае успеха - тело ответа будет содержать JSON вида:
```javascript
{ link: 'http://www.adaperio.ru/engine.html#/success?InvId=3269587&OutSum=100.000000&SignatureValue=1813ba713a5ee12abf0b7bb3e669d072' }
```

### Подпись Signature для метода #3 генерируется следующим образом:

```javascript
function generateSignature(orderId){
     // например: ”100.000000:3269587:ABCDEFG123QWERTY_SUPER_SECRET”

     var s = '' + SUM + ':' + orderId + ':' + PASSWORD;
     var hash = crypto.createHash('md5').update(s).digest("hex");
     return hash;
}
```

**link** и будет являться конечной ссылкой, которую можно передать пользователю. SignatureValue в ссылке не имеет ничего общего с секретной подписью из метода 2. 
Время жизни такой ссылки - месяц. 

ВНИМАНИЕ: вызов этого метода возможен только из защищенного модуля (вашего backend’а), так как ни сумма, ни секретный пароль не должны быть доступны простому конечному пользователю. Если вызывать этот метод (даже с учетом https) из незащищенного приложения - пользователь сможет дизассемблировать его или просто перехватить пароль в памяти.


