@startuml
title Доставка курьерами Яндекс.Еда
actor Курьер as Курьер
actor Ресторан as Ресторан 
actor Клиент as Клиент
actor Яндекс.Маршрутизация as Яндекс.Маршрутизация
boundary Frontend as Frontend 
MS_Заказы -> MS_Доставка : заказ (id) "В работе", время приготовления
loop ежеминутно до выполнения одного из условий
MS_Доставка -> Яндекс.Маршрутизация : выполнить поиск активных курьеров (онлайн) рядом с рестораном (адрес) 
Яндекс.Маршрутизация --> MS_Доставка : id, местоположение 
MS_Доставка -> MS_Доставка : проверка: \n1. до конца приготовления меньше t минут? \n2. курьеров рядом с рестораном  меньше n?
note right
Значения переменных t и n Яндексом не разглашаются ))
"Рядом" определяется по разному для разных 
тарифов доставки (выбирается курьером):
Тариф Курьер (пеший курьер, вело курьер) - 3 км
Тариф Доставка (авто курьер) - 10 км
end note
end
MS_Доставка -> Яндекс.Маршрутизация : выполнить поиск ближайшего курьера 
MS_Доставка -> MS_Заказы : статус заказа (id) "Ищем курьера"
Яндекс.Маршрутизация --> MS_Доставка : результаты поиска (id, координаты или ошибка)
alt курьер найден
MS_Доставка -> MS_Доставка : Назначить заказ (id) курьеру (id)
else курьер не найден
MS_Доставка -> MS_Заказы : заказ (id) отменить, причина (id)
end
MS_Доставка -> Frontend : заказ (данные) назначен курьеру (id)
Frontend -> Курьер : отобразить новый заказ 
MS_Доставка -> MS_Уведомления : заказ (данные) назначен курьеру (id)
MS_Уведомления -> Telegram : заказ (данные) назначен курьеру (id)
Telegram -> Курьер : уведомление
alt курьер принимает заказ
Курьер -> Frontend : Принять заказ
else курьер не принял заказ в течение 1,5 минут
note over Курьер, Frontend
возврат на шаг "выполнить поиск курьера"
end note
end
Frontend -> Курьер : отобразить таймер (нормативное \nвремя прибытия в ресторан)
MS_Доставка -> Яндекс.Маршрутизация : построить маршрут до ресторана 
Яндекс.Маршрутизация --> Курьер : отобразить варианты маршрута до ресторана, навигацию по маршруту
Frontend -> MS_Доставка : заказ (id) принят курьером (id)
MS_Доставка -> MS_Заказы: заказ (id) принят курьером (id) 
Курьер -> Frontend : прибыл в ресторан
Frontend -> MS_Доставка : курьер (id) прибыл в ресторан (id) 
MS_Заказы -> Frontend : статус заказа (id) "забрать заказ" 
Курьер -> Ресторан : забрать заказ (id)
Ресторан --> Курьер : передать заказ (id)
alt курьер отметил получение заказа
Курьер -> Frontend : забрал заказ
Frontend -> MS_Доставка : заказ (id) у курьера
Курьер -> Frontend : отметить полученные позиции заказа
note right
здесь я не усложняю диаграмму случаем, 
если не всё оказалось в наличии
end note
Frontend -> MS_Доставка : позиции (id) заказа (id) получены курьером
else курьер не отметил получение заказа
Ресторан -> Frontend : передан курьеру
Frontend -> MS_Доставка: заказ (id) у курьера
end
Frontend -> Курьер : отобразить таймер \n(нормативное время доставки клиенту)
MS_Доставка -> Яндекс.Маршрутизация : построить маршрут до клиента
Яндекс.Маршрутизация -> Курьер : отобразить варианты маршрута до клиента, навигацию по маршруту
Курьер -> Frontend : у клиента
Frontend -> MS_Доставка: курьер (id) у клиента
alt заказ успешно передан
opt 
Курьер -> Клиент : позвонить
end
Курьер -> Клиент : передать заказ
Курьер -> Frontend : отдал заказ
Frontend -> MS_Доставка: заказ (id), код доставки (N)
MS_Доставка -> MS_Заказы: заказ (id), код доставки (N)
else Клиента нет по адресу доставки
Курьер -> Клиент : позвонить
...Курьер ждёт 10 минут...
Курьер -> Frontend : клиент не вышел на связь
Frontend -> MS_Доставка: заказ (id), код доставки (M)
note right
заказ технически считается выполненным
end note
MS_Доставка -> MS_Заказы: заказ (id), код доставки (M)
end
@enduml