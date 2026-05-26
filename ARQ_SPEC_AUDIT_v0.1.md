# ARQ Core Agent Spec — Audit vs Foundation v0.1

> Аудит `bcs_core_agent_spec.md` (от 2026-02-27) против `BS_PROTOCOL_FOUNDATION_v0.1.md` (от 2026-05-26). Дата аудита: 2026-05-26.

## TL;DR

**Текущая spec ARQ реализует gamified learning platform, а не runtime BS-протокола.** Архитектура, профили и инфраструктурный слой — пригодны как есть. **Core primitive (Quest)** противоречит трём инвариантам foundation и должен быть заменён на Q/S/T до начала кодирования.

Аудит не предлагает переписать spec целиком сейчас. Предлагает **аддендум** (`ARQ_SPEC_ADDENDUM_v0.2.md`), который патчит несовместимые места и переопределяет core primitive.

---

## 1. Проверка по инвариантам

### Инвариант 1: Question ≠ task. Q = напряжение.

**Статус: ❌ нарушен.**

Spec строит всю операционку вокруг `Quest`:
- §2.4 "Quest Module" — основной модуль работы пользователя
- §5.1 YAML-формат квестов с полями `quest_type`, `difficulty`, `xp_reward`, `deadline`
- §2.6 Quest verification: `auto / peer / committee`

`Quest` в spec — это **предварительно сформулированная задача**, которую пользователь "берёт" и "сдаёт". Это в точности то, что инвариант 1 запрещает: тикет-система. Пользователь не производит Q, он потребляет уже сформулированный.

**Что нужно:** заменить `Quest` на `Question` как entry primitive. Пользователь приносит **сформулированное напряжение** (или ARQ помогает сформулировать из тумана). "Сдача результата" не существует — есть Assembly Trace, см. инвариант 3.

### Инвариант 2: Spark must carry cost-of-error.

**Статус: ❌ отсутствует.**

Spec не содержит концепции falsifiability в submissions. Verification в §2.5 — это `auto/peer/committee`, что отвечает на вопрос "хорошо ли сделано", а не "что станет видимым, если эта гипотеза неверна".

Reputation в §2.5 считается из количества действий + completion rate. Это count-based metric, а не quality-of-hypothesis metric.

**Что нужно:** добавить Spark как первый-класс примитив с обязательным полем `cost_of_error`. Trace Engine метрика должна включать долю Spark, чья cost-of-error реально проверилась (validated/falsified count), а не просто action count.

### Инвариант 3: Initiator ≠ Assembler.

**Статус: ❌ структурно отсутствует.**

Spec не содержит Trace lifecycle. Workflow в §3.3:
```
User picks quest → submits → gets verified (auto/peer/committee)
```

Здесь:
- Quest создан core team (одной стороной)
- Submitter — другая сторона
- Verifier — третья
- Но это разделение **внутри одного task'а сверху-вниз**, не Initiator/Assembler одного цикла

Trace в текущем spec — это `ActionRecord` (см. §2.3): запись факта, не закрытый цикл сотрудничества.

**Что нужно:** добавить полный Q→S→T lifecycle с явным запретом "Initiator закрывает свой Trace". Это не social rule, это check в коде Quest Module (после rename → Question Module).

---

## 2. Что хорошо и должно остаться

| Компонент | Где | Почему сохраняем |
|---|---|---|
| Архитектура модулей | §1.1 | Router/Knowledge/Profile/Analytics — корректная декомпозиция |
| `ActionRecord` структура | §2.3 | `evidence_hash`, timestamp, domain — готов под будущий blockchain anchoring |
| Decay-математика репутации | §2.5 | Двухскоростной decay (activity vs contribution) корректно отражает 3 временных режима из `BS_SEED_DOCUMENT_v0.3.md` |
| Knowledge Module system prompt | §2.2 | "Никаких вождей и знатоков. Тональность инженер коллеге, не продавец клиенту" — идеально соответствует foundation Vocabulary §6 |
| "Пустые троны" pattern | §3.2 | "В домене X — 0 человек. Ты можешь быть первым" — корректный filter-механизм |
| Tech stack | §4 | Python + aiogram + Claude + Postgres + Qdrant — sane defaults |
| Profile data model | §2.3 | Сохранить, переименовать `actions` → `traces`, `quest_history` → `participation_log` |

## 3. Что заменить (краткий план патча)

| Что | Что становится | Затрагивает |
|---|---|---|
| `Quest` module | `Question` + `Spark` + `Trace` modules | §2.4 переписать |
| `XP reward` | `Trace contribution score` (Spark validated/falsified ratio) | §2.5 расширить, не упрощать |
| Top-down quest creation (`scripts/seed_quests.py`) | User-formulated Q + AI-assisted formulation | §5 переписать, удалить `quests/` директорию из структуры |
| Quest YAML формат | Trace markdown формат (`open/`, `sealed/`, `stale/`) | §5.1 удалить, добавить новый раздел |
| "Submit result" workflow | Spark publish → Assembly → Trace seal | §3.3 переписать полностью |
| Verification `auto/peer/committee` | Spark cost-of-error validation lifecycle | §2.4 новая логика |
| `Action Record` для onboarding | `Trace #0001` для первой Q пользователя (например "Каково моё напряжение") | §3.2 onboarding flow адаптировать |

## 4. Что не входит в этот аудит (но требует решения отдельно)

- **Cross-cell routing.** Когда Q одной cell может стать Signal для другой cell — отложено до Phase 1.
- **Cell membership management.** Как формируется cell, кто добавляет/исключает — out of scope ARQ MVP. Возможно решается через kChat invite + ARQ только обслуживает Q/S/T внутри.
- **Encyclopedia publication pipeline.** Из sealed Trace → публикация на `mathr.ch/traces/`. Это отдельный workflow, не часть ARQ bot.
- **Conflict resolution.** Что если два Sparker'а противоречат друг другу — оба идут в Trace, Assembler синтезирует, это OK. Но что если Initiator не согласен с Assembly? — Reformulation как валидный исход (см. foundation §2 step 8).

## 5. Рекомендация по реализации

**Не переписывать `bcs_core_agent_spec.md` сейчас.** Он содержит много валидной инфраструктурной информации, переписывать его — это убить рабочий артефакт.

**Что сделать:**

1. Написать `ARQ_SPEC_ADDENDUM_v0.2.md` — short document, который:
   - Объявляет foundation v0.1 как авторитетный источник core primitives
   - Списком переопределяет несовместимые части §2.4, §5 и §3.3 оригинального spec
   - Оставляет всё остальное в силе

2. В коде реализации использовать addendum как primary source, оригинальный spec — как secondary reference для архитектуры.

3. После того, как первая cell прошла 3-5 Trace, написать `bcs_core_agent_spec_v0.3.md` — консолидированный rewrite. Не раньше.

---

## 6. Открытые вопросы

1. **Domains.** Spec в §2.4 фиксирует 6 доменов сверху (Разработка/Координация/Мышление/Образование/Коммуникация/Арбитраж). Foundation говорит "домены эмерджентны". Конфликт. Решение: оставить 6 доменов как **seed** для cold start, разрешить cell вводить новые через Rule Update.

2. **MVP-репутация без Trace Score.** Foundation §8 явно выводит Trace Score за scope v0.1. Spec §2.5 включает компиляцию reputation_vector. Решение: оставить вычисление, но **не отображать в боте** до тех пор, пока не накопится 100+ закрытых Trace. До этого — только "X Trace participated", "Y Trace assembled".

3. **Onboarding flow.** Текущий §3.2 завязан на quest selection. После замены на Q-flow онбординг должен быть: пользователь приводит первое напряжение, ARQ помогает сформулировать его как Q, это становится первым Trace пользователя.

---

*v0.1 — 2026-05-26.*
