# Digital Barometer

Сервис анализа медиа-упоминаний по теме: собирает данные из нескольких
источников, размечает тональность и эмоции через LLM, строит тренды и
графики. Веб-интерфейс и Telegram — в перспективе.

## Архитектура

Три независимых репозитория, разворачиваются вместе через общую docker-сеть:

| Репозиторий | Стек | Роль |
| --- | --- | --- |
| [**backend**](https://github.com/digital-barometer/backend) | FastAPI, dishka (DI), PostgreSQL, LangChain (LLM) | REST API, сбор и анализ данных |
| [**frontend**](https://github.com/digital-barometer/frontend) | React 18, TypeScript, Vite, Tailwind CSS, Recharts | Веб-интерфейс: топики, источники, графики анализа |
| [**infra**](https://github.com/digital-barometer/infra) | Traefik, PostgreSQL, Docker Compose | Reverse-proxy (TLS через Let's Encrypt) и база данных |

Backend построен слоями `api → services → repositories → db`, с отдельным
пакетом `digital-barometer-db` (модели SQLAlchemy + Alembic-миграции).

## Источники данных

| Источник | Через |
| --- | --- |
| GDELT Doc API | встроенный коннектор |
| NewsAPI | встроенный коннектор |
| RSS | произвольные ленты |
| Google Trends | SerpApi |

Источник подключается через `ConnectorFactory` по типу; запросы идут с
ограничением конкурентности и опциональным исходящим прокси. Секреты
(API-ключи, `Authorization`) вычищаются из логов при ошибках источника.

## Как работает анализ

1. Пользователь создаёт топик и выбирает источники.
2. Backend строит поисковый план (`search_plan`) и параллельно тянет данные
   из источников (`source_fetch`).
3. LLM (LangChain, OpenAI-совместимый эндпоинт) размечает тональность и
   эмоции упоминаний батчами (`sentiment`, `llm`).
4. Результаты агрегируются в тренды и метрики (`metrics`, `analysis`) и
   отдаются фронтенду для графиков.

## Инфраструктура и деплой

Каждый сервис — свой `.gitlab-ci.yml` с одинаковой структурой:
`test → build → deploy (stage/prod по SSH)`. Backend дополнительно собирает
и катит отдельные образы для миграций и сидирования БД. Reverse-proxy
(Traefik) и PostgreSQL живут в `infra` и общей внешней сети `web_network`,
к которой подключаются `backend` и `frontend`.

---

Подробности запуска — в README каждого репозитория.
