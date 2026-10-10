# Архитектура наблюдаемости и мониторинга (Observability)

Платформа Africa Delivery внедряет модель мониторинга на базе **Четырех золотых сигналов Google SRE (4 Golden Signals)** и открытых стандартов OpenTelemetry.

---

## 1. Четыре золотых сигнала Google SRE (Four Golden Signals)

| Золотой сигнал | Что измеряется | Ключевые метрики (Prometheus / OpenTelemetry) | Критические пороги (Alert Thresholds) | Бизнес-влияние |
|---|---|---|---|---|
| **1. Latency (Задержка)** | Время обработки запросов пользователей | `http_request_duration_seconds{quantile="0.95"}`<br>`db_query_duration_seconds{quantile="0.99"}` | - p95 API $> 500$ мс (Warning)<br>- p95 API $> 1.5$ с (Critical) | Замедление витрины приводит к потере заказов. |
| **2. Traffic (Трафик)** | Объем входящего спроса и транзакций | `http_requests_total` (RPS)<br>`orders_created_total` (заказов/мин) | - Скачок трафика $> 300\%$ за 5 мин (Auto-scale)<br>- Падение заказов до 0 в пик (Incident) | Контроль нагрузки рекламных кампаний и выявление аномалий. |
| **3. Errors (Ошибки)** | Частота и доля сбойных запросов | `http_requests_total{status=~"5.."}`<br>`payment_failures_total{reason=~".*"}` | - Error Rate $> 1\%$ за 5 мин (Warning)<br>- Error Rate $> 5\%$ за 1 мин (Critical / Page On-Call) | Сбои при оплате и оформлении заказов. |
| **4. Saturation (Насыщение)** | Утилизация ресурсов инфраструктуры | `node_cpu_utilization_percent`<br>`node_memory_utilization_percent`<br>`pg_stat_activity_count` (пул соединений) | - CPU / RAM $> 75\%$ (Trigger Auto-scale)<br>- DB Connection Pool $> 85\%$ (Critical) | Предотвращение каскадных отказов из-за исчерпания памяти/пулов. |

---

## 2. Стек инструментов наблюдаемости (Observability Stack)

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              ИСТОЧНИКИ ТЕЛЕМЕТРИИ (SOURCES)                            │
│  [Мобильный клиент PWA]    [ECS Fargate Containers]    [RDS PostgreSQL & ElastiCache]  │
└───────────────────────────────────────────┬────────────────────────────────────────────┘
                                            │ OTLP / Prometheus Metrics / JSON Logs
┌───────────────────────────────────────────▼────────────────────────────────────────────┐
│                    СБОР И АГРЕГАЦИЯ (OpenTelemetry Collector / Vector)                │
└─────────────────────┬─────────────────────┬─────────────────────┬──────────────────────┘
                      │                     │                     │
                      ▼                     ▼                     ▼
┌───────────────────────────┐ ┌───────────────────────────┐ ┌───────────────────────────┐
│     МЕТРИКИ (METRICS)     │ │        ЛОГИ (LOGS)        │ │    ТРАССИРОВКА (TRACES)   │
│   Prometheus / Grafana    │ │       Grafana Loki        │ │       Grafana Tempo     │
│       Cloud Metrics       │ │  (Структурированный JSON) │ │   (Distributed Tracing) │
└─────────────────────┬─────┘ └─────────────┬─────────────┘ └─────────────┬─────────────┘
                      │                     │                             │
                      └─────────────────────┼─────────────────────────────┘
                                            ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                   ВИЗУАЛИЗАЦИЯ И ОПОВЕЩЕНИЯ (Grafana Cloud + Alertmanager)             │
│   [Executive & SRE Dashboards] ──► [Alertmanager] ──► [Telegram / PagerDuty / Email]   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

Диаграмма архитектуры мониторинга: [observability-architecture.png](observability-architecture.png).

---

## 3. Политика логирования и алертинга

1. **Структурированные логи**: Все сервисы пишут логи в stdout в формате JSON (`timestamp`, `level`, `trace_id`, `span_id`, `module`, `user_id_hash`, `message`).
2. **Маскирование ПДн (PII Masking)**: Запрещено логировать пароли, номера банковских карт (PCI DSS) и полные телефонные номера (маскируются: `+234***1234`).
3. **Маршрутизация алертов**:
   - **P1 (Critical / Блокировка бизнеса)**: Сбой БД, Error Rate $> 5\%$, отказ M-Pesa $
ightarrow$ Мгновенный звонок/Push в PagerDuty/Telegram дежурному инженеру (SLA реакции $< 5$ минут);
   - **P2 (Warning / Деградация)**: CPU $> 75\%$, рост задержки p95 $
ightarrow$ Оповещение в Slack/Telegram-канал команды (SLA реакции $< 30$ минут).
