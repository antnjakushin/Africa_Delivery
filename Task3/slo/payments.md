# SLO-контракт: Сервис процессинга платежей (Payment Service)

Документ оформлен в соответствии со спецификацией **OpenSLO v1alpha**.

```yaml
apiVersion: openslo/v1alpha
kind: SLO
metadata:
  name: payment-processing-success
  displayName: Успешность инициации и верификации транзакций оплаты
  service: payment-service
spec:
  description: Обеспечивает бесперебойную инициацию платежей M-Pesa, Flutterwave и Cash-on-Delivery без потери транзакций.
  timeWindow:
    - duration: 30d
      isRolling: true
  budgetingMethod: Occurrences
  indicator:
    metadata:
      name: payment-success-ratio
    spec:
      ratioMetric:
        good:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(payment_transactions_total{status="SUCCESS", error_type="NONE"}[5m]))
        total:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(payment_transactions_total[5m])) - sum(rate(payment_transactions_total{error_type="USER_CANCELLED"}[5m]))
  objectives:
    - target: 0.995
      displayName: 99.5% успешных платежных транзакций (за вычетом пользовательских отказов)
```

---

## 1. Спецификация контракта

### 1.1. SLI (Service Level Indicator)
Доля платежных транзакций, успешно инициированных и обработанных платформой без внутренних технических ошибок (HTTP 5xx, внутренние таймауты, сбои стейт-машины):
$$	ext{SLI} = rac{\sum 	ext{Transactions}(	ext{Success})}{\sum 	ext{Total Transactions} - \sum 	ext{User Abandonments}} 	imes 100\%$$

### 1.2. SLO (Service Level Objective)
* **Целевой уровень**: **99.5%** за скользящее окно **30 дней**.
* **Бюджет ошибок (Error Budget)**: **0.5%** (допустимо не более 5 сбоев на 1 000 оформлений заказов).

### 1.3. SLA (Service Level Agreement для бизнеса)
* **Финансовая гарантия**: **Zero Loss Policy (RPO = 0)**. Ни одна подтвержденная списанием M-Pesa/картой транзакция не может быть потеряна. При расхождении баланса — автоматический триггер ночной сверки (Reconciliation Job).
* **Смягчение при сбоях провайдеров**: Если шлюз M-Pesa или Flutterwave возвращает ошибки более 30 секунд, срабатывает Circuit Breaker, и клиентам принудительно предлагается опция **Cash on Delivery (COD)**, чтобы не терять продажи.

---

## 2. Политика оповещения

* **P1 Alert (Critical)**: Доля ошибок платежного шлюза $> 3\%$ за 5 минут $ightarrow$ Немедленный алерт дежурному инженеру (PagerDuty) и переключение чекаута на COD.
* **P2 Alert (Warning)**: Рост времени подтверждения вебхука M-Pesa $> 10$ секунд $ightarrow$ Оповещение в канал интеграций.
