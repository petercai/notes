
Python’s **parallel computing** system: `multiprocessing`, `Manager` and proxy objects like `proxy_dict`.

---

## 🧠 1. What is `multiprocessing`?

`multiprocessing` is a **Python standard library** for running **multiple processes in parallel** — each with its own **Python interpreter** and **memory space**.

This is different from `threading`:

|Feature|`threading`|`multiprocessing`|
|---|---|---|
|Runs in same process|✅ Yes|❌ No|
|Shares memory directly|✅ Yes|❌ No (each process has its own memory)|
|Avoids Global Interpreter Lock (GIL)|❌ No|✅ Yes|
|Suitable for|I/O-bound tasks|CPU-bound tasks|

---

## ⚙️ 2. Basic Example

Here’s a minimal `multiprocessing` example:

```python
from multiprocessing import Process

def worker(name):
    print(f"Worker {name} is running")

if __name__ == "__main__":
    p1 = Process(target=worker, args=("A",))
    p2 = Process(target=worker, args=("B",))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

Each `Process` runs in its **own Python interpreter**, fully parallelized across CPU cores.

---

## 🧩 3. Data Sharing Problem

Since each process has its **own memory**, you **can’t directly share variables**:

```python
x = 0
def worker():
    global x
    x += 1  # This modifies a *copy* of x in a separate process
```

To share data, you need **inter-process communication (IPC)**.

---

## 🧮 4. Ways to Share Data

`multiprocessing` provides several tools:

|Tool|Description|Example Use|
|---|---|---|
|`Queue`|Thread/process-safe FIFO|Pass messages or data|
|`Pipe`|Two-way communication channel|Simple data passing|
|`Value`, `Array`|Shared C types (low-level)|Simple numbers or arrays|
|`Manager`|High-level shared objects|Shared dicts, lists, etc. ✅|

---

## 🧰 5. `Manager` — Shared Objects Manager

`multiprocessing.Manager()` creates a **server process** that holds shared objects.  
Other processes get **proxies** (remote references) to these objects — meaning they can interact with shared data safely.

### Example:

```python
from multiprocessing import Process, Manager

def worker(shared_dict, key, value):
    shared_dict[key] = value

if __name__ == "__main__":
    with Manager() as manager:
        shared_dict = manager.dict()  # shared across processes
        processes = [
            Process(target=worker, args=(shared_dict, i, i*i))
            for i in range(5)
        ]
        for p in processes: p.start()
        for p in processes: p.join()
        print(shared_dict)
```

**Output:**

```
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

Each worker updates the **same shared dictionary**, even though they’re in separate processes.

---

## 🧩 6. Proxy Objects (like `proxy_dict`)

When you use a `Manager`, Python doesn’t give you the real object — instead, it gives you a **proxy** object.

For example:

```python
from multiprocessing import Manager
m = Manager()
d = m.dict()
print(type(d))
```

Output:

```
<class 'multiprocessing.managers.DictProxy'>
```

That’s a **proxy_dict** (or more precisely, a `DictProxy`) —  
it **looks like a normal dict**, but every operation (like `d[key] = value`) is sent through an **IPC channel** to the manager’s server process.

---

### ⚠️ Performance note

Because all access goes through IPC, **Manager objects are slower** than normal ones.  
Use them **only when sharing data is necessary**.

---

## 🧱 7. Common Shared Object Types from Manager

|Type|Proxy Class|Description|
|---|---|---|
|`manager.list()`|`ListProxy`|Shared list|
|`manager.dict()`|`DictProxy`|Shared dictionary|
|`manager.Queue()`|`QueueProxy`|Shared queue|
|`manager.Namespace()`|`NamespaceProxy`|Simple attribute container|
|`manager.Value()`, `manager.Array()`|`ValueProxy`, `ArrayProxy`|Shared primitives|

---

## ⚡ 8. Example: Multiple Workers Updating Shared State

```python
from multiprocessing import Process, Manager
import time

def increment(shared_dict, key):
    for _ in range(5):
        shared_dict[key] += 1
        time.sleep(0.1)

if __name__ == "__main__":
    with Manager() as manager:
        data = manager.dict({'count': 0})
        workers = [Process(target=increment, args=(data, 'count')) for _ in range(3)]

        for w in workers: w.start()
        for w in workers: w.join()

        print(data)  # {'count': 15}
```

---

## 🧠 Summary

|Concept|Purpose|
|---|---|
|`multiprocessing`|Run multiple processes in parallel|
|`Process`|Represents one process|
|`Queue` / `Pipe`|Pass data between processes|
|`Manager`|Create shared objects|
|`Proxy objects` (e.g., `DictProxy`)|Remote-controlled shared objects (IPC-based)|
![[multiprocessing-20251018-1.png]]
---

