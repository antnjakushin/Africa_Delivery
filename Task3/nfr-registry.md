# Реестр нефункциональных требований (NFR Registry)

Реестр составлен на основе анализа бизнес-целей, правовых норм (NDPR/POPIA), ограничений бюджета CFO и Cloud-Native чек-листа CTO (CNCF / 12-factor apps).

---

## Таблица нефункциональных требований

| ID | Категория | Формулировка требования | Метрика | Целевое значение | Способ проверки / Валидация | Источник требования |
|---|---|---|---|---|---|---|
| **NFR-PERF-01** | **Производительность** | Время загрузки первого экрана витрины (First Meaningful Paint) на мобильных устройствах в сетях 3G. | Time to First Meaningful Paint (LCP / FMP) | $\mathbf{< 1.8	ext{ секунды}}$ (p95) при скорости канала 512 кбит/с | Lighthouse CI / WebPageTest в эмуляции Fast 3G | Маркетинговый бриф (бенчмарк «до 2 сек») |
| **NFR-PERF-02** | **Производительность** | Время отклика бэкенда на запросы чтения каталога и поиска товаров. | Server Latency (TTFB / API Response Time) | $\mathbf{< 150	ext{ мс}}$ (p95), $\mathbf{< 400	ext{ мс}}$ (p99) | Нагрузочное тестирование k6 / Grafana Tempo tracing | Спецификация API / Маркетинг |
| **NFR-PERF-03** | **Производительность** | Время отклика на оформление заказа и инициацию платежа (Checkout). | Checkout Request Duration | $\mathbf{< 1.0	ext{ секунды}}$ (p95) | k6 e-commerce сценарий / APM Tracing | Продуктовые метрики конверсии |
| **NFR-AVAIL-01** | **Доступность** | Общая доступность витрины и каталога для покупателей. | High Availability (Uptime %) | $\mathbf{\ge 99.8\%}$ (MVP), $\mathbf{\ge 99.95\%}$ (Scale) | Prometheus Blackbox Exporter / BetterUptime | Инвестор / TCO-модель доступности |
| **NFR-AVAIL-02** | **Доступность** | Доступность транзакционного ядра оформления заказов (Order Checkout). | Order Placement Success Rate | $\mathbf{\ge 99.5\%}$ (бюджет ошибок $\le 216$ мин/мес) | Бизнес-метрики Prometheus (`orders_failed_rate`) | CFO / Инвестор |
| **NFR-AVAIL-03** | **Отказоустойчивость** | Время автоматического восстановления при отказе инстанса приложения или зоны доступности (Multi-AZ). | Recovery Time Objective (RTO) | $\mathbf{\le 60	ext{ секунд}}$ | Chaos Engineering (LitmusChaos / AWS Fault Injection) | CTO Cloud-Native чек-лист |
| **NFR-AVAIL-04** | **Целостность данных** | Допустимая потеря зафиксированных транзакций при аварии ЦОД (Disaster Recovery). | Recovery Point Objective (RPO) | $\mathbf{0	ext{ секунд}}$ (для фин. транзакций), $\mathbf{< 60	ext{ с}}$ (для сессий) | Синхронная репликация PostgreSQL Multi-AZ | CFO / Аудит безопасности |
| **NFR-SCAL-01** | **Масштабируемость** | Поддержка пиковой нагрузки в день старта рекламной кампании (15 октября). | Peak Throughput (RPS) | $\mathbf{250	ext{ RPS}}$ (каталог), $\mathbf{30	ext{ RPS}}$ (чекаут) без деградации | Стресс-тестирование k6 (ramp-up до 300 RPS) | Маркетинг / Инвестор |
| **NFR-SCAL-02** | **Масштабируемость** | Горизонтальное масштабирование вычислительных инстансов при росте нагрузки. | Auto-scaling Provisioning Time | Ввод нового инстанса $\mathbf{\le 90	ext{ секунд}}$ при CPU $> 75\%$ | AWS ECS Auto Scaling Policy | CTO CN.ADL.01 |
| **NFR-SEC-01** | **Безопасность** | Изоляция и токенизация персональных данных граждан Нигерии и Кении (NDPR / DPA / POPIA). | PII Exposure Rate / Tokenization | $\mathbf{0\%}$ открытых ПДн в аналитической и операционной БД заказов | SAST/DAST аудит, статический анализ схем БД | Lead Legal (Амина Диоп) / NDPR |
| **NFR-SEC-02** | **Безопасность** | Шифрование конфиденциальных данных при хранении и передаче. | Encryption Standard | **AES-256 (At Rest)**, **TLS 1.3 (In Transit)** | SSL Labs (A+ rating), AWS KMS Policies | CTO CN.SEC.03 |
| **NFR-SEC-03** | **Безопасность** | Изоляция платежных данных карт от инфраструктуры платформы (Out-of-Scope). | PCI DSS Compliance Scope | Africa Delivery **не хранит и не видит PAN/CVV карт** | Архитектурный аудит токенизации Flutterwave | PCI DSS Level 1 Requirement |
| **NFR-OBS-01** | **Наблюдаемость** | Сбор телеметрии по 4 золотым сигналам Google SRE (Latency, Traffic, Errors, Saturation). | Telemetry Coverage | $\mathbf{100\%}$ API эндпоинтов покрыты OTel трейсами и метриками | OpenTelemetry SDK + Grafana Dashboards | CTO CN.OBS.01 / CN.OBS.02 |
| **NFR-CN-01** | **Cloud Native / CI/CD** | Развертывание новых версий без прерывания обслуживания (Zero Downtime). | Deployment Downtime | $\mathbf{0	ext{ секунд}}$ простоя при деплое (Rolling Update) | GitLab CI / ECS Deployment Controller | CTO CN.ADL.05 |
| **NFR-CN-02** | **Cloud Native** | Хранение конфигурации приложений в репозитории отдельно от кода (12-Factor App). | Configuration-as-Code | $\mathbf{100\%}$ настроек в ENV / AWS Secrets Manager | Валидация линтеров в CI/CD пайплайне | CTO CN.ADL.04 |
| **NFR-FIN-01** | **Финансовый лимит** | Ограничение ежемесячных расходов на облачную инфраструктуру на этапе запуска MVP. | Monthly Cloud Infrastructure Cost | $\mathbf{\le \$5\,000	ext{ / месяц}}$ (цель $\mathbf{\$2\,000	ext{ / мес}}$) | AWS Cost Explorer / Billing Alarms | CFO (Алекс Вебер) |
