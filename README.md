# Проект 2 спринта

## Задание 1. Планирование
Ссылка на схемы проектов 1, 2, 3
https://drive.google.com/file/d/1RqW3syzpTDPZi7Tsq7PYrvppCGi98jZi/view?usp=sharing
Вкладки: Sharding_01, Replica_02, Cached_03

## Задание 2. Шардирование
Содержание в папке: mongo-sharding

## Задание 3. Репликация
Содержание в папке: mongo-sharding-repl
Создание/удаление внешней сети:
> docker network create mongo-cluster
> docker network rm mongo-cluster       

## Задание 4. Кеширование
Содержание в папке: sharding-repl-cache
Для работы с проектом используйте утилиту make

## Задание 5. Service Discovery и балансировка с API Gateway
Ссылка на схемы проектов 4
https://drive.google.com/file/d/1RqW3syzpTDPZi7Tsq7PYrvppCGi98jZi/view?usp=sharing
Вкладка: Gateway_04

## Задание 6. CDN
Ссылка на схемы проектов 6
https://drive.google.com/file/d/1RqW3syzpTDPZi7Tsq7PYrvppCGi98jZi/view?usp=sharing
Вкладка: CDN_05

## Внимание, если возникают ошибки при запуске проекта
Ошибка может возникнуть когда происходит обращение к еще не готовому сервису, поэтому я ставил задержки.
В виду этого я сделал поэтапную инициализацию см. описание
