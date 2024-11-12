# Смартпроцессы

1. **Расчет стоимости доставки**
    - Описание: Реализовать смарт-процесс для расчета стоимости доставки.
    - Взаимодействие:
        - Передача параметров отправления и получения груза.
        - Указание характеристик груза (объем, вес, класс опасности и т.д.).
        - Получение ответа с расчетом стоимости, включающим все возможные тарифы и опции.

2. **Получение печатных форм**
    - Описание: Получение печатных документов по заказу, таких как товарно-транспортные накладные, квитанции и т.д.
    - Взаимодействие:
        - Запрос к API с идентификатором заказа.
        - Указание формата документа (PDF, DOCX и др.).
        - Получение ссылки для скачивания документа или содержимого в виде base64.

3. **История статуса заказа**
    - Описание: Отслеживание истории изменения статусов заказа для мониторинга этапов выполнения.
    - Взаимодействие:
        - Запрос истории статусов с указанием `orderID`.
        - Получение данных о смене статусов с временными метками.
        - Поддержка уведомлений об изменении статуса, при необходимости (например, через WebSocket или push-уведомления).

# Бизнес процессы

1. **Оформление доставки сборного груза**

### Поля, передаваемые 

| Параметр                            | Обязательный | Тип данных    | Описание                                                                                                       |
|-------------------------------------|--------------|---------------|----------------------------------------------------------------------------------------------------------------|
| `dateSend`                          | Нет          | string        | Предпочтительная дата отправки груза, если не передается то подбираю по своей логике                           |
| `inOrder`                           | Да           | boolean       | Для мультизаявки - `false`. True - заказ без мультизаявки                                                      |
| `deliveryType`                      | Да           | string        | Вид доставки: `auto`, `express`, `letter`, `avia`, `small`.                                                    |
| `accompanyingDocuments`             | Нет          | array         | Сопроводительные документы. "отправить", "прислать"                                                            |
| `delivery.smsback`                  | Нет          | string        | Телефон для SMS-уведомлений. Формат: 7XXXXXXXXXX.                                                              |
| `delivery.comment`                  | Нет          | string        | Комментарий к заказу (максимум 500 символов).                                                                  |
| Информация о получателе             | Да           | структура     | структура ниже задается как `arrival`                                                                          |
| `arrival.variant`                   | Да           | string        | Вариант получения: `terminal` или `address`.                                                                   |
| `arrival.address`                   | Да           | string        | Адрес либо терминала, либо адресса.                                                                            |
| `arrival.timeEnd`                   | Да           | string        | Время окончания приема доставки.                                                                               |
| `arrival.timeStart`                 | Да           | string        | Время начала приема доставки.                                                                                  |
| `arrival.timeBreakStart`            | Нет          | string        | Время начала перерыва.                                                                                         |
| `arrival.timeBreakEnd`              | Нет          | string        | Время окончания перерыва.                                                                                      |
| `arrival.handling`                  | Нет          | object        | Погрузо-разгрузочные работы на адресе получения. Здесь расстояние которое нужно нести, есть ли лифт какой этаж |
| `arrival.requirements`              | Нет          | array         | Дополнительные требования к транспорту при получении.                                                          |
| `arrival.payer`                     | Нет          | string        | Плательщик за получение: `sender`, `receiver` или `third`. (отправитель, получатель или третье лицо)           |
| Участники перевозки                 | Да           | структура     | структура ниже задается как `members`                                                                          |
| `members.sender.contactPersons`     | Нет          | array(string) | Контактные лица отправителя. Фио                                                                               |
| `members.sender.phoneNumbers`       | Нет          | array(string) | Телефоны отправителя. номера телефона(<7<номер телефона>(<добавочный номер>)>)                                 |
| `members.receiver.counteragent.inn` | Нет          | string        | ИНН получателя.                                                                                                |
| `members.receiver.contactPersons`   | Нет          | array(string) | Контактные лица получателя. Фио                                                                                |
| `members.receiver.phoneNumbers`     | Нет          | array         | Телефоны получателя. номера телефона(<7<номер телефона>(<добавочный номер>)>)                                  |
| `members.thirdcounteragent.inn`     | Нет          | string        | ИНН третьего лица.                                                                                             |
| `dataForReceipt.phone`              | Нет          | string        | Номер телефона для отправки чека  плательщику                                                                  |
| `dataForReceipt.email`              | Нет          | string        | Адерс почты для отправки чека   плательшику                                                                    |
| Информация о грузе                  | Да           | структура     | структура ниже задается как `cargo`                                                                            |
| `cargo.quantity`                    | Да           | integer       | Количество мест.                                                                                               |
| `cargo.length`                      | Да           | number        | Длина самого длинного грузового места в метрах.                                                                |
| `cargo.width`                       | Да           | number        | Ширина самого широкого грузового места в метрах.                                                               |
| `cargo.height`                      | Да           | number        | Высота самого высокого грузового места в метрах.                                                               |
| `cargo.totalVolume`                 | Да           | number        | Общий объем груза в кубических метрах.                                                                         |
| `cargo.Weight`                      | Да           | number        | Вес самого тяжелого грузового места в килограммах.                                                             |
| `cargo.totalWeight`                 | Да           | number        | Общий вес груза в килограммах.                                                                                 |
| `cargo.hazardClass`                 | Нет          | float         | Класс опасности груза.                                                                                         |
| `cargo.CargoInfo`                   | Да           | string        | Характер груза (что перевозится)                                                                               |
| Информация об оплате                | Да           | структура     | структура ниже задается как `payment`                                                                          |
| `payment.type`                      | Да           | string        | Тип оплаты: `cash` или `noncash`.                                                                              |
| `payment.primaryPayer`              | Да           | string        | Основной плательщик: `sender`, `receiver` или `third`.                                                         |
| `payment.cashOnDelivery`            | Нет          | array         | Параметры наложенного платежа.                                                                                 |
### Поля, возвращаемые в ответе


#### Успешный ответ
| Параметр        | Тип данных | Описание                                           |
|-----------------|------------|----------------------------------------------------|
| status          | string     | статус заказа                                      |
| derival.address | string     | Найденный адреса отправитея                        |
| arrival.address | string     | Внутренний идентификатор заказа в системе «Деловые Линии». |
| request.ID      | string     | Номер заказа, если оформлена мультиотправка.       |
| orderId         | string     | Номер заказа, для отслеживания статуса             |
| orderedAt       | string     | Дата забора груза "2020-10-01 08:36:27"            |
| docId           | string     | номер накладной                                    |

#### Пример успешного ответа
