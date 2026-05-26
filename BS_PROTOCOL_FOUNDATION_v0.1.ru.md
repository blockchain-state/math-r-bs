# BS PROTOCOL — FOUNDATION v0.1

> Operating layer фундамента. Не онтология (она в `BS_SEED_DOCUMENT_v0.3.md`), не tech spec (она в `bcs_core_agent_spec.md`). Это **минимальный контур**, который любой первый node должен понимать одинаково с инвестором и грантовым экспертом.

---

## 1. Operating definition

**BS Protocol / ARQ MVP** — минимальный протокол сотрудничества, где субъекты сами производят вопросы, решения, следы и правила взаимодействия, а система помогает маршрутизировать путь от смутной проблемы до зафиксированного результата.

*EN-версия для Innosuisse / лендинга:*
> An AI-guided protocol for adaptive collaboration. Helps small groups formulate problems, distribute attention, record decisions, and adapt their rules of working together as complexity grows.

## 2. Практический маршрут

Один цикл сотрудничества — восемь шагов:

```
Problem  →  Question  →  Trace  →  Signal  →  Coordination  →  Action  →  Result  →  Rule Update
```

| Шаг | Что происходит | Кто двигает | Где это |
|---|---|---|---|
| **Problem** | Внутренняя неясность | Любой человек | До системы |
| **Question** | Превращение неясности в сформулированный Q | ARQ помогает | ARQ bot |
| **Trace** | Q + первые ходы фиксируются как артефакт | ARQ + автор | Persistent storage |
| **Signal** | Trace становится сигналом для других | Система маршрутизирует | ARQ + cell channel |
| **Coordination** | Несколько человек подхватывают, распределяют внимание | Cell (3–5 ppl) | ARQ + kChat |
| **Action** | Выполнение того, что было решено | Cell | Реальный мир |
| **Result** | Зафиксированное изменение | Cell | Trace updated |
| **Rule Update** | Если выявилась нужная правка процедуры — она вносится | Cell через BS-форк | Foundation v.NEXT |

MVP должен показать **хотя бы один полный цикл** на 3–5 человек.

## 3. Три структурных инварианта

Эти три правила не подлежат форку — без них протокол схлопывается:

1. **Question ≠ task.** Q — это сформулированное напряжение, не запрос на работу. Без этого Q деградирует в Jira-тикет, и протокол становится task manager'ом.

2. **Spark must carry cost-of-error.** Любое предложение направления разрешения обязано фиксировать "что станет видимым, если гипотеза неверна". Без этого Trace накапливает мнения, а не верифицируемые ходы.

3. **Initiator ≠ Assembler.** Тот, кто сформулировал Q, не может быть тем, кто закрывает Trace. Это структурный анти-self-validation invariant. Защищает цикл от схлопывания на одного человека независимо от размера cell.

Всё остальное — окна, форматы, роли, дедлайны, storage layout — определяется самой cell и эволюционирует через шаг **Rule Update** (см. §2).

## 4. Self-amending principle

Cell применяет Q/S/T к самой процедуре сотрудничества. Когда что-то ломается или работает плохо — это становится Q, проходит цикл, и Rule Update изменяет процедуру для следующих циклов.

**Что эволюционирует:** окна сбора, форматы Trace, состав ролей, способы маршрутизации, критерии закрытия.

**Что не эволюционирует:** три инварианта из §3. Они защищают сам факт того, что эволюция остаётся протокольной, а не превращается в произвол.

## 5. Где живёт что

| Слой | Артефакт | Назначение |
|---|---|---|
| **Constitutional** | этот документ + `BS_SEED_DOCUMENT_v0.3.md` | Инварианты, философия |
| **Runtime** | ARQ Live bot (см. `bcs_core_agent_spec.md`) | Алгоритм маршрутизации, исполнение протокола |
| **Surface** | mathr.ch | Объяснение того, что это, для кого, как войти |
| **Cell conventions** | Каждая cell ведёт свой `CELL_LOG.md` | Локальные правила, эволюционирующие через Rule Update |

ARQ Live bot — это не "красивый собеседник". Это **проводник из внутренней неясности в координируемое действие.**

## 6. Vocabulary: internal ↔ external

Слова, на которых мы разговариваем внутри — не всегда работают наружу.

| Internal | External (Innosuisse / landing / cold contacts) |
|---|---|
| Стая | Protocol cell / guided collaboration group |
| Нода | Participant / contributor |
| Trace | Recorded decision cycle |
| Spark | Proposed direction |
| Guided elevation of protocol | AI-guided protocol onboarding for adaptive collaboration |
| BS Protocol | Adaptive collaboration protocol (in technical contexts) |
| Forking the rule | Updating cell conventions |
| Свадхарма / призвание | Vocation alignment (в текстах для исследовательской аудитории) |

Правило: внутри сохраняем живой язык, наружу переводим в архитектурный/методологический. Никогда не сводим к "ещё одной DAO" или "AI-tool".

## 7. Phase 0 — что нужно доказать

Не "что протокол работает на тысячах". Только три вещи:

1. **Single-user route.** Один человек проходит через ARQ от тумана до точного Q и Trace.
2. **Small pack flow.** 3–5 человек проходят полный цикл §2 на одной реальной задаче.
3. **Rule Update в реале.** Cell обнаруживает разрыв в процедуре, проводит Q/S/T по самой процедуре, обновляет свой `CELL_LOG.md`.

Если эти три демо есть — мы можем разговаривать с Innosuisse, первыми нодами и серьёзной аудиторией не как утопия, а как механизм.

## 8. Что НЕ в этом документе

- Trace Score математика
- Cross-cell routing
- Blockchain anchoring, tokens, anti-Sybil
- Goverence между cells
- ARQ tech architecture (это в `bcs_core_agent_spec.md`)

Эти слои открываются после того, как Phase 0 §7 закрыт.

---

*v0.1 — 2026-05-26. Open for fork по правилам Rule Update (§2 + §4).*
