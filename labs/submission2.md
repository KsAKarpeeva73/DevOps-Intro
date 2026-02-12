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

