<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" width="220" alt="Fill Me In: человек выглядывает из-за двери, всё пропустив">
  </picture>
</p>

<h1 align="center">Fill Me In</h1>

<p align="center">
  <em>Ты зашёл на середине. Он введёт в курс.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/quiqqe/fill-me-in?style=flat-square&color=111111&label=stars" alt="Stars">
  <img src="https://img.shields.io/badge/claude%20code-plugin-111111?style=flat-square" alt="Плагин Claude Code">
  <img src="https://img.shields.io/badge/eval-62%2F72%20vs%2050%2F72-111111?style=flat-square" alt="Эвал: 62/72 понятных ответов с плагином против 50/72 без">
  <img src="https://img.shields.io/github/v/release/quiqqe/fill-me-in?display_name=release&style=flat-square&color=111111&label=release" alt="Release">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="Лицензия MIT">
</p>

<p align="center">
  <strong>Сначала суть &middot; что, почему, что делать &middot; глубина по ходу разговора &middot; <code>/fill-me-in</code>, если ответ не зашёл</strong>
</p>

<p align="center">
  <sub><a href="README.md">English</a></sub>
</p>

---

Знакомо. Агент отчитывается о том, что только что сделал:

> Fixed: dropped `set $php_fpm`, moved to upstream with zone + resolve, valid=1s. Recreating php no longer needs a reload.

Каждое слово знакомо. Смысла ноль. Агент всё это время был в комнате: читал файлы, отбросил две гипотезы, как-то называл вещи по ходу. И пишет так, будто ты был рядом. Пишешь «не понял» — и получаешь то же самое, только длиннее.

Fill Me In заставляет агента писать для того, кто только что вошёл.

## До / после

На сообщение выше отвечаешь «I don't get it. What does that mean?». Настоящие ответы из прогона эвалов, одна модель, один запрос (начало ответов; целиком — в [examples/](examples/fill-me-in-unclear.md)):

**Без Fill Me In** — суть закопана под извинением и описанием проблемы:

> Sorry, that was too terse. Here it is in plain terms.
>
> **The problem:** nginx forwards PHP requests to your `php` container by its name, `php`. When the container is recreated (after a rebuild or a `docker compose up` that replaces it), Docker often gives it a new IP address. Out of the box, nginx looks up the name once at startup and keeps using that IP…

**С Fill Me In** — сначала суть, потом механизм на конкретных адресах:

> **In short:** nginx used to lose track of your PHP container whenever that container was recreated. I changed the config so nginx re-checks the container's address on its own, so you no longer have to reload nginx after recreating PHP.
>
> **What was going wrong.** nginx passes PHP requests to the PHP container by name, something like `php:9000`. Docker turns that name into an IP address, for example `172.18.0.5`. By default nginx does that lookup once, at startup, and keeps using that IP. When you recreate the PHP container, it usually comes back with a new IP, say `172.18.0.7`…

В этой ситуации судья признал понятными **0 из 12** ответов без плагина и **8 из 12** с ним.

## Цифры

Шесть сценариев, где агент отчитывается о работе, которую ты не видел. Каждый прогнан 12 раз с плагином и без. Судья Sonnet оценивает каждый ответ по одному вопросу: *поймёт ли человек, не видевший работы, что было не так, почему и что дальше?*

| ситуация | понятно с плагином | без | шанс, что это случайность |
|---|--:|--:|--:|
| «не понял» после скупого ответа | **8/12** | 0/12 | **0.1%** |
| пользователь — эксперт (не разжёвывать) | 7/12 | 4/12 | 41% |
| нестабильный тест: «это был баг?» | 11/12 | 10/12 | — |
| отчёты про миграцию, nginx, очередь | 36/36 | 36/36 | — |
| **всего** | **62/72** (86%) | 50/72 (69%) | **2.7%** |

Что это показывает и что нет:

- **Доказано:** когда ответ не зашёл и ты переспрашиваешь, плагин превращает пересказ теми же словами в настоящее объяснение.
- **Не доказано:** что первые отчёты становятся понятнее. В этих сценариях модель и без него объясняет хорошо, а оставшиеся разницы в пределах шума.
- **Не проверено:** приоритет над плагинами «пиши короче» (ниже) и настоящие многочасовые сессии — эвал из одного сообщения и имитирует потерю контекста рабочими заметками агента.
- **Цена:** ответы длиннее примерно на 40% (338 слов против 241).

Методика, результаты по кейсам и ограничения: [evals/results](evals/results/README.md). Повторить: `claude plugin eval . --runs 12 --judge-model sonnet` (около $12).

## Как это работает

Две части в одном плагине.

**Правило на каждый ответ.** Хук `SessionStart` (а также после `/clear` и сжатия контекста) загружает его, а напоминание в одну строку едет с каждым твоим сообщением, чтобы другие инструкции его не заглушили:

```
1. Сначала суть        → одна простая фраза: что произошло
2. Потом ситуация       → что происходило, что агент нашёл
3. Каждый вывод        → что, почему, что это значит для тебя
4. Без личного контекста → «тот баг», «вторая гипотеза» объяснены или убраны
5. Конкретика           → «запрос X вернёт Y, потому что Z»
6. Глубина по тебе      → по теме, по твоим словам, а не по ощущению агента «это очевидно»
```

Глубину агенты обычно угадывают неправильно, поэтому она берётся из разговора, а не из догадки:

| Ты… | Агент… |
|---|---|
| сам используешь термины темы, поправляешь по существу | пропускает азы, не пересказывает тебе твои же слова |
| спрашиваешь, что значит термин, пишешь «не понял» | объясняет механизм и держит этот уровень по теме |
| никак не показываешь | объясняет: лишняя фраза дешевле непонятного ответа |

**`/fill-me-in`, если ответ всё же не зашёл.** Агент находит невидимый тебе контекст, на который опирался, перепроверяет ключевые факты по источнику, а не по памяти, и перерассказывает: суть в одной строке, механизм, пример по шагам, что из этого следует. `/fill-me-in <место>` разбирает только это место.

## Уживается с «пиши короче»

Стоят [ponytail](https://github.com/DietrichGebert/ponytail), [caveman](https://github.com/JuliusBrussee/caveman) или своё правило «максимум три строки»? Оставь. Fill Me In говорит агенту: пусть они режут код, повторы и вежливость, но не контекст, причины и механизм, — и повторяет это с каждым сообщением, чтобы не заглушили. Прямая просьба в сообщении («только код») главнее обоих. Этот приоритет — инструкция модели, а не гарантия, и вместе с такими плагинами эвал его пока не измерял.

## Что он запускает

Два хука типа `command`, оба — `cat` текстового файла внутри плагина: `hooks/rule.md` при старте сессии и `hooks/reminder.md` на каждое сообщение. Их вывод попадает в контекст агента. Никаких сетевых запросов, скриптов и зависимостей, ничего не собирается и не хранится. Skill `/fill-me-in` — обычный `SKILL.md`.

## Установка

### Claude Code

```
/plugin marketplace add quiqqe/fill-me-in
/plugin install fill-me-in@quiqqe
```

Начинает работать со следующей сессии. Обновить до свежей версии (в терминале):

```
claude plugin update fill-me-in@quiqqe
```

### Только skill, в любом агенте

Один `/fill-me-in`, без постоянного правила — для агентов, которые понимают `SKILL.md`:

```bash
npx skills add quiqqe/fill-me-in
```

### Удаление

```
/plugin uninstall fill-me-in@quiqqe
```

<details>
<summary><strong>Почему плагин, а не просто skill?</strong></summary>

Skill загружается, только когда агент решит, что задача под него подходит. «Пиши понятно» нужно в каждом ответе, и как skill оно промахнулось бы ровно на быстрых ответах, где контекст и теряется. Плагин через хук загружает правило при каждом старте сессии и при этом даёт `/fill-me-in` про запас.

</details>

## Лицензия

MIT
