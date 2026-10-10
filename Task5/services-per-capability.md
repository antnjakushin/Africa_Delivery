# Маппинг возможностей на сервисы и внешние зависимости (Services per Capability)

---

## Таблица распределения возможностей, сервисов и внешних зависимостей

| Бизнес-возможность (Capability L2) | Реализующий программный сервис / модуль | Владелец сервиса (Команда) | Тип реализации | Внешние интеграционные зависимости | SLA / Требования к зависимости |
|---|---|---|---|---|---|
| **1.1. Каталог товаров и номенклатура** | `Catalog Module` | Core Backend (Dev 1) | In-House (Go) | S3 Media Storage, Redis | Доступность 99.8%, latency < 50 мс |
| **1.2. Поисковый движок** | `Search & Filter Engine` | Core Backend (Dev 1) | In-House (PostgreSQL Trigram / Elastic) | Redis Cache | Время выполнения запроса < 100 мс |
| **1.3. Стейт-машина заказов** | `Order Processing Engine` | Core Backend (Dev 2) | In-House (Go) | PostgreSQL Master, SMS Gateway | Строгая транзакционность (ACID) |
| **2.1. Клиентская витрина** | `Client PWA / Mobile Web` | Frontend (Dev 3) | In-House (React / Vite PWA) | CloudFront CDN, Sentry RUM | Размер бандла < 3.5 МБ |
| **2.3. Нотификации покупателям** | `Notification Dispatcher` | Core Backend (Dev 4) | Hybrid Adapter | **Africa's Talking API** (Primary), Twilio (Backup) | Гарантия доставки SMS < 10 секунд |
| **3.1–3.3. Кабинет мерчанта** | `Merchant Web Portal` | Fullstack (Dev 5) | In-House (Next.js / Go) | PostgreSQL Master, CloudFront | Простота работы на слабых ноутбуках |
| **4.1. Мобильные платежи** | `M-Pesa Gateway Adapter` | Payments (Dev 6) | Partner Integration | **Safaricom Daraja API** (REST) | Время ответа STK Push < 3 секунд |
| **4.2. Карточный эквайринг** | `Card Payments Adapter` | Payments (Dev 6) | Partner Integration | **Flutterwave / Paystack API** | PCI DSS Level 1 соответствие |
| **4.3. Оплата наличными (COD)** | `COD Settlement Handler` | Payments (Dev 6) | In-House | SMS Confirmation Gateway | Простая смена статуса в 1 клик |
| **5.3. Координация доставки** | `Logistics Dispatcher` | Core Backend (Dev 2) | Partner Adapter | **Sendy / Kobo360 API**, WBA Fleet | Webhook обновление трекинг-статуса |
| **6.1. Защищенный контур ПДн** | `PII Token Vault` | Security & Lead Arch | In-House Isolated Service | AWS KMS / Local Encrypted DB | 100% изоляция персональных данных |
