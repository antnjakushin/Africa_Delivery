# SLO-контракт: Сервис нотификаций (Notification Service)

Документ оформлен в соответствии со спецификацией **OpenSLO v1alpha**.

```yaml
apiVersion: openslo/v1alpha
kind: SLO
metadata:
  name: notification-delivery-timeliness
  displayName: Надежность и своевременность доставки OTP и сервисных нотификаций
  service: notification-service
spec:
  description: Гарантирует быструю доставку одноразовых паролей (OTP) и статусов заказов через Africa's Talking и SMS-шлюзы.
  timeWindow:
    - duration: 30d
      isRolling: true
  budgetingMethod: Occurrences
  indicator:
    metadata:
      name: sms-delivery-latency-ratio
    spec:
      ratioMetric:
        good:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(sms_dispatch_duration_seconds_bucket{le="10.0", status="ACCEPTED"}[5m]))
        total:
          metricSource:
            type: Prometheus
            spec:
              query: sum(rate(sms_dispatch_duration_seconds_count[5m]))
  objectives:
    - target: 0.990
      displayName: 99.0% нотификаций передано оператору за время <= 10 секунд
```

---

## 1. Спецификация контракта

### 1.1. SLI (Service Level Indicator)
Доля SMS-сообщений и OTP-кодов подтверждения, успешно переданных в шлюз телеком-провайдера (Africa's Talking / Twilio) за время **$\le 10.0$ секунд**:
$$	ext{SLI} = rac{\sum 	ext{Notifications}(	ext{Dispatched} \le 10	ext{s} \land 	ext{Status} = 200)}{\sum 	ext{Total Notifications}} 	imes 100\%$$

### 1.2. SLO (Service Level Objective)
* **Целевой уровень**: **99.0%** за скользящее окно **30 дней**.
* **Бюджет ошибок**: **1.0%** (не более 1 задержки на 100 отправок).

### 1.3. SLA (Service Level Agreement для бизнеса)
* **Критический путь OTP**: Доставка одноразового пароля для входа в приложение должна занимать не более **20 секунд**. При превышении таймаута клиенту автоматически предлагается альтернатива — голосовой звонок или WhatsApp-сообщение.
* **Сервисные SMS о доставке**: Допускается буферизация в очереди до 15 минут в часы пиковой нагрузки сотовых вышек в Лагосе/Найроби без влияния на работу платформы.
