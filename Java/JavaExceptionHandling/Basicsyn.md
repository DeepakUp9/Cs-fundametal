# Java Exception Handling — Try, Catch & Finally Edge Cases
> *Understand the tricky behaviours of exception handling in Java.*

---

## Introduction

Exception handling in Java follows very specific rules about how `try`, `catch`, and `finally` blocks interact — especially when exceptions are thrown inside them. These rules can produce surprising results if you don't know them. This guide walks through 7 carefully chosen code puzzles, each designed to expose a different edge case of Java's exception handling mechanism.

### 🎯 What You'll Learn

- ✅ How `finally` always executes — and what happens when it throws too
- ✅ What happens to an exception thrown inside `catch`
- ✅ Why `finally` can override a `return` statement from `catch`
- ✅ The correct order of `try` → `catch` → `finally`
- ✅ Why only one `finally` block is allowed per `try`
- ✅ How array references work through `catch` and `finally`

---

## Core Rules Before We Begin

| Rule | Detail |
|---|---|
| **`finally` always runs** | Executes whether or not an exception occurs — even after a `return` |
| **`finally` can suppress exceptions** | If `finally` throws an exception, it replaces any exception from `try` or `catch` |
| **`finally` can override `return`** | A `return` inside `finally` replaces the value returned by `catch` |
| **Only one `finally` per `try`** | Multiple `finally` blocks cause a compile error |
| **`catch` must follow `try`** | A `finally` block between `try` and `catch` is invalid syntax |
| **Object references persist** | If `catch` returns an object reference, `finally` can still mutate it |

---

## Puzzle 1 — Exception in `finally` Wins

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new ArithmeticException();
        } catch (Exception e) {
            System.out.println("Catch");
            throw new NullPointerException();
        } finally {
            System.out.println("Finally");
            throw new NumberFormatException();
        }
    }
}
```

### Output

```
Catch
Finally
Exception in thread "main" java.lang.NumberFormatException
```

### Explanation

**Step by step:**

1. `try` throws `ArithmeticException`
2. `catch(Exception e)` catches it (since `Exception` is the parent of all exceptions) → prints `"Catch"`
3. `catch` then throws `NullPointerException` — but before it can propagate...
4. `finally` **always runs**, so it executes next → prints `"Finally"`
5. `finally` throws `NumberFormatException` — this **replaces** the `NullPointerException` that was in flight

> ⚠️ **Key Rule:** If `finally` throws an exception, it **silently discards** any exception that was already being propagated from `try` or `catch`. The `NullPointerException` is simply lost.

---

## Puzzle 2 — Only the `finally` Exception Survives

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new ArithmeticException();
        } catch (Exception e) {
            throw new NullPointerException();
        } finally {
            throw new NumberFormatException();
        }
    }
}
```

### Output

```
Exception in thread "main" java.lang.NumberFormatException
```

### Explanation

**Step by step:**

1. `try` throws `ArithmeticException`
2. `catch(Exception e)` catches it and throws `NullPointerException`
3. `finally` runs (always!) and throws `NumberFormatException`
4. The `NullPointerException` is **discarded** — only `NumberFormatException` propagates

This is identical to Puzzle 1, but with no print statements. The key takeaway is the same:

> ✅ **Rule confirmed:** An exception thrown inside `finally` always wins over any exception thrown in `try` or `catch`.

### Comparison: Puzzle 1 vs Puzzle 2

| | Puzzle 1 | Puzzle 2 |
|---|---|---|
| `catch` prints? | Yes — `"Catch"` | No |
| `finally` prints? | Yes — `"Finally"` | No |
| Final exception | `NumberFormatException` | `NumberFormatException` |
| `NullPointerException` survives? | ❌ No | ❌ No |

---

## Puzzle 3 — `catch` Cannot Come After `finally`

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            System.out.println("A");
            throw new Exception();
        } finally {
            System.out.println("B");
        } catch (Exception e) {       // ← COMPILE ERROR
            System.out.println("C");
        }
    }
}
```

### Output

```
Compile Error: catch block must come before finally block
```

### Explanation

Java enforces a strict **block order** for `try`-`catch`-`finally`:

```
✅ Correct order:    try → catch → finally
❌ Wrong order:      try → finally → catch
```

Once `finally` appears, no more `catch` blocks are allowed. The `catch` block **must** come immediately after `try` (or after other `catch` blocks), never after `finally`.

### Valid Structures

```java
// ✅ Structure 1: try + catch
try { ... }
catch (Exception e) { ... }

// ✅ Structure 2: try + finally (no catch)
try { ... }
finally { ... }

// ✅ Structure 3: try + catch + finally
try { ... }
catch (Exception e) { ... }
finally { ... }

// ❌ Structure 4: try + finally + catch (ILLEGAL)
try { ... }
finally { ... }
catch (Exception e) { ... }  // Compile error
```

---

## Puzzle 4 — `try` + `finally` Without `catch`

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            System.out.println("A");
            throw new Exception();
        } finally {
            System.out.println("B");
        }
    }
}
```

### Output

```
A
B
Exception in thread "main" java.lang.Exception
```

### Explanation

**Step by step:**

1. `try` executes → prints `"A"`
2. `try` throws `Exception`
3. There is no `catch` block — the exception is **not caught**
4. `finally` still runs → prints `"B"`
5. After `finally` completes, the uncaught exception propagates and crashes the program

> ✅ **Key Rule:** `catch` is **optional** — a `try` block can have just `finally`. But if an exception is thrown and there's no matching `catch`, it still propagates after `finally` runs.

### Real-World Use Case

`try`-`finally` without `catch` is commonly used for **resource cleanup**:

```java
FileInputStream file = null;
try {
    file = new FileInputStream("data.txt");
    // process file...
} finally {
    if (file != null) file.close(); // always close, even on exception
}
```

---

## Puzzle 5 — `finally` Overrides the `return` Value

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        System.out.print("Returned Value is: " + method1());
    }

    public static int method1() {
        int value = 1;
        try {
            throw new ArrayIndexOutOfBoundsException();
        } catch (ArrayIndexOutOfBoundsException e) {
            value = 2;
            return value;     // ← wants to return 2
        } finally {
            value += 2;       // value becomes 4
            return value;     // ← actually returns 4
        }
    }
}
```

### Output

```
Returned Value is: 4
```

### Explanation

This is one of the most surprising behaviours in Java exception handling.

**Step by step:**

| Step | Code | `value` |
|---|---|---|
| 1 | `int value = 1;` | 1 |
| 2 | `ArrayIndexOutOfBoundsException` thrown | — |
| 3 | `catch` executes: `value = 2;` | 2 |
| 4 | `catch` says `return value;` (wants to return 2) | 2 |
| 5 | **Before returning**, `finally` runs: `value += 2;` | 4 |
| 6 | `finally` has its own `return value;` — returns 4 | 4 |
| 7 | The `catch`'s `return 2` is **discarded** | — |

> ⚠️ **Key Rule:** A `return` statement inside `finally` **overrides** any `return` in `try` or `catch`. The value prepared by `catch` is thrown away.

### What If `finally` Had No `return`?

```java
// finally WITHOUT return:
finally {
    value += 2;  // value becomes 4, but no return here
}
// catch's return value (2) would be used → prints "Returned Value is: 2"
```

> 💡 When `finally` modifies a **primitive** but doesn't `return`, the original `catch` return value is used. When `finally` has its own `return`, it wins.

---

## Puzzle 6 — Only One `finally` Block Allowed

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            System.out.println("A");
            throw new Exception();
        } catch (Exception e) {
            System.out.println("B");
        } finally {
            System.out.println("C");
        } finally {               // ← COMPILE ERROR
            System.out.println("D");
        }
    }
}
```

### Output

```
Compile Error: only one finally block is allowed per try statement
```

### Explanation

Java's syntax permits **exactly one** `finally` block per `try` statement. Having two `finally` blocks is a compile-time error — there is no workaround except nesting try blocks.

### If You Need Behaviour from Two "Finally" Blocks

```java
// ✅ Workaround: nested try blocks
try {
    try {
        throw new Exception();
    } catch (Exception e) {
        System.out.println("B");
    } finally {
        System.out.println("C");  // inner finally
    }
} finally {
    System.out.println("D");      // outer finally
}
```

---

## Puzzle 7 — `finally` Mutates an Array Returned by `catch`

### Code

```java
class parent {
    public static void main(String[] args) {
        child c = new child();
        System.out.print("Returned Value is " + c.method1()[0]);
    }
}

class child {
    public int[] method1() {
        int[] arr = {1};
        try {
            System.out.println(arr[4]); // index 4 doesn't exist
        } catch (ArrayIndexOutOfBoundsException e) {
            return arr;       // ← wants to return arr (arr[0] = 1)
        } finally {
            arr[0] += 2;      // arr[0] becomes 3
        }
        return arr;
    }
}
```

### Output

```
Returned Value is 3
```

### Explanation

This puzzle looks similar to Puzzle 5 but has a crucial difference — the return type is an **array (object reference)**, not a primitive.

**Step by step:**

| Step | Code | `arr[0]` |
|---|---|---|
| 1 | `int[] arr = {1};` | 1 |
| 2 | `arr[4]` → `ArrayIndexOutOfBoundsException` | — |
| 3 | `catch` executes: `return arr;` | 1 |
| 4 | Java saves the **reference** to `arr` to return later | — |
| 5 | `finally` runs: `arr[0] += 2;` → modifies the actual array | 3 |
| 6 | `catch`'s saved reference now points to the **modified** array | — |
| 7 | Returns the array → `[0]` is `3` | 3 |

> ✅ **Key Rule:** When `catch` returns an object reference, `finally` can still mutate the object that reference points to. The `return` in `catch` stores the reference — not a copy of the object. So `finally`'s changes to the array are visible in the returned result.

### Puzzle 5 vs. Puzzle 7 — Primitive vs. Object Reference

| | Puzzle 5 (int) | Puzzle 7 (int[]) |
|---|---|---|
| Type returned | Primitive `int` | Object reference `int[]` |
| `finally` has `return`? | ✅ Yes | ❌ No |
| `catch`'s return value used? | ❌ No (`finally` return wins) | ✅ Yes (same reference) |
| `finally` mutation visible? | N/A | ✅ Yes — same array object |
| Final output | `4` | `3` |

---

## Summary of All 7 Puzzles

| # | Key Concept | Result |
|---|---|---|
| 1 | `finally` throws → overrides `catch`'s exception | `Catch`, `Finally`, `NumberFormatException` |
| 2 | `finally` throws → only that exception survives | `NumberFormatException` |
| 3 | `catch` after `finally` is illegal | **Compile Error** |
| 4 | `try`+`finally` without `catch` — exception still propagates | `A`, `B`, then exception |
| 5 | `finally` with `return` → overrides `catch`'s `return` | Prints `4` |
| 6 | Two `finally` blocks → illegal | **Compile Error** |
| 7 | `finally` mutates object returned by `catch` | Prints `3` |

---

## The Golden Rules of `try`-`catch`-`finally`

```
┌─────────────────────────────────────────────────────────┐
│  GOLDEN RULES                                           │
│                                                         │
│  1. finally ALWAYS runs (except System.exit())          │
│                                                         │
│  2. Exception in finally KILLS the previous exception   │
│                                                         │
│  3. return in finally OVERRIDES return in catch         │
│                                                         │
│  4. Order must be: try → catch → finally                │
│                                                         │
│  5. Only ONE finally block per try                      │
│                                                         │
│  6. Object references returned by catch CAN be          │
│     mutated by finally (same reference)                 │
│                                                         │
│  7. catch is optional — try + finally is valid          │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Test Yourself

Before looking at answers, predict the output:

**Question A:**
```java
try {
    return 1;
} finally {
    return 2;
}
```
<details>
<summary>Answer</summary>
Returns <code>2</code> — <code>finally</code>'s <code>return</code> always wins.
</details>

**Question B:**
```java
try {
    throw new RuntimeException("A");
} finally {
    throw new RuntimeException("B");
}
```
<details>
<summary>Answer</summary>
Throws <code>RuntimeException("B")</code> — exception from <code>finally</code> replaces the one from <code>try</code>.
</details>

**Question C:**
```java
try {
    throw new Exception();
} catch (Exception e) {
    System.out.println("Caught");
} finally {
    System.out.println("Finally");
}
System.out.println("After");
```
<details>
<summary>Answer</summary>
Prints: <code>Caught</code>, <code>Finally</code>, <code>After</code> — exception is caught, so execution continues normally after the block.
</details>

---

