# Java Exception Handling — Try & Catch Fundamentals
> *Understand the rules, variations, and edge cases of try-catch blocks in Java.*

---

## Introduction

The `try`-`catch` block is the foundation of exception handling in Java. It lets you protect risky code and gracefully handle problems when they arise — instead of letting the program crash.

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            // code that might throw an exception
        } catch (Exception e) {
            // code to handle the exception
        }
    }
}
```

- The **`try` block** contains code where you believe an exception *might* occur — for example, dividing by a user-supplied value that could be zero.
- The **`catch` block** catches that exception and contains the logic to handle it gracefully.

---

## Case 1 — Can We Throw Multiple Exceptions in a Single `try`?

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new NullPointerException();     // exception 1
            throw new ArithmeticException();      // exception 2 — UNREACHABLE
        } catch (Exception e) {
            System.out.print(e);
        }
    }
}
```

### Output

```
Compile Error: unreachable statement
```

### Explanation

When `throw new NullPointerException()` executes, control **immediately leaves** the `try` block to find a matching `catch`. The second `throw` statement can **never be reached** — it is dead code.

```
try {
    throw new NullPointerException();  ← Control jumps here
    throw new ArithmeticException();   ← NEVER reached
}
```

> ✅ **Rule:** Only one exception can be in flight at a time. Once a `throw` statement executes, the rest of the `try` block is skipped.

### What If You Need to Throw Multiple Exceptions?

You can't throw two exceptions sequentially, but you can use **multi-catch** (covered in Case 4) or **nested try blocks** to handle multiple exception scenarios.

---

## Case 2 — Does the Order of Multiple `catch` Blocks Matter?

### Code (Wrong Order — Will Not Compile)

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new NumberFormatException();
        } catch (Exception e) {                     // ← parent class FIRST
            System.out.print("A");
        } catch (NumberFormatException e) {         // ← UNREACHABLE
            System.out.print("B");
        } catch (ArithmeticException e) {           // ← UNREACHABLE
            System.out.print("C");
        }
    }
}
```

### Output

```
Compile Error: exception NumberFormatException has already been caught
```

### Explanation

`NumberFormatException` and `ArithmeticException` are both subclasses of `Exception`. When Java checks each `catch` block from **top to bottom**, `catch(Exception e)` matches **any** exception — so `NumberFormatException` and `ArithmeticException` can never be reached.

```
Exception (parent)
├── NumberFormatException (child)
└── ArithmeticException (child)
```

Java detects this at **compile time** and refuses to compile. This prevents writing catch blocks that can never execute.

---

### Code (Correct Order — Works)

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            throw new NumberFormatException();
        } catch (NumberFormatException e) {   // ← child class first
            System.out.print("A");
        } catch (ArithmeticException e) {     // ← sibling class
            System.out.print("B");
        } catch (Exception e) {               // ← parent class LAST
            System.out.print("C");
        }
    }
}
```

### Output

```
A
```

### Explanation

Java matches exceptions by checking `catch` blocks **from top to bottom** and picks the **first match**. With the correct ordering:

1. `NumberFormatException` is thrown
2. `catch(NumberFormatException e)` matches → prints `"A"`
3. The other `catch` blocks are skipped

> ✅ **Rule — Bottom-Up Hierarchy:** When using multiple `catch` blocks with related exceptions (IS-A relationship), always put the **most specific (child) class first** and the **most general (parent) class last**.

### Catch Order Quick Reference

| Order | Valid? | Reason |
|---|---|---|
| Child → Parent | ✅ Yes | Specific catches first, generic fallback last |
| Parent → Child | ❌ No | Compile error — child is unreachable |
| Unrelated exceptions (any order) | ✅ Yes | No IS-A relationship, no overlap |

---

## Case 3 — Nested `try`-`catch` Blocks

### Puzzle A — Inner `catch` Handles the Exception

```java
class Exceptions {
    public static void main(String args[]) {
        int a = 5;
        int b = 0;
        try {
            try {
                a = a / b;                        // throws ArithmeticException
            } catch (ArithmeticException e) {
                System.out.print("B");            // ← caught here
            }
        } catch (ArithmeticException e) {
            System.out.print("A");                // ← never reached
        }
    }
}
```

### Output

```
B
```

### Explanation

`a / b` throws an `ArithmeticException`. Java searches for a matching `catch` starting with the **innermost** enclosing `try` block. The inner `catch(ArithmeticException e)` matches — so it handles the exception and prints `"B"`. The outer `catch` is never reached.

```
Outer try
└── Inner try
    └── ArithmeticException thrown
        └── Inner catch matches → "B" printed
            → Outer catch never reached
```

---

### Puzzle B — Inner `finally` + Outer `catch`

```java
class Exceptions {
    public static void main(String args[]) {
        int a = 5;
        int b = 0;
        try {
            try {
                a = a / b;              // throws ArithmeticException
            } finally {
                System.out.print("B"); // ← always runs
            }
        } catch (ArithmeticException e) {
            System.out.print("A");     // ← catches the propagated exception
        }
    }
}
```

### Output

```
BA
```

### Explanation

1. `a / b` throws `ArithmeticException` inside the inner `try`
2. The inner `try` has no `catch` block — the exception is **not caught**
3. The inner `finally` runs first → prints `"B"`
4. After `finally`, the exception continues propagating upward
5. The outer `catch(ArithmeticException e)` catches it → prints `"A"`

```
Inner try → exception thrown (no inner catch)
Inner finally → "B" printed
Exception propagates to outer catch → "A" printed
```

> ✅ **Rule:** When an exception is not caught by the inner `try`, it propagates to the **outer `try`** for a matching `catch`. The inner `finally` still runs before propagation.

### Nested Try — How Propagation Works

```
try (outer)
└── try (inner)         ← exception thrown here
    └── finally         ← runs first, always
    exception propagates upward
└── catch (outer)       ← catches the propagated exception
```

---

## Case 4 — Catching Multiple Exceptions in a Single `catch`

Since Java 7, multiple exception types can be caught in a **single `catch` block** using the pipe `|` operator.

### Code

```java
class Exceptions {
    public static void main(String args[]) {
        int a = 1;
        int b = 0;

        try {
            System.out.print(a / b);              // throws ArithmeticException
        } catch (ArithmeticException | NullPointerException e) {
            System.out.print("A");
        }
    }
}
```

### Output

```
A
```

### Explanation

`a / b` throws `ArithmeticException`. The `catch` block handles **either** `ArithmeticException` or `NullPointerException` — since `ArithmeticException` matches, the block runs and prints `"A"`.

### Multi-Catch Syntax Rules

```java
// ✅ Correct: unrelated exception types
catch (ArithmeticException | NullPointerException e) { }

// ✅ Correct: any number of unrelated types
catch (IOException | SQLException | ClassNotFoundException e) { }

// ❌ Wrong: parent and child in same multi-catch
catch (Exception | NumberFormatException e) { }  // compile error
// NumberFormatException IS-A Exception — redundant
```

> ✅ **Rule:** In a multi-catch block, you cannot list a parent and its child class together — the child is already covered by the parent, so it's flagged as a compile error.

### Multi-Catch vs. Separate Catch Blocks

| Feature | Multi-catch `A | B` | Separate catch blocks |
|---|---|---|
| **Code brevity** | ✅ More concise | ⚠️ More verbose |
| **Different handling per type** | ❌ Same handler for all | ✅ Different code per type |
| **IS-A rule** | ❌ Cannot mix parent/child | ✅ Allowed (child first) |
| **Since** | Java 7 | All versions |

---

## Bonus — Throwing Exceptions Inside `catch`

You can throw a new exception inside a `catch` block. This is common when you want to wrap a low-level exception in a more meaningful one.

```java
class Exceptions {
    public static void main(String args[]) {
        try {
            try {
                throw new ArithmeticException("Division failed");
            } catch (ArithmeticException e) {
                System.out.println("Caught: " + e.getMessage());
                throw new RuntimeException("Wrapped exception", e); // re-throw
            }
        } catch (RuntimeException e) {
            System.out.println("Re-caught: " + e.getMessage());
        }
    }
}
```

### Output

```
Caught: Division failed
Re-caught: Wrapped exception
```

> 💡 This pattern — catching one exception and throwing another — is called **exception chaining** or **exception wrapping**. It preserves the original cause while providing a more meaningful exception at the higher level.

---

## All Cases — Quick Summary

| Case | Key Rule | Outcome |
|---|---|---|
| **Two throws in one `try`** | Only one exception can be in flight | Compile error — unreachable statement |
| **Wrong `catch` order (parent first)** | Child exceptions become unreachable | Compile error |
| **Correct `catch` order (child first)** | Most specific match wins | Correct — first matching `catch` runs |
| **Nested `try` — inner catch** | Exception caught at innermost level | Only inner `catch` runs |
| **Nested `try` — no inner catch** | Exception propagates after `finally` | `finally` runs, then outer `catch` |
| **Multi-catch `A | B`** | Handles either type in one block | Clean, concise error handling |
| **Throw inside `catch`** | Starts a new exception chain | New exception propagates upward |

---

## Key Highlights

- The **`try` block** contains code where an exception might occur
- The **`catch` block**, following `try`, catches and handles the exception
- There can be **multiple `catch` clauses** after a single `try`
- When using related exceptions, follow **child-first, parent-last** ordering
- **Multi-catch** (`|`) handles multiple unrelated types in one block — since Java 7
- Exceptions can also be **thrown inside `catch`** to wrap or re-throw errors
- In nested `try` blocks, exceptions propagate **outward** if not caught at the inner level
- `finally` always runs before an exception propagates to an outer `catch`

---

## Test Yourself

**Question 1:** What is printed?
```java
try {
    throw new NullPointerException();
} catch (RuntimeException e) {
    System.out.print("A");
} catch (NullPointerException e) {
    System.out.print("B");
}
```
<details>
<summary>Answer</summary>
<strong>Compile Error</strong> — <code>NullPointerException</code> extends <code>RuntimeException</code>, so the second <code>catch</code> is unreachable. Put <code>NullPointerException</code> first.
</details>

**Question 2:** What is printed?
```java
try {
    try {
        throw new Exception();
    } catch (NullPointerException e) {
        System.out.print("A");
    }
} catch (Exception e) {
    System.out.print("B");
}
```
<details>
<summary>Answer</summary>
<strong>B</strong> — The inner <code>catch</code> doesn't match <code>Exception</code> (it only catches <code>NullPointerException</code>). The exception propagates to the outer <code>catch(Exception e)</code>.
</details>

**Question 3:** What is printed?
```java
try {
    System.out.print("1");
    throw new ArithmeticException();
} catch (ArithmeticException | NullPointerException e) {
    System.out.print("2");
} catch (Exception e) {
    System.out.print("3");
}
```
<details>
<summary>Answer</summary>
<strong>12</strong> — Prints "1" from the <code>try</code> block, then the multi-catch handles <code>ArithmeticException</code> and prints "2". The final <code>catch</code> is never reached.
</details>

---

