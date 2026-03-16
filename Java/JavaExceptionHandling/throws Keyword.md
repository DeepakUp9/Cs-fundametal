# Java Exception Handling — The `throws` Keyword
> *Learn how to declare, propagate, and override exceptions using `throws`.*

---

## Introduction

So far we've used `throw` to manually raise exceptions inside a method. But what if a method itself doesn't want to handle an exception — it just wants to **warn callers** that an exception might occur and let them deal with it? That's exactly what the `throws` keyword is for.

### `throw` vs. `throws` — The Key Difference

| Keyword | Used In | Purpose | How Many? |
|---|---|---|---|
| `throw` | Method body | Actually raises/throws an exception | One at a time |
| `throws` | Method signature | Declares that a method *might* throw an exception | Multiple allowed |

```java
// throw — used INSIDE the method body
throw new IOException();

// throws — used in the method SIGNATURE
public void readFile(String path) throws IOException { ... }
```

---

## Syntax

```java
accessModifier returnType methodName(params) throws ExceptionType1, ExceptionType2 {
    // method body
}
```

The `throws` clause goes after the parameter list and before the method body. It is the method's way of saying: *"I might produce this exception — caller, you deal with it."*

---

## Checked vs. Unchecked Exceptions

Before diving into examples, it's important to understand **why** `throws` matters differently for different exception types.

```
Throwable
├── Error (serious JVM issues — never caught normally)
└── Exception
    ├── RuntimeException   ← UNCHECKED (compiler doesn't enforce handling)
    │   ├── NullPointerException
    │   ├── ArithmeticException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── ...
    └── Checked Exceptions ← CHECKED (compiler ENFORCES handling or declaration)
        ├── IOException
        ├── SQLException
        ├── FileNotFoundException
        └── ...
```

| Exception Type | Must handle or declare? | Examples |
|---|---|---|
| **Checked** | ✅ Yes — compile error if you don't | `IOException`, `SQLException` |
| **Unchecked** | ❌ No — optional | `NullPointerException`, `ArithmeticException` |

---

## Case 1 — Declaring an Unchecked Exception with `throws`

```java
class Exceptions {
    public static void main(String args[]) {
        readFile("filename.txt");        // ← caller doesn't need to handle it
    }

    public static void readFile(String path) throws RuntimeException {
        System.out.println("Reading File");
    }
}
```

### Output

```
Reading File
```

### Explanation

`RuntimeException` is **unchecked** — the compiler does not force the caller to handle it. Declaring it with `throws` is technically optional but can be useful as documentation to let callers know the method might throw it.

> 💡 Declaring unchecked exceptions in `throws` is a **documentation choice**, not a compiler requirement.

---

## Case 2 — Declaring a Checked Exception with `throws`

```java
import java.io.IOException;

class Exceptions {
    public static void main(String args[]) {
        readFile("filename.txt");        // ← COMPILE ERROR — not handled
    }

    public static void readFile(String path) throws IOException {
        System.out.println("Reading File");
    }
}
```

### Output

```
Compile Error: unreported exception IOException; must be caught or declared to be thrown
```

### Explanation

`IOException` is a **checked exception**. When a method declares `throws IOException`, the compiler enforces that every caller **must** either:

1. Handle it with a `try`-`catch` block, **OR**
2. Also declare `throws IOException` in its own signature (propagating it further up)

```
readFile declares: throws IOException
↓
main calls readFile
↓
Compiler: "main, did you handle IOException?" → NO → COMPILE ERROR
```

### Fix — Handle It in the Caller

```java
import java.io.IOException;

class Exceptions {
    public static void main(String args[]) {
        try {
            readFile("filename.txt");    // ← now handled
        } catch (IOException e) {
            System.out.println("Caught: " + e);
        }
    }

    public static void readFile(String path) throws IOException {
        System.out.println("Reading File");
    }
}
```

### Alternative Fix — Propagate It

```java
// main also declares throws — passes the buck to whoever calls main
public static void main(String args[]) throws IOException {
    readFile("filename.txt");
}
```

---

## Case 3 — Declaring Multiple Exceptions with `throws`

While `throw` can only throw one exception at a time, `throws` can declare **multiple exceptions** separated by commas.

```java
import java.io.IOException;
import java.sql.SQLException;

class Exceptions {
    public static void main(String args[]) {
        try {
            readFile("filename.txt");
        } catch (Exception e) {
            System.out.print("Exception caught: " + e);
        }
    }

    public static void readFile(String path) throws IOException, SQLException {
        System.out.println("Reading File");
        throw new IOException();   // actually throws one at a time
    }
}
```

### Output

```
Reading File
Exception caught: java.io.IOException
```

### Explanation

`readFile` declares it might throw **either** `IOException` **or** `SQLException`. The caller handles both with a single `catch(Exception e)` since `Exception` is the parent of both.

> 💡 Declaring multiple exceptions in `throws` is a contract to the caller: *"Be prepared to handle any of these."*

---

## Overriding Methods with `throws`

When a child class overrides a parent method that has a `throws` clause, strict rules apply.

### Rule Summary

| Scenario | Valid? |
|---|---|
| Child throws **nothing** (removes the exception) | ✅ Yes |
| Child throws the **same exception** as parent | ✅ Yes |
| Child throws a **subclass** of parent's exception | ✅ Yes |
| Child throws a **parent/broader** exception than parent declared | ❌ No — Compile Error |
| Child throws a **new unrelated checked exception** | ❌ No — Compile Error |

---

### ✅ Legal — Child Removes the Exception

```java
class Parent {
    public void readFile(String path) throws IOException {
        // reads file
    }
}

class Child extends Parent {
    @Override
    public void readFile(String path) {    // ← no throws — perfectly legal
        // overridden method
    }
}
```

A child method can always be **stricter** (declare fewer exceptions) than the parent.

---

### ❌ Illegal — Child Adds a Broader Exception

```java
class Parent {
    public void readFile(String path) {    // ← no throws clause
        // reads file
    }
}

class Child extends Parent {
    @Override
    public void readFile(String path) throws IOException {    // ← COMPILE ERROR
        // overridden method
    }
}
```

### Output

```
Compile Error: readFile(String) in Child cannot override readFile(String) in Parent;
overriding method does not throw IOException
```

### Why This Is Illegal

If a parent method doesn't throw any checked exceptions, callers of that method won't have `try`-`catch` blocks around it. If a child could suddenly throw a broader exception, it would break all that calling code.

---

### ✅ Legal — Child Throws a Subclass

```java
class Parent {
    public void readFile(String path) throws Exception {
        // reads file
    }
}

class Child extends Parent {
    @Override
    public void readFile(String path) throws IOException {   // ← subclass of Exception
        // overridden method
    }
}
```

`IOException` **IS-A** `Exception` — so this is allowed. The child is being **more specific** about what it throws, which is safe for callers that already handle `Exception`.

```
Exception (parent's declared exception)
└── IOException (child's narrower, more specific exception) ✅
```

---

## Interfaces and `throws`

The same overriding rules apply when implementing interfaces.

```java
import java.io.*;

interface Filing {
    public void readFile(String path) throws IOException;
}

class Parent implements Filing {

    // ❌ ILLEGAL — Exception is broader than IOException
    // public void readFile(String path) throws Exception { }

    // ✅ LEGAL — FileNotFoundException is a subclass of IOException
    @Override
    public void readFile(String path) throws FileNotFoundException {
        // implementation
    }
}
```

```
IOException (interface declares)
└── FileNotFoundException (implementation narrows it) ✅

Exception (broader than IOException)
└── Cannot use — would widen the contract ❌
```

### Full Working Example

```java
import java.io.*;

class Exceptions {
    public static void main(String args[]) {
        Parent p = new Parent();
        try {
            p.readFile("filename.txt");
        } catch (Exception e) {
            System.out.print("Exception caught: " + e);
        }
    }
}

interface Filing {
    public void readFile(String path) throws IOException;
}

class Parent implements Filing {
    @Override
    public void readFile(String path) throws FileNotFoundException {
        // FileNotFoundException is a subclass of IOException — valid
        System.out.println("Reading file...");
    }
}
```

---

## The Propagation Chain

When a method uses `throws` instead of handling, the exception travels up the call stack:

```
method3() throws IOException       ← declares, doesn't catch
    ↑ propagates to
method2() throws IOException       ← declares, doesn't catch
    ↑ propagates to
method1() {                        ← finally handles it
    try {
        method2();
    } catch (IOException e) {
        // handled here
    }
}
```

Each method in the chain must either handle it or declare `throws`. The exception keeps bubbling up until it's caught.

---

## `throws` Overriding Rules — Visual Summary

```
Parent method: throws Exception
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
  throws nothing   throws Exception   throws IOException
     ✅ OK            ✅ OK           ✅ OK (subclass)

Parent method: throws IOException
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
  throws nothing   throws IOException  throws Exception
     ✅ OK            ✅ OK           ❌ ILLEGAL (broader)
```

---

## Key Highlights

| Rule | Detail |
|---|---|
| `throw` vs `throws` | `throw` raises an exception; `throws` declares one in a method signature |
| `throws` with checked exceptions | Caller must handle or also declare `throws` |
| `throws` with unchecked exceptions | Optional — for documentation purposes |
| Multiple exceptions | `throws IOException, SQLException` — comma-separated |
| Overriding — allowed | No exception, same exception, or subclass exception |
| Overriding — not allowed | Broader or unrelated checked exceptions |
| Interfaces | Same overriding rules apply when implementing interface methods |

---

## Test Yourself

**Question 1:** Will this compile?
```java
class Parent {
    public void doSomething() throws RuntimeException { }
}
class Child extends Parent {
    public void doSomething() throws Exception { }  // Exception is broader
}
```
<details>
<summary>Answer</summary>
<strong>No — Compile Error.</strong> <code>RuntimeException</code> is unchecked, but <code>Exception</code> is a checked exception that is broader. The child cannot widen a checked exception beyond what the parent declared.
</details>

**Question 2:** Will this compile?
```java
import java.io.*;
class MyClass {
    public static void main(String[] args) {
        riskyMethod();
    }
    public static void riskyMethod() throws IOException {
        System.out.println("Risky!");
    }
}
```
<details>
<summary>Answer</summary>
<strong>No — Compile Error.</strong> <code>IOException</code> is a checked exception. <code>main</code> calls <code>riskyMethod()</code> but doesn't handle or declare <code>throws IOException</code>.
</details>

**Question 3:** What is the output?
```java
import java.io.*;
class Test {
    public static void main(String[] args) {
        try {
            riskyMethod();
        } catch (IOException e) {
            System.out.print("Caught: " + e.getClass().getSimpleName());
        }
    }
    public static void riskyMethod() throws IOException {
        throw new FileNotFoundException();
    }
}
```
<details>
<summary>Answer</summary>
<strong>Caught: FileNotFoundException</strong> — <code>FileNotFoundException</code> is a subclass of <code>IOException</code>, so the <code>catch(IOException e)</code> block catches it. <code>e.getClass().getSimpleName()</code> returns the actual runtime type.
</details>

---

