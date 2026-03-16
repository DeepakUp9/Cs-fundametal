# Java Exception Handling — The `finally` Block
> *Understand when, why, and how `finally` always executes.*

---

## Introduction

The `finally` block executes **no matter what** — whether an exception was thrown or not, whether it was caught or not. It is most commonly used for cleanup tasks like:

- Closing file streams
- Releasing database connections
- Freeing up resources

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            // code that might throw an exception
        } catch (Exception e) {
            // exception is caught here
        } finally {
            // ALWAYS executes — optional block
        }
    }
}
```

> 💡 `finally` is **optional** — a `try` block can exist with just `catch`, just `finally`, or both.

---

## When Does `finally` Execute?

| Scenario | `catch` runs? | `finally` runs? |
|---|---|---|
| No exception thrown | ❌ No | ✅ Yes |
| Exception thrown and caught | ✅ Yes | ✅ Yes |
| Exception thrown but not caught | ❌ No | ✅ Yes |
| `return` statement in `catch` | ✅ Yes | ✅ Yes (before returning) |
| `try`-`finally` (no `catch`) | N/A | ✅ Yes |

> ✅ **The bottom line:** `finally` always runs. The only exception to this rule is `System.exit()` — calling it shuts down the JVM before `finally` can execute.

---

## Case 1 — Exception Thrown and Handled

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new NullPointerException();
        } catch (Exception e) {
            System.out.println("Exception caught");   // ← runs
        } finally {
            System.out.println("Finally executed");   // ← also runs
        }
    }
}
```

### Output

```
Exception caught
Finally executed
```

### Explanation

1. `NullPointerException` is thrown in `try`
2. `catch(Exception e)` catches it → prints `"Exception caught"`
3. `finally` runs after `catch` completes → prints `"Finally executed"`

```
try → exception → catch → finally
```

---

## Case 2 — Exception Thrown but NOT Handled

```java
class Exceptions {
    public static void main(String args[]) {
        int months = 5;
        int salary = 0;
        try {
            System.out.print(months / salary);   // throws ArithmeticException
        } catch (NullPointerException e) {        // ← wrong type, won't catch it
            System.out.println("Exception caught");
        } finally {
            System.out.println("Finally executed"); // ← still runs!
        }
    }
}
```

### Output

```
Finally executed
Exception in thread "main" java.lang.ArithmeticException: / by zero
```

### Explanation

1. `months / salary` → `5 / 0` throws `ArithmeticException`
2. `catch(NullPointerException e)` does **not** match — wrong exception type
3. `finally` still runs → prints `"Finally executed"`
4. After `finally`, the uncaught `ArithmeticException` propagates and crashes the program

```
try → ArithmeticException → catch (no match) → finally → exception propagates
```

> ✅ **Key point:** Even when an exception goes uncaught, `finally` still executes before the program terminates.

---

## Case 3 — `try`-`finally` Without `catch`

A `try` block can be followed directly by `finally` — no `catch` is required.

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            // code here
        } finally {
            System.out.println("Finally executed");
        }
    }
}
```

### Output

```
Finally executed
```

This is valid Java. If no exception is thrown, `try` completes and `finally` runs. If an exception is thrown, `finally` runs before the exception propagates.

### With an Exception

```java
class Exceptions {
    public static void main(String args[]) {
        int month = 5;
        int salary = 0;
        try {
            System.out.print(month / salary);  // throws ArithmeticException
        } finally {
            System.out.println("Finally executed");  // ← still runs
        }
    }
}
```

### Output

```
Finally executed
Exception in thread "main" java.lang.ArithmeticException: / by zero
```

### Explanation

There is no `catch` here — the `ArithmeticException` is never caught. But `finally` still runs before the program terminates. This is the same behaviour as Case 2 — the only difference is there's no `catch` block at all.

> 💡 **Real-world use:** `try`-`finally` without `catch` is commonly used to guarantee resource cleanup when you don't need to handle the exception yourself — you just want to clean up and let the exception propagate.

```java
FileInputStream file = null;
try {
    file = new FileInputStream("data.txt");
    // process file...
} finally {
    if (file != null) file.close();  // always close, even on exception
}
```

---

## Case 4 — `return` in `catch` with `finally`

What happens when `catch` has a `return` statement? Does `finally` still run?

```java
class Exceptions {
    public static void main(String args[]) {
        int value = 5;
        System.out.println("Value returned: " + findNumber(value));
    }

    public static int findNumber(int value) {
        Integer[] arr = null;
        try {
            System.out.println(arr[0]);   // NullPointerException — arr is null
        } catch (NullPointerException e) {
            return value;                 // ← wants to return here
        } finally {
            System.out.println("Finally executed");  // ← runs BEFORE returning
        }
        return value;
    }
}
```

### Output

```
Finally executed
Value returned: 5
```

### Explanation

1. `arr[0]` throws `NullPointerException` (arr is null)
2. `catch(NullPointerException e)` catches it and reaches `return value`
3. **Before actually returning**, `finally` executes → prints `"Finally executed"`
4. After `finally` completes, the `return value` from `catch` sends `5` back to `main`
5. `main` prints `"Value returned: 5"`

```
try → NullPointerException
catch → return value (paused) → finally → "Finally executed"
return value = 5 → main → "Value returned: 5"
```

> ✅ **Key Rule:** `finally` runs even when there is a `return` inside `catch`. The `return` is **paused** until `finally` completes. Then the return value is sent back to the caller.

---

## Can There Be Multiple `finally` Blocks?

**No.** While a `try` can have multiple `catch` blocks, it can have **only one `finally`** block.

```java
// ✅ Valid — one finally
try { }
catch (Exception e) { }
finally { }

// ❌ Invalid — two finally blocks (compile error)
try { }
catch (Exception e) { }
finally { System.out.println("First"); }
finally { System.out.println("Second"); }  // COMPILE ERROR
```

---

## All Cases Side by Side

```
Case 1: Exception thrown AND caught
  try → throw → catch (handles it) → finally → continues

Case 2: Exception thrown but NOT caught
  try → throw → catch (no match) → finally → exception propagates

Case 3: try-finally (no catch)
  try → (exception or not) → finally → (exception propagates if thrown)

Case 4: return in catch
  try → throw → catch (return paused) → finally → return executes
```

---

## `finally` and the Exception Flow

```
                      ┌─────────────┐
                      │   try block │
                      └──────┬──────┘
                             │
               Exception?    │   No Exception?
              ┌──────────────┴──────────────┐
              ▼                             ▼
     ┌────────────────┐            ┌────────────────┐
     │  catch block   │            │  (skip catch)  │
     │  (if matches)  │            └───────┬────────┘
     └────────┬───────┘                    │
              │                            │
              └───────────┬────────────────┘
                          ▼
                  ┌───────────────┐
                  │ finally block │  ← ALWAYS runs
                  └───────┬───────┘
                          │
              Caught?     │     Not caught?
         ┌────────────────┴────────────────┐
         ▼                                 ▼
  ┌─────────────┐                 ┌─────────────────┐
  │  Continue   │                 │ Exception        │
  │  normally   │                 │ propagates up    │
  └─────────────┘                 └─────────────────┘
```

---

## Key Highlights

| Rule | Detail |
|---|---|
| `finally` is optional | `try`-`catch`, `try`-`finally`, or `try`-`catch`-`finally` are all valid |
| `finally` always executes | Whether or not an exception is thrown or caught |
| `finally` runs before `return` | A `return` in `catch` is delayed until `finally` completes |
| Only one `finally` allowed | Multiple `finally` blocks cause a compile error |
| `System.exit()` bypasses `finally` | Calling `System.exit()` shuts the JVM immediately |

---

## Test Yourself

**Question 1:** What is printed?
```java
try {
    System.out.println("Try");
} finally {
    System.out.println("Finally");
}
System.out.println("After");
```
<details>
<summary>Answer</summary>
<strong>Try, Finally, After</strong> — No exception is thrown. <code>try</code> runs, then <code>finally</code>, then execution continues normally.
</details>

**Question 2:** What is printed?
```java
try {
    throw new Exception();
} catch (Exception e) {
    System.out.println("Catch");
    return;
} finally {
    System.out.println("Finally");
}
```
<details>
<summary>Answer</summary>
<strong>Catch, Finally</strong> — The <code>return</code> in <code>catch</code> is paused until <code>finally</code> runs. <code>finally</code> prints before the method actually returns.
</details>

**Question 3:** What is printed?
```java
int a = 10, b = 0;
try {
    System.out.println(a / b);
} catch (NullPointerException e) {
    System.out.println("Null");
} finally {
    System.out.println("Done");
}
```
<details>
<summary>Answer</summary>
<strong>Done</strong>, then an <code>ArithmeticException</code> — The <code>catch</code> doesn't match (wrong type), but <code>finally</code> still runs and prints <code>"Done"</code>. Then the uncaught exception crashes the program.
</details>

---

