## Task 1 — Git Object Model Exploration

### 1.1: Создание тестового коммита

Команды:

```bash
echo "Test content" > test.txt
git add test.txt
git commit -m "Add test file"
```

Проверка истории:

```bash
git log --oneline -3 --decorate
```

Вывод:

```text
1373a59 (HEAD -> feature/lab2) Add test file
ad54f82 (origin/main, main) Merge pull request #1 from KsAKarpeeva73/feature/lab1
4a33850 (origin/feature/lab1) finish 2 task
```

---

### 1.2: Просмотр объектов Git (commit / tree / blob)

#### 1) Commit hash

Команда:

```bash
git rev-parse HEAD
```

Вывод:

```text
1373a5901995d2bd2885181902ac2f04f08087a2
```

#### 2) Содержимое commit-объекта и tree hash

Команда:

```bash
git cat-file -p HEAD
```

Вывод

```text
tree 97be9be53e014c66ed6b64a3c3cc194a64f1702a
parent ad54f82a98d81aee74303486830fca3054b5ab4c
author KsAKarpeeva73 <ksenia.karpeeva@icloud.com> 1770922916 +0300
committer KsAKarpeeva73 <ksenia.karpeeva@icloud.com> 1770922916 +0300
gpgsig -----BEGIN SSH SIGNATURE-----
 U1NIU0lHAAAAAQAAADMAAAALc3NoLWVkMjU1MTkAAAAgMN/NduiMfE+9e5+BjP0SiXiWGt
 Nse+k+QK2UKL9tz44AAAADZ2l0AAAAAAAAAAZzaGE1MTIAAABTAAAAC3NzaC1lZDI1NTE5
 AAAAQOPF8cHDdAYbSUr2an1mazUIUOqpTzdicSZbsPR+UCJ3VWEOWQGFMk79ME6W1EYSmk
 bysJDJXq0IpEF9ur1jKgg=
 -----END SSH SIGNATURE-----

Add test file
```

#### 3) Содержимое tree-объекта (структура репозитория) и blob hash для test.txt

Команда:

```bash
git cat-file -p 97be9be53e014c66ed6b64a3c3cc194a64f1702a
```

Вывод:

```text
040000 tree 6a760a8deafb7e27f0eef7130e46b23f9c8e63d5    .github
100644 blob a38dbc03f5f790c14c955f7cc235f89bef651095    .gitignore
100644 blob 6e60bebec0724892a7c82c52183d0a7b467cb6bb    README.md
040000 tree a1061247fd38ef2a568735939f86af7b1000f83c    app
040000 tree 09e15257766447b3ee5129c5de404ab0da3b66d4    labs
040000 tree d3fb3722b7a867a83efde73c57c49b5ab3e62c63    lectures
100644 blob 2eec599a1130d2ff231309bb776d1989b97c6ab2    test.txt
```

Здесь видно, что файл `test.txt` хранится как **blob** с хэшем: `2eec599a1130d2ff231309bb776d1989b97c6ab2`

#### 4) Содержимое blob-объекта

Команда:

```bash
git cat-file -p 2eec599a1130d2ff231309bb776d1989b97c6ab2
```

Вывод:

```text
Test content
```

---

### Ответ

* **Blob** — это содержимое файла. Git хранит данные файла отдельно, и blob не знает про имя файла или папку, только про байты.
* **Tree** — это снимок структуры: список файлов и подпапок с их именами и ссылками на blob/tree-объекты.
* **Commit** — это снимок состояния проекта: он ссылается на корневой tree, хранит автора/дату/сообщение и указывает на родительский коммит.

### Как Git хранит данные в репо

Цепочка такая:

* Коммит `1373a590...` ссылается на дерево `97be9be5...`
* В дереве `97be9be5...` есть запись `test.txt`, которая ссылается на blob `2eec599a...`
* Blob `2eec599a...` содержит текст `Test content`

То есть Git хранит не “файлы в папках” как обычная ОС, а набор объектов (blob/tree/commit), связанных между собой хэшами.

---

## Task 2 — Reset and Reflog Recovery

### 2.1: Создание ветки

Команды:

```bash
git status
git switch feature/lab2
git status
git switch -c git-reset-practice
git status
```

Вывод:

```text
On branch feature/lab2
nothing to commit, working tree clean
```

Дальше я сделала три последовательных коммита в ветке `git-reset-practice`:

```bash
echo "First commit" > file.txt
git add file.txt
git commit -m "First commit"

echo "Second commit" >> file.txt
git add file.txt
git commit -m "Second commit"

echo "Third commit" >> file.txt
git add file.txt
git commit -m "Third commit"
```

---

### 2.2: reset --soft, reset --hard и восстановление через reflog

#### reset --soft HEAD~1

Команды:

```bash
git switch git-reset-practice
git log --oneline -5 --decorate
git status
cat file.txt

git reset --soft HEAD~1

git log --oneline -5 --decorate
git status
cat file.txt
```

Вывод:

```text
M       labs/submission2.md
Already on 'git-reset-practice'
291bd08 (HEAD -> git-reset-practice) First commit
6eac8d4 Second commit
7ec3d00 First commit
e57a2d9 (feature/lab2) finish 1 task
1373a59 Add test file
On branch git-reset-practice
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   labs/submission2.md

First commit
6eac8d4 (HEAD -> git-reset-practice) Second commit
7ec3d00 First commit
e57a2d9 (feature/lab2) finish 1 task
1373a59 Add test file
ad54f82 (origin/main, main) Merge pull request #1 from KsAKarpeeva73/feature/lab1
On branch git-reset-practice
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file.txt
        modified:   labs/submission2.md

First commit
```
После `git reset --soft HEAD~1` HEAD сдвигается назад по истории, но изменения не пропадают.
По `git status` видно, что изменения остаются в индексе и считаются подготовленными к коммиту, в моём случае staged были `file.txt` и `labs/submission2.md`.

---

#### reset --hard HEAD~1

Команды:

```bash
git reset --hard HEAD@{2}
git log --oneline -5 --decorate
git status
cat file.txt

git reset --hard HEAD~1

git log --oneline -5 --decorate
git status
cat file.txt
```

Вывод:

```text
fatal: Cannot do hard reset with paths.
6eac8d4 (HEAD -> git-reset-practice) Second commit
7ec3d00 First commit
e57a2d9 (feature/lab2) finish 1 task
1373a59 Add test file
ad54f82 (origin/main, main) Merge pull request #1 from KsAKarpeeva73/feature/lab1
On branch git-reset-practice
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   file.txt
        modified:   labs/submission2.md

First commit
HEAD is now at 7ec3d00 First commit
7ec3d00 (HEAD -> git-reset-practice) First commit
e57a2d9 (feature/lab2) finish 1 task
1373a59 Add test file
ad54f82 (origin/main, main) Merge pull request #1 from KsAKarpeeva73/feature/lab1
4a33850 (origin/feature/lab1) finish 2 task
On branch git-reset-practice
nothing to commit, working tree clean
First commit
```

После `git reset --hard HEAD~1` Git откатил HEAD назад и очистил рабочую папку: `git status` показал `nothing to commit, working tree clean`.
По `cat file.txt` видно, что содержимое реально откатилось и осталось только `First commit`.

---

#### reflog и восстановление состояния

Команды:

```bash
git reflog -10
git reset --hard HEAD@{2}
git log --oneline -5 --decorate
git status
cat file.txt
```

Вывод:

```text
7ec3d00 (HEAD -> git-reset-practice) HEAD@{0}: reset: moving to HEAD~1
6eac8d4 HEAD@{1}: reset: moving to HEAD~1
291bd08 HEAD@{2}: checkout: moving from git-reset-practice to git-reset-practice
291bd08 HEAD@{3}: reset: moving to HEAD~1
59f8fe7 HEAD@{4}: checkout: moving from git-reset-practice to git-reset-practice
59f8fe7 HEAD@{5}: commit (amend): finish 2 task
c85b86a HEAD@{6}: commit: finish 1 task
291bd08 HEAD@{7}: reset: moving to HEAD~1
a7c55ba HEAD@{8}: reset: moving to HEAD~1
bfcfb22 HEAD@{9}: commit: Third commit
HEAD is now at 291bd08 First commit
291bd08 (HEAD -> git-reset-practice) First commit
6eac8d4 Second commit
7ec3d00 First commit
e57a2d9 (feature/lab2) finish 1 task
1373a59 Add test file
On branch git-reset-practice
nothing to commit, working tree clean
First commit
```


`git reflog` реально помогает, потому что он показывает, куда двигался HEAD, даже если коммиты уже не видны в обычном `git log`.
В моём reflog есть запись `bfcfb22 ... commit: Third commit` — это нужная точка, чтобы вернуть состояние с третьим коммитом.
Я попробовала восстановиться командой `git reset --hard HEAD@{2}`, но по reflog видно, что `HEAD@{2}` у меня сейчас относится к checkout, поэтому я в итоге вернулась на `291bd08 First commit`.

Чтобы восстановиться именно на `Third commit`, нужно сброситься на `bfcfb22` (он у меня в reflog как `HEAD@{9}`):


```bash
git reset --hard bfcfb22
```


```bash
git log --oneline -5 --decorate
git status
cat file.txt
```

---

### Что меняется в working tree, индексе и истории

* `git reset --soft HEAD~1` двигает HEAD назад, но изменения остаются подготовленными к коммиту, то есть остаются в индексе.
* `git reset --hard HEAD~1` двигает HEAD назад и полностью приводит рабочую директорию и индекс к состоянию того коммита, на который откатились.
* `git reflog` позволяет найти прошлые состояния HEAD и восстановиться, даже если я уже “ушла назад” reset-ом.




## Task 3 — Visualize Commit History

### Команды

```bash
git status
git switch feature/lab2
git status
git add labs/submission2.md
git commit -m "docs: update submission2 tasks 1-2"
git switch -c side-branch
echo "Branch commit" >> history.txt
git add history.txt
git commit -m "Side branch commit"
git switch -
git log --oneline --graph --all --decorate -15
```

### Вывод

```text
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
Already on 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
```

```text
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
```

```text
Switched to a new branch 'side-branch'
[side-branch 5302521] Side branch commit
 1 file changed, 1 insertion(+)
 create mode 100644 history.txt
```

```text
Switched to branch 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
```

```text
* 5302521 (side-branch) Side branch commit
* fef7d67 (HEAD -> feature/lab2, origin/feature/lab2) finish 2 task
| * 291bd08 (git-reset-practice) First commit
| * 6eac8d4 Second commit
| * 7ec3d00 First commit
|/  
* e57a2d9 finish 1 task
* 1373a59 Add test file
*   ad54f82 (origin/main, main) Merge pull request #1 from KsAKarpeeva73/feature/lab1
|\  
| * 4a33850 (origin/feature/lab1) finish 2 task
| * 99e184b finish 1 task
| * 990680f docs: add commit signing summary
| | * 08dabbc (tag: v1.0.0) Add test file
| |/  
|/|   
* | c9679b3 chore: add PR template
|/  
* d6b6a03 Update lab2
* 87810a0 feat: remove old Exam Exemption Policy
```

### Вывод

Граф `git log --graph --all` удобно показывает, где именно разошлись ветки и какой коммит к какой ветке относится. По нему сразу видно параллельные ветки `side-branch` и `git-reset-practice` и то, что они не слиты обратно в `feature/lab2`.



## Task 4 — Tagging Commits

### Команды

```bash
git switch feature/lab2
git status
git tag --list
git show-ref --tags
git ls-remote --tags origin
git push origin v1.0.0
git rev-parse --short HEAD
git tag v1.1.0
git push origin v1.1.0
git show v1.1.0 --oneline --no-patch
```

### Вывод

```text
Already on 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
v1.0.0
08dabbc0e01ca71487f6b14d542789e9f99be544 refs/tags/v1.0.0
```

```text
08dabbc0e01ca71487f6b14d542789e9f99be544        refs/tags/v1.0.0
```

```text
Everything up-to-date
```

```text
855b5d5
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/KsAKarpeeva73/DevOps-Intro.git
 * [new tag]         v1.1.0 -> v1.1.0
```

```text
855b5d5 (HEAD -> feature/lab2, tag: v1.1.0, origin/feature/lab2) finish 3 task
```

### Теги и к каким коммитам привязаны

* v1.0.0 → 08dabbc0e01ca71487f6b14d542789e9f99be544
* v1.1.0 → 855b5d5

### Зачем вообще нужны теги

Теги нужны, чтобы помечать конкретные стабильные версии кода и потом легко на них ссылаться. Это удобно для релизов, для заметок к версиям и для CI/CD, когда сборка или релиз запускаются по тегу.


## Task 5 — git switch vs git checkout vs git restore

### Команды

```bash
git switch feature/lab2
git status
git branch

git switch -c cmd-compare
git branch
git status
git switch -
git branch
git status

git checkout -b cmd-compare-2
git branch
git status
git switch -
git branch
git status

echo "base" > demo.txt
git add demo.txt
git commit -m "chore: add demo file for restore"
git status

echo "scratch" >> demo.txt
git status
cat demo.txt
git restore demo.txt
git status
cat demo.txt

echo "staged-change" >> demo.txt
git add demo.txt
git status
cat demo.txt
git restore --staged demo.txt
git status
cat demo.txt

git add demo.txt
git commit -m "chore: update demo file"
git log --oneline -3 --decorate
cat demo.txt

git restore --source=HEAD~1 demo.txt
git status
cat demo.txt
```

### Вывод

```text
Already on 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
* feature/lab2
  git-reset-practice
  main
  side-branch
```

```text
Switched to a new branch 'cmd-compare'
* cmd-compare
  feature/lab2
  git-reset-practice
  main
  side-branch
On branch cmd-compare
nothing to commit, working tree clean
Switched to branch 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
  cmd-compare
* feature/lab2
  git-reset-practice
  main
  side-branch
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
```

```text
Switched to a new branch 'cmd-compare-2'
  cmd-compare
* cmd-compare-2
  feature/lab2
  git-reset-practice
  main
  side-branch
On branch cmd-compare-2
nothing to commit, working tree clean
Switched to branch 'feature/lab2'
Your branch is up to date with 'origin/feature/lab2'.
  cmd-compare
  cmd-compare-2
* feature/lab2
  git-reset-practice
  main
  side-branch
On branch feature/lab2
Your branch is up to date with 'origin/feature/lab2'.

nothing to commit, working tree clean
```

```text
[feature/lab2 f2a27d5] chore: add demo file for restore
 1 file changed, 1 insertion(+)
 create mode 100644 demo.txt
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

```text
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   demo.txt

no changes added to commit (use "git add" and/or "git commit -a")
base
scratch
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
base
```

```text
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   demo.txt

base
staged-change
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   demo.txt

no changes added to commit (use "git add" and/or "git commit -a")
base
staged-change
```

```text
[feature/lab2 8b2b2d4] chore: update demo file
 1 file changed, 1 insertion(+)
8b2b2d4 (HEAD -> feature/lab2) chore: update demo file
f2a27d5 chore: add demo file for restore
bd20ea6 (origin/feature/lab2, cmd-compare-2, cmd-compare) finish 4 task
base
staged-change
```

```text
On branch feature/lab2
Your branch is ahead of 'origin/feature/lab2' by 2 commits.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   demo.txt

no changes added to commit (use "git add" and/or "git commit -a")
base
```

### Когда что использовать

`git switch` я использую, когда мне нужно только работать с ветками: создать ветку и перейти на неё или быстро вернуться на прошлую ветку через `git switch -`.
`git checkout` делает то же самое, но он старый и у него слишком много ролей, поэтому его легко перепутать с восстановлением файлов.
`git restore` я использую, когда нужно откатить изменения в файлах: убрать изменения из рабочей папки, снять файл из staged или вернуть файл из другого коммита.
