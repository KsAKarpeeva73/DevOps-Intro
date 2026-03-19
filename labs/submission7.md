# Lab 7 — GitOps Fundamentals

## Task 1 — Git State Reconciliation

### 1.1 Initial desired state and current state

`desired-state.txt`

```text
version: 1.0
app: myapp
replicas: 3
````

`current-state.txt`

```text
version: 1.0
app: myapp
replicas: 3
```

Изначально текущее состояние было синхронизировано с желаемым, то есть `current-state.txt` был создан как копия `desired-state.txt`.

---

### 1.2 Reconciliation script

`reconcile.sh`

```bash
#!/bin/bash
# reconcile.sh - GitOps reconciliation loop

DESIRED=$(cat desired-state.txt)
CURRENT=$(cat current-state.txt)

if [ "$DESIRED" != "$CURRENT" ]; then
    echo "$(date) - ⚠️  DRIFT DETECTED!"
    echo "Reconciling current state with desired state..."
    cp desired-state.txt current-state.txt
    echo "$(date) - ✅ Reconciliation complete"
else
    echo "$(date) - ✅ States synchronized"
fi
```

Скрипт сравнивает желаемое состояние с текущим. Если есть расхождение, он считает это drift и возвращает текущее состояние к содержимому `desired-state.txt`.

---

### 1.3 Manual drift detection and reconciliation

Сначала drift был создан вручную изменением `current-state.txt`.

`current-state.txt` before reconciliation:

```text
version: 2.0
app: myapp
replicas: 5
```

Output of manual reconciliation:

```text
Thu Mar 19 16:52:29 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:52:29 UTC 2026 - ✅ Reconciliation complete
```

Output of diff after reconciliation:

```text
diff exit code: 0
```

Код возврата `0` означает, что после выполнения `reconcile.sh` файлы `desired-state.txt` и `current-state.txt` снова стали одинаковыми.

`current-state.txt` after reconciliation:

```text
version: 1.0
app: myapp
replicas: 3
```

---

### 1.4 Continuous reconciliation and auto-healing

Для имитации непрерывной работы GitOps-оператора был запущен цикл:

```bash
watch -n 5 ./reconcile.sh
```

После этого в другом терминале в `current-state.txt` была внесена несанкционированная правка. Цикл автоматически обнаружил drift и восстановил состояние.

Output from continuous reconciliation loop:

```text
Every 5.0s: ./reconcile.sh                                                        server-3-pdf: Thu Mar 19 16:53:35 2026

Thu Mar 19 16:53:35 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:53:35 UTC 2026 - ✅ Reconciliation complete
```

`current-state.txt` after auto-healing:

```text
version: 1.0
app: myapp
replicas: 3
```

---

### Analysis

Логика reconciliation loop здесь простая: есть файл с желаемым состоянием, который играет роль source of truth, и есть файл с текущим состоянием. Скрипт регулярно сравнивает их содержимое. Если обнаруживается расхождение, текущее состояние принудительно приводится к желаемому. По сути это и есть базовая идея GitOps: система не ждет ручного вмешательства, а сама постоянно сверяет реальное состояние с тем, что описано декларативно.

Такой подход хорошо предотвращает configuration drift. Даже если кто-то вручную поменял состояние, эти изменения не считаются легитимными, пока они не отражены в source of truth. Поэтому при следующей проверке система откатывает несанкционированные изменения и возвращается к корректной конфигурации.

---

### Reflection

Декларативная конфигурация в продакшене удобнее императивных команд по нескольким причинам. Во-первых, всегда есть одно понятное описание того, как система должна выглядеть. Во-вторых, это легче версионировать, ревьюить и откатывать через Git. В-третьих, декларативный подход лучше масштабируется: не нужно помнить последовательность ручных действий, достаточно хранить итоговое желаемое состояние. Плюс это уменьшает число человеческих ошибок, потому что система сама доводит инфраструктуру до нужного состояния, а не полагается на ручные шаги администратора.

---

## Task 2 — GitOps Health Monitoring

### 2.1 Health check script

`healthcheck.sh`

```bash
#!/bin/bash
# healthcheck.sh - Monitor GitOps sync health

DESIRED_MD5=$(md5sum desired-state.txt | awk '{print $1}')
CURRENT_MD5=$(md5sum current-state.txt | awk '{print $1}')

if [ "$DESIRED_MD5" != "$CURRENT_MD5" ]; then
    echo "$(date) - ❌ CRITICAL: State mismatch detected!" | tee -a health.log
    echo "  Desired MD5: $DESIRED_MD5" | tee -a health.log
    echo "  Current MD5: $CURRENT_MD5" | tee -a health.log
else
    echo "$(date) - ✅ OK: States synchronized" | tee -a health.log
fi
```

Скрипт считает MD5-хэши для желаемого и текущего состояний. Если хэши отличаются, значит содержимое файлов различается, и это фиксируется как критическое отклонение в `health.log`.

---

### 2.2 Testing health monitoring

#### Healthy state

Output of healthy state check:

```text
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
```

`health.log` after healthy check:

```text
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
```

#### Drifted state

Для проверки drift в `current-state.txt` была добавлена строка:

```text
unapproved-change: true
```

`current-state.txt` with drift:

```text
version: 1.0
app: myapp
replicas: 3
unapproved-change: true
```

Output of health check on drifted state:

```text
Thu Mar 19 16:54:35 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
```

`health.log` after drift detection:

```text
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:35 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
```

#### Fixing drift and verifying health again

Output after reconciliation and repeated health check:

```text
Thu Mar 19 16:54:35 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:54:35 UTC 2026 - ✅ Reconciliation complete
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
```

`health.log` after fix:

```text
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:35 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
```

---

### 2.3 Continuous health monitoring

`monitor.sh`

```bash
#!/bin/bash
# monitor.sh - Combined reconciliation and health monitoring

echo "Starting GitOps monitoring..."
for i in {1..10}; do
    echo
    echo "--- Check #$i ---"
    ./healthcheck.sh
    ./reconcile.sh
    sleep 3
done
```

Output of `monitor.sh`:

```text
Starting GitOps monitoring...

--- Check #1 ---
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:35 UTC 2026 - ✅ States synchronized

--- Check #2 ---
Thu Mar 19 16:54:38 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:38 UTC 2026 - ✅ States synchronized

--- Check #3 ---
Thu Mar 19 16:54:41 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:41 UTC 2026 - ✅ States synchronized

--- Check #4 ---
Thu Mar 19 16:54:44 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:44 UTC 2026 - ✅ States synchronized

--- Check #5 ---
Thu Mar 19 16:54:47 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:47 UTC 2026 - ✅ States synchronized

--- Check #6 ---
Thu Mar 19 16:54:50 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:50 UTC 2026 - ✅ States synchronized

--- Check #7 ---
Thu Mar 19 16:54:53 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:53 UTC 2026 - ✅ States synchronized

--- Check #8 ---
Thu Mar 19 16:54:56 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:56 UTC 2026 - ✅ States synchronized

--- Check #9 ---
Thu Mar 19 16:54:59 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:59 UTC 2026 - ✅ States synchronized

--- Check #10 ---
Thu Mar 19 16:55:02 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:55:02 UTC 2026 - ✅ States synchronized
```

Complete `health.log`:

```text
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:35 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:35 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:38 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:41 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:44 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:47 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:50 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:53 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:56 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:54:59 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:55:02 UTC 2026 - ✅ OK: States synchronized
```

---

### Analysis

Checksums нужны для быстрого контроля целостности и синхронности состояния. Если содержимое двух файлов отличается хотя бы на один символ, их MD5-хэши будут разными. За счет этого можно очень быстро понять, совпадает текущее состояние с желаемым или нет, не разбирая файл построчно вручную. Для такой учебной задачи этого более чем достаточно.

При этом важно понимать, что MD5 здесь используется не как криптографическая защита, а именно как простой механизм сравнения состояний. В GitOps-сценарии это похоже на быстрый индикатор того, что live state ушел от desired state.

---

### Comparison with real GitOps tools

Это прямо похоже на то, как работают инструменты вроде ArgoCD. У ArgoCD есть понятие `Sync Status`: он сравнивает состояние, описанное в Git, с тем, что реально существует в кластере. Если они совпадают, система находится в состоянии `Synced`. Если нет — состояние становится `OutOfSync`, и оператор может либо показать проблему, либо автоматически привести систему к нужному виду.

В этой лабораторной мы сделали ту же самую идею, только в очень упрощенном виде и без Kubernetes. `desired-state.txt` играет роль Git-репозитория с эталонной конфигурацией, `current-state.txt` — роль live state, `reconcile.sh` — роль reconciliation loop, а `healthcheck.sh` — роль мониторинга статуса синхронизации.

---

## Conclusion

В ходе лабораторной были реализованы два ключевых принципа GitOps: автоматическая реконсиляция состояния и мониторинг синхронизации. В первой части удалось показать, как система обнаруживает drift и автоматически восстанавливает корректную конфигурацию. Во второй части был добавлен health monitoring через MD5-хэши и логирование, что позволило явно фиксировать как нормальное состояние, так и отклонения.



