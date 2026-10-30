# Александр Шевченко (@kocheregov)
**Senior DevOps / SRE / DevSecOps Engineer & AI Agents Architect**

- **Production-опыт (5+ лет):** Сбербанк (Главный инженер по разработке, DevOps / DevSecOps), САМСОН Групп (SRE, распределенная инфраструктура 170+ серверов), Компас Плюс (FinTech DevOps).
- **Bug Bounty & Offensive Security:** **Топ-60** рейтинга [BI.ZONE Bug Bounty](https://bugbounty.bi.zone/profile/kocheregov) и активный исследователь на Standoff 365 (35+ принятых отчетов об уязвимостях в Kubernetes, Cloud, DBaaS, IAM; >1.7 млн руб выплат).
- **Соревнования и алгоритмы:** **3 место** на Альфа ЦТФ 2026 (команда HillBilly), полуфиналист чемпионата мира по программированию ICPC (2019-2020), ПМИ ЮУрГУ.

---

## Мультиагентные системы и R&D (Architecture Showcase)

Исходный код боевых пайплайнов находится в приватных репозиториях (содержит активные правила аудита, скоупы и интеграции). Ниже приведена архитектура и контракты разработанных систем, работающих в продакшене.

### 1. FeralOps - автономный мультиагентный оркестратор для DevSecOps-аудита и анализа инцидентов
Python-движок с жестким разделением ролей, изоляцией контекстов и принципом наименьших привилегий (Least Privilege) для агентов:

~~~mermaid
flowchart TD
    W[Watcher Daemon<br/>Мониторинг диффов API / JS / K8s] -->|Алерт + контекст| C[Conductor<br/>Декомпозиция и бюджет токенов]
    C -->|Постановка задачи в lane| H[Hunter Agent<br/>Полевой сбор в Docker-песочнице]
    H -->|Сырые артефакты и логи без выводов| J[Blind Judge<br/>Слепая верификация без галлюцинаций]
    J -->|Подтвержденный кейс| O[Operator HITL<br/>Подтверждение опасных действий в TG]
~~~

- **Conductor (Дирижер):** декомпозирует скоуп, планирует исследовательские линии (lanes), маршрутизирует задачи между моделями (тяжелые reasoning-модели vs быстрые полевые агенты) и контролирует бюджет токенов.
- **Hunter (Хантер):** полевой агент-исполнитель в изолированной контейнерной песочнице. Анализирует сетевые сервисы, API-схемы, манифесты Kubernetes/облаков и клиентские бандлы через строго типизированный tool-calling.
- **Blind Judge (Слепой Судья):** независимый агент-верификатор с изолированным контекстом. Не видит гипотез и "убеждений" Хантера, получает только сырые дампы/логи выполнения, проверяет воспроизводимость и отсекает галлюцинации (false positives) до эскалации человеку.
- **Watcher:** сервис непрерывного мониторинга инфраструктуры. Отслеживает изменения поверхности (обновления бандлов, API, сертификатов, чейнджлогов) и отправляет диффы оператору в Telegram с запуском проверок через reply (Human-in-the-Loop).
- **Результат в бою:** обнаружено и подтверждено 35+ уязвимостей в инфраструктуре крупных компаний (облачные провайдеры, банки, DBaaS, Kubernetes multi-tenancy).

#### Пример контракта передачи артефактов (Hunter -> Blind Judge)
~~~json
{
  "lane_id": "dbaas-exporter-transfer-check",
  "sandbox_run_id": "run_2026_10_25_0412",
  "proposed_action_tier": "read_only_verification",
  "raw_evidence": {
    "target_service": "managed-dbaas-metrics-exporter",
    "observed_state": "exporter_bearer_token_remains_active_post_tenant_transfer",
    "http_trace_hash": "sha256:9f86d081884c7d659a2feaa0c55ad015",
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
Продакшн-экосистема ИИ-агентов на выделенном Linux-сервере с разделением прав доступа (RBAC между контуром владельца и гостевым контуром):
- Двухуровневая структурированная долговременная память (`profile` + `topic` с типизацией фактов, решений и правил) и агентный поиск по архиву (`recall`).
- Изолированный раннер инструментов (разбор PDF/аудио/изображений, генерация диаграмм и графиков) + проактивный планировщик (`nanny.py`), выбирающий момент и повод для пинга по контексту и дедлайнам.

### 4. Sotto - real-time система поддержки принятия решений с низкой задержкой
Десктопный стриминговый ассистент с захватом аудиопотока и контекста в реальном времени, выводом подсказок на прозрачный click-through оверлей и синхронизацией с мобильным пультом по WebSockets.

---

## Стек и инфраструктурная экспертиза
- **AI & Agents:** Python (`asyncio`), Multi-Agent Orchestration, MCP, Tool-Calling, RAG, `vLLM` (GPU inference, AWQ/FP8), Google Vertex AI (Gemini), Claude, DeepSeek, OpenAI.
- **DevOps / SRE / DevSecOps:** Kubernetes, Docker, Linux (диагностика на уровне ядра и ФС, бесшовная миграция 170+ серверов на Astra/ALT Linux), Ansible, Terraform, Helm, Argo CD, GitLab CI, Jenkins Shared Libraries (блокирующие Quality Gates: Semgrep, Trivy, Gitleaks, SonarQube), Prometheus, Grafana, Loki, OpenSearch, PostgreSQL/Patroni, Kafka, Redis.
