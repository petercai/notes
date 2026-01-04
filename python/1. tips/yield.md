# yield()

`yield` is one of Python’s most powerful but misunderstood features.  
Let’s go step by step 👇

---

## 🧩 What `yield` Does

- `yield` **turns a function into a generator**.
    
- Instead of running from start to finish, the function can **pause** and **resume** later.
    
- Each time it reaches `yield`, it **returns a value** to the caller — but keeps its local state for the next time it’s called.
    

So, a **generator** is like a _lazy iterator_: it produces items one by one, only when requested.

---

## ⚙️ Basic Example

```python
def count_up_to(n):
    for i in range(1, n + 1):
        yield i
```

Using it:

```python
counter = count_up_to(5)
print(next(counter))  # 1
print(next(counter))  # 2
print(list(counter))  # [3, 4, 5]
```

Notice how:

- The generator “remembers” its position between `next()` calls.
    
- After the final yield, it raises `StopIteration` automatically.
    

---

## 🌀 Typical Use Cases

### 1. Iterating large datasets

```python
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()
```

✅ Efficient — loads one line at a time instead of reading the whole file.

---

### 2. Infinite sequences

```python
def even_numbers():
    n = 0
    while True:
        yield n
        n += 2
```

You can use it like:

```python
evens = even_numbers()
for i in range(5):
    print(next(evens))
# Output: 0, 2, 4, 6, 8
```

---

### 3. Using `yield from` to delegate to another generator

```python
def sub_gen():
    yield from range(3)

def main_gen():
    yield "start"
    yield from sub_gen()
    yield "end"

print(list(main_gen()))
# ['start', 0, 1, 2, 'end']
```

---

### 4. Combining with `send()`

Generators can also **receive** data using `.send(value)`.

```python
def echo():
    while True:
        received = yield
        print("You said:", received)

g = echo()
next(g)           # Start generator
g.send("Hello!")  # You said: Hello!
```

---

## 🔍 Summary

|Concept|What it does|
|---|---|
|`yield`|Pause function and return a value|
|Generator|Function that yields values on demand|
|`next(gen)`|Resume generator until next yield|
|`yield from`|Yield all values from another generator|
|`send(value)`|Send a value back into the generator|

---

Perfect 👍 — let’s explore how `yield` helps you **save memory and boost performance**, especially when working with _large data_.

---

## 💡 Problem: Lists use lots of memory

Suppose you want to process a 10 million-line file.

### ❌ Normal (using a list)

```python
def read_lines_list(filename):
    with open(filename) as f:
        lines = [line.strip() for line in f]
    return lines
```

- This loads **every line into memory** at once.
    
- If the file is 500 MB, your program may use **>500 MB RAM**.
    

---

## ✅ Generator solution (using `yield`)

```python
def read_lines_gen(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()
```

Now when you do:

```python
for line in read_lines_gen("big.txt"):
    process(line)
```

➡️ Only **one line** exists in memory at a time.  
This is called **lazy evaluation** — data is produced _on demand_.

---

## ⚙️ Another Example — Generating squares

### List version

```python
def make_squares(n):
    return [i * i for i in range(n)]
```

This builds a list of all squares in memory.

### Generator version

```python
def gen_squares(n):
    for i in range(n):
        yield i * i
```

Usage:

```python
for sq in gen_squares(10_000_000):
    if sq > 100:
        break
```

✅ **No list**, **no memory blow-up** — only the current `sq` is kept.

---

## 🧮 Compare memory footprint

You can check the difference easily:

```python
import sys

list_data = [i for i in range(1000000)]
gen_data = (i for i in range(1000000))  # generator expression

print(sys.getsizeof(list_data))  # ~8,000,000 bytes
print(sys.getsizeof(gen_data))   # ~112 bytes
```

💥 A generator uses **thousands of times less memory**!

---

## 🧩 When to use `yield`

Use it when:

- You process **large / infinite sequences**
    
- You only need **one item at a time**
    
- You want **streaming / pipeline** behavior (e.g., process-then-discard)
    

Don’t use it when:

- You truly need all data in memory at once
    
- You need random access (generators are one-way iterators)
    

---

## 🧱 Bonus: Pipeline pattern with generators

You can chain them for efficient data pipelines:

```python
def read_numbers(filename):
    with open(filename) as f:
        for line in f:
            yield int(line)

def filter_even(numbers):
    for n in numbers:
        if n % 2 == 0:
            yield n

def double(numbers):
    for n in numbers:
        yield n * 2

# Combine lazily — nothing stored in memory
for num in double(filter_even(read_numbers("numbers.txt"))):
    print(num)
```

Each stage yields one value at a time.  
This pattern is extremely powerful for streaming data.

---

Awesome 😎 — let’s cover **both** topics:  
1️⃣ how to convert a generator to a list, and  
2️⃣ how to use `yield` inside a class (with practical examples).

---

## 1️⃣ Converting a generator → list

Sometimes, after processing, you **want to collect all results at once** — even if you used `yield`.

### Example

```python
def squares(n):
    for i in range(n):
        yield i * i
```

#### Convert to list:

```python
nums = list(squares(5))
print(nums)
# Output: [0, 1, 4, 9, 16]
```

✅ `list()` calls `next()` repeatedly until the generator is exhausted.

You can also use:

```python
nums = [x for x in squares(5)]  # same as list()
```

---

### ⚠️ Important:

Once a generator is exhausted, it’s **done**:

```python
g = squares(3)
print(list(g))  # [0, 1, 4]
print(list(g))  # []  (empty — already used up)
```

If you need to reuse it, call the generator function again.

---

## 2️⃣ Using `yield` inside a class

You can use `yield` in **methods** the same way as in regular functions.

---

### 🧩 Example 1: Simple class generator

```python
class Counter:
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def generate(self):
        for i in range(self.start, self.end + 1):
            yield i
```

Usage:

```python
c = Counter(3, 6)
for num in c.generate():
    print(num)
# Output: 3, 4, 5, 6
```

---

### 🧩 Example 2: Using `yield` in `__iter__`

If you want your class to be **iterable** (so you can use it directly in a for-loop),  
implement `__iter__()` with `yield`:

```python
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit

    def __iter__(self):
        for i in range(0, self.limit + 1, 2):
            yield i
```

Usage:

```python
for n in EvenNumbers(10):
    print(n)
# Output: 0, 2, 4, 6, 8, 10
```

✅ You can now treat `EvenNumbers(10)` like a built-in iterable.

---

### 🧩 Example 3: Chaining class-based generators

```python
class Fibonacci:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        a, b = 0, 1
        for _ in range(self.n):
            yield a
            a, b = b, a + b

for num in Fibonacci(8):
    print(num)
# Output: 0 1 1 2 3 5 8 13
```

This pattern is used in **data streaming**, **custom iterators**, or **simulation models**.

---

## 🧠 Summary

|Concept|Example|Result|
|---|---|---|
|Convert generator → list|`list(gen)`|All values collected|
|Reuse generator|Call the function again|New generator each time|
|Use `yield` in class|Inside `__iter__()`|Makes class iterable|
|Chaining|Use `yield from`|Combine generators easily|

---

Perfect 🤝 — now we’re entering the _advanced zone_ of Python generators:  
using `yield` not just to **produce** data, but also to **receive** data — turning your function into a **coroutine**.

Let’s go step by step 👇

---

## 🧩 What makes it special

Normally, `yield` is one-way:

- It _sends_ data **out** of a generator (like a return).
    

But when combined with `.send(value)`,  
`yield` can also **receive** data back **into** the generator — resuming it with new input.

---

## ⚙️ Basic coroutine example

```python
def listener():
    print("Starting listener...")
    while True:
        data = yield  # yield pauses and waits for .send()
        print(f"Received: {data}")
```

### How to use it

```python
gen = listener()    # create generator
next(gen)           # start it (run until first yield)

gen.send("Hello")   # Received: Hello
gen.send("World!")  # Received: World!
```

> 💡 You must call `next(gen)` (or `gen.send(None)`) **once first**  
> to move execution to the first `yield`, otherwise `.send()` will raise an error.

---

## 🧱 Example: class-based coroutine generator

Let’s combine this with a class.

```python
class ChatBot:
    def __init__(self, name):
        self.name = name

    def conversation(self):
        print(f"{self.name} is ready to chat!")
        while True:
            message = yield
            if message.lower() == "bye":
                print(f"{self.name}: Goodbye!")
                break
            print(f"{self.name}: You said '{message}'")
```

### Use it:

```python
bot = ChatBot("Echo")
conv = bot.conversation()
next(conv)  # start

conv.send("Hello")
conv.send("How are you?")
conv.send("bye")
```

Output:

```
Echo is ready to chat!
Echo: You said 'Hello'
Echo: You said 'How are you?'
Echo: Goodbye!
```

---

## 🔄 Bidirectional communication pattern

You can also use the return value of `yield` to control logic dynamically.

```python
def accumulator():
    total = 0
    while True:
        x = yield total
        if x is None:
            break
        total += x

gen = accumulator()
print(next(gen))      # 0
print(gen.send(5))    # 5
print(gen.send(10))   # 15
print(gen.send(None)) # stops
```

---

## 🧠 Summary table

|Action|Description|Example|
|---|---|---|
|`yield`|Pause & return value|`x = yield y`|
|`next(gen)`|Start / resume until next yield|First activation|
|`.send(value)`|Resume and inject value into `yield`|`x = yield` receives it|
|`.close()`|Gracefully stop generator|`gen.close()`|
|`.throw(Exception)`|Inject exception into generator|Rare, for cleanup|

---

## 💬 Real-world uses

- **Asynchronous pipelines** (before `async/await` existed)
    
- **Streaming parsers** (receive partial data chunks)
    
- **Event handling loops**
    
- **Custom reactive systems**
    

---

# generator


---

## 🧩 What is a _generator_ in Python?

A **generator** is a _special kind of iterator_ that produces values **on the fly**, instead of storing them all in memory.

It’s created either by:

1. A **function** that uses the keyword `yield`, or
    
2. A **generator expression**, like `(x*x for x in range(5))`.
    

---

## ⚙️ Key idea

A **normal function** runs top-to-bottom and returns once (with `return`).  
A **generator function** can _pause_ every time it hits `yield`,  
then _resume_ later — remembering all its local variables.

---

### 🔍 Example: a simple generator function

```python
def count_up_to(n):
    print("Starting...")
    for i in range(1, n + 1):
        yield i
        print("Yielded", i)
    print("Done!")
```

#### Use it:

```python
counter = count_up_to(3)
print(next(counter))  # Starting... → 1
print(next(counter))  # Yielded 1 → 2
print(next(counter))  # Yielded 2 → 3
print(next(counter))  # StopIteration (done)
```

Each call to `next()` resumes execution right _after_ the last `yield`.

---

## 💡 Generator vs List

|Feature|List|Generator|
|---|---|---|
|Stores data|All at once (in memory)|One at a time (on demand)|
|Memory use|High|Very low|
|Performance|Fast for small data|Better for big / infinite data|
|Example|`[x*x for x in range(5)]`|`(x*x for x in range(5))`|

---

## 🧮 Generator expression example

Instead of writing:

```python
nums = [x*x for x in range(10)]
```

You can write a **generator expression**:

```python
nums = (x*x for x in range(10))
```

Now `nums` is a generator — you can iterate lazily:

```python
for n in nums:
    print(n)
```

✅ Uses almost no memory — computes values _as needed_.

---

## 🔁 How generators behave

1. **`next(gen)`** — get the next value.
    
2. **Automatic iteration** — `for` loops call `next()` internally.
    
3. **StopIteration** — when the generator finishes, it raises this signal.
    
4. **Can be restarted** — only by calling the generator function again (not resuming the same one).
    

---

## 🧠 Why use generators?

✅ **Save memory** — handle big data streams easily.  
✅ **Lazy evaluation** — don’t compute until needed.  
✅ **Pipeline building** — chain multiple steps efficiently.  
✅ **Readable async-like logic** — pause/resume naturally.

---

### 🔧 Real example — processing a large file

```python
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()
```

Usage:

```python
for line in read_lines("data.txt"):
    print(line)
```

Only **one line** is in memory at a time → super efficient.

---

## ⚙️ Summary

|Concept|Description|
|---|---|
|`yield`|Returns a value and pauses the function|
|Generator function|A function containing `yield`|
|Generator object|The result of calling that function|
|`next()`|Resume execution to next `yield`|
|Generator expression|Compact syntax: `(x for x in y)`|

---

Perfect 😄 — let’s visualize exactly how a **Python generator** runs, _pauses_, and _resumes_ at each `yield`.

---

## 🧩 Example to visualize

```python
def demo_gen():
    print("Step 1: Start")
    yield "A"

    print("Step 2: After first yield")
    yield "B"

    print("Step 3: Done")
```

---

## 🕹️ When you use it:

```python
g = demo_gen()
print(next(g))
print(next(g))
print(next(g))
```

---

## 🧠 Execution Timeline (visual diagram)

```
┌──────────────────────────────────────────────┐
│ Call demo_gen()                              │
│  → Creates generator object (no code runs yet)│
└──────────────────────────────────────────────┘
                     │
                     ▼
        next(g) ─────────────► Runs until first yield
                              print("Step 1: Start")
                              yield "A"
                              ───────────────┬─────────────
                                             │ pauses here
Returned value: "A"                          │
                                             ▼
        next(g) ─────────────► Resumes right after yield
                              print("Step 2: After first yield")
                              yield "B"
                              ───────────────┬─────────────
Returned value: "B"                         │ pauses again
                                             ▼
        next(g) ─────────────► Resumes again
                              print("Step 3: Done")
                              Reaches end → StopIteration
```

💡 **Each `next()` resumes execution exactly where it left off.**  
All local variables are preserved automatically.

---

## 📊 Visual analogy

Think of a generator like a **movie with a pause button** 🎬

- `yield` = press **pause**, hand out the current scene
    
- `next()` = press **play**, continue where you paused
    
- `StopIteration` = movie ends
    

---

## 🧮 Example with state memory

```python
def counter():
    n = 1
    while n <= 3:
        print("Yielding:", n)
        yield n
        n += 1
```

Output:

```
>>> g = counter()
>>> next(g)
Yielding: 1
1
>>> next(g)
Yielding: 2
2
>>> next(g)
Yielding: 3
3
>>> next(g)
StopIteration
```

Even though we exited and resumed three times,  
`n` kept its value between calls — **that’s the generator’s memory**.

---

## 🔁 Summary diagram

|Step|Code runs until|Returns|State saved?|
|---|---|---|---|
|1|First `yield`|Value 1|✅|
|2|Next `yield`|Value 2|✅|
|3|End|StopIteration|❌ done|

---

Would you like me to create a **graphic image (flowchart)** of this generator timeline (with arrows, boxes, and yields visually marked)?