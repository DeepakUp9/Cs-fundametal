# Java Exception Handling — Creating Custom Exceptions
> *Learn how to define, throw, and structure your own exception classes.*

---

## Introduction

Java's built-in exceptions cover many common scenarios, but real applications often have business rules that don't map neatly to a `NullPointerException` or `IOException`. For example:

- A banking app might need `InsufficientFundsException`
- A library system might need `BookNotFoundException`
- An e-commerce app might need `OutOfStockException`

**Custom exceptions** let you create meaningful, domain-specific exceptions that make your code clearer, your error messages more helpful, and your application logic easier to follow.

---

## Two Types of Custom Exceptions

Just like built-in exceptions, custom exceptions can be either **checked** or **unchecked** depending on what they extend.

### Checked Custom Exception

```java
// Extend Exception → checked (caller MUST handle or declare throws)
class MyCustomException extends Exception {

    MyCustomException(String message) {
        super(message);   // passes message to Exception's constructor
    }
}
```

### Unchecked Custom Exception

```java
// Extend RuntimeException → unchecked (handling is optional)
class MyCustomException extends RuntimeException {

    MyCustomException(String message) {
        super(message);   // passes message to RuntimeException's constructor
    }
}
```

### Which Should You Choose?

| Use Checked When... | Use Unchecked When... |
|---|---|
| The caller can reasonably be expected to recover | The error represents a programming bug |
| The exception is due to external factors (file, DB, network) | The error is due to invalid arguments or state |
| You want the compiler to enforce handling | You don't want to force callers to write try-catch |
| Example: `BookNotFoundException` in a library search | Example: `InvalidAgeException` for a negative age |

---

## Rules for Creating Custom Exceptions

| Rule | Detail |
|---|---|
| **Name ending** | Always end the class name with `Exception` (e.g., `BookNotFoundException`) |
| **Extends** | Must extend `Exception` (checked) or `RuntimeException` (unchecked) |
| **Constructor** | Create at least one constructor that accepts a `String message` |
| **`super(message)`** | Call `super(message)` to pass the message to the parent class |

---

## Creating and Throwing a Custom Exception

### Step 1 — Define the Exception Class

```java
// Checked custom exception
class MyCustomException extends Exception {

    MyCustomException(String message) {
        super(message);
    }
}
```

### Step 2 — Throw and Catch It

```java
class MyClass {
    public static void main(String args[]) {
        try {
            throw new MyCustomException("Result not found");
        } catch (MyCustomException e) {
            System.out.print(e.getMessage());
        }
    }
}
```

### Output

```
Result not found
```

### Explanation

1. `throw new MyCustomException("Result not found")` creates and throws the exception
2. The `String "Result not found"` is passed to the constructor
3. The constructor calls `super(message)` — this stores the message in the `Exception` parent class
4. `catch(MyCustomException e)` catches it
5. `e.getMessage()` retrieves the stored message string

---

## Real-World Example — Domain-Specific Custom Exceptions

### Example 1 — Bank Account Application

```java
// Custom checked exception
class InsufficientFundsException extends Exception {

    private double amount;   // extra context about the error

    InsufficientFundsException(String message, double amount) {
        super(message);
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }
}

class BankAccount {
    private double balance = 100.0;

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(
                "Insufficient funds. Requested: $" + amount + ", Available: $" + balance,
                amount
            );
        }
        balance -= amount;
        System.out.println("Withdrawal successful. New balance: $" + balance);
    }
}

class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        try {
            account.withdraw(200.0);
        } catch (InsufficientFundsException e) {
            System.out.println("Error: " + e.getMessage());
            System.out.println("Attempted amount: $" + e.getAmount());
        }
    }
}
```

### Output

```
Error: Insufficient funds. Requested: $200.0, Available: $100.0
Attempted amount: $200.0
```

> 💡 Custom exceptions can carry **extra data fields** beyond just the message. Here, `amount` provides context that a plain `Exception` message can't express as cleanly.

---

### Example 2 — Age Validation (Unchecked)

```java
// Unchecked custom exception — no try-catch required
class InvalidAgeException extends RuntimeException {

    InvalidAgeException(String message) {
        super(message);
    }
}

class Person {
    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new InvalidAgeException("Invalid age: " + age + ". Age must be between 0 and 150.");
        }
        this.age = age;
    }
}

class Main {
    public static void main(String[] args) {
        Person p = new Person();
        p.setAge(-5);  // throws InvalidAgeException
    }
}
```

### Output

```
Exception in thread "main" InvalidAgeException: Invalid age: -5. Age must be between 0 and 150.
```

---

## Wrapping Custom Exceptions (Exception Hierarchy)

In larger applications — especially RESTful APIs or layered architectures — custom exceptions are often organized into a hierarchy. A **parent custom exception** represents the application's general error category, and **child custom exceptions** represent specific error types.

### Example — Library Application

```java
// Parent custom exception (application-level)
class LibraryApplicationException extends Exception {

    LibraryApplicationException(String message) {
        super(message);
    }
}

// Child custom exception (specific error)
class BookNotFoundException extends LibraryApplicationException {

    BookNotFoundException(String message) {
        super(message);
    }
}

// Another child custom exception
class BookAlreadyCheckedOutException extends LibraryApplicationException {

    BookAlreadyCheckedOutException(String message) {
        super(message);
    }
}
```

### Using the Hierarchy

```java
class Library {
    public void findBook(String title) throws LibraryApplicationException {
        // Simulating book not found
        throw new BookNotFoundException("Book not found: '" + title + "'");
    }
}

class Main {
    public static void main(String[] args) {
        Library lib = new Library();
        try {
            lib.findBook("Java Programming");
        } catch (BookNotFoundException e) {
            System.out.println("Specific error: " + e.getMessage());
        } catch (LibraryApplicationException e) {
            System.out.println("General library error: " + e.getMessage());
        }
    }
}
```

### Output

```
Specific error: Book not found: 'Java Programming'
```

### The Exception Hierarchy

```
Exception
└── LibraryApplicationException          ← all library errors
    ├── BookNotFoundException             ← specific: book not found
    └── BookAlreadyCheckedOutException    ← specific: book unavailable
```

> ✅ **Advantage:** Callers can catch a **specific** child exception for fine-grained handling, or catch the **parent** exception as a catch-all for all library errors. This is the same pattern Java uses with `IOException` and `FileNotFoundException`.

---

## Custom Exception with Multiple Constructors

It's a good practice to provide multiple constructors to match the flexibility of Java's built-in exceptions:

```java
class ProductNotFoundException extends Exception {

    // Constructor 1: message only
    ProductNotFoundException(String message) {
        super(message);
    }

    // Constructor 2: message + original cause (for exception chaining)
    ProductNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }

    // Constructor 3: cause only
    ProductNotFoundException(Throwable cause) {
        super(cause);
    }
}
```

### Why Include a `cause` Constructor?

```java
try {
    // database lookup fails with SQLException
} catch (SQLException e) {
    // wrap the low-level exception in a higher-level one
    throw new ProductNotFoundException("Product lookup failed", e);
}
```

This is **exception chaining** — the original `SQLException` is preserved as the "cause" and can be retrieved with `e.getCause()`.

---

## Custom Exceptions in REST APIs

In RESTful applications, custom exceptions are commonly mapped to HTTP status codes:

```java
// HTTP 404 Not Found
class ResourceNotFoundException extends RuntimeException {
    ResourceNotFoundException(String message) {
        super(message);
    }
}

// HTTP 400 Bad Request
class InvalidRequestException extends RuntimeException {
    InvalidRequestException(String message) {
        super(message);
    }
}

// HTTP 403 Forbidden
class UnauthorizedAccessException extends RuntimeException {
    UnauthorizedAccessException(String message) {
        super(message);
    }
}
```

Clients receive specific, meaningful errors like `"Product with ID 42 not found"` instead of a generic `500 Internal Server Error`.

---

## Best Practices for Custom Exceptions

| Practice | Why |
|---|---|
| **End name with `Exception`** | Makes it clear it's an exception class, not a regular class |
| **Extend the right parent** | `Exception` for checked, `RuntimeException` for unchecked |
| **Always call `super(message)`** | Enables `getMessage()` to work correctly |
| **Add extra fields when needed** | Carry contextual data (amount, ID, etc.) beyond just the message |
| **Provide multiple constructors** | Match built-in exception flexibility (`message`, `cause`, both) |
| **Organize into a hierarchy** | Use a parent application exception with specific child exceptions |
| **Document with Javadoc** | Explain when and why the exception is thrown |

---

## Common Mistakes to Avoid

> ⚠️ **Not calling `super(message)`** — if you forget, `getMessage()` will return `null` instead of your custom message.

> ⚠️ **Extending `Throwable` directly** — always extend `Exception` or `RuntimeException`, not `Throwable`. Extending `Throwable` directly is unconventional and breaks the standard exception handling model.

> ⚠️ **Naming without `Exception` suffix** — a class called `BookError` or `BookProblem` won't signal to readers that it's an exception.

> ⚠️ **Making everything a checked exception** — overusing checked exceptions forces callers to write verbose try-catch boilerplate. Use unchecked for programming errors and bugs.

---

## Key Highlights

| Rule | Detail |
|---|---|
| Extend `Exception` | Creates a **checked** custom exception |
| Extend `RuntimeException` | Creates an **unchecked** custom exception |
| Name ends with `Exception` | Naming convention to identify exception classes |
| Call `super(message)` | Enables `getMessage()` to return your custom text |
| Extra fields allowed | Custom exceptions can carry additional context |
| Exception hierarchies | Parent + child custom exceptions for organized error handling |
| Exception chaining | Pass the original `cause` to preserve the full error chain |

---

## Test Yourself

**Question 1:** Will this compile and run correctly?
```java
class MyException extends Exception {
    MyException(String msg) {
        super(msg);
    }
}
class Test {
    public static void main(String[] args) {
        throw new MyException("test error");
    }
}
```
<details>
<summary>Answer</summary>
<strong>No — Compile Error.</strong> <code>MyException</code> extends <code>Exception</code>, making it a checked exception. <code>main</code> must either wrap it in <code>try-catch</code> or declare <code>throws MyException</code>.
</details>

**Question 2:** What does `e.getMessage()` return?
```java
class OrderException extends RuntimeException {
    OrderException(String message) {
        super(message);
    }
}
// somewhere in code:
throw new OrderException("Order #1234 not found");
```
<details>
<summary>Answer</summary>
<strong>"Order #1234 not found"</strong> — The message passed to the constructor is forwarded to <code>RuntimeException</code> via <code>super(message)</code> and stored there. <code>getMessage()</code> retrieves it.
</details>

**Question 3:** Which catch block runs?
```java
class AppException extends Exception {
    AppException(String m) { super(m); }
}
class UserException extends AppException {
    UserException(String m) { super(m); }
}
// ...
try {
    throw new UserException("user not found");
} catch (UserException e) {
    System.out.println("User: " + e.getMessage());
} catch (AppException e) {
    System.out.println("App: " + e.getMessage());
}
```
<details>
<summary>Answer</summary>
<strong>User: user not found</strong> — <code>UserException</code> is the most specific match, so the first <code>catch</code> block runs. The second <code>catch(AppException)</code> is never reached (but is valid since it's the parent).
</details>

---

