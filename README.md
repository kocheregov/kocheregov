# Александр Шевченко (@kocheregov)
**Senior DevOps / SRE / DevSecOps Engineer & AI Agents Architect**

- **Production-опыт (5+ лет):** Сбербанк (Главный инженер по разработке, DevOps / DevSecOps), САМСОН Групп (SRE, распределенная инфраструктура 170+ серверов), Компас Плюс (FinTech DevOps).
- **Bug Bounty & Offensive Security:** **Топ-60** рейтинга [BI.ZONE Bug Bounty](https://bugbounty.bi.zone/profile/kocheregov) и активный исследователь на Standoff 365 (35+ принятых отчетов об уязвимостях в Kubernetes, Cloud, DBaaS, IAM; >1.7 млн руб выплат).
- **Соревнования и алгоритмы:** **3 место** на Альфа ЦТФ 2026 (команда HillBilly), полуфиналист чемпионата мира по программированию ICPC (2019-2020), ПМИ ЮУрГУ.

---

## Мультиагентные системы и R&D (Architecture Showcase)

Исходный код внутренних пайплайнов находится в приватных репозиториях. Ниже приведена архитектура и контракты разработанных систем.

### 1. FeralOps - мультиагентный оркестратор для DevSecOps-аудита и диагностики инфраструктуры
Python-движок с жестким разделением ролей, изоляцией контекстов и принципом наименьших привилегий (Least Privilege) для агентов:

~~~mermaid
flowchart TD
    W[Watcher Daemon<br/>Мониторинг изменений инфры / API / K8s] -->|Алерт + дифф| C[Conductor<br/>Декомпозиция и бюджет токенов]
    C -->|Постановка задачи в lane| H[Hunter Agent<br/>Сбор диагностических данных в песочнице]
    H -->|Сырые артефакты и логи без выводов| J[Blind Judge<br/>Слепая верификация без галлюцинаций]
    J -->|Подтвержденный кейс| O[Operator HITL<br/>Подтверждение действий в Telegram]
~~~

- **Conductor (Дирижер):** декомпозирует задачу, планирует диагностические линии (lanes), маршрутизирует запросы между моделями (reasoning-модели vs быстрые полевые агенты) и контролирует бюджет токенов.
- **Hunter (Хантер):** полевой агент-исполнитель в изолированной контейнерной песочнице. Анализирует манифесты Kubernetes, облачные конфигурации, сетевые политики и API-контракты через строго типизированный tool-calling.
- **Blind Judge (Слепой Судья):** независимый агент-верификатор с изолированным контекстом. Не видит гипотез и рассуждений Хантера, получает только сырые дампы/логи выполнения, проверяет детерминированную воспроизводимость и отсекает галлюцинации (false positives) до эскалации человеку.
- **Watcher:** сервис непрерывного мониторинга конфигураций и релизов. Отслеживает дрейф инфраструктуры, обновления API-схем и чейнджлогов, отправляя диффы инженеру в Telegram с запуском проверок через подтверждение (Human-in-the-Loop).

#### Пример контракта верификации (Hunter -> Blind Judge)
~~~json
{
  "lane_id": "k8s-netpol-namespace-isolation-check",
  "sandbox_run_id": "run_2026_10_25_0412",
  "proposed_action_tier": "read_only_verification",
  "raw_evidence": {
    "target_resource": "ns/prod-billing/serviceaccount/metrics-reader",
    "observed_state": "cross_namespace_egress_allowed_to_control_plane_api",
    "manifest_diff_hash": "sha256:9f86d081884c7d659a2feaa0c55ad015",
    "repro_steps_deterministic": true
  },
  "blind_judge_verdict": {
    "status": "CONFIRMED",
    "false_positive_risk": "LOW",
    "requires_operator_approval": true
  }
}
~~~

---

### 2. feral_eval - бенчмарк и фреймворк оценки LLM-агентов на задачах DevOps / SecOps
Фреймворк для автоматизированного тестирования проприетарных и open-weights моделей на многошаговых инженерных сценариях:
- Аудит манифестов Kubernetes (`RBAC`, `NetworkPolicy`, изоляция неймспейсов), поиск мисконфигов инфраструктуры, анализ системных логов и надежность вызова инструментов (`tool use`).
- Автоматический замер `pass-rate`, доли ложных срабатываний, латентности и стоимости успешного расследования на гибридном стеке (`vLLM` на GPU с `AWQ/FP8` и continuous batching + облачные `Vertex AI Gemini`, `Claude`, `DeepSeek`).

### 3. Chiefix - многопользовательская мультимодальная агентная платформа
Продакшн-экосистема ИИ-агентов на выделенном Linux-сервере с разделением прав доступа (RBAC между контуром администратора и гостевым контуром):
- Двухуровневая структурированная долговременная память (`profile` + `topic` с типизацией фактов, решений и правил) и семантический поиск по архиву (`recall`).
- Изолированный раннер инструментов (разбор PDF/аудио/изображений, генерация архитектурных диаграмм и графиков) + проактивный планировщик задач по контексту и дедлайнам.

### 4. Sotto - real-time стриминговый ассистент для разбора инцидентов
Инженерный копайлот с низкой задержкой для работы на траблшутинг-сессиях (on-call war rooms): потоковая обработка аудио и системного контекста в реальном времени, быстрый поиск по базе знаний/ранбукам и синхронизация заметок через WebSockets.

---

## Стек и инфраструктурная экспертиза
- **AI & Agents:** Python (`asyncio`), Multi-Agent Orchestration, MCP, Tool-Calling, RAG, `vLLM` (GPU inference, AWQ/FP8), Google Vertex AI (Gemini), Claude, DeepSeek, OpenAI.
- **DevOps / SRE / DevSecOps:** Kubernetes, Docker, Linux (диагностика на уровне ядра и ФС, бесшовная миграция 170+ серверов на Astra/ALT Linux), Ansible, Terraform, Helm, Argo CD, GitLab CI, Jenkins Shared Libraries (блокирующие Quality Gates: Semgrep, Trivy, Gitleaks, SonarQube), Prometheus, Grafana, Loki, OpenSearch, PostgreSQL/Patroni, Kafka, Redis.
