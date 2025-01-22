# <a name="up" />Проект Яндекс.Прилавка (API)

Яндекс Прилавок - это сервис, который предоставляет предпринимателям и компаниям возможность создавать собственные виртуальные магазины в онлайн-пространстве. Этот инструмент облегчает процесс создания и настройки интернет-магазинов, управления продажами и обработки заказов. Он также обеспечивает интеграцию с онлайн-платежами, что делает его полезным для развития электронной коммерции и совершения успешных онлайн-сделок.

## Содержание
- [Задачи тестировщика](#задачи-тестировщика)
- [Требования к проекту](#требования-к-проекту)
- [Инструменты](#инструменты)
- [Проектирование тестовой документации](#проектирование-тестовой-документации)
- [Выполнение тестов](#выполнение-тестов)

## Задачи тестировщика

<details>
<summary> Задачи </summary> 
  
1. Проанализировать требования к новой функциональности бэкенда Яндекс.Прилавка. 
2. Изучить документацию к API в Apidoc. 
3. Спроектировать тесты в виде чек-листа, чтобы покрыть функциональность:
- Работа с наборами: возможность добавлять продукты в набор — ручка POST /api/v1/kits/{id}/products.
- Работа с курьерами: возможность проверить, есть ли доставка курьерской службой «Привезём быстро» и сколько она стоит. Ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml. 
- Работа с корзиной:
  
Ручка GET /api/v1/orders/:id - возможность получить список продуктов, которые добавили в корзину; 

Ручка PUT /api/v1/orders/:id - возможность добавлять продукты в корзину;

Ручка DELETE/api/v1/orders/:id -  возможность удалять корзину.

4. Протестировать API через Postman и завести баг-репорты.

  ***

</details>

## Требования по проекту

<details>
<summary> Требования к бэкенду приложения Яндекс.Прилавка </summary>

### Описание общей логики
#### Авторизация и данные для заказа
Пользователь может зарегистрироваться. Если пользователь не зарегистрировался, то форма заполнения: имя, e-mail, телефон, адрес, комментарий — появляется, когда пользователь уже сформировал корзину и хочет оформить заказ. Пользователь не может сделать заказ, если не ввёл обязательные поля. Если пользователь зарегистрировался, то ему не нужно вновь вводить данные, однако он может их изменить. Пользователь может оформить несколько заказов.
Для заказа нужно ввести:  
- имя;  
- телефон;  
- адрес;  
- e-mail (необязательно);  
- комментарий к заказу (необязательно).

<img width="649" alt="Снимок экрана 2025-01-22 в 20 33 59" src="https://github.com/user-attachments/assets/557cb6a3-5258-48e7-a379-35411883b45e" />

#### Главное меню заказа
На выбор даётся 3 карточки:  
- «Под ситуацию» (под кино и сериалы, на пикник, вкусы Парижа);  
- «Приготовь блюдо» (сырники, борщ, карбонара, штрудель);  
- «Создать свой набор» (пользователь сам называет и добавляет туда продукты).  
Переходишь в карточку — видишь варианты наборов.   
Переходишь в набор — видишь перечень возможных продуктов. Каждый продукт относится к определённой категории (например, «Напитки»). В перечне пользователь видит название продукта, его массу, цену. Когда клиент нажимает на продукт, ему даётся возможность выбрать количество продуктов. При этом появляется кнопка «Корзина», которая отображает сумму выбранных товаров и время доставки.
При нажатии на кнопку пользователь может посмотреть свою корзину.

#### Корзина
Отображает наименование продукта, его количество, цену для этого продукта с учётом количества, итог. Если доставка платная, то отображает сумму доставки и итоговую сумму заказа. Пользователь может удалить корзину, добавить новые продукты, убрать выбранные продукты.
Создание корзины:  
- При создании корзины должна быть возможность указать время доставки продуктов  
- При создании корзины проверять, что все службы доставки могут обработать заказ в указанное время  
- Если время доставки не указано - брать, как текущее время с сервера.  
При удалении и просмотре корзины время доставки не учитывается.

#### Создание своего набора  
Пользователь может создать свой набор и выбрать продукты. Он обязательно даёт имя набору и выбирает продукты.   Пользователь может изменить название набора, удалить набор, удалить выбранные продукты, добавить новые. Если данные при создании или изменении набора введены неверно — выводится сообщение об ошибке. 

<img width="659" alt="Снимок экрана 2025-01-22 в 20 35 24" src="https://github.com/user-attachments/assets/c051f9f7-99f6-4ee2-b443-53f485b15f3c" />

#### Работа с курьерами  
Работа с курьерами предполагает два режима работы:  
1. При оформлении заказа: URL
`POST /api/v1/orders`  
Логика выбора курьерской службы при оформлении заказа пользователем: служба должна работать в указанное в заказе время и должна быть самой дешёвой.   
Пользователю отображается время доставки в зависимости от выбранной службы.  
В заказе доставка становится платной, если соблюдается хотя бы одно условие:   
- превышено максимальное количество товаров;  
- превышен максимальный вес;  
- сумма заказа меньше 150 рублей.  
Логика расчёта стоимости доставки заказа для пользователя:  
- Если вес или количество превысили максимальное значение, стоимость доставки для пользователя становится 99 рублей.  
- Если вес или количество в заказе не превышают максимального, берётся
`price`
этих продуктов, и если их сумма меньше 150 рублей, то стоимость доставки для пользователя также становится 99 рублей.  
Стоимость доставки прибавляется в итоговую сумму заказа.  
Если ни одно из обозначенных условий не соблюдается, то цена доставки курьерской службы не включается в итоговую сумму заказа.   

2. Узнать возможность доставки продуктов и цену для отдельной курьерской службы: у каждой курьерской службы свой URL.   
Цена доставки курьерской службой рассчитывается по количеству и весу указанных продуктов.
Детальные требования к расчёту стоимости доставки курьерскими службами можно посмотреть [тут](https://code.s3.yandex.net/qa/files/delivery_requirements.pdf)

#### Работа со складом
Имеется 4 складских отделения.   
У каждого склада своя ручка. 
У каждого свой ограниченный набор продуктов. Когда пользователь сделал заказ, ручка уточняет, какой склад сформирует заказ.  
Логика выбора склада: есть продукты на складе, должен работать во время заказа и самый дешёвый.   
Пользователь может заказать только те продукты и их количество, которые есть в полной мере хотя бы на одном складе (то есть ситуации, где он набрал корзину, а ему пишут «Не привезём» — нет). 

<img width="642" alt="Снимок экрана 2025-01-22 в 20 37 40" src="https://github.com/user-attachments/assets/d2f09605-8cb3-4383-b662-10ce0f352320" />

#### Список URL реализованных в API
Подробнее о самих URL и параметрах смотри в документации к API

#### URL для авторизации
- `POST /api/v1/users - создать пользователя`

#### URL для наборов
- `POST /api/v1/kits - создать набор`
- `GET /api/v1/kits - получить список наборов`
- `DELETE /api/v1/kits - удалить набор`
- `PUT /api/v1/kits - переименовать набор, изменить список продуктов в наборе`
- `GET /api/v1/kits/search - получить список продуктов в наборе`

#### URL для продуктов
- `POST /api/v1/products/kits - получить список наборов по продуктам`
- `POST /api/v1/kits/{id}/products - добавить продукты в набор`
- `PUT /api/v1/products/:id - изменить цену продукта`
- `POST /api/v1/orders - посчитать сумму продуктов`
- `POST /api/v1/warehouse/check - проверить наличие продуктов на складах`

#### URL для складов
- `GET /api/v1/warehouses - получить список складов`
- `POST /api/v1/warehouses/amount - получить информацию о количестве продуктов на складах`
- `/api/wsdl - получить информацию о количестве продуктов на складах`
- `POST /api/v1/orders - получить информацию, какой склад возьмет заказ`

#### URL для курьерских служб
- `GET /api/v1/couriers - получить список курьерских служб`
- `POST /api/v1/couriers/check - узнать информацию доступна ли курьерская служба для доставки заказа`
- `POST /api/v1/orders - узнать информацию, какой курьер возьмет заказ`
- `POST /moscow-delivery/v1/calculate - возможность доставки и её стоимость курьерской «Доставка Москва»`
- `POST /on-a-broomstick/v1/delivery - возможность доставки и её стоимость курьерской службой «На метле уюта»`
- `POST /fast-delivery/v3.1.1/calculate-delivery.xml - возможность доставки и её стоимость` `курьерской службой «Привезём быстро»`
- `POST /train-noises/wsdl - возможность доставки и её стоимость курьерской службой «Чух-чух и уже у вас»`

#### URL для заказов
- `POST /api/v1/orders - посчитать итоговую сумму заказа (вместе с доставкой)`
- `POST /api/v1/orders - посчитать стоимость доставки с учётом различных курьерских служб`
- `GET /api/v1/orders - получить список заказов`

#### URL для корзины
- `GET /api/v1/orders/id - получить список продуктов в корзине`
- `POST /api/v1/orders - создать корзину`
- `PUT /api/v1/orders/:id - добавить продукты в корзину`
- `DELETE/api/v1/orders/:id - удалить корзину`

#### Описание содержимого базы данных

<img width="820" alt="Снимок экрана 2025-01-22 в 20 39 47" src="https://github.com/user-attachments/assets/04eb4d63-b2c7-4fa5-a547-d1a6758681bf" />
<img width="514" alt="Снимок экрана 2025-01-22 в 20 39 09" src="https://github.com/user-attachments/assets/f6bfd3dc-c2f7-4370-b9da-ebd655f7107c" />
<img width="792" alt="Снимок экрана 2025-01-22 в 20 39 53" src="https://github.com/user-attachments/assets/76b060f2-ef13-4e15-9284-3657d7a6cbc2" />
<img width="732" alt="Снимок экрана 2025-01-22 в 20 40 05" src="https://github.com/user-attachments/assets/2b3a2e2b-2b58-407d-adf1-8122f146ee09" />
<img width="659" alt="Снимок экрана 2025-01-22 в 20 40 10" src="https://github.com/user-attachments/assets/d160a7fa-d2ff-47da-ac37-ffb7974856fd" />

***

</details>

<details>
<summary> API Яндекс.Прилавка </summary>

### API Яндекс.Прилавок 3.1.1 

### Warehouses
**Warehouses - [SOAP] Проверить количество товаров на складах**
**POST**
`/api/wsdl`  
Параметр:  
| Название | Тип | Описание |
| ------------------- | ------------------- | ------------------- |
| ids       | number[]      | Массив идентификаторов продуктов (после id в таблице product_model).       |

[XML] Проверяем количество товаров
```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="WebServices.MainWsdl">
    <soap:Body>
        <Request xmlns="WebServices.MainWsdl">
            <ids>1</ids>
            <ids>4</ids>
            <ids>44</ids>
        </Request>
    </soap:Body>
</soap:Envelope>
```

Ответ: На каком складе, что есть и сколько
```
    HTTP/1.1 200 OK
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="WebServices.MainWsdl">
    <soap:Body>
        <Response>
            <name>Имеется всё</name>
            <products>
                <name>Sprite классический</name>
                <quantity>9</quantity>
            </products>
            <products>
                <name>Чипсы кукурузные классик Salto Nachos</name>
                <quantity>6</quantity>
            </products>
        </Response>
        <Response>
            <name>Чердак</name>
            <products>
                <name>Сок Jumex апельсин без сахара</name>
                <quantity>3</quantity>
            </products>
            <products>
                <name>Sprite классический</name>
                <quantity>12</quantity>
            </products>
        </Response>
        <Response>
            <name>Большой мир</name>
            <products>
                <name>Сок Jumex апельсин без сахара</name>
                <quantity>1</quantity>
            </products>
        </Response>
        <Response>
            <name>Шведский дом</name>
            <products>
               <name>Сок Jumex апельсин без сахара</name>
                <quantity>3</quantity>
            </products>
            <products>
                <name>Sprite классический</name>
                <quantity>12</quantity>
            </products>
        </Response>
    </soap:Body>
</soap:Envelope>
```

**Warehouses - Получить список складов**
**GET**
`/api/v1/warehouses`  

Ответ: Успешное получение складов
```
HTTP/1.1 200 OK
[
    {
           "name": "Имеется всё",
           "workingHours": {
               "start": 7,
               "end": 23
           }
       },
    {
           "name": "Шведский дом",
           "workingHours": {
               "start": 8,
               "end": 23
           }
       },
    {
           "name": "Чердак",
           "workingHours": {
               "start": 8,
               "end": 21
           }
       },
    {
           "name": "Большой мир",
           "workingHours": {
               "start": 5,
               "end": 20
           }
       }
    ]
```

**Warehouses - Проверить количество товаров на складах**  
Версия этого эндпоинта для SOAP называется - [SOAP] Проверить количество товаров на складах  
**POST**
`/api/v1/warehouses/amount`  

[JSON] Пример заголовков
```
{
    "Content-Type": "application/json"
}
```
[XML] Пример заголовков
```
{
    "Content-Type": "application/xml"
}
```

Параметр:  
| Название | Тип | Описание |
| ------------------- | ------------------- | ------------------- |
| ids       | number[]      | Массив идентификаторов продуктов (после id в таблице product_model).       |
| dataType      | string      | Формат входных данных. Может принимать значения: "json" - тело запроса ожидается в JSON-формате "xml" - тело запроса ожидается в XML-формате По умолчанию: json     |

[JSON] Проверяем количество товаров
```
{
    "ids": [
        1,
        4,
        44
    ]
}
```
[XML] Проверяем количество товаров
```
<root>
    <id>1</id>
    <id>4</id>
    <id>44</id>
</root>
```

Ответ: На каком складе, что есть и сколько
```
HTTP/1.1 200 OK
{
       "Имеется всё": {
           "Sprite классический": 9,
           "Чипсы кукурузные классик Salto Nachos": 6
       },
       "Чердак": {
           "Сок Jumex апельсин без сахара": 3,
           "Sprite классический": 12
       },
       "Большой мир": {
           "Сок Jumex апельсин без сахара": 1
       },
       "Шведский дом": {
           "Сок Jumex апельсин без сахара": 3,
           "Sprite классический": 12
       }
   }
```

***

</details>

## Инструменты

<p align="left"> 
  <a href="https://docs.google.com/" target="_blank" rel="noreferrer"><img src="https://w7.pngwing.com/pngs/240/1015/png-transparent-g-suite-google-docs-google-angle-rectangle-logo.png" width="36" height="36" alt="Google Sheets" /></a>
  <a href="https://www.figma.com/" target="_blank" rel="noreferrer"><img src="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/skills/figma-colored.svg" width="36" height="36" alt="Figma" /></a>
  <a href="https://tracker.yandex.ru/pages/my" target="_blank" rel="noreferrer"><img src="https://upload.wikimedia.org/wikipedia/commons/d/d2/Yandex_Tracker_logo.svg" width="36" height="36" alt="Яндекс Трекер" /></a>
  <a href="https://www.postman.com/" target="_blank" rel="noreferrer"><img src="https://seeklogo.com/images/P/postman-logo-0087CA0D15-seeklogo.com.png" title="Postman" width="36" height="36" alt="Postman" /></a>
</p>

## Процесс работы

<details>
<summary> Проектирование тестовой документации </summary>
  
Чек-лист

<img width="762" alt="Снимок экрана 2025-01-22 в 20 46 09" src="https://github.com/user-attachments/assets/6d5942ed-4ab6-472c-a681-2c362d1dcffb" />

***

</details>

### Выполнение тестов

<details>
<summary> Выполнение тестов </summary>
  
[Тестовая документация с кликабельными ссылками на баг-репорты](https://docs.google.com/spreadsheets/d/1AeKZiU-iO1_I2YEBDO0Tl0sKq04iX72dwOP9e4Labv4/edit?gid=2006427015#gid=2006427015)

### Баг репорты
<img width="1259" alt="Снимок экрана 2025-01-22 в 20 55 11" src="https://github.com/user-attachments/assets/94e29472-ecd3-4d8a-9100-308ea91d568f" />

***

</details>
