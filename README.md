## Группа 2304 
## Команда: Пашков Г.М. & Кроткина З.Э.

## Лабораторная работа
Docker: докеризация приложения

Цель лабораторной: собрать из исходного когда и запустить в докере рабочее приложение с базой данных (любое опенсорс - Java, python/django/flask, golang).

1. Образ должен быть легковесным
2. Использовать базовые легковестные образы - alpine
3. Вся конфигурация приложения должна быть через переменные окружения
4. Статика (зависимости) должна быть внешним томом `volume`
5. Создать файл `docker-compose` для старта и сборки
6. В `docker-compose` нужно использовать базу данных (postgresql,mysql,mongodb etc.)
7. При старте приложения должно быть учтено выполнение автоматических миграций
8. Контейнер должен запускаться от непривилегированного пользователя
9. После установки всех нужных утилит, должен очищаться кеш

## Пояснение к коду:
Наш маленький стартап интернет-магазина перерос в серьезный бизнес, и нам нужно научиться в учет проданного товара, а также (опционально) в аналитику продаваемого товара.

В проде у нас есть сервис заказов, который при создании заказа отправляет в топик Kafka событие об этом (в рамках лабораторной работы его роль выполняет консольное приложение, которое нещадно генерирует события в этот топик — KafkaHomework.OrderEventGenerator).

Вот формат этого события:
```json
{
    "moment": "timestamp",
    "order_id": "long",
    "user_id": "long",
    "warehouse_id": "long",
    "positions": {
        "item_id": "long",
        "quantity": "int",
        "price": {
            "currency": "RUR|KZT",
            "units": "long",
            "nanos": "int"
        }
    },
    "status": "Created|Canceled|Delivered"
}
```

Пояснения к контракту:
* Первое событие по заказу всегда будет created
* Второе событие — всегда canceled или delivered. Но второго может и не быть
* Объект price повторяет тип google.type.money.
 * * currency — валюта,
 * * units — целая часть,
 * * nanos — дробная часть в диапазоне 0..1, умноженная на 1,000,000,000.
 * * Пример: цена в 5316 рублей 78 копеек будет отражена как currency="RUB", units=5316, nanos = 780_000_000

