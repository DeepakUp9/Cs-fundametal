# Java Exception Handling — Return Statements in Try-Catch
> *Understand what gets returned when both `catch` and `finally` have return statements.*

---

## Introduction

When both `catch` and `finally` contain `return` statements, Java has specific rules about which value actually gets sent back to the caller. The answer depends on whether you're returning a **primitive** or a **reference type** — and whether `finally` has its own `return` or just mutates the object.

### 🎯 What You'll Learn

- ✅ What happens when both `catch` and `finally` return a **primitive**
- ✅ What happens when `catch` returns a **reference type** and `finally` mutates it
- ✅ The golden rule: `finally`'s `return` always wins over `catch`'s `return`

---

## The Core Rule

> **When both `catch` and `finally` have `return` statements, the value from `finally` is always returned to the caller. The `catch` return is discarded.**

---

## Case 1 — Returning Primitive Values

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
            System.out.println("Exception caught, value is " + value);
            return value;   // ← wants to return 2
        } finally {
            value = 3;
            System.out.println("Finally, value is now " + value);
            return value;   // ← actually returns 3
        }
    }
}
```

### Output

```
Exception caught, value is 2
Finally, value is now 3
Returned Value is: 3
```

### Step-by-Step Breakdown

| Step | Code | `value` |
|---|---|---|
| 1 | `int value = 1` | `1` |
| 2 | `ArrayIndexOutOfBoundsException` thrown | — |
| 3 | `catch` runs: `value = 2` | `2` |
| 4 | `catch` prints: `"Exception caught, value is 2"` | `2` |
| 5 | `catch` reaches `return value` — **paused** | `2` |
| 6 | `finally` runs: `value = 3` | `3` |
| 7 | `finally` prints: `"Finally, value is now 3"` | `3` |
| 8 | `finally` executes `return value` → returns `3` | `3` |
| 9 | `catch`'s `return 2` is **discarded** | — |
| 10 | `main` prints: `"Returned Value is: 3"` | — |

### Key Insight

The `catch` block's `return value` doesn't execute immediately. Java **pauses** it, runs `finally`, and if `finally` has its own `return`, that value wins. The `catch` return is permanently discarded.

```
catch → return 2 (PAUSED)
finally → value = 3; return 3 (WINS)
catch's return 2 → DISCARDED
```

---

## Case 2 — Returning Reference Types

### Code

```java
class Parent {
    public static void main(String[] args) {
        Child c = new Child();
        System.out.print("Returned Value is " + c.method1()[0]);
    }
}

class Child {
    public int[] method1() {
        int[] arr = {1};
        try {
            System.out.println(arr[4]);           // index 4 doesn't exist
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Exception caught, value is " + arr[0]);
            return arr;    // ← returns reference to arr (arr[0] = 1)
        } finally {
            arr[0] = 2;    // ← mutates the actual array object
            System.out.println("Finally, value is " + arr[0]);
            // NO return statement here
        }
        return arr;
    }
}
```

### Output

```
Exception caught, value is 1
Finally, value is 2
Returned Value is 2
```

### Step-by-Step Breakdown

| Step | Code | `arr[0]` |
|---|---|---|
| 1 | `int[] arr = {1}` | `1` |
| 2 | `arr[4]` → `ArrayIndexOutOfBoundsException` | — |
| 3 | `catch` prints: `"Exception caught, value is 1"` | `1` |
| 4 | `catch` reaches `return arr` — Java saves the **reference** | `1` |
| 5 | `finally` runs: `arr[0] = 2` — **mutates the actual object** | `2` |
| 6 | `finally` prints: `"Finally, value is 2"` | `2` |
| 7 | No `return` in `finally` — `catch`'s saved reference is used | `2` |
| 8 | The reference points to the **same array** — `arr[0]` is now `2` | `2` |
| 9 | `main` accesses `[0]` → prints `"Returned Value is 2"` | — |

### Key Insight

When `catch` returns an object reference, Java saves **the reference** — not a copy of the object. The reference points to the same `arr` object in memory. When `finally` mutates `arr[0]`, it's changing the exact same object that the saved reference points to. So the caller receives a reference to the already-modified array.

```
catch → saves reference to arr (arr[0] still = 1 at save time)
finally → arr[0] = 2  (mutates the SAME object in memory)
returned reference still points to arr → arr[0] is now 2
caller sees 2
```

---

## The Critical Difference — Primitive vs. Reference

This is the most important distinction in this topic:

| Aspect | Primitive (`int`, `double`, etc.) | Reference (`int[]`, `Object`, etc.) |
|---|---|---|
| **What `catch` saves** | A **copy** of the value | A **reference** (pointer) to the object |
| **Can `finally` change the returned value without its own `return`?** | ❌ No — copy is independent | ✅ Yes — mutates the same object |
| **Does `finally`'s `return` override `catch`'s?** | ✅ Yes | ✅ Yes |
| **Example outcome** | `finally` must use `return` to override | `finally` just needs to mutate the object |

---

## All Combinations — What Gets Returned?

| `catch` has `return`? | `finally` has `return`? | What is returned? |
|---|---|---|
| ✅ Yes | ✅ Yes | `finally`'s value (catch's is discarded) |
| ✅ Yes | ❌ No (primitive) | `catch`'s value (finally can't change it) |
| ✅ Yes | ❌ No (reference, mutated) | `catch`'s reference — but pointing to mutated object |
| ❌ No | ✅ Yes | `finally`'s value |
| ❌ No | ❌ No | The `return` at the end of the method |

---

## Side-by-Side Comparison: Case 1 vs. Case 2

| | Case 1 (Primitive `int`) | Case 2 (Reference `int[]`) |
|---|---|---|
| Type | Primitive | Object reference |
| `catch` returns | `value` (= 2 at time of catch) | `arr` (reference) |
| `finally` does | `value = 3; return value` | `arr[0] = 2` (no return) |
| `finally` has `return`? | ✅ Yes | ❌ No |
| Final result | `3` (`finally` return wins) | `2` (reference mutated) |
| Why? | `finally`'s `return` overrides `catch`'s | Mutation affects the shared object |

---

## Visual Memory Model for Case 2

```
catch block executes:
  return arr;
  ┌─────────────────────────┐
  │ Java saves: ref → [ arr ]│  arr[0] = 1 at this moment
  └─────────────────────────┘

finally block executes:
  arr[0] = 2;
  ┌────────────────────────────────────────┐
  │ SAME object in memory — arr[0] is now 2│
  └────────────────────────────────────────┘

Returned to caller:
  saved ref → [ arr ]   arr[0] = 2   ← caller sees 2
```

---

## Practical Advice

> ⚠️ **Avoid putting `return` statements in `finally` blocks.** It is widely considered bad practice because it silently suppresses exceptions and discards the `catch` return value, making code very hard to debug.

```java
// ❌ Confusing and bug-prone
finally {
    return someValue;  // swallows exceptions and overrides catch
}

// ✅ Better — only use finally for cleanup
finally {
    connection.close();  // cleanup, no return
}
```

> ⚠️ **Be careful mutating reference types in `finally`.** If `catch` returns a reference and `finally` modifies the object, the caller will receive the modified version — which may not be what you intended.

---

## Key Highlights

| Rule | Detail |
|---|---|
| `finally` always runs before `return` | Even a `return` in `catch` is paused until `finally` completes |
| `finally`'s `return` wins | When both `catch` and `finally` return, `finally`'s value is sent to the caller |
| Primitive returns are copies | `finally` must have its own `return` to change what `catch` returns |
| Reference returns share memory | `finally` can change the returned value by mutating the object — no `return` needed |
| Avoid `return` in `finally` | It silently suppresses exceptions and is hard to reason about |

---

## Test Yourself

**Question 1:** What is printed?
```java
public static int test() {
    try {
        return 1;
    } finally {
        return 2;
    }
}
System.out.println(test());
```
<details>
<summary>Answer</summary>
<strong>2</strong> — <code>finally</code>'s <code>return 2</code> overrides <code>try</code>'s <code>return 1</code>.
</details>

**Question 2:** What is printed?
```java
public static int test() {
    int x = 10;
    try {
        return x;
    } finally {
        x = 20;
        // no return here
    }
}
System.out.println(test());
```
<details>
<summary>Answer</summary>
<strong>10</strong> — <code>try</code> saves a copy of <code>x = 10</code> for return. <code>finally</code> changes <code>x</code> to 20 but has no <code>return</code>, so the original saved value <code>10</code> is returned.
</details>

**Question 3:** What is printed?
```java
public static int[] test() {
    int[] arr = {10};
    try {
        return arr;
    } finally {
        arr[0] = 20;
    }
}
System.out.println(test()[0]);
```
<details>
<summary>Answer</summary>
<strong>20</strong> — <code>try</code> saves a reference to <code>arr</code>. <code>finally</code> mutates <code>arr[0]</code> to 20. The returned reference still points to the same object, so the caller sees 20.
</details>

---

