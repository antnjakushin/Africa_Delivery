# SLO-контракт: Сервис каталога товаров (Catalog Service)

Документ оформлен в соответствии со спецификацией **OpenSLO v1alpha**.

```yaml
apiVersion: openslo/v1alpha
kind: SLO
metadata:
  name: catalog-service-availability-latency
  displayName: Доступность и производительность витрины каталога товаров
  service: catalog-service
spec:
  description: Гарантирует быстрый отклик каталога (< 500 мс) и доступность витрины для покупателей в условиях 3G сетей.
  timeWindow:
    - duration: 30d
      isRolling: true
  budgetingMethod: Occurrences
  indicator:
    metadata:
      name: catalog-good-requests-ratio
    spec:
      ratioMetric:
        good:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(http_request_duration_seconds_bucket{service="catalog", route=~"/v1/products.*", le="0.5", status!~"5.."}[5m]))
        total:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(http_request_duration_seconds_count{service="catalog", route=~"/v1/products.*"}[5m]))
  objectives:
    - target: 0.998
      displayName: 99.8% успешных быстрых запросов к каталогу
```

---

## 1. Спецификация контракта

### 1.1. SLI (Service Level Indicator)
Доля HTTP-запросов к каталогу товаров (`GET /v1/products/*`, `GET /v1/categories/*`), завершившихся с кодом отличным от 5xx за время **$\le 500$ миллисекунд**:
$$	ext{SLI} = rac{\sum 	ext{Requests}(	ext{Status} 
e 5xx \land 	ext{Latency} \le 500	ext{ms})}{\sum 	ext{Total Requests}} 	imes 100\%$$

### 1.2. SLO (Service Level Objective)
* **Целевой уровень**: **99.8%** за скользящее окно **30 дней**.
* **Бюджет ошибок (Error Budget)**: **0.2%** (эквивалентно суммарному окну недоступности не более **86.4 минут в месяц**).

### 1.3. SLA (Service Level Agreement для бизнеса)
* **Бизнес-гарантия**: Доступность витрины не менее **99.5%**.
* **Последствия нарушения**:
  - При исчерпании $> 50\%$ бюджета ошибок за неделю: автоматическая заморозка выкатки новых фичей (Feature Freeze); команда занимается оптимизацией кэша и индексов БД;
  - При падении ниже 99.5%: запуск процедуры P1 Incident с привлечением Lead Architect.

---

## 2. Политика оповещения (Multi-Window Multi-Burn-Rate Alerts)

| Уровень алерта | Окно оценки | Скорость сгорания бюджета (Burn Rate) | Израсходованный бюджет | Канал эскалации | Время реакции (MTTA) |
|---|---|---|---|---|---|
| **Critical (P1)** | 1 час | **14.4x** | 2% бюджета | PagerDuty / Telegram Call дежурному | $\le 5$ минут |
| **Warning (P2)** | 6 часов | **6.0x** | 5% бюджета | Slack / Telegram уведомление | $\le 30$ минут |
| **Ticket (P3)** | 3 дня | **1.0x** | 10% бюджета | Jira задача в бэклог техдолга | Следующий рабочий день |
