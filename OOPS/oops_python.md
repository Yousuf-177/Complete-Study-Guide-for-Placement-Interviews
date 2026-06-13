# 🧩 Object-Oriented Programming (OOP) — Complete Study Guide for Placement Interviews

> **How to use this guide:** This is written for *learning*, not just revision. Every concept is explained with *why it exists*, *how it works internally*, and *how to answer it confidently in an interview*. Work through each section in order — later sections build on earlier ones. Use the checklists and one-liners at the end to self-test.

---

## Table of Contents

1. [What is OOP and Why Does It Exist?](#1-what-is-oop-and-why-does-it-exist)
2. [Classes and Objects](#2-classes-and-objects)
3. [The Four Pillars of OOP](#3-the-four-pillars-of-oop)
4. [Encapsulation — Protecting Your Data](#4-encapsulation--protecting-your-data)
5. [Abstraction — Hiding Complexity](#5-abstraction--hiding-complexity)
6. [Inheritance — Reusing and Extending](#6-inheritance--reusing-and-extending)
7. [Polymorphism — One Interface, Many Forms](#7-polymorphism--one-interface-many-forms)
8. [Constructors and Destructors](#8-constructors-and-destructors)
9. [Access Modifiers](#9-access-modifiers)
10. [Abstract Classes and Interfaces](#10-abstract-classes-and-interfaces)
11. [Static, Final, and Other Keywords](#11-static-final-and-other-keywords)
12. [Method Overloading vs Method Overriding](#12-method-overloading-vs-method-overriding)
13. [The `this` and `super` Keywords](#13-the-this-and-super-keywords)
14. [Object Relationships — Association, Aggregation, Composition](#14-object-relationships--association-aggregation-composition)
15. [Coupling and Cohesion](#15-coupling-and-cohesion)
16. [SOLID Principles](#16-solid-principles)
17. [Design Patterns — Overview](#17-design-patterns--overview)
18. [Exception Handling in OOP](#18-exception-handling-in-oop)
19. [OOP in Java vs C++ vs Python — Key Differences](#19-oop-in-java-vs-c-vs-python--key-differences)
20. [Common Interview Traps](#20-common-interview-traps)
21. [Scenario and Coding Questions](#21-scenario-and-coding-questions)
22. [Must-Know Essentials — Final Checklist](#22-must-know-essentials--final-checklist)

---

## 1. What is OOP and Why Does It Exist?

### The Problem Before OOP

In the early days of programming, code was written **procedurally** — as a list of instructions executed one after another. As programs grew larger, this became a nightmare:
- Data and functions were completely separate
- Changing one function broke others
- Code couldn't be reused without copy-pasting
- Modelling real-world problems felt forced

**Example of the pain:** Imagine writing a banking system procedurally. You'd have global variables for `accountNumber`, `balance`, `ownerName`, and functions like `deposit()`, `withdraw()` scattered everywhere. Any function could accidentally modify any variable. There was no protection, no structure.

### The OOP Solution

**Object-Oriented Programming (OOP)** is a programming paradigm that organizes software around **objects** — self-contained units that bundle **data** (attributes) and **behavior** (methods) together.

The core idea: model software the same way we think about the real world. A `BankAccount` object knows its own balance and knows how to deposit/withdraw. Nobody else should be able to reach in and change the balance directly.

### Four Goals OOP Achieves

| Goal | What It Means |
|---|---|
| **Modularity** | Code is organized into independent, self-contained units (classes) |
| **Reusability** | Write once, use in many places through inheritance and composition |
| **Maintainability** | Change one class without breaking others (loose coupling) |
| **Real-world Modeling** | Software structure mirrors the problem domain naturally |

> 🎯 **Interview answer template:** *"OOP is a paradigm that organizes code around objects — entities that combine state (data) and behavior (methods). It solves problems of procedural programming like uncontrolled data access, lack of reuse, and difficulty scaling — through the four pillars: Encapsulation, Abstraction, Inheritance, and Polymorphism."*

---

## 2. Classes and Objects

### Class — The Blueprint

A **class** is a **template or blueprint** that defines what properties and behaviors an object of that type will have. A class by itself doesn't occupy memory for the data — it's just a definition.

Think of a class as the architectural drawing of a house — it describes what a house looks like, but isn't a house itself.

```java
// Java example
class BankAccount {
    // Attributes (state)
    private String accountNumber;
    private double balance;
    private String ownerName;

    // Methods (behavior)
    public void deposit(double amount) {
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= balance) {
            balance -= amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

### Object — The Instance

An **object** is a **concrete instance** of a class — it occupies actual memory and has actual values for the attributes.

```java
// Creating objects (instances) of BankAccount
BankAccount acc1 = new BankAccount();   // Object 1
BankAccount acc2 = new BankAccount();   // Object 2

// Each object has its own independent state
acc1.deposit(5000);   // acc1.balance = 5000
acc2.deposit(1000);   // acc2.balance = 1000
// acc1 and acc2 are separate objects — they don't interfere
```

### Key Terminology

| Term | Meaning |
|---|---|
| **Class** | Blueprint / template for objects |
| **Object** | Instance of a class; lives in heap memory |
| **Attribute / Field / Member variable** | Data stored in an object |
| **Method / Member function** | Behavior an object can perform |
| **Instantiation** | The process of creating an object from a class (`new` keyword) |
| **Instance** | Another word for object — an instantiated class |
| **Reference** | A variable that holds the memory address of an object |

### Memory: Stack vs Heap

```
Stack Memory                    Heap Memory
────────────                    ───────────
acc1  ───────────────────────→  [BankAccount object: balance=5000, ...]
acc2  ───────────────────────→  [BankAccount object: balance=1000, ...]
```

- **Reference variables** (`acc1`, `acc2`) live on the **stack**
- **Object data** lives on the **heap**
- When you write `acc1 = acc2`, you're copying the *reference*, not the object — both variables now point to the same heap object (**shallow copy**)

> 🎯 **Interview question:** *"What is the difference between a class and an object?"*
> **Answer:** A class is a blueprint that defines structure and behavior. An object is a concrete instance of that blueprint, existing in memory with actual values.

---

## 3. The Four Pillars of OOP

These are the foundation of every OOP interview. You must not only define them but explain *why* each one matters and give examples.

```
┌─────────────────────────────────────────────────────────┐
│               THE FOUR PILLARS OF OOP                   │
│                                                         │
│  1. ENCAPSULATION  →  Bundle data + protect it          │
│  2. ABSTRACTION    →  Hide complexity, show essentials  │
│  3. INHERITANCE    →  Reuse and extend existing code    │
│  4. POLYMORPHISM   →  One interface, many behaviors     │
└─────────────────────────────────────────────────────────┘
```

> 💡 **Mnemonic:** **"A PIE"** — **A**bstraction, **P**olymorphism, **I**nheritance, **E**ncapsulation

Each pillar is covered in depth in the sections below.

---

## 4. Encapsulation — Protecting Your Data

### What It Is

**Encapsulation** is the practice of **bundling data (fields) and methods that operate on that data into a single unit (class)**, and **restricting direct access to the data** from outside.

The key mechanism: **private fields + public getter/setter methods**.

### Why It Matters (The Real Reason)

Without encapsulation:
```java
// BAD — no encapsulation
class BankAccount {
    public double balance;  // anyone can modify this!
}

BankAccount acc = new BankAccount();
acc.balance = -99999;  // ← illegal! But no one stops this.
```

With encapsulation:
```java
// GOOD — encapsulated
class BankAccount {
    private double balance;  // only this class can access directly

    public void deposit(double amount) {
        if (amount > 0) {           // validation!
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;             // read-only access
    }
    // No setBalance() — balance can only change through deposit/withdraw
}
```

### Benefits of Encapsulation

| Benefit | Explanation |
|---|---|
| **Data Validation** | Setters can validate input before changing state |
| **Read-only / Write-only fields** | Expose only getter OR only setter as needed |
| **Flexibility** | Internal implementation can change without affecting external code |
| **Security** | Sensitive data (password, balance) can't be directly modified |
| **Maintainability** | All logic related to a field is in one place |

### Getters and Setters

```java
class Student {
    private String name;
    private int age;

    // Getter — read access
    public String getName() { return name; }

    // Setter — write access with validation
    public void setAge(int age) {
        if (age > 0 && age < 150) {   // validation
            this.age = age;
        } else {
            throw new IllegalArgumentException("Invalid age");
        }
    }
}
```

> 🎯 **Interview one-liner:** *"Encapsulation bundles data and methods into a class and controls access using access modifiers — it's like a capsule that protects its contents."*

### Encapsulation vs Information Hiding

These are related but distinct:
- **Encapsulation** = the mechanism (bundling + access control)
- **Information Hiding** = the principle/goal (hide internal implementation details from outside users)

Encapsulation is how you *achieve* information hiding.

---

## 5. Abstraction — Hiding Complexity

### What It Is

**Abstraction** means **exposing only what is necessary and hiding the underlying complexity**. You interact with an object through a simple interface without needing to know how it works internally.

### Real-World Analogy

Think of driving a car:
- You use the **steering wheel, accelerator, brakes** — the interface
- You don't need to know about fuel injection, engine timing, ABS algorithms
- The complexity is **abstracted away**

### Abstraction in Code

```java
// You use List without caring if it's ArrayList or LinkedList
List<String> names = new ArrayList<>();
names.add("Alice");
names.get(0);
// You don't know (or care) how add() or get() work internally

// Similarly with a payment system:
interface PaymentGateway {
    boolean processPayment(double amount);
}

// You call processPayment() — you don't care if it's Stripe, PayPal, or Razorpay internally
```

### How Abstraction is Achieved in Java

| Mechanism | How it helps abstraction |
|---|---|
| **Abstract classes** | Define a partial implementation; subclasses fill in specifics |
| **Interfaces** | Define a contract (what) without any implementation (how) |
| **Access modifiers** | Hide private implementation details |
| **Packages** | Organize and hide internal classes |

### Encapsulation vs Abstraction — The Crucial Distinction

This is the #1 most confused OOP pair in interviews.

| Aspect | Encapsulation | Abstraction |
|---|---|---|
| **Focus** | *How* to protect data | *What* to expose to the user |
| **Goal** | Data protection and controlled access | Simplification and hiding complexity |
| **Mechanism** | Private fields + getters/setters | Abstract classes, interfaces |
| **Level** | Implementation level | Design level |
| **Analogy** | A capsule protecting medicine | A TV remote hiding circuit complexity |
| **Example** | `private int balance` with `getBalance()` | `interface Shape { double area(); }` |

> 💡 **Think of it this way:**
> - **Encapsulation:** "You can't touch my data directly."
> - **Abstraction:** "You don't need to know how I work — just use me."

> 🎯 **Interview answer:** *"Encapsulation hides data using access modifiers; abstraction hides complexity using abstract classes and interfaces. Encapsulation is at the object level; abstraction is at the design level."*

---

## 6. Inheritance — Reusing and Extending

### What It Is

**Inheritance** allows a new class (**child/subclass/derived class**) to **acquire the properties and behaviors** of an existing class (**parent/superclass/base class**). The child class gets everything the parent has and can add its own features or modify inherited behavior.

### Why It Exists

Without inheritance: if you have `Dog`, `Cat`, and `Bird` classes, you'd write `eat()`, `sleep()`, `name`, `age` in all three. That's **code duplication**. With inheritance, you write those once in an `Animal` class.

```
              Animal
             /  |  \
           Dog Cat  Bird
```

```java
// Parent class
class Animal {
    String name;
    int age;

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }
}

// Child class — inherits everything from Animal, adds its own
class Dog extends Animal {
    String breed;

    public void bark() {
        System.out.println(name + " says: Woof!");
    }

    // Can also OVERRIDE eat() to customize behavior
    @Override
    public void eat() {
        System.out.println(name + " is eating dog food.");
    }
}

// Usage
Dog d = new Dog();
d.name = "Bruno";  // inherited from Animal
d.eat();           // uses Dog's overridden eat()
d.bark();          // Dog's own method
d.sleep();         // inherited from Animal
```

### Types of Inheritance

| Type | Structure | Support |
|---|---|---|
| **Single** | A → B (one parent, one child) | Java ✅, C++ ✅, Python ✅ |
| **Multilevel** | A → B → C (chain) | Java ✅, C++ ✅, Python ✅ |
| **Hierarchical** | A → B, A → C (one parent, many children) | Java ✅, C++ ✅, Python ✅ |
| **Multiple** | A + B → C (two parents, one child) | Java ❌ (via interfaces ✅), C++ ✅, Python ✅ |
| **Hybrid** | Combination of above | Java ❌ (partially via interfaces), C++ ✅, Python ✅ |

> 🎯 **Why Java doesn't support multiple inheritance with classes:**
> The **Diamond Problem**. If class C inherits from both A and B, and both A and B have the same method `display()`, which one does C use? Java avoids ambiguity by disallowing multiple class inheritance. Interfaces solve this because they don't provide implementation (until default methods in Java 8+).

### The Diamond Problem (Visual)

```
        A
       / \
      B   C        Both B and C override method foo()
       \ /
        D          ← Which foo() does D inherit? Ambiguous!
```

### IS-A Relationship

Inheritance models an **IS-A** relationship:
- `Dog` IS-A `Animal` ✅
- `Car` IS-A `Vehicle` ✅
- `Manager` IS-A `Employee` ✅
- `Engine` IS-A `Car` ❌ (Engine is part of Car, not a type of Car → use Composition)

> If you can say "X IS-A Y" naturally, inheritance is appropriate. If you say "X HAS-A Y", use composition.

### Key Concepts in Inheritance

**Method Overriding:** Child class redefines a method from the parent (same name, same signature). The child's version replaces the parent's for child objects. (Covered in depth in Section 12.)

**`super` keyword:** Used in child class to call parent class methods or constructors.

**Inheritance Chain (Method Resolution Order):** When a method is called, Java searches from the most-specific class upward until it finds the method.

### When NOT to Use Inheritance

Inheritance is often **overused**. Use it only when:
- There's a true IS-A relationship
- The child genuinely is a more specific version of the parent

**Prefer composition over inheritance** when:
- You want to reuse code but the IS-A relationship is forced
- The parent class might change in ways that break children
- You need to combine behaviors from multiple sources

---

## 7. Polymorphism — One Interface, Many Forms

### What It Is

**Polymorphism** (from Greek: *poly* = many, *morph* = forms) means the ability of **different objects to respond to the same method call in different ways**.

### Real-World Analogy

The `+` operator:
- `3 + 4` → adds numbers (returns 7)
- `"Hello" + " World"` → concatenates strings (returns "Hello World")

Same operator (`+`), different behavior based on context. That's polymorphism.

### Two Types of Polymorphism

```
Polymorphism
├── Compile-time (Static) Polymorphism  → Method Overloading
└── Run-time (Dynamic) Polymorphism     → Method Overriding
```

---

### Compile-Time Polymorphism (Method Overloading)

The compiler decides **which method to call** at compile time, based on the method signature (number and types of parameters).

```java
class Calculator {
    // Same method name, different parameters
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

Calculator calc = new Calculator();
calc.add(2, 3);        // calls add(int, int) → 5
calc.add(2.5, 3.5);   // calls add(double, double) → 6.0
calc.add(1, 2, 3);    // calls add(int, int, int) → 6
```

**Why is it compile-time?** The compiler sees the argument types at compile time and binds the call to the correct method before the program runs. This is also called **static binding** or **early binding**.

---

### Run-Time Polymorphism (Method Overriding)

The JVM decides **which method to call** at run time, based on the actual type of the object (not the reference type). This is the more powerful and interesting form.

```java
class Shape {
    public double area() {
        return 0;  // default
    }
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    double width, height;
    Rectangle(double w, double h) { width = w; height = h; }

    @Override
    public double area() {
        return width * height;
    }
}

// The magic of runtime polymorphism:
Shape s1 = new Circle(5);       // Shape reference, Circle object
Shape s2 = new Rectangle(4, 6); // Shape reference, Rectangle object

s1.area();  // calls Circle's area() → 78.54  ← JVM decides at RUNTIME
s2.area();  // calls Rectangle's area() → 24.0 ← JVM decides at RUNTIME
```

**Why is it run-time?** The reference type (`Shape`) is known at compile time, but the actual object type (`Circle` or `Rectangle`) is only known when the program runs. The JVM uses a **virtual method table (vtable)** to dispatch the correct method. This is called **dynamic binding** or **late binding**.

### The Power: Polymorphic Arrays and Collections

```java
// Process many different shapes with ONE loop
Shape[] shapes = {
    new Circle(5),
    new Rectangle(4, 6),
    new Triangle(3, 4, 5)
};

for (Shape s : shapes) {
    System.out.println("Area: " + s.area());
    // Each object calls its OWN area() — polymorphism in action!
}
```

Without polymorphism, you'd need `if (instanceof Circle)`, `if (instanceof Rectangle)` — messy, not extensible.

### How Dynamic Dispatch Works (vtable)

Every class has a **virtual method table (vtable)** — a lookup table mapping method names to their implementations.

```
Circle's vtable:        Rectangle's vtable:
area → Circle.area()   area → Rectangle.area()
```

When `s1.area()` is called on a `Shape` reference pointing to a `Circle`, the JVM looks up Circle's vtable → finds `Circle.area()` → calls it.

### Compile-Time vs Run-Time Polymorphism Summary

| Feature | Compile-Time (Overloading) | Run-Time (Overriding) |
|---|---|---|
| **How** | Same method name, different parameters | Same method name, same parameters, different class |
| **Binding** | Static (early) binding | Dynamic (late) binding |
| **Decided at** | Compile time | Runtime |
| **Requires** | Same class | Inheritance |
| **Return type** | Can differ (if params differ) | Must be same (or covariant) |
| **Performance** | Slightly faster | Slight overhead (vtable lookup) |
| **Also called** | Static polymorphism, ad-hoc polymorphism | Dynamic polymorphism, subtype polymorphism |

---

## 8. Constructors and Destructors

### Constructor — Setting Up an Object

A **constructor** is a special method that is **automatically called when an object is created**. Its job is to **initialize the object's state**.

**Rules:**
- Same name as the class
- No return type (not even `void`)
- Called automatically with `new`
- Can be overloaded

```java
class Student {
    String name;
    int age;

    // Default constructor (no parameters)
    public Student() {
        name = "Unknown";
        age = 0;
    }

    // Parameterized constructor
    public Student(String name, int age) {
        this.name = name;  // 'this' distinguishes field from parameter
        this.age = age;
    }
}

Student s1 = new Student();             // calls default constructor
Student s2 = new Student("Alice", 20);  // calls parameterized constructor
```

### Types of Constructors

| Type | Description |
|---|---|
| **Default Constructor** | No parameters. Java provides one automatically if you write none. |
| **Parameterized Constructor** | Takes arguments to initialize with specific values. |
| **Copy Constructor** | Takes an object of the same class and creates a copy (C++ has this natively; Java simulates it). |

> ⚠️ **Important:** If you write *any* constructor, Java **stops providing** the default constructor. You must explicitly write it if you need it.

### Constructor Chaining

Calling one constructor from another using `this()` or `super()`:

```java
class Employee {
    String name;
    int id;
    String department;

    Employee(String name, int id) {
        this(name, id, "General");   // calls the 3-arg constructor
    }

    Employee(String name, int id, String department) {
        this.name = name;
        this.id = id;
        this.department = department;
    }
}
```

**Rules for `this()` and `super()`:**
- Must be the **first statement** in a constructor
- Cannot use both `this()` and `super()` in the same constructor
- `super()` is automatically added by Java if you don't write it (calls parent's default constructor)

### Copy Constructor

```java
class Point {
    int x, y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // Copy constructor
    Point(Point other) {
        this.x = other.x;
        this.y = other.y;
    }
}

Point p1 = new Point(3, 4);
Point p2 = new Point(p1);   // p2 is an independent copy of p1
p2.x = 10;                  // doesn't affect p1.x
```

### Shallow Copy vs Deep Copy

| Type | What is copied | Shared references? |
|---|---|---|
| **Shallow Copy** | Field values; object references are copied (both point to same heap object) | ✅ Yes — dangerous |
| **Deep Copy** | Field values; new objects created for all referenced objects | ❌ No — fully independent |

```java
// Shallow copy — BAD for mutable objects
Point p1 = new Point(3, 4);
Point p2 = p1;            // p2 and p1 point to the SAME object
p2.x = 10;                // This ALSO changes p1.x!

// Deep copy — GOOD
Point p3 = new Point(p1); // copy constructor creates new object
p3.x = 99;                // p1 is unaffected
```

### Destructors

A **destructor** is called when an object is about to be destroyed, to release resources.

| Language | Mechanism |
|---|---|
| **C++** | Explicit destructor `~ClassName()` — called deterministically when object goes out of scope |
| **Java** | No explicit destructor. JVM has **garbage collector** (GC). `finalize()` method is deprecated and unreliable. Use `try-with-resources` or `AutoCloseable`. |
| **Python** | `__del__()` method, but GC is non-deterministic |

```java
// Java — proper resource cleanup
class FileHandler implements AutoCloseable {
    @Override
    public void close() {
        // cleanup code — called when try-with-resources block exits
    }
}

try (FileHandler fh = new FileHandler()) {
    // use fh
} // fh.close() called automatically here
```

---

## 9. Access Modifiers

### Why They Matter

Access modifiers control **who can access a class, field, method, or constructor**. They are the enforcement mechanism for encapsulation.

### Java Access Modifiers

| Modifier | Same Class | Same Package | Subclass (diff pkg) | Everywhere |
|---|---|---|---|---|
| **`private`** | ✅ | ❌ | ❌ | ❌ |
| **`default`** (no keyword) | ✅ | ✅ | ❌ | ❌ |
| **`protected`** | ✅ | ✅ | ✅ | ❌ |
| **`public`** | ✅ | ✅ | ✅ | ✅ |

> 💡 **Rule of thumb:** Use the **most restrictive** access level that still works. Start with `private`; only relax if necessary.

### Best Practices

```java
class BankAccount {
    private double balance;        // private — only this class touches it
    protected String accountType;  // protected — subclasses may need it
    public String bankName;        // public — ok for everyone to see (generally avoid)

    private void calculateInterest() { ... }  // internal helper — private
    public void deposit(double amt) { ... }   // external interface — public
}
```

### C++ Access Modifiers

C++ has the same three keywords: `private`, `protected`, `public`.

Additional C++ concept: **`friend`** — a class or function declared as `friend` of a class gets access to its private and protected members. Use sparingly.

```cpp
class Box {
    private:
        double width;
    friend class BoxPrinter;  // BoxPrinter can access private width
};
```

### Python's Approach

Python doesn't enforce access modifiers — it's convention-based:
- `name` — public (accessible everywhere)
- `_name` — "protected" by convention (single underscore — please don't use this directly)
- `__name` — "private" by convention (double underscore — triggers name mangling: `_ClassName__name`)

```python
class Student:
    def __init__(self):
        self.name = "Alice"     # public
        self._grade = "A"       # protected by convention
        self.__id = 12345       # name-mangled to _Student__id
```

---

## 10. Abstract Classes and Interfaces

### Abstract Class

An **abstract class** is a class that **cannot be instantiated** and may contain **abstract methods** (methods with no body). Subclasses must implement all abstract methods.

**When to use:** When you want to provide a partial implementation and force subclasses to fill in the rest.

```java
abstract class Shape {
    String color;  // concrete field

    // Abstract method — no body, must be overridden
    public abstract double area();

    // Concrete method — has body, inherited as-is
    public void printColor() {
        System.out.println("Color: " + color);
    }
}

class Circle extends Shape {
    double radius;

    @Override
    public double area() {             // MUST implement
        return Math.PI * radius * radius;
    }
}

// Shape s = new Shape();  // ❌ ERROR — can't instantiate abstract class
Shape s = new Circle();    // ✅ OK — reference is Shape, object is Circle
```

### Interface

An **interface** is a **pure contract** — it defines what a class must do (method signatures) without saying how. It's like a legally binding agreement: "If you implement this interface, you MUST provide these methods."

```java
interface Flyable {
    void fly();                    // implicitly public abstract
    int MAX_HEIGHT = 10000;        // implicitly public static final
}

interface Swimmable {
    void swim();
}

// A class can implement MULTIPLE interfaces (Java's answer to multiple inheritance)
class Duck extends Bird implements Flyable, Swimmable {
    @Override
    public void fly() { System.out.println("Duck flying"); }

    @Override
    public void swim() { System.out.println("Duck swimming"); }
}
```

### Java 8+ Interface Features

Java 8 added **default methods** and **static methods** to interfaces, blurring the line somewhat:

```java
interface Printable {
    void print();  // abstract — must implement

    default void printWithBorder() {   // default — CAN override, not required
        System.out.println("---");
        print();
        System.out.println("---");
    }

    static void version() {            // static — belongs to interface, not instance
        System.out.println("v1.0");
    }
}
```

### Abstract Class vs Interface — The Key Comparison

| Feature | Abstract Class | Interface |
|---|---|---|
| **Instantiation** | ❌ Cannot instantiate | ❌ Cannot instantiate |
| **Methods** | Can have abstract + concrete | Abstract by default; default + static in Java 8+ |
| **Fields** | Can have instance fields | Only `public static final` constants |
| **Constructors** | ✅ Yes | ❌ No |
| **Access modifiers** | Any | Methods implicitly `public` |
| **Inheritance** | `extends` (single parent) | `implements` (multiple interfaces) |
| **Multiple inheritance** | ❌ No (one class only) | ✅ Yes (many interfaces) |
| **Use when** | Shared code + IS-A relationship | Pure contract / capability |
| **Speed** | Slightly faster | Slight overhead |

### When to Choose Which?

**Use Abstract Class when:**
- Classes share common code (avoid duplication)
- You want to provide a default implementation for some methods
- Subclasses are closely related (strong IS-A relationship)

**Use Interface when:**
- You want to define a capability that unrelated classes can share
- You need multiple inheritance
- You're defining an API contract
- `Comparable`, `Serializable`, `Runnable` — all interfaces

> 🎯 **Classic interview example:**
> - `Animal` is an abstract class (animals share `eat()`, `sleep()`)
> - `Flyable`, `Swimmable` are interfaces (capabilities that any unrelated class might have)
> - A `Duck` extends `Animal` and implements `Flyable, Swimmable`

---

## 11. Static, Final, and Other Keywords

### `static` Keyword

`static` means the member **belongs to the class** rather than to any specific object. There's only one copy, shared by all instances.

```java
class Counter {
    static int count = 0;    // ONE shared variable for all objects
    int id;

    Counter() {
        count++;             // increments the shared counter
        id = count;
    }
}

Counter c1 = new Counter();  // count = 1, c1.id = 1
Counter c2 = new Counter();  // count = 2, c2.id = 2
Counter c3 = new Counter();  // count = 3, c3.id = 3

System.out.println(Counter.count);  // 3 — accessed via class, not object
```

**Static members:**
- `static` field — shared variable (e.g., count of instances)
- `static` method — utility method, no `this` reference, can't access instance fields
- `static` block — runs once when class is first loaded, used for static initialization
- `static` nested class — doesn't need outer class instance to exist

> ⚠️ `static` methods cannot call non-static methods or access non-static fields directly (they don't have `this`).

### `final` Keyword

`final` means "cannot be changed/extended/overridden."

| Applied to | Effect |
|---|---|
| **`final` variable** | Becomes a constant — must be initialized once, cannot be reassigned |
| **`final` method** | Cannot be overridden in subclasses |
| **`final` class** | Cannot be subclassed (e.g., `String`, `Integer` in Java) |

```java
final int MAX = 100;       // constant — can't change MAX
MAX = 200;                 // ❌ COMPILE ERROR

final class ImmutablePoint {  // can't extend this
    final int x, y;           // x and y can't change after construction
    ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
}
```

> **`final` ≠ immutable for objects.** A `final` reference can't be reassigned, but the object it points to can still be mutated.
> ```java
> final List<String> list = new ArrayList<>();
> list.add("Hello");   // ✅ OK — modifying the object
> list = new ArrayList<>();  // ❌ ERROR — can't reassign the reference
> ```

### `static final` — Constants

```java
public static final double PI = 3.14159;  // class-level constant
public static final int MAX_SIZE = 1000;
```

Convention: UPPER_SNAKE_CASE for constants.

### `this` Keyword (Preview — detailed in Section 13)

- Refers to the **current object**
- Distinguishes instance fields from parameters with the same name
- Used for constructor chaining: `this(args)`

### `super` Keyword (Preview — detailed in Section 13)

- Refers to the **parent class**
- `super.method()` calls parent's method
- `super()` calls parent's constructor

### `instanceof` Operator

Tests if an object is an instance of a class or interface:

```java
Animal a = new Dog();
System.out.println(a instanceof Animal);  // true
System.out.println(a instanceof Dog);     // true
System.out.println(a instanceof Cat);     // false
```

---

## 12. Method Overloading vs Method Overriding

This is one of the most commonly tested OOP topics. Know the differences cold.

### Method Overloading

**Definition:** Multiple methods in the **same class** with the **same name** but **different parameter lists** (different number, type, or order of parameters).

```java
class MathUtils {
    public int square(int n)       { return n * n; }
    public double square(double n) { return n * n; }
    public long square(long n)     { return n * n; }
}
```

**Rules:**
- Must differ in parameter list (number, type, or order)
- Return type alone is NOT enough to distinguish
- Access modifier can differ
- Can be in the same class or a subclass

```java
// ❌ NOT overloading — only return type differs (compile error)
public int getValue() { return 1; }
public double getValue() { return 1.0; }

// ✅ IS overloading
public void print(int x) { }
public void print(int x, int y) { }
public void print(String s) { }
```

### Method Overriding

**Definition:** A subclass provides a **specific implementation** of a method that is **already defined in its parent class** — same name, same return type, same parameter list.

```java
class Vehicle {
    public void start() {
        System.out.println("Vehicle starting with key...");
    }
}

class ElectricCar extends Vehicle {
    @Override
    public void start() {   // same name, same params, same return type
        System.out.println("Electric car starting silently...");
    }
}

Vehicle v = new ElectricCar();
v.start();  // prints "Electric car starting silently..." — overriding!
```

**Rules for Overriding:**
1. Method signature must be **identical** (name + parameters)
2. Return type must be the **same or covariant** (subtype)
3. Access modifier must be **same or more permissive** (can't narrow access)
4. Cannot override `static`, `final`, or `private` methods
5. `@Override` annotation is optional but recommended (catches errors)
6. Overriding method can throw **fewer or narrower checked exceptions**

### Covariant Return Type

A subclass can override a method and return a more specific type:

```java
class Animal {
    public Animal create() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    public Dog create() { return new Dog(); }  // Dog is a subtype of Animal ✅
}
```

### Complete Comparison Table

| Feature | Overloading | Overriding |
|---|---|---|
| **Definition** | Same name, different params in same class | Same name, same params in child class |
| **Polymorphism type** | Compile-time (static) | Run-time (dynamic) |
| **Binding** | Early (compile time) | Late (runtime) |
| **Class requirement** | Same class | Parent + Child class (inheritance) |
| **Parameters** | Must differ | Must be identical |
| **Return type** | Can differ | Must be same or covariant |
| **Access modifier** | Can differ | Same or more permissive |
| **`static` methods** | Can be overloaded | Cannot be overridden (can be hidden) |
| **`private` methods** | Can be overloaded | Cannot be overridden |
| **`final` methods** | Can be overloaded | Cannot be overridden |
| **Annotations** | None required | `@Override` recommended |

### Method Hiding (static methods)

Static methods cannot be overridden — they are **hidden** instead:

```java
class Parent {
    public static void display() { System.out.println("Parent static"); }
}

class Child extends Parent {
    public static void display() { System.out.println("Child static"); }
    // This is METHOD HIDING, not overriding
}

Parent p = new Child();
p.display();  // prints "Parent static" — static binding, not dynamic!
```

---

## 13. The `this` and `super` Keywords

### `this` — Referring to Current Object

`this` is a reference to the **current object** — the object on which a method is being invoked.

**Five uses of `this`:**

```java
class Employee {
    String name;
    int salary;
    static int count = 0;

    // Use 1: Disambiguate field from parameter
    Employee(String name, int salary) {
        this.name = name;       // this.name = field; name = parameter
        this.salary = salary;
    }

    // Use 2: Call another constructor in same class (constructor chaining)
    Employee(String name) {
        this(name, 50000);      // must be FIRST statement
    }

    // Use 3: Pass current object as argument
    void register(EmployeeDB db) {
        db.add(this);           // pass this object to another method
    }

    // Use 4: Return current object (builder pattern)
    Employee setSalary(int salary) {
        this.salary = salary;
        return this;            // enables method chaining
    }

    // Use 5: Can't use this in static context (no current object)
    static void staticMethod() {
        // this.name = "X";  // ❌ ERROR — static has no 'this'
    }
}

// Method chaining using return this:
Employee e = new Employee("Alice")
    .setSalary(70000);
```

### `super` — Referring to Parent Class

`super` is a reference to the **parent class** of the current object.

**Three uses of `super`:**

```java
class Animal {
    String name;
    int age;

    Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    void describe() {
        System.out.println("Animal: " + name + ", Age: " + age);
    }
}

class Dog extends Animal {
    String breed;

    // Use 1: Call parent constructor
    Dog(String name, int age, String breed) {
        super(name, age);        // MUST be FIRST statement
        this.breed = breed;
    }

    // Use 2: Call parent method (when you've overridden it)
    @Override
    void describe() {
        super.describe();        // calls Animal's describe()
        System.out.println("Breed: " + breed);  // adds Dog-specific info
    }

    // Use 3: Access parent field (if same-named field exists in child)
    void printNames() {
        System.out.println(super.name);  // parent's name field
        // (rare — avoid hiding fields in child)
    }
}
```

### `this()` vs `super()` Rules

| Rule | `this()` | `super()` |
|---|---|---|
| **Purpose** | Call another constructor in same class | Call parent class constructor |
| **Position** | Must be first statement in constructor | Must be first statement in constructor |
| **Together?** | ❌ Cannot use both in same constructor | ❌ Cannot use both in same constructor |
| **Auto-inserted?** | ❌ No | ✅ Yes — Java inserts `super()` if you don't write any super/this call |

---

## 14. Object Relationships — Association, Aggregation, Composition

Beyond inheritance, objects relate to each other in three key ways. These describe **HAS-A** relationships.

### Association

**Definition:** A **general relationship** between two classes where they are aware of each other and can interact, but neither owns the other. Both can exist independently.

```java
class Teacher {
    String name;
    void teach(Student s) {
        System.out.println(name + " teaches " + s.name);
    }
}

class Student {
    String name;
}

// Teacher and Student can exist independently
// A teacher can teach many students; a student can have many teachers
Teacher t = new Teacher();
Student s = new Student();
t.teach(s);  // association — they interact but don't own each other
```

**Real-world examples:** Teacher-Student, Doctor-Patient, Driver-Car (if car is rented)

---

### Aggregation — Weak HAS-A

**Definition:** A **HAS-A relationship** where the child can **exist independently** of the parent. If the parent is destroyed, the child lives on.

```java
class Department {
    String name;
    List<Employee> employees;  // Department HAS-A list of Employees

    Department(String name, List<Employee> employees) {
        this.employees = employees;  // employees passed in — they pre-exist
    }
}

class Employee {
    String name;
    // Employee can exist without a Department
}

List<Employee> emp = new ArrayList<>();
emp.add(new Employee("Alice"));
emp.add(new Employee("Bob"));

Department dept = new Department("Engineering", emp);
// If dept is destroyed, employees still exist
```

**Key signal:** The contained object is **passed in** (not created inside) — it was created elsewhere and can continue after the container dies.

---

### Composition — Strong HAS-A

**Definition:** A **HAS-A relationship** where the child **cannot exist independently**. The child is created by and destroyed with the parent.

```java
class Heart {
    void beat() { System.out.println("Beating..."); }
}

class Human {
    private Heart heart;    // Human HAS-A Heart

    Human() {
        heart = new Heart();  // Heart CREATED inside Human
    }
    // When Human object is destroyed, Heart object is destroyed too
}
```

**Key signal:** The contained object is **created inside** the container's constructor — it has no existence outside.

### Comparison: Association vs Aggregation vs Composition

| Feature | Association | Aggregation | Composition |
|---|---|---|---|
| **Relationship type** | "uses" or "knows" | Weak HAS-A | Strong HAS-A |
| **Ownership** | No ownership | Partial ownership | Full ownership |
| **Lifecycle dependency** | Independent | Independent | Dependent |
| **UML** | Plain arrow | Hollow diamond | Filled diamond |
| **Object creation** | Separate | Separate, passed in | Created inside parent |
| **Example** | Teacher-Student | Department-Employee | Human-Heart |

### Composition Over Inheritance

> *"Favor composition over inheritance"* — Design Patterns (GoF)

**Why?** Inheritance creates tight coupling. Changes to the parent class can break all subclasses. Composition is more flexible — you can swap components.

```java
// Inheritance approach — tightly coupled
class Car extends Engine { }   // Car IS-A Engine? No, that's wrong!

// Composition approach — loosely coupled
class Car {
    private Engine engine;      // Car HAS-A Engine ✅

    Car(Engine e) {
        this.engine = e;
    }
}
// Can swap ElectricEngine, GasEngine, HybridEngine without changing Car
```

---

## 15. Coupling and Cohesion

### Cohesion — How focused is a class?

**Cohesion** measures how closely related the responsibilities of a class or module are.

- **High cohesion** (good): A class has a single, well-defined responsibility. All its methods work toward one purpose.
- **Low cohesion** (bad): A class has many unrelated responsibilities — it does too many things.

```java
// ❌ Low cohesion — UserManager does too many things
class UserManager {
    void createUser() { }
    void deleteUser() { }
    void sendEmail() { }         // unrelated to user management
    void generateReport() { }   // unrelated to user management
    void connectToDatabase() { } // unrelated to user management
}

// ✅ High cohesion — each class has one focus
class UserRepository  { void create() {} void delete() {} }
class EmailService    { void sendEmail() {} }
class ReportGenerator { void generate() {} }
```

### Coupling — How dependent are classes?

**Coupling** measures how much one class depends on another.

- **Loose coupling** (good): Classes interact through interfaces/abstractions; changes in one don't break others.
- **Tight coupling** (bad): Classes depend on each other's internal details; changes ripple everywhere.

```java
// ❌ Tight coupling — OrderProcessor directly creates MySQLDatabase
class OrderProcessor {
    MySQLDatabase db = new MySQLDatabase();  // hard-coded dependency
    void processOrder() { db.save(order); }
}
// If you switch to PostgreSQL, you must change OrderProcessor!

// ✅ Loose coupling — depend on abstraction (interface)
interface Database { void save(Object o); }

class OrderProcessor {
    Database db;                    // depends on interface, not concrete class
    OrderProcessor(Database db) { this.db = db; }
    void processOrder() { db.save(order); }
}
// Inject MySQLDatabase or PostgreSQLDatabase — OrderProcessor doesn't care!
```

### The Goal

> **High Cohesion + Loose Coupling = Well-designed OOP system**

---

## 16. SOLID Principles

SOLID is an acronym for five design principles that make OOP code easier to maintain, extend, and understand. These appear frequently in senior-level placement interviews.

```
S — Single Responsibility Principle
O — Open/Closed Principle
L — Liskov Substitution Principle
I — Interface Segregation Principle
D — Dependency Inversion Principle
```

---

### S — Single Responsibility Principle (SRP)

> *"A class should have only one reason to change."*

Every class should have **exactly one responsibility**. If a class has two reasons to change, it has two responsibilities — split it.

```java
// ❌ Violates SRP — Invoice does too much
class Invoice {
    void calculateTotal() { }
    void printInvoice() { }    // printing responsibility
    void saveToDB() { }        // persistence responsibility
}

// ✅ Follows SRP
class Invoice        { void calculateTotal() { } }
class InvoicePrinter { void print(Invoice i) { } }
class InvoiceRepo    { void save(Invoice i)  { } }
```

---

### O — Open/Closed Principle (OCP)

> *"Software entities should be open for extension but closed for modification."*

You should be able to add new functionality by **adding new code** (extending), not by **changing existing code** (modifying). This protects existing, tested code.

```java
// ❌ Violates OCP — every new shape requires modifying AreaCalculator
class AreaCalculator {
    double calculate(Object shape) {
        if (shape instanceof Circle) { ... }
        else if (shape instanceof Rectangle) { ... }
        // Must add new if-else for every new shape
    }
}

// ✅ Follows OCP — add new shapes without touching AreaCalculator
interface Shape { double area(); }
class Circle    implements Shape { public double area() { return Math.PI * r * r; } }
class Rectangle implements Shape { public double area() { return w * h; } }
class Triangle  implements Shape { public double area() { ... } } // just add this — no other changes

class AreaCalculator {
    double calculate(Shape shape) { return shape.area(); }  // works for any Shape
}
```

---

### L — Liskov Substitution Principle (LSP)

> *"Objects of a subclass should be replaceable with objects of the parent class without breaking the program."*

If `Dog` is a subclass of `Animal`, you should be able to use a `Dog` wherever an `Animal` is expected and everything should still work correctly.

```java
// ❌ Violates LSP — classic Square-Rectangle problem
class Rectangle {
    int width, height;
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
    int area() { return width * height; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w)  { width = height = w; }  // must keep square!
    @Override
    void setHeight(int h) { width = height = h; }  // must keep square!
}

// This breaks with Square:
Rectangle r = new Square();
r.setWidth(4);
r.setHeight(5);
System.out.println(r.area());  // Expected 20, gets 25! LSP violated.

// ✅ Fix: Square and Rectangle should NOT have inheritance relationship
//    Use a common Shape interface instead
```

---

### I — Interface Segregation Principle (ISP)

> *"Clients should not be forced to implement interfaces they do not use."*

Split large interfaces into smaller, more specific ones. A class should only implement methods that are relevant to it.

```java
// ❌ Violates ISP — fat interface forces irrelevant implementations
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work() { ... }    // ✅ relevant
    public void eat() { /* ??? */ }  // ❌ Robots don't eat!
    public void sleep() { /* ??? */ } // ❌ Robots don't sleep!
}

// ✅ Follows ISP — segregated interfaces
interface Workable { void work(); }
interface Eatable  { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable { ... }
class Robot implements Workable { ... }  // only what's relevant
```

---

### D — Dependency Inversion Principle (DIP)

> *"High-level modules should not depend on low-level modules. Both should depend on abstractions."*

Don't depend on concrete implementations — depend on interfaces/abstractions. This enables loose coupling and easy swapping of implementations.

```java
// ❌ Violates DIP — high-level depends on low-level concrete class
class NotificationService {
    EmailSender emailSender = new EmailSender();  // tightly coupled
    void notify(String msg) { emailSender.send(msg); }
}

// ✅ Follows DIP — both depend on abstraction
interface MessageSender { void send(String message); }
class EmailSender implements MessageSender { public void send(String msg) { ... } }
class SMSSender   implements MessageSender { public void send(String msg) { ... } }

class NotificationService {
    MessageSender sender;
    NotificationService(MessageSender sender) { this.sender = sender; }
    void notify(String msg) { sender.send(msg); }
}

// Inject any implementation at runtime — email, SMS, push notification
NotificationService ns = new NotificationService(new SMSSender());
```

---

## 17. Design Patterns — Overview

Design patterns are **reusable solutions to commonly occurring design problems**. They're not code — they're templates. Knowing even 5-6 patterns deeply will set you apart in interviews.

### Three Categories

```
Design Patterns
├── Creational  → How objects are created
├── Structural  → How objects are composed/structured
└── Behavioral  → How objects communicate
```

---

### Creational Patterns

#### Singleton Pattern

**Problem:** You need exactly one instance of a class (e.g., database connection pool, logger, config manager).

```java
class DatabaseConnection {
    private static DatabaseConnection instance;  // the single instance

    private DatabaseConnection() { }  // private constructor — nobody can call new

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();  // create only once
        }
        return instance;
    }
}

// Usage
DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
db1 == db2;  // true — same object!
```

> ⚠️ **Thread-safe Singleton** — The above has a race condition. In multithreaded environments, use `synchronized` or the "Bill Pugh" holder pattern.

#### Factory Pattern

**Problem:** You want to create objects without exposing creation logic to the client.

```java
interface Animal { void speak(); }
class Dog implements Animal { public void speak() { System.out.println("Woof"); } }
class Cat implements Animal { public void speak() { System.out.println("Meow"); } }

class AnimalFactory {
    public static Animal create(String type) {
        switch (type) {
            case "dog": return new Dog();
            case "cat": return new Cat();
            default: throw new IllegalArgumentException("Unknown animal: " + type);
        }
    }
}

Animal a = AnimalFactory.create("dog");  // client doesn't know Dog class details
a.speak();
```

#### Builder Pattern

**Problem:** Object construction is complex with many optional parameters.

```java
class Pizza {
    String size, crust, sauce;
    List<String> toppings;

    private Pizza(Builder b) {
        this.size = b.size;
        this.crust = b.crust;
        this.sauce = b.sauce;
        this.toppings = b.toppings;
    }

    static class Builder {
        String size, crust = "thin", sauce = "tomato";
        List<String> toppings = new ArrayList<>();

        Builder size(String s) { this.size = s; return this; }
        Builder crust(String c) { this.crust = c; return this; }
        Builder topping(String t) { toppings.add(t); return this; }
        Pizza build() { return new Pizza(this); }
    }
}

Pizza p = new Pizza.Builder()
    .size("large")
    .crust("thick")
    .topping("cheese")
    .topping("pepperoni")
    .build();
```

---

### Structural Patterns

#### Adapter Pattern

**Problem:** Two incompatible interfaces need to work together.

```java
// Legacy interface you can't change
interface OldPrinter { void printText(String text); }

// New interface you want to use
interface NewPrinter { void print(String text, int copies); }

// Adapter makes OldPrinter usable where NewPrinter is expected
class PrinterAdapter implements NewPrinter {
    OldPrinter old;
    PrinterAdapter(OldPrinter old) { this.old = old; }

    @Override
    public void print(String text, int copies) {
        for (int i = 0; i < copies; i++) {
            old.printText(text);  // delegates to old interface
        }
    }
}
```

#### Decorator Pattern

**Problem:** Add behavior to individual objects without changing their class.

```java
interface Coffee { double cost(); }
class SimpleCoffee implements Coffee { public double cost() { return 1.0; } }

class MilkDecorator implements Coffee {
    Coffee coffee;
    MilkDecorator(Coffee c) { this.coffee = c; }
    public double cost() { return coffee.cost() + 0.5; }  // adds milk cost
}

Coffee c = new MilkDecorator(new SimpleCoffee());  // 1.5
```

---

### Behavioral Patterns

#### Observer Pattern

**Problem:** When one object changes, many dependent objects need to be notified automatically.

```java
interface Observer { void update(String event); }

class EventSystem {
    List<Observer> observers = new ArrayList<>();

    void subscribe(Observer o) { observers.add(o); }
    void notify(String event) {
        for (Observer o : observers) o.update(event);
    }
}

// Usage
EventSystem events = new EventSystem();
events.subscribe(e -> System.out.println("Logger: " + e));
events.subscribe(e -> System.out.println("Email: " + e));
events.notify("UserRegistered");  // both observers called
```

#### Strategy Pattern

**Problem:** You want to swap algorithms/behaviors at runtime.

```java
interface SortStrategy { void sort(int[] arr); }
class BubbleSort implements SortStrategy { public void sort(int[] arr) { ... } }
class QuickSort  implements SortStrategy { public void sort(int[] arr) { ... } }

class Sorter {
    SortStrategy strategy;
    Sorter(SortStrategy s) { this.strategy = s; }
    void sort(int[] arr) { strategy.sort(arr); }
}

Sorter s = new Sorter(new QuickSort());
s.sort(data);  // uses QuickSort — can swap to BubbleSort without changing Sorter
```

### Pattern Summary Table

| Pattern | Category | Problem Solved | Key Idea |
|---|---|---|---|
| **Singleton** | Creational | Ensure one instance | Private constructor + static getInstance() |
| **Factory** | Creational | Hide object creation | Factory method returns interface |
| **Builder** | Creational | Complex construction | Fluent API, step-by-step building |
| **Prototype** | Creational | Clone objects | Implement `clone()` method |
| **Adapter** | Structural | Incompatible interfaces | Wrapper that translates calls |
| **Decorator** | Structural | Add behavior dynamically | Wrapper that extends behavior |
| **Facade** | Structural | Simplify complex subsystem | Single class as simple interface |
| **Composite** | Structural | Tree structures | Treat individual and group uniformly |
| **Observer** | Behavioral | Event notification | Publish-subscribe mechanism |
| **Strategy** | Behavioral | Swap algorithms | Interface + multiple implementations |
| **Command** | Behavioral | Encapsulate requests | Request as object (undo/redo) |
| **Template Method** | Behavioral | Define algorithm skeleton | Abstract base + concrete steps |

---

## 18. Exception Handling in OOP

### Why OOP and Exceptions Go Together

In OOP, exceptions are **objects** — they are instances of exception classes. This fits naturally into the OOP model and allows for exception hierarchies.

### Java Exception Hierarchy

```
Throwable
├── Error (JVM errors — don't catch these normally)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (Unchecked — compiler doesn't enforce handling)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   ├── ArithmeticException (divide by zero)
    │   └── IllegalArgumentException
    └── Checked Exceptions (compiler FORCES you to handle)
        ├── IOException
        ├── FileNotFoundException
        ├── SQLException
        └── ClassNotFoundException
```

### Checked vs Unchecked Exceptions

| Type | Checked | Unchecked |
|---|---|---|
| **Base class** | `Exception` (not RuntimeException) | `RuntimeException` |
| **Compiler enforced?** | ✅ Yes — must catch or declare | ❌ No — optional |
| **Represents** | External, recoverable issues (file missing, network down) | Programming errors (null pointer, array bounds) |
| **Example** | `IOException`, `SQLException` | `NullPointerException`, `ArrayIndexOutOfBoundsException` |
| **Handling** | `try-catch` or `throws` declaration | Fix the code; optionally catch |

### Custom Exception Classes

```java
// Custom checked exception
class InsufficientFundsException extends Exception {
    private double amount;

    InsufficientFundsException(double amount) {
        super("Insufficient funds: needed " + amount);
        this.amount = amount;
    }

    double getAmount() { return amount; }
}

// Usage in a class
class BankAccount {
    private double balance;

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount);
        }
        balance -= amount;
    }
}

// Catching the custom exception
try {
    account.withdraw(1000);
} catch (InsufficientFundsException e) {
    System.out.println("Can't withdraw: " + e.getMessage());
}
```

### Exception Handling Best Practices in OOP

```java
// 1. Catch specific exceptions, not generic Exception
try { ... }
catch (FileNotFoundException e) { ... }   // ✅ specific
catch (IOException e) { ... }             // ✅ broader fallback

// 2. Never swallow exceptions silently
catch (Exception e) { }  // ❌ BAD — hides problems!

// 3. Use finally for cleanup (or try-with-resources)
try (FileReader fr = new FileReader("file.txt")) {  // auto-closed
    // use fr
}  // fr.close() called automatically

// 4. Throw exceptions with meaningful messages
throw new IllegalArgumentException("Age must be positive, got: " + age);

// 5. Don't use exceptions for normal flow control
// ❌ BAD: Using exception as control flow
try { int i = 0; while (true) arr[i++]... } catch (ArrayIndexOutOfBoundsException e) { }

// ✅ GOOD: Use a loop condition
for (int i = 0; i < arr.length; i++) { ... }
```

---

## 19. OOP in Java vs C++ vs Python — Key Differences

### Java OOP Specifics

- All classes implicitly extend `Object` (root of all classes)
- **Single inheritance** for classes; multiple via interfaces
- **Garbage collection** — no manual memory management
- `@Override` annotation (optional but recommended)
- No destructors — use `try-with-resources` or `AutoCloseable`
- All methods are **virtual by default** (can be overridden unless `final`)

### C++ OOP Specifics

- **Multiple inheritance** supported natively (with diamond problem)
- Manual memory management (`new`/`delete`)
- Explicit **destructors** `~ClassName()`
- Methods are **NOT virtual by default** — must declare `virtual`
- **`virtual`** enables runtime polymorphism; without it, static binding
- `override` and `final` keywords (C++11+)
- **`friend`** classes and functions
- Abstract class: class with at least one **pure virtual function** (`= 0`)

```cpp
class Shape {
public:
    virtual double area() = 0;  // pure virtual — makes class abstract
    virtual ~Shape() { }        // virtual destructor — important!
};

class Circle : public Shape {
public:
    double area() override { return 3.14 * r * r; }
};
```

### Python OOP Specifics

- **Multiple inheritance** supported (MRO — Method Resolution Order using C3 linearization)
- Everything is an object (even functions, classes)
- **Dynamic typing** — no explicit type declarations
- No access modifiers — convention-based (`_` and `__`)
- `__init__()` is the constructor
- `__del__()` is the destructor (non-deterministic)
- `self` is explicit (Java's `this` is implicit)
- **Duck typing** — "if it quacks like a duck, it's a duck" — polymorphism without inheritance

```python
class Animal:
    def __init__(self, name):
        self.name = name       # 'self' is explicit
        self._age = 0          # protected by convention
        self.__id = 123        # name-mangled to _Animal__id

    def speak(self):           # no return type declared
        pass                   # abstract-like (Python doesn't force override)

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

# Duck typing — polymorphism without common parent
class Robot:
    def speak(self):
        return "Beep boop"

animals = [Dog("Bruno"), Cat("Whiskers"), Robot()]  # Robot isn't Animal!
for a in animals:
    print(a.speak())  # works because all have speak() — duck typing
```

### Comparison Table

| Feature | Java | C++ | Python |
|---|---|---|---|
| **Multiple inheritance** | ❌ (interfaces only) | ✅ | ✅ |
| **Memory management** | Automatic (GC) | Manual (new/delete) | Automatic (GC) |
| **Destructor** | `finalize()` (deprecated) | `~Class()` | `__del__()` |
| **Default virtual** | ✅ All methods | ❌ Must declare `virtual` | ✅ All methods |
| **Access enforcement** | Compiler-enforced | Compiler-enforced | Convention-based |
| **Abstract class** | `abstract` keyword | Pure virtual method | `ABC` module or convention |
| **Interface** | `interface` keyword | Pure virtual class (convention) | `ABC` with abstractmethod |
| **Constructor name** | Same as class | Same as class | `__init__()` |
| **`this`/`self`** | `this` (implicit) | `this` (implicit) | `self` (explicit) |
| **Type system** | Static typing | Static typing | Dynamic typing |
| **Operator overloading** | ❌ (limited) | ✅ | ✅ |

---

## 20. Common Interview Traps

Know these — interviewers use them to separate prepared candidates from unprepared ones.

---

### ❌ Trap 1: "Abstraction and Encapsulation are the same thing"
✅ **Correct:** They are related but different.
- Encapsulation = **how**: bundle data + restrict access with private fields
- Abstraction = **what**: show only the essential interface, hide implementation complexity
- Encapsulation is at the **object level**; Abstraction is at the **design level**

---

### ❌ Trap 2: "Method overriding and method hiding are the same"
✅ **Correct:** Instance methods are **overridden** (dynamic dispatch, runtime). Static methods are **hidden** (static dispatch, compile time). You cannot override a static method — calling it through a parent reference uses the parent's version regardless of actual object type.

---

### ❌ Trap 3: "An abstract class can't have a constructor"
✅ **Correct:** Abstract classes **CAN** have constructors. They're called via `super()` from subclass constructors to initialize the abstract class's fields. You just can't instantiate the abstract class directly.

---

### ❌ Trap 4: "Interfaces can't have any method bodies"
✅ **Correct (pre-Java 8):** Since **Java 8**, interfaces can have `default` methods (with body) and `static` methods (with body). Since **Java 9**, also `private` methods. The distinction: `default` methods can be overridden; `static` methods belong to the interface itself.

---

### ❌ Trap 5: "A final variable is immutable"
✅ **Correct (for primitives):** For **primitive** types, `final` makes the value immutable. For **reference** types, `final` only means the reference can't be reassigned — the object it points to CAN still be mutated.
```java
final List<String> list = new ArrayList<>();
list.add("hello");  // ✅ modifying the object — allowed
list = new ArrayList<>();  // ❌ reassigning the reference — not allowed
```

---

### ❌ Trap 6: "Constructors are inherited"
✅ **Correct:** Constructors are **NOT inherited**. If you want a parent class constructor in the child class, you explicitly call it with `super()`. The child must define its own constructors.

---

### ❌ Trap 7: "Private methods can be overridden"
✅ **Correct:** Private methods are **not visible** in subclasses, so they **cannot be overridden**. If a subclass defines a method with the same name, it's a completely new, unrelated method (not overriding). No dynamic dispatch for private methods.

---

### ❌ Trap 8: "Polymorphism only means method overriding"
✅ **Correct:** Polymorphism has two forms:
1. **Compile-time** (method overloading)
2. **Run-time** (method overriding)
Also, in some contexts: operator overloading (C++, Python), parametric polymorphism (generics/templates).

---

### ❌ Trap 9: "Multiple inheritance in Java is impossible"
✅ **Correct (for classes):** Java doesn't allow multiple **class** inheritance. But a class **CAN** implement multiple interfaces. And interfaces can `extend` multiple interfaces. With Java 8's default methods, this can cause diamond-problem-like issues — resolved by requiring the implementing class to override the conflicting default method.

---

### ❌ Trap 10: "Composition and Aggregation are the same"
✅ **Correct:** Both are HAS-A relationships, but:
- **Aggregation:** Child can exist independently. Child is passed in (created externally). Weak lifecycle dependency.
- **Composition:** Child cannot exist without parent. Child is created inside parent. Strong lifecycle dependency.

---

### ❌ Trap 11: "`super()` is optional in child constructors"
✅ **Correct:** Java **automatically inserts** `super()` (parent's default constructor) as the first line of a child constructor if you don't write any `super(...)` or `this(...)` call. But if the parent class has NO default constructor (only parameterized), you MUST explicitly call `super(args)` — otherwise it's a compile error.

---

### ❌ Trap 12: "Singleton is always created eagerly"
✅ **Correct:** There are two approaches:
- **Eager initialization:** Instance created when class loads (simple, thread-safe but wastes memory if never used)
- **Lazy initialization:** Instance created on first request (saves memory, needs synchronization for thread safety)

---

## 21. Scenario and Coding Questions

### Scenario 1: Design a Library Management System

**Entities to identify:**
```
Book          — title, ISBN, author, available/checked-out
Member        — memberId, name, borrowedBooks[]
Librarian     — manages books and members
Library       — contains books, manages members

Relationships:
- Library HAS-A collection of Books (composition)
- Library HAS-A collection of Members (aggregation)
- Member BORROWS Book (association)
- Librarian IS-A Employee (inheritance)
```

**Key OOP decisions:**
- `Book` class — encapsulate all book data
- `Borrowable` interface — `borrow()`, `returnBook()` — so any borrowable item follows same contract
- Abstract `Person` class — `name`, `id` common to both Member and Librarian

---

### Scenario 2: Implement a Singleton Thread-Safe

```java
class Singleton {
    // Volatile ensures visibility across threads
    private static volatile Singleton instance;

    private Singleton() { }

    // Double-checked locking
    public static Singleton getInstance() {
        if (instance == null) {               // first check (no lock)
            synchronized (Singleton.class) {  // lock only once
                if (instance == null) {       // second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

---

### Scenario 3: Why does this code print "Animal" and not "Dog"?

```java
class Animal {
    static void display() { System.out.println("Animal"); }
}
class Dog extends Animal {
    static void display() { System.out.println("Dog"); }
}

Animal a = new Dog();
a.display();  // prints "Animal" — WHY?
```

**Answer:** `display()` is a **static method**. Static methods use **static binding** (compile-time). At compile time, the reference type is `Animal`, so `Animal.display()` is called — regardless of the actual object being `Dog`. This is **method hiding**, not overriding. Dynamic dispatch only works for **instance** methods.

---

### Scenario 4: Explain this output

```java
class Parent {
    int x = 10;
    void show() { System.out.println("Parent: " + x); }
}
class Child extends Parent {
    int x = 20;
    @Override
    void show() { System.out.println("Child: " + x); }
}

Parent p = new Child();
p.show();    // prints "Child: 20"
System.out.println(p.x);  // prints 10 — WHY?
```

**Answer:**
- `p.show()` → **dynamic dispatch** → calls `Child.show()` → prints "Child: 20" ✅
- `p.x` → **fields are NOT polymorphic** → resolved at compile time based on reference type (`Parent`) → prints `Parent.x = 10`

> Fields are statically bound; methods are dynamically bound. This is a classic interview trick.

---

### Scenario 5: Design Pattern in the Wild

**Question:** "Which design pattern does `Collections.sort(list, comparator)` use?"

**Answer:** **Strategy Pattern**. The sorting algorithm is fixed, but the comparison strategy (how to compare elements) is passed as a parameter (`Comparator`). Different comparators (sort by name, by age, by salary) can be plugged in without changing the sorting code.

---

### Quick Coding Exercises

**1. Write a class that enforces that only positive values can be set:**
```java
class PositiveNumber {
    private double value;
    void setValue(double v) {
        if (v <= 0) throw new IllegalArgumentException("Must be positive");
        this.value = v;
    }
    double getValue() { return value; }
}
```

**2. Show method overloading with print:**
```java
class Printer {
    void print(int i)    { System.out.println("int: " + i); }
    void print(String s) { System.out.println("String: " + s); }
    void print(double d) { System.out.println("double: " + d); }
}
```

**3. Implement a simple Observer pattern:**
```java
interface Observer { void update(String msg); }
class EventBus {
    List<Observer> observers = new ArrayList<>();
    void subscribe(Observer o) { observers.add(o); }
    void publish(String event) { observers.forEach(o -> o.update(event)); }
}
```

---

## 22. Must-Know Essentials — Final Checklist

Before your interview, verify you can answer each without hesitation.

---

### Fundamentals

- [ ] What is OOP and what problems does it solve over procedural programming?
- [ ] Difference between a class and an object
- [ ] What is instantiation? What is a reference vs an object?
- [ ] What is the difference between stack and heap memory in context of objects?

### The Four Pillars

- [ ] Define Encapsulation and explain WHY it's useful (not just "it hides data")
- [ ] Define Abstraction and explain the difference from Encapsulation
- [ ] Define Inheritance — what is IS-A relationship? When NOT to use it?
- [ ] Define Polymorphism — explain both compile-time and run-time types with examples
- [ ] The Diamond Problem — what is it and how do Java/C++/Python handle it?

### Constructors

- [ ] What is a constructor? How is it different from a regular method?
- [ ] What happens if you write no constructor? What if you write a parameterized one?
- [ ] Difference between shallow copy and deep copy
- [ ] What is constructor chaining (`this()` and `super()`)?
- [ ] Rules: `this()` and `super()` must be the first statement; can't use both together

### Access Modifiers

- [ ] 4 Java access modifiers and their scope (private, default, protected, public)
- [ ] Python's convention-based approach (`_` and `__`)

### Abstract Classes & Interfaces

- [ ] Abstract class: can it be instantiated? Can it have constructors? Can it have concrete methods?
- [ ] Interface: what methods can it have in Java 8+? What fields?
- [ ] When to choose abstract class vs interface?
- [ ] A class can extend ONE class but implement MULTIPLE interfaces

### Overloading vs Overriding

- [ ] Overloading = same name, different params, same class, compile-time
- [ ] Overriding = same name, same params, child class, run-time
- [ ] Can you override static/private/final methods? (No)
- [ ] What is covariant return type?
- [ ] What is method hiding?

### Keywords

- [ ] `static` — belongs to class, not instance; no `this`; shared across all objects
- [ ] `final` — variable (constant), method (no override), class (no subclass)
- [ ] `this` — current object; disambiguate; constructor chaining; can't use in static
- [ ] `super` — parent class; call parent constructor; call overridden method

### Relationships

- [ ] Association vs Aggregation vs Composition — lifecycle dependency is the key
- [ ] Composition over Inheritance — when and why
- [ ] High Cohesion + Loose Coupling = well-designed system

### SOLID

- [ ] S: Single Responsibility — one reason to change
- [ ] O: Open/Closed — open for extension, closed for modification
- [ ] L: Liskov Substitution — subclass should work wherever parent is used
- [ ] I: Interface Segregation — don't force irrelevant method implementations
- [ ] D: Dependency Inversion — depend on abstractions, not concrete classes

### Design Patterns (Know at least these)

- [ ] Singleton — one instance, private constructor, static getInstance()
- [ ] Factory — hide object creation behind a method
- [ ] Builder — step-by-step complex object construction
- [ ] Observer — publish-subscribe, event notification
- [ ] Strategy — swap algorithms at runtime via interface

### Exception Handling

- [ ] Checked vs Unchecked exceptions — which does compiler enforce?
- [ ] Custom exception — extend Exception (checked) or RuntimeException (unchecked)
- [ ] Never swallow exceptions; use specific catch blocks

---

## 📚 Quick Reference: Interview One-Liners

| Question | One-liner Answer |
|---|---|
| What is OOP? | Paradigm organizing code around objects that combine state and behavior |
| What are the 4 pillars? | Encapsulation, Abstraction, Inheritance, Polymorphism (A PIE) |
| Encapsulation vs Abstraction? | Encapsulation: restrict data access. Abstraction: hide complexity. Different levels. |
| What is Polymorphism? | One interface, many forms — compile-time (overloading) and run-time (overriding) |
| What is Inheritance? | Child class acquires parent's properties/methods; models IS-A relationship |
| Abstract class vs Interface? | Abstract: partial implementation, single inheritance. Interface: pure contract, multiple. |
| Overloading vs Overriding? | Overloading: same class, different params, compile-time. Overriding: child class, same params, runtime. |
| Can we override static methods? | No — static methods are hidden, not overridden. Static binding, not dynamic. |
| Can abstract class have constructor? | Yes — called via super() from subclasses to initialize fields |
| What is the Diamond Problem? | Ambiguity when inheriting from two classes with same method. Java avoids with no multiple class inheritance. |
| Composition vs Aggregation? | Aggregation: child lives independently (passed in). Composition: child dies with parent (created inside). |
| What is Singleton? | Design pattern ensuring only one instance; private constructor + static getInstance() |
| What is the SOLID 'O'? | Open/Closed: extend behavior by adding code (not modifying existing) |
| What is Liskov Substitution? | Subclass objects must be replaceable for parent objects without breaking behavior |
| `final` in Java? | Variable: constant. Method: no override. Class: no subclass. |
| What is `this`? | Reference to current object. Used to disambiguate fields and chain constructors. |
| Checked vs Unchecked exception? | Checked: compiler forces handling (IOException). Unchecked: programming errors (NullPointerException). |
| What is coupling? | Degree of dependency between classes. Loose coupling = good design. |
| What is cohesion? | How focused a class's responsibilities are. High cohesion = good design. |
| What is Duck Typing? | Python polymorphism: "if it has the right methods, it works" — no inheritance needed |

---

*📌 Remember: In an interview, always explain the **why** behind each concept — why it exists, what problem it solves. That transforms a good answer into a great one. Good luck with your placements!*
