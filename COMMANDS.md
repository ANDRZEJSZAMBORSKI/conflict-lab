# conflict-lab — справочник команд

Мини-проект: Git branching, PR (`gh`), conflict resolution.  
Сценарий: `theme=light` → ветки `dark` / `auto` → PR#1 без конфликта → PR#2 с конфликтом → resolve на feature → `gh pr merge` (финиш на master).

Стандарт учёбы: зачем → что меняется → команда → проверка глазами.

---

## 0. Карта: когда что трогает

| Команда | Диск (файлы/ветки) | `.git` история | GitHub |
|--------|--------------------|----------------|--------|
| `add` / `commit` | да (commit фиксирует) | да | нет |
| `switch` / `branch` | да (рабочие файлы) | указатели веток | нет |
| `fetch` | нет* | обновляет `origin/*` | читает |
| `merge` | да | да | нет (пока не push) |
| `pull` | да | да | читает |
| `push` | нет | нет | пишет |
| `gh pr ...` | иногда (gh может подтянуть master) | иногда | да |

\*После одного `fetch` файлы текущей ветки не меняются, пока не сделаешь `merge`.

---

## 1. Навигация / файлы (shell)

### `cd /d/pyton_2/git/conflict-lab`
Перейти в папку проекта. Git всегда смотрит на **текущую** папку.  
Урок: `git init` без `cd` в `conflict-lab` завёл репо в родителе `/d/pyton_2/git` — пришлось откатывать.

### `pwd`
Показать текущий путь. Проверка «я там, где думаю».

### `ls -la`
Список файлов, включая скрытые (`.git`).

### `mkdir conflict-lab`
Создать папку проекта.

### `echo "theme=light" > SETTINGS.txt`
Записать одну строку в файл (перезаписать файл целиком).  
Также для resolve: `echo "theme=auto" > SETTINGS.txt`.

### `cat SETTINGS.txt`
Показать содержимое. Обязательная проверка до/после merge и конфликта.

### `rm -rf .git` / `rm -f SETTINGS.txt`
Удаление (откат ошибочного репо в родителе).  
**Опасно**, если `pwd` не тот — всегда проверяй `pwd` перед `rm -rf .git`.

---

## 2. Создать репозиторий

### `git init -b master`

| Часть | Смысл |
|-------|--------|
| `init` | создать `.git` |
| `-b master` | имя первой ветки `master` |

**Альтернативы:**

```bash
git init
git branch -M master    # переименовать текущую ветку в master
```

или сразу `main`:

```bash
git init -b main
```

---

## 3. Смотреть состояние и историю

### `git status`
Главная проверка: ветка, clean/dirty, staged, `|MERGING`, tracking.

### `git log --oneline`
Короткая история.

Частые добавки:

```bash
git log --oneline -1          # последний
git log --oneline -3          # три
git log --oneline -4
git log --oneline --graph --all -6   # граф всех веток
git log --oneline -1 origin/master   # где remote master
```

**Альтернатива** подробнее:

```bash
git log --oneline --decorate --graph --all
```

### `git branch`
Локальные ветки, `*` = текущая.

### `git branch -vv`
Плюс tracking (`[origin/...]`) и последний коммит.

### `git remote -v`
Имя remote → URL (fetch/push).

---

## 4. Коммиты

### `git add SETTINGS.txt`
В staging.

**Альтернативы:**

```bash
git add .                 # всё в папке (осторожно)
git add -A                # включая удаления
git add -p                # кусками (позже)
```

### `git commit -m "сообщение"`
Снимок staging в историю **локально**.

**Альтернативы:**

```bash
git commit                # откроет редактор
git commit --no-edit      # взять готовое merge-сообщение
git commit -am "msg"      # add+commit только уже tracked файлов (новые не возьмёт)
```

---

## 5. Remote

### `git remote add origin URL`
Привязать GitHub как `origin`.

Пример:

```bash
git remote add origin https://github.com/ANDRZEJSZAMBORSKI/conflict-lab.git
```

**Альтернативы:**

```bash
git remote rename origin old-origin
git remote remove origin
git remote set-url origin NEW_URL
```

---

## 6. Ветки: создать / перейти / удалить

### `git switch master`
Перейти на существующую ветку.

### `git switch -c feature/theme-dark`
Создать и перейти (`-c` = create).

**Альтернативы (старый стиль, то же по смыслу):**

```bash
git checkout master
git checkout -b feature/theme-dark
```

Современный Git:

- `switch` — ветки
- `restore` — откат файлов

### `git branch -d feature/theme-dark`
Удалить локальную ветку (безопасно, если влито).

**Альтернативы:**

```bash
git branch -D feature/theme-dark              # принудительно (опасно)
git push origin --delete feature/theme-dark   # удалить на GitHub
```

### `git fetch --prune`
Обновить remote-ссылки и убрать призраки удалённых веток (`origin/feature/...`).

---

## 7. Обмен с GitHub: push / fetch / merge / pull

### `git push -u origin master`

| Часть | Смысл |
|-------|--------|
| `push` | отправить коммиты |
| `-u` | запомнить upstream |
| `origin` | remote |
| `master` | ветка |

Потом достаточно:

```bash
git push
```

если upstream уже есть.

То же для feature:

```bash
git push -u origin feature/theme-dark
git push origin feature/theme-auto    # после resolve
```

---

### `git fetch origin`
Скачать обновления в `origin/master`, `origin/feature/...`.  
**Рабочие файлы текущей ветки не меняет.**

Проверка «что пришло»:

```bash
git log --oneline -1 origin/master
git log --oneline master..origin/master
```

---

### `git merge origin/master`
Влить `origin/master` **в текущую** ветку (в мини-проекте — в `feature/theme-auto` при конфликте).

Также:

```bash
git merge feature/theme-dark   # влить локальную ветку в текущую
```

При конфликте: правишь файл → `git add` → `git commit`.

---

### `git pull` vs `git fetch` + `git merge`

**Эквивалент (когда ты на `master` и tracking настроен):**

```bash
git pull origin master
```

≈

```bash
git fetch origin
git merge origin/master
```

Ещё короче, если `-u` уже был:

```bash
git pull
```

≈ fetch + merge upstream текущей ветки.

#### Когда лучше `fetch` + `merge` (учебный стиль)

- нужна **пауза** между «скачал» и «влил»
- сначала посмотреть:

  ```bash
  git fetch origin
  git log --oneline --graph master origin/master -8
  git diff master origin/master
  ```

  и только потом `git merge origin/master`
- перед опасным слиянием / конфликтом
- когда не уверен, что пришло с remote

#### Когда нормален `git pull`

- ветка простая, ты на `master`, просто «подтянуть после merge PR на сайте»
- нет сомнений, не нужно смотреть diff заранее
- бытовой sync:

  ```bash
  git switch master
  git pull
  ```

#### Чего `pull` не заменяет

- **не** создаёт PR
- **не** делает `gh pr merge`
- **не** пушит твои коммиты (`push` отдельно)
- если ты на `feature/theme-auto` и пишешь `git pull`, подтянется upstream **этой** ветки (если есть), а не обязательно «обновить master»

#### Где в сценарии взаимозаменяемо

| Место в сценарии | Делали | Можно было `pull`? |
|------------------|--------|---------------------|
| После `gh pr merge 1`, обновить локальный master | `fetch` + `merge origin/master` | да: `git switch master` && `git pull` |
| Перед конфликтом на auto | `fetch` + `merge origin/master` **в auto** | `git pull origin master` **находясь на auto** ≈ то же. Явный fetch+merge нагляднее |
| После resolve | `push` feature | pull тут ни при чём |
| После `gh pr merge 2` | `fetch` + `merge` на master | да: `git pull` на master |

**Формулировка:**

```text
git pull          = обновить ТЕКУЩУЮ ветку из её remote
git fetch         = узнать, что на GitHub, ничего не вливая
git merge origin/master = влить remote-master в ТЕКУЩУЮ ветку
```

Конфликт устраивали так: **стоим на auto**, делаем `merge origin/master` (или `pull origin master` с auto) — вливаем dark-master **в** auto.

---

## 8. Конфликт: команды починки

```bash
git switch feature/theme-auto
git fetch origin
git merge origin/master          # CONFLICT
# правка файла / echo "theme=auto" > SETTINGS.txt
git add SETTINGS.txt
git commit -m "Resolve conflict: keep theme=auto"
git push origin feature/theme-auto
```

Пока `|MERGING` и не сделан commit — merge не завершён.  
Пока нет `push` — GitHub PR остаётся CONFLICTING.

**Маркеры конфликта:**

```text
<<<<<<< HEAD
theme=auto          ← текущая ветка (feature/theme-auto)
=======
theme=dark          ← origin/master
>>>>>>> origin/master
```

**Альтернатива:** кнопка *Resolve conflicts* на сайте PR (локально прозрачнее для учёбы).

**Отменить незавершённый merge:**

```bash
git merge --abort
```

---

## 9. Полуфинал vs финиш (обязательно помнить)

| Момент | `feature/theme-auto` | `origin/master` | PR #2 |
|--------|----------------------|-----------------|-------|
| До починки | auto (старый) | **dark** | CONFLICTING |
| После resolve + push | auto + знает про dark | **ещё dark** | MERGEABLE |
| После `gh pr merge 2` | (можно удалить ветку) | **auto** | MERGED |

- **Полуфинал:** починить head-ветку PR + push → снять CONFLICTING  
- **Финиш:** `gh pr merge` → реально меняется master  

---

## 10. `gh` — Pull Request

### `gh auth status`
Проверить логин.

### `gh pr create --base master --head feature/theme-dark --title "..." --body "..."`
Создать PR: куда (`base`) / откуда (`head`).

**Альтернативы:**

- кнопка *Compare & pull request* на сайте
- интерактивно:

  ```bash
  gh pr create
  ```

### `gh pr list`
Открытые PR.

### `gh pr view 2`

```bash
gh pr view 2 --web
gh pr view 2 --json number,title,mergeable,mergeStateStatus,state,mergedAt
```

| Значение | Смысл |
|----------|--------|
| `CONFLICTING` / `DIRTY` | нельзя авто-merge |
| `MERGEABLE` / `CLEAN` | можно вливать |
| `MERGED` | уже влит |
| `UNKNOWN` иногда после merge | смотри `state` |

Баннер *This branch has conflicts* — на вкладке **Conversation** (нужен логин).  
Вкладка **Files changed** показывает diff ветки, не всегда баннер конфликта.

### `gh pr merge 2 --merge`
Влить PR #2 merge-commit’ом на GitHub.  
**Финиш:** `origin/master` меняется (у нас → `auto`).

```bash
gh pr merge 2 --merge                 # как в мини-проекте
gh pr merge 2 --merge --delete-branch # + удалить head-ветку на GitHub (часто и local)
gh pr merge 2 --squash
gh pr merge 2 --rebase
```

---

## 11. Полный сценарий conflict-lab (порядок)

```text
# старт
git init -b master
echo / add / commit SETTINGS light
git remote add origin URL
git push -u origin master

# две ветки
git switch -c feature/theme-dark → dark → commit → push -u
git switch master
git switch -c feature/theme-auto → auto → commit → push -u

# PR1 dark — без конфликта
gh pr create --base master --head feature/theme-dark ...
gh pr merge 1 --merge
git switch master && git fetch && git merge origin/master
#   альтернатива: git pull

# PR2 auto — конфликт
gh pr create --base master --head feature/theme-auto ...
gh pr view 2 --json mergeable,...   # CONFLICTING

# полуфинал
git switch feature/theme-auto
git fetch origin
git merge origin/master             # маркеры
#   альтернатива: git pull origin master
echo auto → add → commit
git push origin feature/theme-auto
gh pr view 2 ...                    # MERGEABLE

# финиш
gh pr merge 2 --merge
git switch master && git fetch && git merge origin/master
#   альтернатива: git pull
cat SETTINGS.txt                    # auto
```

Фактическая линия коммитов (пример):

```text
418c971  light
    ├─ dark ──PR#1──► b96f9d4  master=dark
    └─ auto → resolve eebd665 → PR#2 merge d449546  master=auto
```

---

## 12. Шпаргалка «что взамен чего»

| Хочу | Команда |
|------|---------|
| Создать ветку | `git switch -c name` ≡ `git checkout -b name` |
| Перейти на ветку | `git switch name` ≡ `git checkout name` |
| Обновить master с GitHub просто | `git pull` (на master) ≡ `fetch` + `merge origin/master` |
| Влить master в feature перед PR | стоя на feature: `fetch` + `merge origin/master` или `git pull origin master` |
| Отправить ветку | `git push -u origin name` |
| Создать PR | `gh pr create ...` или сайт |
| Влить PR | `gh pr merge N --merge` |
| Увидеть конфликт | `gh pr view N --json mergeable` + Conversation на сайте |
| Отменить незавершённый merge | `git merge --abort` |

---

## 13. Почему PR#1 без конфликта, а PR#2 с конфликтом

- **PR#1 (dark → master/light):** после развилки файл меняла только dark; master остался как предок → авто-merge.
- **PR#2 (auto → master/dark):** обе линии по-разному изменили одну строку относительно общего предка light → CONFLICTING, пока не resolve на feature + push.

---

*Файл для учёбы. Коммит в `conflict-lab` — по желанию (`git add COMMANDS.md && git commit && git push`).*
