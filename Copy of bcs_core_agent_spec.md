# BCS Core Agent v0.1 — Техническое Задание
## ARQ LIVE: MVP Telegram Bot
### Для разработчика: vell_01
### Версия: 0.1 | Дата: 27 февраля 2026

---

## 0. КОНТЕКСТ ПРОЕКТА

### Что такое BCS
Blockchain State (BCS) — экспериментальный протокол доказуемости вклада. Система регистрирует верифицированные действия и их каузальные связи с идеями. Подробнее — см. приложенный файл `BCS_Knowledge_Base_v1.1.md`.

### Что такое ARQ LIVE
Telegram-бот, который является первым живым интерфейсом BCS. Не «чат с функциями», а операционная система входа: гид, навигатор, диспетчер задач и проводник репутации.

### Задача этого ТЗ
Спроектировать и реализовать MVP (Level 0 — Echo), способный к поэтапному расширению до полноценного BCS-агента.

### Ключевой принцип UX
Участник должен чувствовать: «Система прозрачна. Я понимаю правила. Я расту. Моё действие имеет вес.»

---

## 1. АРХИТЕКТУРА MVP

### 1.1 Общая схема

```
[Telegram User] 
    ↓
[Telegram Bot API (python-telegram-bot / aiogram)]
    ↓
[ARQ Core Engine]
    ├── Router (intent classification)
    ├── Knowledge Module (RAG over BCS KB)
    ├── Profile Module (user state)
    ├── Quest Module (onboarding + tasks)
    ├── Reputation Module (read-only v0.1)
    └── Analytics Module (heatmap)
    ↓
[Storage Layer]
    ├── PostgreSQL (user profiles, quest progress, analytics)
    ├── Redis (session state, cache)
    └── Vector DB — Qdrant / Chroma (KB embeddings для RAG)
    ↓
[AI Layer]
    ├── Claude API (dialogue, reasoning)
    └── Embedding model (для RAG-поиска по KB)
```

### 1.2 Принципы архитектуры

- **Модульность**: каждый модуль — независимый сервис с чётким API. Можно заменять, обновлять, масштабировать отдельно.
- **Stateful conversations**: бот помнит контекст диалога в рамках сессии (Redis), профиль участника — в PostgreSQL.
- **AI-first, rule-fallback**: основная логика через LLM с system prompt + RAG. Жёсткие правила только для критических операций (регистрация, репутация).
- **Extensible**: архитектура рассчитана на добавление блокчейн-слоя (Level 2+), Agora (Level 1+), Trace Engine (Level 3+).

---

## 2. МОДУЛИ — ДЕТАЛЬНОЕ ОПИСАНИЕ

### 2.1 Router Module (Маршрутизатор)

**Назначение**: классификация намерения пользователя и маршрутизация к нужному модулю.

**Реализация (MVP)**:
- Intent classification через LLM (Claude) с structured output
- Fallback: keyword-based routing для базовых команд

**Интенты v0.1**:
| Intent | Описание | Целевой модуль |
|--------|----------|----------------|
| `explain_bcs` | Вопросы о BCS, принципах, архитектуре | Knowledge |
| `register` | Регистрация, создание профиля | Profile |
| `my_profile` | Просмотр своего профиля / репутации | Profile + Reputation |
| `get_quest` | Запрос задачи / следующего шага | Quest |
| `quest_submit` | Отправка результата задачи | Quest |
| `explore_domains` | Обзор доменов, кто где работает | Analytics |
| `dao_explain` | Вопросы о голосовании, управлении | Knowledge |
| `feedback` | Обратная связь, предложения | Analytics |
| `unknown` | Не классифицировано | Knowledge (общий) |

**Команды Telegram**:
```
/start — начало, онбординг
/profile — мой профиль
/quest — текущая задача / получить новую
/domains — обзор доменов BCS
/help — помощь
/about — о BCS
```

---

### 2.2 Knowledge Module (База знаний)

**Назначение**: отвечать на любые вопросы о BCS на основе Knowledge Base.

**Реализация**:
1. BCS Knowledge Base v1.1 разбивается на chunks (~500 токенов)
2. Chunks эмбеддятся (модель: `voyage-3` или `text-embedding-3-large`)
3. Хранятся в Qdrant / Chroma
4. При вопросе: embedding query → top-k chunks → Claude с контекстом

**System prompt для Knowledge Module**:
```
Ты — ARQ, навигатор Blockchain State. Твоя задача — объяснять BCS 
простым языком. 

Правила:
1. Никогда не уговаривай вступить. Объясняй и показывай.
2. Всегда заканчивай конкретным действием: "Вот что ты можешь сделать."
3. Если не знаешь ответа — скажи честно.
4. Используй аналогии из повседневной жизни.
5. Никаких вождей и знатоков. BCS — протокол, не организация.
6. Провоцируй agency: "Ты можешь быть первым."
7. Аргументируй от очевидности: быстрее, дешевле, надёжнее.

Тональность: уважительная, конкретная, без пафоса. 
Как инженер объясняет коллеге, не как продавец клиенту.
```

**Обновление KB**: при обновлении файла Knowledge Base — автоматический re-embedding. Скрипт: `update_kb.py`.

---

### 2.3 Profile Module (Профиль участника)

**Назначение**: создание, хранение и управление цифровым профилем.

**Структура данных профиля**:
```python
class UserProfile:
    # Идентификация
    telegram_id: int          # Telegram user ID
    display_name: str         # Имя в системе
    registered_at: datetime   # Timestamp регистрации
    
    # Онбординг
    onboarding_step: int      # 0-5 (0 = не начат, 5 = завершён)
    interests: list[str]      # Выбранные домены интересов
    
    # Активность (MVP — внутренняя БД, позже → блокчейн)
    actions: list[ActionRecord]  # Зафиксированные действия
    quest_history: list[QuestResult]  # История квестов
    
    # Репутация (v0.1 — упрощённая, read-only)
    reputation_vector: dict    # {domain: score}
    total_actions: int
    streak_days: int           # Серия активных дней
    
    # Мета
    last_active: datetime
    language: str              # ru / en
    
class ActionRecord:
    action_id: str             # UUID
    action_type: str           # quest_complete, idea_seed, forum_post, etc.
    domain: str                # Домен компетенции
    timestamp: datetime
    evidence_hash: str         # SHA256 доказательства (для будущей миграции в блокчейн)
    verified: bool
    
class QuestResult:
    quest_id: str
    completed_at: datetime
    score: float               # 0.0 — 1.0
    reviewer: str              # auto / peer / mentor
```

**Регистрация (flow)**:
1. Пользователь нажимает `/start`
2. Бот приветствует, кратко объясняет BCS (3 предложения)
3. Спрашивает имя (или берёт из Telegram)
4. Создаёт профиль в PostgreSQL
5. Генерирует `evidence_hash` для genesis action: "Регистрация в BCS"
6. Переходит к онбордингу

**Важно**: на MVP wallet и SBT НЕ создаются. Профиль хранится в PostgreSQL. Структура данных спроектирована так, чтобы миграция в блокчейн (Level 2+) была бесшовной — все хэши уже вычисляются.

---

### 2.4 Quest Module (Квесты и онбординг)

**Назначение**: структурированные задачи для участников. Онбординг + развитие.

#### Онбординг-квест (обязательный, 5 шагов):

**Шаг 1 — "Понимание"**
Бот задаёт 3 простых вопроса о BCS (multiple choice). Не экзамен — проверка что человек прочитал/послушал. При неправильном ответе — объясняет, не штрафует.

Пример:
```
Вопрос: Что определяет твоё влияние в BCS?
A) Количество денег
B) Доказанный вклад
C) Голосование большинства
D) Решение администратора
→ Правильно: B
```

**Шаг 2 — "Интерес"**
Выбор 1-3 доменов из списка:
- Разработка (код, архитектура, тестирование)
- Координация (управление проектами, DAO, governance)
- Мышление (философия, системный анализ, стратегия)
- Образование (создание курсов, менторство)
- Коммуникация (контент, дизайн, медиа)
- Арбитраж (разрешение споров, медиация)

**Шаг 3 — "Первое действие"**
Микро-задача по выбранному домену:
- Разработка: "Найди одну ошибку в этом псевдокоде" (простой пример)
- Координация: "Опиши в 3 предложениях, как бы ты организовал команду из 5 человек для задачи X"
- Мышление: "Назови одно слабое место BCS и предложи решение"
- И т.д.

Результат сохраняется как первый `ActionRecord`. Evidence hash вычисляется.

**Шаг 4 — "Обзор"**
Бот показывает профиль: "Вот твой первый action record. Timestamp: [дата]. Это навсегда. Твои правнуки увидят, что ты был здесь, когда нас было [N] человек."

**Шаг 5 — "Навигация"**
Бот показывает: текущие открытые квесты в выбранных доменах, количество участников в каждом домене, пустые ниши ("В домене X — 0 человек. Ты можешь быть первым. Навсегда.").

#### Регулярные квесты (после онбординга):

**Типы квестов**:
| Тип | Описание | Верификация | XP |
|-----|----------|-------------|-----|
| `micro` | 5-10 минут, ежедневные | auto | 1-5 |
| `standard` | 1-3 часа, еженедельные | peer review | 10-30 |
| `mission` | 1-2 недели, проектные | committee | 50-100 |
| `genesis` | Создание нового домена/модуля | governance vote | 200+ |

**Генерация квестов (MVP)**:
- v0.1: квесты создаются вручную (Alex / core team) и загружаются в БД
- v0.2+: AI-генерация на основе gaps (где много обсуждений, мало действий)

**Структура квеста в БД**:
```python
class Quest:
    quest_id: str
    title: str
    description: str
    domain: str
    quest_type: str           # micro / standard / mission / genesis
    difficulty: int           # 1-5
    xp_reward: int
    prerequisites: list[str]  # Требуемые завершённые квесты
    verification: str         # auto / peer / committee
    max_participants: int     # 0 = unlimited
    deadline: datetime | None
    created_by: str           # author ID
    status: str               # open / in_progress / completed / expired
```

---

### 2.5 Reputation Module (Репутация)

**Назначение**: вычисление и отображение репутационного вектора участника.

**MVP-реализация (упрощённая)**:

В v0.1 репутация — НЕ на блокчейне. Вычисляется из внутренней БД действий.

```python
def compute_reputation(user: UserProfile) -> dict:
    """
    Упрощённый reputation vector для MVP.
    Полная версия (Shapley, ZK) — Level 3+.
    """
    rep = {}
    for domain in user.interests:
        domain_actions = [a for a in user.actions if a.domain == domain]
        
        # Activity score (быстрый decay)
        activity = sum(
            1.0 * (0.99 ** days_since(a.timestamp))
            for a in domain_actions
        )
        
        # Contribution score (медленный decay)
        contribution = sum(
            a.score * (0.9999 ** days_since(a.timestamp))
            for a in domain_actions
            if a.verified
        )
        
        # Quest completion rate
        quests = [q for q in user.quest_history if q.domain == domain]
        completion_rate = (
            len([q for q in quests if q.score > 0.5]) / max(len(quests), 1)
        )
        
        rep[domain] = {
            "activity": round(activity, 2),
            "contribution": round(contribution, 2),
            "completion_rate": round(completion_rate, 2),
            "total_actions": len(domain_actions),
            "level": compute_level(contribution)  # 1-10
        }
    
    return rep

def compute_level(contribution: float) -> int:
    """Уровень от 1 до 10 по логарифмической шкале."""
    if contribution <= 0: return 1
    import math
    return min(10, 1 + int(math.log2(contribution + 1)))
```

**Отображение в боте**:
```
📊 Твой профиль — @username

Участник с: 25.02.2026 (Genesis Phase)
Действий зафиксировано: 47
Серия активности: 12 дней 🔥

Компетенции:
┌─────────────────────────────┐
│ Разработка     ████████░░ L7│
│ Координация    █████░░░░░ L5│
│ Мышление       ███░░░░░░░ L3│
└─────────────────────────────┘

Следующий шаг: Завершение Quest #12 
даёт +15 XP в домене "Координация" 
и открывает доступ к Mission-квестам.
```

---

### 2.6 Analytics Module (Аналитика)

**Назначение**: сбор и визуализация метаданных активности. Тепловая карта интересов. Данные для контент-стратегии канала.

**Что собирается (анонимно)**:
```python
class AnalyticsEvent:
    event_type: str    # question, registration, quest_start, quest_complete, domain_view
    domain: str | None
    timestamp: datetime
    # НЕ собирается: telegram_id, имя, содержание вопроса
```

**Агрегации для канала**:
- Топ-5 тем вопросов за неделю
- Количество участников по доменам
- "Пустые троны" — домены с 0 участников
- Скорость роста (новые регистрации / день)
- Активные квесты и их статус

**Публикация**: бот автоматически публикует в канал ARQ LIVE еженедельный дайджест "State of the State".

---

## 3. FLOW-ДИАГРАММЫ

### 3.1 Первый контакт

```
User: /start
  ↓
Bot: "Привет. Я ARQ — навигатор Blockchain State.
      BCS — система, где твои действия говорят за тебя.
      Не слова, не дипломы — дела. 
      Хочешь разобраться?"
  ↓
[Кнопки]:
  🔍 Разобраться — что это    → Knowledge flow
  🎯 Начать — создать профиль  → Registration flow
  📊 Посмотреть — что тут      → Analytics flow
```

### 3.2 Регистрация + Онбординг

```
User: [Нажал "Начать"]
  ↓
Bot: "Как тебя называть?" 
  ↓
User: "Alex"
  ↓
Bot: "Alex, добро пожаловать. Профиль создан.
      Timestamp: 2026-02-27T14:30:00Z
      Ты участник #[N]. Это зафиксировано.
      
      Пройдём 5 шагов, чтобы ты понял систему
      и сделал первое действие."
  ↓
[Онбординг: шаги 1-5]
  ↓
Bot: "Готово. Твой первый Action Record:
      🔗 Hash: 7a3f...b2e1
      📅 2026-02-27T14:35:00Z
      📌 Тип: onboarding_complete
      
      Сейчас в BCS [N] участников.
      В домене [X] — [M] человек.
      В домене [Y] — 0. Ноль. Первым будешь ты?"
  ↓
[Кнопки]:
  🎯 Получить первый квест
  📊 Посмотреть домены
  ❓ Задать вопрос
```

### 3.3 Цикл задач

```
User: /quest
  ↓
Bot: [Анализирует профиль: домены, уровень, историю]
  ↓
Bot: "Для тебя есть 3 задачи:

      1. 🟢 [micro] Проверь этот фрагмент кода на ошибки
         Домен: Разработка | XP: 5 | ~10 мин
      
      2. 🟡 [standard] Предложи улучшение для Quest Module
         Домен: Координация | XP: 20 | ~2 часа
      
      3. 🔴 [mission] Спроектируй систему peer-review для Agora
         Домен: Мышление | XP: 75 | ~1 неделя
         ⚠️ Требуется: Координация L4+"
  ↓
User: [Выбирает квест]
  ↓
Bot: "Квест принят. Дедлайн: [дата].
      Отправь результат командой /submit
      или напиши мне — помогу, если застрял."
  ↓
User: /submit [результат]
  ↓
Bot: [Верификация: auto / направляет на peer review]
  ↓
Bot: "✅ Квест завершён!
      +20 XP в домене Координация
      Уровень: 4 → 5
      📊 Новые возможности: Mission-квесты открыты
      
      Следующая цель: ещё 2 standard-квеста
      и ты можешь участвовать в DAO-голосовании
      по вопросам координации."
```

### 3.4 Вопрос о BCS

```
User: "Как работает репутация?"
  ↓
Router: intent = explain_bcs, topic = reputation
  ↓
Knowledge Module: [RAG → top-3 chunks из KB] 
  ↓
Claude: [Генерирует ответ на основе контекста + system prompt]
  ↓
Bot: "Репутация в BCS — не оценка и не рейтинг. 
      Это вычисляемый профиль твоих реальных действий.
      
      Представь: вместо резюме, где ты пишешь 
      «умею программировать», система видит: 
      47 завершённых задач по коду, 
      3 peer-review других участников, 
      1 архитектурное предложение, принятое в работу.
      
      Два типа памяти:
      • Свежая активность — затухает, если не действуешь
      • Исторический вклад — сохраняется надолго
      
      Хочешь посмотреть свой текущий профиль?"
  ↓
[Кнопки]:
  📊 Мой профиль
  🎯 Как повысить репутацию?
  ❓ Другой вопрос
```

---

## 4. ТЕХНИЧЕСКИЙ СТЕК

### 4.1 Рекомендуемый стек

| Компонент | Технология | Причина выбора |
|-----------|-----------|----------------|
| Язык | Python 3.11+ | Экосистема AI/ML, быстрая разработка |
| Telegram | aiogram 3.x | Async, modern, хорошая документация |
| AI | Claude API (Anthropic) | Качество рассуждений, system prompt |
| Embeddings | voyage-3 / OpenAI ada-002 | Для RAG |
| Vector DB | Qdrant (self-hosted) или Chroma | RAG-поиск по KB |
| Database | PostgreSQL 16 | Профили, квесты, аналитика |
| Cache | Redis | Сессии, rate limiting |
| Hosting | VPS (Hetzner / DigitalOcean) | Начальный этап |
| CI/CD | GitHub Actions | Автодеплой |
| Мониторинг | Sentry + простой dashboard | Ошибки, метрики |

### 4.2 Структура проекта

```
bcs-core-agent/
├── bot/
│   ├── __init__.py
│   ├── main.py              # Entry point, bot startup
│   ├── config.py             # Environment variables, settings
│   ├── router.py             # Intent classification
│   ├── handlers/
│   │   ├── start.py          # /start, registration
│   │   ├── profile.py        # /profile
│   │   ├── quest.py          # /quest, /submit
│   │   ├── knowledge.py      # Free-form questions
│   │   ├── domains.py        # /domains
│   │   └── admin.py          # Admin commands
│   ├── modules/
│   │   ├── knowledge.py      # RAG engine
│   │   ├── profile.py        # Profile CRUD
│   │   ├── quest.py          # Quest logic
│   │   ├── reputation.py     # Reputation computation
│   │   └── analytics.py      # Event tracking
│   ├── models/
│   │   ├── user.py           # SQLAlchemy models
│   │   ├── quest.py
│   │   └── action.py
│   ├── services/
│   │   ├── claude.py         # Claude API wrapper
│   │   ├── embeddings.py     # Embedding + vector search
│   │   └── publisher.py      # Channel publishing
│   └── utils/
│       ├── hashing.py        # SHA256, evidence hashes
│       └── formatting.py     # Telegram message formatting
├── data/
│   ├── knowledge_base/       # KB markdown files
│   ├── quests/               # Quest definitions (YAML)
│   └── prompts/              # System prompts
├── scripts/
│   ├── update_kb.py          # Re-embed KB
│   ├── seed_quests.py        # Load quests into DB
│   └── migrate.py            # DB migrations
├── tests/
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── .env.example
└── README.md
```

### 4.3 Environment Variables

```env
# Telegram
TELEGRAM_BOT_TOKEN=xxx
TELEGRAM_CHANNEL_ID=@arq_live

# AI
ANTHROPIC_API_KEY=xxx
EMBEDDING_API_KEY=xxx
CLAUDE_MODEL=claude-sonnet-4-5-20250929

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/bcs
REDIS_URL=redis://localhost:6379

# Vector DB
QDRANT_URL=http://localhost:6333
QDRANT_COLLECTION=bcs_knowledge

# App
LOG_LEVEL=INFO
ADMIN_IDS=123456789  # Telegram IDs админов
```

---

## 5. ДАННЫЕ И КВЕСТЫ

### 5.1 Формат квестов (YAML)

```yaml
# data/quests/onboarding.yml
- id: onboarding_01
  title: "Понимание BCS"
  type: micro
  domain: general
  difficulty: 1
  xp: 5
  verification: auto
  content:
    type: quiz
    questions:
      - text: "Что определяет твоё влияние в BCS?"
        options:
          - "Количество денег"
          - "Доказанный вклад"  # correct
          - "Голосование большинства"
          - "Решение администратора"
        correct: 1
      - text: "Что происходит с репутацией, если ты неактивен?"
        options:
          - "Ничего"
          - "Удаляется"
          - "Постепенно снижается"  # correct
          - "Передаётся другому"
        correct: 2

# data/quests/dev_micro.yml
- id: dev_micro_01
  title: "Найди баг"
  type: micro
  domain: development
  difficulty: 1
  xp: 5
  verification: auto
  prerequisites: [onboarding_05]
  content:
    type: code_review
    code: |
      def reputation_decay(score, days):
          return score * 0.99 ** days
          if score < 0:
              return 0
    task: "Найди проблему в этом коде и опиши её."
    check_keywords: ["unreachable", "dead code", "после return", "недостижимый"]
```

### 5.2 Начальный набор квестов (для запуска)

Нужно подготовить минимум:
- 5 онбординг-квестов (обязательные)
- 10 micro-квестов (по 2 на каждый домен)
- 3 standard-квеста (для продвинутых)
- 1 genesis-квест ("Предложи новый домен для BCS")

---

## 6. МЕХАНИКА ОБНОВЛЕНИЯ

### 6.1 Knowledge Base

```
1. Обновить .md файл(ы) в data/knowledge_base/
2. Запустить: python scripts/update_kb.py
   → Разбивает на chunks
   → Эмбеддит
   → Загружает в Qdrant
   → Логирует: "KB updated: N chunks, M new"
3. Бот автоматически использует обновлённую KB
```

### 6.2 Квесты

```
1. Добавить/обновить YAML в data/quests/
2. Запустить: python scripts/seed_quests.py
   → Загружает в PostgreSQL
   → Логирует: "Quests seeded: N new, M updated"
3. Бот видит новые квесты немедленно
```

### 6.3 System Prompts

```
1. Обновить файл в data/prompts/
2. Перезапуск бота не требуется — prompts загружаются при каждом запросе
```

### 6.4 Код бота

```
1. Push в main branch → GitHub Actions → Docker build → Deploy
2. Zero-downtime: новый контейнер стартует, старый останавливается
```

---

## 7. УРОВНИ АВТОНОМНОСТИ

### Level 0 — Echo (MVP, этот документ)
- Отвечает на вопросы (RAG)
- Регистрирует участников
- Проводит онбординг
- Выдаёт квесты из готового набора
- Показывает профиль и репутацию (из внутренней БД)
- Собирает аналитику

**Бот НЕ умеет**: создавать квесты автоматически, проводить peer review, запускать DAO-голосование, работать с блокчейном.

### Level 1 — Pulse (+ 1-2 месяца)
- AI-генерация квестов на основе gaps
- Автоматическое сведение участников ("Вы оба интересуетесь X")
- Тематические треды в канале по данным аналитики
- Peer review для standard-квестов (участники оценивают друг друга)

### Level 2 — Nerve (+ 3-6 месяцев)
- Подключение блокчейна (DID, SBT)
- Миграция Action Records из PostgreSQL в on-chain
- Wallet integration
- DAO-голосование через бота (простое: за/против/воздержался)
- Quest system с bounties (токены)

### Level 3 — Cortex (+ 6-12 месяцев)
- Trace Engine (PageRank для каузальных цепочек)
- Agora integration (Seed → Fork → Challenge → Build)
- Полноценный reputation vector на блокчейне
- ZK-proofs для selective disclosure
- Арбитраж через бота

### Level 4 — Autonomy (+ 12 месяцев)
- Экосистема AI-агентов
- Cross-domain reputation
- Governance через weighted voting
- Self-evolving quest generation
- Bridge к внешним системам (GitHub, LinkedIn, etc.)

---

## 8. МЕТРИКИ УСПЕХА MVP

| Метрика | Цель (1 месяц) | Цель (3 месяца) |
|---------|-----------------|-------------------|
| Регистрации | 100 | 1000 |
| Онбординг completion rate | >70% | >80% |
| Daily Active Users | 20 | 200 |
| Квестов выполнено | 500 | 5000 |
| Среднее время ответа бота | <3 сек | <2 сек |
| KB accuracy (ручная проверка) | >85% | >95% |
| Retention (неделя) | 30% | 50% |

---

## 9. БЕЗОПАСНОСТЬ

- Telegram Bot Token — только в .env, не в коде
- API ключи — через environment variables
- Rate limiting: max 10 сообщений/минуту на пользователя (Redis)
- Input sanitization: все пользовательские данные проходят валидацию
- SQL injection protection: SQLAlchemy ORM, никаких raw queries
- Admin commands: только для ADMIN_IDS
- Данные пользователей: не продаются, не передаются, удаляются по запросу
- Аналитика: только агрегированные данные, без PII

---

## 10. ПЛАН РАЗРАБОТКИ (sprints)

### Sprint 1 (неделя 1): Скелет
- [ ] Настройка проекта (repo, docker, CI/CD)
- [ ] Telegram bot scaffold (aiogram)
- [ ] PostgreSQL schema + migrations
- [ ] Базовые команды: /start, /help, /about
- [ ] Заглушки для всех модулей

### Sprint 2 (неделя 2): Мозг
- [ ] Knowledge Module: RAG pipeline
- [ ] Загрузка и эмбеддинг KB v1.1
- [ ] Claude integration (dialogue)
- [ ] Router (intent classification)
- [ ] Свободный диалог о BCS работает

### Sprint 3 (неделя 3): Профиль + Онбординг
- [ ] Profile Module: регистрация, хранение
- [ ] Онбординг flow (5 шагов)
- [ ] Action Record создание + evidence hash
- [ ] Отображение профиля (/profile)

### Sprint 4 (неделя 4): Квесты + Репутация
- [ ] Quest Module: YAML загрузка, выдача, submission
- [ ] Reputation computation (упрощённая)
- [ ] Analytics Module: сбор событий
- [ ] Автопубликация в канал (еженедельный дайджест)

### Sprint 5 (неделя 5): Полировка + Запуск
- [ ] Тестирование (unit + integration)
- [ ] Нагрузочное тестирование (100 concurrent users)
- [ ] Копирайт всех сообщений бота (тональность)
- [ ] Подготовка первых 20+ квестов
- [ ] Soft launch (приглашённые участники)

---

## 11. КОНТАКТЫ И КОММУНИКАЦИЯ

- **Product Owner**: Alex (Oleh)
- **Разработчик**: vell_01
- **Коммуникация**: [указать канал — Telegram группа / Discord]
- **Код**: [указать repo URL]
- **Knowledge Base**: прилагается (BCS_Knowledge_Base_v1.1.md)
- **Обновления ТЗ**: через Pull Requests в repo

---

## ПРИЛОЖЕНИЯ

### A. Knowledge Base
Файл: `BCS_Knowledge_Base_v1.1.md` (прилагается отдельно)

### B. System Prompts
Файл: `prompts/arq_system.txt` (будет создан в Sprint 2)

### C. Quest Database
Файл: `data/quests/*.yml` (будет создан в Sprint 4)

### D. Глоссарий ключевых терминов
См. BCS Knowledge Base v1.1, раздел "Глоссарий"
