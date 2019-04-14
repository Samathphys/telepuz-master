# doka
## Доступ к дневнику
### Шаг 1. useType
GET на `https://app.eschool.center/ec-server/esia/useType`<br>
headers:`Cookie: site_ver=app; _pk_ses.1.81ed=*; _pk_id.1.81ed=de563a6425e21a4f.1553009060.16.1554146944.1554139340.`
В Response Cookie (или header Set-Cookie) прилетает route и JSESSIONID.
### Шаг 2. login
POST на `https://app.eschool.center/ec-server/login`<br>
В headers прилагаешь только что полученные JSESSIONID и route, а также те же куки из useType.<br>
В запрос пишешь `username=LOGIN&password=HASH`<br>
В ответ приходит, собственно кука JSESSIONID, которую вместе с route и теми ты прикладываешь везде.<br>
### Доступ к расписанию
GET на `https://app.eschool.center/ec-server/student/diary?userId=`%USER_ID%`&d1=1554066000000&d2=1554645636682`<br>
d1 = дата начала периода (в миллисекундах), d2 = дата конца периода <br>
Единственный рабочий способ перевести дату в миллисекунды:
```
DateTimeFormatter dateFormatter
                    = DateTimeFormatter.ofPattern("d-M-uuuu", Locale.ENGLISH);
            String stringDate = "1-4-2019";
            long millis = LocalDate.parse(stringDate, dateFormatter)
                    .atStartOfDay(ZoneId.systemDefault())
                    .toInstant()
                    .toEpochMilli();
            System.out.println(millis);
```
в куки прилагаешь<br>
%COOKIE%`;site_ver=app;route=`%ROUTE%`;_pk_id.1.81ed=de563a6425e21a4f.1553009060.16.1554146944.1554139340.`<br>
В ответ прилетает много мусора в формате
``` javascript
{
  <...>
  dateBegin_s: "2019-4-1"
  dateEnd_s: "2019-4-7"
  lesson: // массив уроков в формате
  [{
    date_d: "2019-03-31T21:00:00Z"
    numInDay: 1
    part: [{<...>}] // дз
    subject: "решение задач"
    teacher: {
      <...>
      factTeacherIN: "Селякова М.В."
    }
    unit: {
      <...>
      name: "Физика"
    }
  }<...>]
}
```
## Доступ к Firebase
1. POST на `https://fcm.googleapis.com/fcm/send`
``` javascript
Headers:
Authorization: key=%KEY%
Content-Type: application/json // обязательно, а то ругнется
{
  "to": "id1" // id юзера в firebase
  "data": { // дублируешь notification
    "title": "TITLE"
    "body": "test text"
  }
  "notification": {
    "title": "TITLE"
    "body": "test text"
  }
}
```
## Формат взаимодействия с сервером
### Запрос о новом юзере
POST на `https://still-cove-90434.herokuapp.com/login`
``` javascript
{
  "login": // login
  "password": // hash
  "id": // firebase_id
} 
// все string
```
### Уведомление о новой оценке
``` javascript
data: {
  "type": "mark"
  "val": 2 // int
  "coef": 0.5 // double
  "unitId": 1416 // int
}
```
### Уведомление о новом сообщении
``` javascript
data: {
  "type": "msg"
  "text": "jqbnkjqekvfnnasffgknm"
  "senderId": 69908
  "senderFio": "Гуров Матвей Вадимович"
  "threadId": 677770
  "date": 1555008702000
  // если есть тема диалога, то еще присылай
  "subject": "ёу"
}
```
