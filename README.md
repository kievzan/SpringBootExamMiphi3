# Hotel Booking Platform

Учебный проект, выполненный в рамках итоговой аттестации МИФИ по дисциплине «Фреймворк Spring и работа с REST API».

**Hotel Booking Platform** - Микросервисная система бронирования 
Распределённое приложение для управления бронированиями номеров в отелях, построенное на архитектуре микросервисов с использованием Spring Boot и Spring Cloud.

**1. Описание проекта**.

Система состоит из четырёх микросервисов на базе **Spring Boot: API Gateway + Eureka (Service Discovery) + Hotel Service + Booking Service**, обеспечивающих полный цикл бронирования: от регистрации пользователя до управления отелями и номерами. Реализована двухшаговая   согласованность данных между сервисами с механизмами компенсации при сбоях.  

**2. Архитектура**.

Основные компоненты:  

API Gateway (Port 8080) - единая точка входа, маршрутизация запросов  
Booking Service - управление бронированиями, аутентификация пользователей (JWT)  
Hotel Service - управление отелями и номерами, контроль доступности  
Eureka Server (Port 8761) - service discovery и регистрация сервисов  

**3. Технологический стек**.

Java 17+  
Spring Boot 3.x  
Spring Cloud (Gateway, Eureka, WebClient)  
Spring Security + JWT (TTL: 1 час)  
Spring Data JPA  
H2 Database (in-memory)  
Swagger/OpenAPI 3.0  

**4. Структура проекта**.

hotel-booking-platform/

├── eureka-server/          # Service Discovery  
├── api-gateway/            # API Gateway  
├── booking-service/        # Сервис бронирований  
└── hotel-service/          # Сервис управления отелями  

**5. Установка и запуск**.

Требования  
JDK 17 или выше  

Maven 3.8+  

**6. Сборка**.

mvn clean install

Запуск:

1. Eureka Server
cd eureka-server
mvn spring-boot:run
2. Hotel Service
cd hotel-service
mvn spring-boot:run
3. Booking Service
cd booking-service
mvn spring-boot:run
4. API Gateway
cd api-gateway
mvn spring-boot:run
Проверка: http://localhost:8080
Eureka Dashboard: http://localhost:8761
Swagger UI: http://localhost:8080/swagger-ui.html

**7. Аутентификация**.

Система использует JWT токены с TTL 1 час. Требуется наличие токена в заголовке Authorization: Bearer <token> для защищённых эндпоинтов.  

** **
**8. API Endpoints**.

**Аутентификация (публичные)**  
POST /user/register - регистрация пользователя  

POST /user/auth - вход в систему

**Бронирования (USER/ADMIN)**

POST /bookings - создать бронирование
GET /bookings - список своих бронирований
GET /booking/{id} - получить конкретное бронирование
DELETE /booking/{id} - отменить бронирование
GET /bookings/all - все бронирования (ADMIN)

**Отели**
GET /hotels - список отелей (USER/ADMIN)
POST /hotels - создать отель (ADMIN)
PUT /hotels/{id} - обновить отель (ADMIN)
DELETE /hotels/{id} - удалить отель (ADMIN)

**Номера**
GET /rooms/{id} - получить номер (USER/ADMIN)
GET /rooms/recommend - рекомендованные номера по загруженности (USER/ADMIN)
POST /rooms - создать номер (ADMIN)
PUT /rooms/{id} - обновить номер (ADMIN)
DELETE /rooms/{id} - удалить номер (ADMIN)

**🎯 Примеры использования**  
Регистрация  

curl -X POST http://localhost:8080/user/register \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "john_doe",
    "password": "password123",
    "admin": false
  }'

Вход в систему  

curl -X POST http://localhost:8080/user/auth \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "john_doe",
    "password": "password123"
  }'

Использование токена  

TOKEN="<ваш_токен>"
curl -H "Authorization: Bearer $TOKEN" http://localhost:8080/bookings

Создание отеля (ADMIN)

curl -X POST http://localhost:8080/hotels \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "Grand Hotel Moscow",
    "city": "Moscow",
    "address": "Red Square, 1"
  }'

Создание номера (ADMIN)  

curl -X POST http://localhost:8080/rooms \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "number": "101",
    "capacity": 2,
    "available": true,
    "hotel": {"id": 1}
  }'

Создание бронирования (ручной выбор)  

curl -X POST http://localhost:8080/bookings \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "requestId": "req-001",
    "roomId": 1,
    "startDate": "2025-11-01",
    "endDate": "2025-11-05",
    "autoSelect": false
  }'

Создание бронирования (автоподбор)  

curl -X POST http://localhost:8080/bookings \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "requestId": "req-002",
    "startDate": "2025-11-01",
    "endDate": "2025-11-05",
    "autoSelect": true
  }'

Система автоматически выберет наименее загруженный номер!  

** **

**9. Распределённые транзакции**.

Двухшаговая согласованность (Saga Pattern)  
PENDING - бронирование создано в Booking Service  

Hold - временная блокировка номера в Hotel Service  
Confirm - подтверждение блокировки  
CONFIRMED - бронирование успешно завершено  
Компенсация при сбоях  
При ошибке на любом этапе выполняется компенсация:  
Release - освобождение временной блокировки  
CANCELLED - бронирование отменено  

Механизмы надёжности  
Идемпотентность: каждый запрос содержит уникальный requestId  
Трассировка: все операции маркируются correlationId  

Retry: 3 попытки с экспоненциальной задержкой (300ms, 600ms, 1200ms)  

Timeout: 5 секунд на запрос к Hotel Service  

**10.️ Структура базы данных**.

Booking Service  
users  
id (PK), username (UNIQUE), passwordHash, role
bookings  
id (PK), userId (FK), roomId, startDate, endDate, status, requestId (UNIQUE), correlationId, createdAt  
Hotel Service  
hotels  
id (PK), name, city, address  
rooms  
id (PK), hotel_id (FK), number, capacity, available, timesBooked  
room_reservation_locks  
id (PK), requestId (UNIQUE), roomId, startDate, endDate, status  

**11.️ Безопасность**.

JWT токены подписываются HMAC ключом  
Разграничение доступа: USER и ADMIN  
HTTPS рекомендуется для продакшна  
Секретный ключ должен быть изменён перед деплоем  

**12. Мониторинг**.

Eureka Dashboard: http://localhost:8761  
Health Check: http://localhost:8080/actuator/health  
Metrics: http://localhost:8080/actuator/metrics  

**13. Troubleshooting**.

Сервисы не регистрируются в Eureka  
Убедитесь, что Eureka Server запущен первым  
Проверьте, что порт 8761 не занят  
Проверьте логи сервиса  
JWT токен недействителен  
Проверьте, что токен не истёк (1 час)  
Убедитесь в правильности формата заголовка  
Ошибка при создании бронирования  
Проверьте, что номер существует и доступен  
Проверьте, что requestId уникален  

** **
👨 Выполнено в рамках учебного задания МИФИ для демонстрации микросервисной архитектуры.
