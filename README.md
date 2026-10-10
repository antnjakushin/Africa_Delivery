# Africa Delivery — Мультикатегорийный маркетплейс в Африке

Проект посвящен проектированию архитектуры мультикатегорийного e-commerce маркетплейса **Africa Delivery** (целевые рынки: Нигерия, Кения, ЮАР) в условиях жестких граничных ограничений:
- **Дедлайн запуска MVP**: строго **15 октября 2026 года** (8 недель на разработку);
- **Бюджет на облако** **\$5\,000$ в месяц** на этапе запуска;
- **Команда разработки**: **6 инженеров-контракторов**, заморозка найма;
- **Регуляторика**: соответствие законам о защите данных **NDPR** (Нигерия), **DPA** (Кения), **POPIA** (ЮАР).

---

## Навигация по проектной работе

### [Task 1: Карта стейкхолдеров и Architecture Canvas](Task1/)
* **[stakeholder-matrix.md](Task1/stakeholder-matrix.md)** — Матрица влияния и интереса (Power/Interest Grid), детальный реестр стейкхолдеров и анализ баланса сил (решающий голос инвестора К. Орлова).
* **[architecture-canvas.md](Task1/architecture-canvas.md)** — Паспорт архитектуры (Architecture Canvas): бизнес-цели, границы MVP, NFR, ключевые решения и ограничения.
* **[contradictions.md](Task1/contradictions.md)** — Реестр 6 ключевых несовместимых требований (CTO vs CFO vs Инвестор vs Legal) с рекомендациями архитектора по их разрешению.
* **[open-questions.md](Task1/open-questions.md)** — Открытые вопросы стейкхолдерам, дедлайны и консервативные допущения (Late Binding).

### [Task 2: Архитектурное видение и выбор архитектурного стиля](Task2/)
* **[options-comparison.md](Task2/options-comparison.md)** — Сравнительный анализ 4 вариантов (Монолит, K8s Микросервисы, Модульный монолит, Serverless) с расчетом стоимости 1 заказа и «стоимости ошибки» для CFO.
* **[tco-model.xlsx](Task2/tco-model.xlsx)** / **[tco-model.md](Task2/tco-model.md)** — Финансовая модель совокупной стоимости владения (TCO 12 мес.): инфраструктура $2 000/мес + ФОТ 6 контракторов.
* **[adr-001-architecture-style.md](Task2/adr-001-architecture-style.md)** — ADR №1: Выбор Модульного монолита (Modular Monolith) на базе AWS ECS Fargate + PostgreSQL Multi-AZ.
* **[architecture-vision.md](Task2/architecture-vision.md)** — Архитектурное видение (Architecture Vision): принципы Late Binding, YAGNI, Cloud Pragmatism, дорожная карта эволюции.
* **[nairobi-slide.pptx](Task2/nairobi-slide.pptx)** / **[nairobi-slide.md](Task2/nairobi-slide.md)** — Презентационный слайд для совета директоров в Найроби.
* **[nairobi-script.md](Task2/nairobi-script.md)** — Полный 10-минутный транскрипт выступления (~1350 слов) перед инвестором, CEO, CTO и CFO.
* **Диаграммы**: [c4-context.png](Task2/c4-context.png) ([.puml](Task2/c4-context.puml), [.png](Task2/c4-context.png)) и [use-cases.png](Task2/use-cases.png) ([.puml](Task2/use-cases.puml), [.png](Task2/use-cases.png)).

### [Task 3: Нефункциональные требования и стратегия доступности](Task3/)
* **[nfr-registry.md](Task3/nfr-registry.md)** — Реестр нефункциональных требований (16 требований) с учетом критериев Cloud Native (CNCF).
* **SLO-контракты (OpenSLO v1alpha)**:
  * **[slo/catalog.md](Task3/slo/catalog.md)** — SLO витрины каталога (99.8% доступность, p95 latency < 500 мс).
  * **[slo/payments.md](Task3/slo/payments.md)** — SLO платежей (99.5% успешность, RPO = 0, фолбэк на COD).
  * **[slo/notifications.md](Task3/slo/notifications.md)** — SLO нотификаций (99.0% доставка OTP < 10 с).
* **[quality-attribute-scenarios.md](Task3/quality-attribute-scenarios.md)** — 5 детальных QAS сценариев по 6 полям SEI (доступность, задержка 3G, сбой M-Pesa, всплеск нагрузки, аудит ПДн).
* **[fmea.md](Task3/fmea.md)** — Failure Mode and Effects Analysis (FMEA): анализ 6 критических точек отказа с расчетом RPN.
* **[adr-002-caching-strategy.md](Task3/adr-002-caching-strategy.md)** — ADR №2: Стратегия кеширования (CloudFront Edge PoP, ElastiCache Redis, Cache-Aside).
* **[observability.md](Task3/observability.md)** — Стратегия наблюдаемости по 4 золотым сигналам SRE (Prometheus, Grafana Loki, Tempo, Alertmanager) и [observability-architecture.png](Task3/observability-architecture.png).
* **[tco-availability-model.md](Task3/tco-availability-model.md)** / **[tco-availability-model.xlsx](Task3/tco-availability-model.xlsx)** — Экономическое обоснование доступности: критика тезиса инвестора о "$12k/мин" vs реальный SLA 99.5%.

### [Task 4: Регуляторные ограничения и стратегия данных](Task4/)
* **[data-residency-map.md](Task4/data-residency-map.md)** — Карта локализации данных: распределение данных по юрисдикциям NDPR (Нигерия), DPA (Кения), POPIA (ЮАР) и комплаенс-шлюз базы 410к номеров.
* **[cloud-regions-comparison.md](Task4/cloud-regions-comparison.md)** — Сравнение облачных регионов: выбор AWS Cape Town (`af-south-1`) + локальный VPS в Лагосе (MainOne).
* **[adr-003-data-residency.md](Task4/adr-003-data-residency.md)** — ADR №3: Стратегия токенизации персональных данных (PII Token Vault) и late binding (100% обратимость).
* **[failover-plan.md](Task4/failover-plan.md)** — План отказоустойчивости и аварийного переключения (RTO < 30 с при падении AZ, RTO < 60 с при сбое БД).
* **Диаграммы**: [c4-container.png](Task4/c4-container.png) ([.puml](Task4/c4-container.puml), [.png](Task4/c4-container.png)) и [deployment-diagram.png](Task4/deployment-diagram.png) ([.puml](Task4/deployment-diagram.puml), [.png](Task4/deployment-diagram.png)).

### [Task 5: MVP Roadmap и Build/Buy/Partner решения](Task5/)
* **[capability-map.md](Task5/capability-map.md)** — Иерархия способностей L1/L2 и приоритизация бэклога из 29 фичей экс-PM (5 в MVP, 24 в Post-MVP).
* **[services-per-capability.md](Task5/services-per-capability.md)** — Матрица маппинга возможностей на сервисы, владельцев и внешние интеграции.
* **[roadmap.md](Task5/roadmap.md)** — Дорожная карта с диаграммой Ганта и 4 контрольными точками (M1 Каркас, M2 MVP 15 Октября, M3 Стабилизация 6 мес, M4 Экспансия 12 мес).
* **[adr-004-payment-provider.md](Task5/adr-004-payment-provider.md)** — ADR №4: Платежная оркестрация (M-Pesa 1.5%, Flutterwave <= 2.2% GMV, Cash on Delivery).
* **[transition-architecture.md](Task5/transition-architecture.md)** — Переходная архитектура: эволюция от MVP к микросервисам (Catalog & Payments) без переписывания кодовой базы.
* **[risk-register.md](Task5/risk-register.md)** — Реестр проектных рисков и организация круглосуточного дежурства On-Call 24/7 (Follow-the-Sun) для 6 удаленных контракторов.
* **Диаграмма Value Stream**: [value-stream.png](Task5/value-stream.png) ([.puml](Task5/value-stream.puml), [.png](Task5/value-stream.png)).

---

## Архитектурные принципы решения

1. **Late Binding (Позднее связывание)**: Архитектурные решения принимаются как обратимые («двусторонние двери»). Изоляция PII Vault позволяет вынести хранилище в Лагос за 2 часа после получения юридического заключения без правки логики маркетплейса.
2. **FinOps & Lean**: Инфраструктура MVP в AWS обходится в **$2 000 / месяц**, оставляя 60% резерва от лимита CFO ($5 000 / месяц). Инфраструктурная стоимость одного заказа составляет всего **$0.013**.
3. **Эволюционность**: Код пишется с жесткими доменными границами, что позволяет на 12-м месяце при достижении 500 000 заказов/день бесшовно вынести высоконагруженные модули в микросервисы на Kubernetes.
