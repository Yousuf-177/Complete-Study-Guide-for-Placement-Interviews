# 🧩 Object-Oriented Programming in C++ — Complete Study Guide for Placement Interviews

> **How to use this guide:** Every concept is explained with *why it exists*, *how C++ implements it differently from other languages*, and *how to answer it confidently in an interview*. All code is pure C++. Read section by section — later sections build on earlier ones. Use the checklists and one-liners at the end to self-test.

---

## Table of Contents

1. [What is OOP and Why C++ for OOP?](#1-what-is-oop-and-why-c-for-oop)
2. [Classes and Objects in C++](#2-classes-and-objects-in-c)
3. [The Four Pillars of OOP](#3-the-four-pillars-of-oop)
4. [Encapsulation in C++](#4-encapsulation-in-c)
5. [Abstraction in C++](#5-abstraction-in-c)
6. [Inheritance in C++ — All Types](#6-inheritance-in-c--all-types)
7. [Polymorphism in C++ — Virtual Functions & vtable](#7-polymorphism-in-c--virtual-functions--vtable)
8. [Constructors in C++ — Every Type](#8-constructors-in-c--every-type)
9. [Destructors in C++ — The Critical Difference](#9-destructors-in-c--the-critical-difference)
10. [Access Modifiers and Inheritance Visibility](#10-access-modifiers-and-inheritance-visibility)
11. [Static Members in C++](#11-static-members-in-c)
12. [Friend Functions and Friend Classes](#12-friend-functions-and-friend-classes)
13. [Operator Overloading](#13-operator-overloading)
14. [Abstract Classes and Pure Virtual Functions](#14-abstract-classes-and-pure-virtual-functions)
15. [Method Overloading vs Overriding in C++](#15-method-overloading-vs-overriding-in-c)
16. [The `this` Pointer in C++](#16-the-this-pointer-in-c)
17. [Memory Management — `new`, `delete`, and RAII](#17-memory-management--new-delete-and-raii)
18. [Copy Constructor and Assignment Operator](#18-copy-constructor-and-assignment-operator)
19. [Move Semantics and Rvalue References (C++11)](#19-move-semantics-and-rvalue-references-c11)
20. [Multiple Inheritance and the Diamond Problem](#20-multiple-inheritance-and-the-diamond-problem)
21. [Templates — Generic Programming in C++](#21-templates--generic-programming-in-c)
22. [Object Relationships — Association, Aggregation, Composition](#22-object-relationships--association-aggregation-composition)
23. [SOLID Principles in C++](#23-solid-principles-in-c)
24. [Design Patterns in C++](#24-design-patterns-in-c)
25. [Exception Handling in C++](#25-exception-handling-in-c)
26. [Common Interview Traps](#26-common-interview-traps)
27. [Scenario and Coding Questions](#27-scenario-and-coding-questions)
28. [Must-Know Essentials — Final Checklist](#28-must-know-essentials--final-checklist)

---

## 1. What is OOP and Why C++ for OOP?

### The Problem Before OOP

In C (procedural), you had:
- **Structs** for grouping data — but no behavior attached
- **Global functions** for behavior — but no protection over data
- Any function could modify any data — zero encapsulation
- Code reuse meant copy-pasting or using error-prone `#include` tricks

```c
// C procedural style — fragile
struct BankAccount {
    int accountNumber;
    double balance;   // anyone can write: account.balance = -99999; legally!
};

void deposit(struct BankAccount* acc, double amount) { acc->balance += amount; }
void withdraw(struct BankAccount* acc, double amount) { acc->balance -= amount; }
```

The problem: no **ownership**, no **protection**, no **extensibility**. As codebases scaled to millions of lines, this became unmaintainable.

### Why C++ for OOP?

C++ was designed by **Bjarne Stroustrup** in 1979 as **"C with Classes"** — it keeps C's raw performance (no garbage collector, direct memory control) while adding full OOP capabilities. This is why C++ is unique:

| Feature | C | C++ |
|---|---|---|
| Classes | ❌ (only structs) | ✅ |
| Encapsulation | ❌ | ✅ |
| Inheritance | ❌ | ✅ (including multiple) |
| Polymorphism | ❌ | ✅ (via virtual functions) |
| Operator Overloading | ❌ | ✅ |
| Templates (Generics) | ❌ | ✅ |
| Manual memory management | ✅ | ✅ (plus RAII) |
| Runtime overhead | Zero | Near-zero |

> 🎯 **Interview answer:** *"C++ is a multi-paradigm language that supports OOP while retaining C's performance. Unlike Java (which forces OOP and has GC), C++ gives the programmer full control — you decide what's on the stack vs heap, and you manage object lifetimes explicitly."*

---

## 2. Classes and Objects in C++

### Class Definition

In C++, a `class` is nearly identical to a `struct`, except that **members are private by default** in a class (public by default in a struct).

```cpp
#include <iostream>
#include <string>
using namespace std;

class BankAccount {
private:                          // private by default in class
    string accountNumber;
    double balance;
    string ownerName;

public:                           // must explicitly mark public interface
    // Constructor
    BankAccount(string accNo, string owner, double initialBalance)
        : accountNumber(accNo), ownerName(owner), balance(initialBalance) { }

    // Methods (behavior)
    void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }

    double getBalance() const { return balance; }  // 'const' = doesn't modify object
    string getOwner() const  { return ownerName; }
};

int main() {
    BankAccount acc("ACC001", "Alice", 5000.0);
    acc.deposit(1000.0);
    cout << acc.getBalance() << endl;  // 6000
}
```

### Class vs Struct in C++

| Feature | `class` | `struct` |
|---|---|---|
| Default access | `private` | `public` |
| Default inheritance | `private` | `public` |
| Use case | Full OOP entities with behavior | Plain data containers (POD types) |

> ✅ **Best practice:** Use `struct` for simple data bundles (no invariants). Use `class` when you have data + behavior + access control.

### Object Creation — Stack vs Heap

```cpp
// Stack allocation — destroyed automatically when goes out of scope
BankAccount acc1("ACC001", "Alice", 1000.0);   // on stack

// Heap allocation — YOU must destroy it
BankAccount* acc2 = new BankAccount("ACC002", "Bob", 2000.0);  // on heap
acc2->deposit(500.0);   // use -> for pointer, . for object
delete acc2;            // MUST manually free — or memory leak!
acc2 = nullptr;         // good practice after delete

// Modern C++ — use smart pointers (no manual delete needed)
#include <memory>
auto acc3 = make_unique<BankAccount>("ACC003", "Carol", 3000.0);
acc3->deposit(100.0);
// acc3 auto-deleted when it goes out of scope
```

### The `->` vs `.` Operator

```cpp
BankAccount  obj = ...;    // object on stack
obj.deposit(100);          // use . to call method

BankAccount* ptr = &obj;   // pointer to object
ptr->deposit(100);         // use -> to call method through pointer
(*ptr).deposit(100);       // equivalent — dereference first, then dot
```

---

## 3. The Four Pillars of OOP

These are the conceptual foundation. In C++, each pillar has specific mechanisms:

```
┌──────────────────────────────────────────────────────────────┐
│                  FOUR PILLARS IN C++                         │
│                                                              │
│  1. ENCAPSULATION  → private/protected + getters/setters     │
│  2. ABSTRACTION    → pure virtual functions + interfaces      │
│  3. INHERITANCE    → : public/private/protected BaseClass     │
│  4. POLYMORPHISM   → virtual functions + vtable dispatch      │
└──────────────────────────────────────────────────────────────┘
```

> 💡 **Mnemonic:** **"A PIE"** — **A**bstraction, **P**olymorphism, **I**nheritance, **E**ncapsulation

---

## 4. Encapsulation in C++

### What It Is

**Encapsulation** bundles data and the methods that operate on it into a single class, and **restricts direct access** to the data using access modifiers.

### Why It Matters

Without encapsulation, any code anywhere can corrupt your object's state:

```cpp
// BAD — no encapsulation (C-style struct)
struct BankAccount {
    double balance;  // public by default in struct
};

BankAccount acc;
acc.balance = -999999;  // completely legal — disaster!
```

With encapsulation:

```cpp
class BankAccount {
private:
    double balance;     // ONLY this class can directly access

public:
    void deposit(double amount) {
        if (amount <= 0) throw invalid_argument("Amount must be positive");
        balance += amount;
    }

    void withdraw(double amount) {
        if (amount <= 0 || amount > balance)
            throw runtime_error("Invalid withdrawal");
        balance -= amount;
    }

    double getBalance() const { return balance; }
    // No setBalance() — balance only changes through deposit/withdraw
};
```

### Getters, Setters, and `const` Correctness

```cpp
class Student {
private:
    string name;
    int age;
    double gpa;

public:
    // Getter — marked const because it doesn't modify the object
    string getName() const { return name; }
    int    getAge()  const { return age;  }
    double getGpa()  const { return gpa;  }

    // Setter — validates before modifying
    void setAge(int a) {
        if (a > 0 && a < 150) age = a;
        else throw out_of_range("Invalid age: " + to_string(a));
    }

    void setGpa(double g) {
        if (g >= 0.0 && g <= 10.0) gpa = g;
        else throw out_of_range("GPA must be between 0 and 10");
    }

    void setName(const string& n) {
        if (!n.empty()) name = n;
    }
};
```

### `const` Member Functions — C++-Specific Concept

A `const` member function **promises not to modify the object**. This is enforced by the compiler. It also allows the method to be called on `const` objects.

```cpp
class Circle {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}

    double area() const {          // const — read-only, safe to call on const object
        return 3.14159 * radius * radius;
    }

    void setRadius(double r) {     // non-const — modifies object
        radius = r;
    }
};

const Circle c(5.0);
c.area();       // ✅ OK — area() is const
c.setRadius(6); // ❌ ERROR — can't call non-const method on const object
```

### `mutable` Keyword

Sometimes you need to modify a field even inside a `const` method (e.g., a cache). Use `mutable`:

```cpp
class ExpensiveCalculation {
private:
    double data;
    mutable double cachedResult;   // mutable — can change even in const method
    mutable bool cacheValid = false;

public:
    double compute() const {
        if (!cacheValid) {
            cachedResult = data * data * 3.14;  // expensive
            cacheValid = true;                  // allowed because mutable
        }
        return cachedResult;
    }
};
```

> 🎯 **Interview one-liner:** *"Encapsulation in C++ uses private/protected members and public methods to control access. `const` member functions enforce the guarantee that a method won't modify the object — enabling safe usage with const objects and expressing intent clearly."*

---

## 5. Abstraction in C++

### What It Is

**Abstraction** means exposing only the essential interface and hiding implementation complexity. In C++, this is achieved through:
1. **Pure virtual functions** (making a class abstract)
2. **Access modifiers** (hiding implementation details)
3. **Header/source file separation** (`.h` declares interface, `.cpp` hides implementation)

### Abstraction via Pure Virtual Functions

```cpp
// Abstract base class — the "what" without the "how"
class Shape {
public:
    virtual double area()      const = 0;  // pure virtual — MUST override
    virtual double perimeter() const = 0;  // pure virtual — MUST override
    virtual void   draw()      const = 0;  // pure virtual — MUST override

    // Concrete method — provided once, reused by all
    void printInfo() const {
        cout << "Area: " << area()
             << " | Perimeter: " << perimeter() << endl;
    }

    virtual ~Shape() {}  // ALWAYS have virtual destructor in abstract classes
};

// Concrete classes — the "how"
class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area()      const override { return 3.14159 * radius * radius; }
    double perimeter() const override { return 2 * 3.14159 * radius; }
    void   draw()      const override { cout << "Drawing circle\n"; }
};

class Rectangle : public Shape {
private:
    double w, h;
public:
    Rectangle(double w, double h) : w(w), h(h) {}
    double area()      const override { return w * h; }
    double perimeter() const override { return 2 * (w + h); }
    void   draw()      const override { cout << "Drawing rectangle\n"; }
};
```

### Header File Separation — Real-World Abstraction

```cpp
// shape.h — the PUBLIC interface (what users see)
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() {}
private:
    // internal fields here — users of the header don't need to know
};

// circle.cpp — the PRIVATE implementation (hidden from users)
#include "shape.h"
class Circle : public Shape {
    double radius;
public:
    Circle(double r);
    double area() const override;
};
```

### Encapsulation vs Abstraction — The C++ Perspective

| | Encapsulation | Abstraction |
|---|---|---|
| **Mechanism in C++** | `private`/`protected` + getters/setters | Pure virtual functions, interfaces |
| **Goal** | Protect data integrity | Simplify usage, hide complexity |
| **Level** | Object / implementation | Design / interface |
| **Analogy** | Medicine in a sealed capsule | TV remote (you press buttons, not circuits) |
| **Example** | `private double balance` + `getBalance()` | `virtual double area() = 0` |

---

## 6. Inheritance in C++ — All Types

### What It Is and Why It Exists

**Inheritance** lets a derived class acquire all members of a base class, avoiding code duplication and modelling IS-A relationships. C++ supports **all five types** of inheritance — including multiple inheritance (unlike Java).

### Basic Syntax

```cpp
class Animal {                     // Base class
protected:
    string name;
    int age;

public:
    Animal(string n, int a) : name(n), age(a) {}

    virtual void speak() const {
        cout << name << " makes a sound\n";
    }

    void eat() const {
        cout << name << " is eating\n";
    }

    virtual ~Animal() {}           // Always virtual destructor in base
};

class Dog : public Animal {        // Derived class — public inheritance
private:
    string breed;

public:
    Dog(string n, int a, string b)
        : Animal(n, a), breed(b) {} // call base constructor via initializer list

    void speak() const override {  // override base method
        cout << name << " barks: Woof!\n";
    }

    void fetch() const {
        cout << name << " fetches the ball\n";
    }
};

int main() {
    Dog d("Bruno", 3, "Labrador");
    d.speak();   // "Bruno barks: Woof!"  — overridden
    d.eat();     // "Bruno is eating"     — inherited
    d.fetch();   // "Bruno fetches..."    — Dog-specific
}
```

### Types of Inheritance in C++

#### 1. Single Inheritance
```cpp
class A {};
class B : public A {};       // B inherits from A
```

#### 2. Multilevel Inheritance
```cpp
class A {};
class B : public A {};       // B inherits A
class C : public B {};       // C inherits B (and transitively A)
```

#### 3. Hierarchical Inheritance
```cpp
class Animal {};
class Dog  : public Animal {};  // Dog inherits Animal
class Cat  : public Animal {};  // Cat inherits Animal
class Bird : public Animal {};  // Bird inherits Animal
```

#### 4. Multiple Inheritance ← C++ exclusive (not in Java)
```cpp
class Flyable {
public:
    virtual void fly() { cout << "Flying\n"; }
    virtual ~Flyable() {}
};

class Swimmable {
public:
    virtual void swim() { cout << "Swimming\n"; }
    virtual ~Swimmable() {}
};

class Duck : public Flyable, public Swimmable {
public:
    void fly()  override { cout << "Duck flying\n"; }
    void swim() override { cout << "Duck swimming\n"; }
};

Duck d;
d.fly();    // Duck flying
d.swim();   // Duck swimming
```

#### 5. Hybrid Inheritance
A combination of multiple inheritance types — leads to the **Diamond Problem** (covered in Section 20).

---

### Inheritance Access Specifiers — C++-Specific

This is **unique to C++** and a common interview topic. The access specifier used when inheriting controls how base class members are seen in the derived class.

```cpp
class Base {
public:    int pub   = 1;
protected: int prot  = 2;
private:   int priv  = 3;   // always inaccessible to derived
};
```

| Base member | `public` inheritance | `protected` inheritance | `private` inheritance |
|---|---|---|---|
| `public` in Base | `public` in Derived | `protected` in Derived | `private` in Derived |
| `protected` in Base | `protected` in Derived | `protected` in Derived | `private` in Derived |
| `private` in Base | ❌ inaccessible | ❌ inaccessible | ❌ inaccessible |

```cpp
class PublicDerived   : public    Base {};  // pub stays public
class ProtectedDerived: protected Base {};  // pub becomes protected
class PrivateDerived  : private   Base {};  // pub becomes private

PublicDerived    pd;
pd.pub = 10;   // ✅ pub is public — accessible outside

PrivateDerived   prd;
prd.pub = 10;  // ❌ ERROR — pub became private in PrivateDerived
```

> 🎯 **Rule of thumb:**
> - **`public` inheritance** = IS-A relationship (Dog IS-A Animal) — use this by default
> - **`private` inheritance** = IMPLEMENTED-IN-TERMS-OF (implementation detail, not IS-A)
> - **`protected` inheritance** = rare; used when children of the derived class need access

### The `protected` Access Specifier

`protected` members are:
- ❌ Not accessible from outside the class (unlike `public`)
- ✅ Accessible inside the class
- ✅ Accessible in **derived classes** (unlike `private`)

```cpp
class Vehicle {
protected:
    int speed;           // accessible in derived classes
private:
    string serialNum;    // inaccessible even in derived classes
public:
    int getSpeed() const { return speed; }
};

class Car : public Vehicle {
public:
    void accelerate(int delta) {
        speed += delta;      // ✅ protected — accessible here
        // serialNum = "X";  // ❌ private — not accessible
    }
};
```

---

## 7. Polymorphism in C++ — Virtual Functions & vtable

### The Two Types

```
Polymorphism in C++
├── Compile-time (Static)  → Function overloading, operator overloading, templates
└── Run-time (Dynamic)     → Virtual functions, vtable dispatch
```

### Why `virtual` is Needed in C++ (Unlike Java)

**Java:** All instance methods are virtual by default (can be overridden).
**C++:** Methods are **NOT virtual by default**. You must explicitly mark them `virtual` to enable dynamic dispatch. This is a deliberate performance decision — virtual calls have a small overhead; non-virtual calls are resolved at compile time.

```cpp
class Animal {
public:
    void sound() {           // NOT virtual — static binding
        cout << "Animal sound\n";
    }

    virtual void speak() {   // virtual — dynamic binding
        cout << "Animal speaks\n";
    }
};

class Dog : public Animal {
public:
    void sound() {           // hides base (no dynamic dispatch)
        cout << "Dog sound\n";
    }

    void speak() override {  // overrides base (dynamic dispatch)
        cout << "Dog speaks: Woof\n";
    }
};

Animal* a = new Dog();

a->sound();   // "Animal sound"  — NOT virtual, base class version called!
a->speak();   // "Dog speaks: Woof" — virtual, derived version called!

delete a;
```

> ⚠️ **Critical Interview Point:** Without `virtual`, calling through a base pointer always calls the **base class version**, even if a derived class "overrides" it. This is called **static binding** (resolved at compile time). With `virtual`, the correct derived version is called — **dynamic binding** (resolved at runtime via vtable).

### How vtable Works Internally

This is a favourite deep-dive question in C++ interviews.

Every class that has at least one `virtual` function gets a hidden **virtual table (vtable)** — an array of function pointers to the most-derived implementation of each virtual function.

Every object of such a class gets a hidden **vptr** (virtual pointer) — a pointer to its class's vtable — added as the first member.

```
Memory layout of Dog object:
┌───────────┐
│  vptr ────────→  Dog's vtable: [speak → Dog::speak, ...]
├───────────┤
│  name     │
├───────────┤
│  age      │
└───────────┘

Memory layout of Animal object:
┌───────────┐
│  vptr ────────→  Animal's vtable: [speak → Animal::speak, ...]
├───────────┤
│  name     │
└───────────┘
```

When `a->speak()` is called on a base pointer:
1. Dereference `a` to find the object
2. Read the `vptr` from the object
3. Look up `speak` in the vtable pointed to by `vptr`
4. Call the function found → always the most-derived version

This is why it's **runtime** polymorphism — the decision of which function to call happens at runtime, not at compile time.

### `override` and `final` Keywords (C++11)

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual void   draw()       = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
public:
    double area() const override { return 3.14 * r * r; }  // override — compiler verifies this actually overrides something
    void   draw() override       { cout << "Circle\n"; }

    // final — prevents further overriding in sub-sub-classes
    virtual void describe() final { cout << "I am a Circle\n"; }
private:
    double r;
};

// Circle itself can be final too
class ImmutableCircle final : public Circle { };
// class SubCircle : public ImmutableCircle { };  // ❌ ERROR — final class
```

**Why use `override`?**
Without it, a typo in the signature creates a new method (not an override) with no compiler warning:

```cpp
class Dog : public Animal {
    void Speak() override { }    // ❌ ERROR — compiler catches the typo (capital S)
    void Speak()          { }    // ❌ No error without override — silently wrong!
};
```

### Runtime Polymorphism in Action

```cpp
#include <vector>
#include <memory>

// Process any shape polymorphically
void processShapes(const vector<unique_ptr<Shape>>& shapes) {
    for (const auto& s : shapes) {
        s->draw();                          // calls the right draw() at runtime
        cout << "Area: " << s->area() << "\n";
    }
}

int main() {
    vector<unique_ptr<Shape>> shapes;
    shapes.push_back(make_unique<Circle>(5.0));
    shapes.push_back(make_unique<Rectangle>(4.0, 6.0));
    shapes.push_back(make_unique<Triangle>(3.0, 4.0, 5.0));

    processShapes(shapes);   // each calls its own draw() and area()
}
```

---

## 8. Constructors in C++ — Every Type

### What a Constructor Does

A **constructor** initializes an object when it is created. In C++, if you don't provide one, the compiler generates a **default constructor** that zero-initializes trivial types. The constructor name is always the same as the class name and has no return type.

### Types of Constructors

#### 1. Default Constructor

```cpp
class Point {
public:
    double x, y;

    Point() : x(0.0), y(0.0) {}   // default constructor

    // OR — let compiler generate: Point() = default;
};

Point p;         // calls default constructor — p.x=0, p.y=0
Point p2 = {};   // also calls default constructor
```

#### 2. Parameterized Constructor

```cpp
class Point {
public:
    double x, y;
    Point(double x, double y) : x(x), y(y) {}
};

Point p(3.0, 4.0);         // calls parameterized constructor
Point p2 = {3.0, 4.0};    // aggregate initialization (C++11)
```

#### 3. Copy Constructor

Called when an object is initialized from another object of the same class.

```cpp
class Point {
public:
    double x, y;

    Point(double x, double y) : x(x), y(y) {}

    // Copy constructor — takes const reference to same type
    Point(const Point& other) : x(other.x), y(other.y) {
        cout << "Copy constructor called\n";
    }
};

Point p1(3.0, 4.0);
Point p2 = p1;        // copy constructor called
Point p3(p1);         // also calls copy constructor
```

**When is the copy constructor automatically called?**
```cpp
Point p1(1, 2);
Point p2 = p1;            // 1. Direct initialization from object
void func(Point p) {}
func(p1);                 // 2. Pass by value to a function
Point func2() { return p1; }
Point p3 = func2();       // 3. Return by value (may be optimized away by RVO)
```

#### 4. Move Constructor (C++11)

Called when an object is initialized from a **temporary (rvalue)** — transfers resources instead of copying them. (Detailed in Section 19.)

```cpp
Point(Point&& other) noexcept : x(other.x), y(other.y) {
    other.x = other.y = 0;   // leave moved-from in valid state
    cout << "Move constructor called\n";
}
```

#### 5. Delegating Constructor (C++11)

One constructor calls another in the same class:

```cpp
class Employee {
    string name;
    int id;
    string dept;

public:
    Employee(string name, int id, string dept)
        : name(name), id(id), dept(dept) {}

    Employee(string name, int id)
        : Employee(name, id, "General") {}  // delegates to 3-arg constructor

    Employee(string name)
        : Employee(name, 0) {}              // chains further
};
```

#### 6. Explicit Constructor — Preventing Implicit Conversions

```cpp
class Circle {
    double radius;
public:
    Circle(double r) : radius(r) {}              // implicit conversion allowed
    // explicit Circle(double r) : radius(r) {}  // explicit — prevents implicit
};

void draw(Circle c) { }

draw(5.0);      // Without explicit: ✅ 5.0 implicitly converted to Circle
                // With explicit:    ❌ ERROR — must write draw(Circle(5.0))
```

> 💡 **Best practice:** Mark single-argument constructors `explicit` unless you intentionally want implicit conversion.

### Member Initializer List — Why Use It?

C++ constructors should initialize members using the **initializer list** (after the `:`) instead of assignments in the constructor body.

```cpp
class MyClass {
    int x;
    string name;
    const int ID;          // const member — MUST use initializer list
    int& ref;              // reference member — MUST use initializer list

public:
    // ✅ Correct — initializer list
    MyClass(int x, string n, int id, int& r)
        : x(x), name(n), ID(id), ref(r) {}

    // ❌ Wrong — assignment in body (works for x and name, fails for const/ref)
    MyClass(int x, string n) {
        this->x = x;     // OK but inefficient (default-constructed, then assigned)
        name = n;        // OK but inefficient
        // ID = id;      // ERROR — can't assign to const
        // ref = r;      // ERROR — can't reassign reference
    }
};
```

**Why is initializer list more efficient?**
For `string name`:
- With body assignment: `string` is **default-constructed** first, then **copy-assigned** — two operations
- With initializer list: `string` is **copy-constructed** directly — one operation

**Order of initialization:** Members are initialized in the **order they are declared** in the class, NOT the order in the initializer list. Always match the two to avoid bugs.

---

## 9. Destructors in C++ — The Critical Difference

### What a Destructor Does

A **destructor** is called automatically when an object is destroyed (goes out of scope on stack, or `delete` is called on heap). Its job is to **release resources** — memory, file handles, network connections, locks.

```cpp
class FileHandler {
private:
    FILE* file;

public:
    FileHandler(const char* path) {
        file = fopen(path, "r");
        if (!file) throw runtime_error("Cannot open file");
        cout << "File opened\n";
    }

    ~FileHandler() {           // destructor — name is ~ClassName
        if (file) {
            fclose(file);
            cout << "File closed\n";
        }
    }
};

{
    FileHandler fh("data.txt");  // constructor called
    // ... use fh ...
}   // ← destructor called automatically here — file closed!
```

### Virtual Destructor — CRITICAL in C++

If you have a **base class pointer** pointing to a **derived class object**, and you `delete` it without a virtual destructor, **only the base class destructor runs** — the derived class destructor is skipped. This is **undefined behavior** and a **resource leak**.

```cpp
class Animal {
public:
    Animal()  { cout << "Animal created\n"; }
    ~Animal() { cout << "Animal destroyed\n"; }  // NOT virtual — PROBLEM!
};

class Dog : public Animal {
    int* data;
public:
    Dog() : data(new int[100]) { cout << "Dog created\n"; }
    ~Dog() {
        delete[] data;                            // this will NOT be called!
        cout << "Dog destroyed\n";
    }
};

Animal* a = new Dog();    // Animal created, Dog created
delete a;                 // ONLY Animal destroyed — Dog's destructor SKIPPED! Memory leak!
```

**Fix: Always declare base class destructor as `virtual`:**

```cpp
class Animal {
public:
    virtual ~Animal() { cout << "Animal destroyed\n"; }  // virtual!
};

Animal* a = new Dog();
delete a;   // NOW: Dog destroyed → Animal destroyed (correct order)
```

> 🎯 **Golden rule:** *"If a class has any virtual function, it must have a virtual destructor."* Any class intended to be used as a base class should have a `virtual` destructor.

### Destructor Rules

| Rule | Detail |
|---|---|
| No return type | Not even `void` |
| No parameters | Cannot be overloaded |
| One per class | Only one destructor |
| Called automatically | On scope exit (stack) or `delete` (heap) |
| Order | Derived destructor first, then base destructor (reverse of construction) |
| `virtual` needed? | Yes, if deleting through base pointer |

### Order of Construction and Destruction

```cpp
class A { public: A() { cout << "A()\n"; } ~A() { cout << "~A()\n"; } };
class B : public A { public: B() { cout << "B()\n"; } ~B() { cout << "~B()\n"; } };
class C : public B { public: C() { cout << "C()\n"; } ~C() { cout << "~C()\n"; } };

C obj;
// Output:
// A()    ← base first
// B()
// C()    ← most derived last
// ~C()   ← most derived destroyed first
// ~B()
// ~A()   ← base last
```

Construction goes **base → derived** (parents set up before children).
Destruction goes **derived → base** (children cleaned up before parents).

---

## 10. Access Modifiers and Inheritance Visibility

### The Three Access Levels

```cpp
class MyClass {
public:
    int pubMember;      // accessible everywhere

protected:
    int protMember;     // accessible in this class + derived classes

private:
    int privMember;     // accessible ONLY in this class
};
```

### Accessibility Summary

| Member | Same Class | Derived Class | Anywhere Outside |
|---|---|---|---|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

### Inheritance + Access Specifier Interaction

Recall from Section 6 — the access specifier in the inheritance declaration (`public`, `protected`, `private`) further restricts visibility:

```cpp
class Base {
public:    int x = 1;
protected: int y = 2;
private:   int z = 3;  // always unreachable in derived
};

class D1 : public    Base { };   // x=public,    y=protected
class D2 : protected Base { };   // x=protected, y=protected
class D3 : private   Base { };   // x=private,   y=private

D1 d1;
d1.x;  // ✅ public remains public
D2 d2;
d2.x;  // ❌ x became protected — inaccessible outside
D3 d3;
d3.x;  // ❌ x became private — inaccessible outside
```

---

## 11. Static Members in C++

### `static` Data Members

A `static` data member belongs to the **class**, not to any instance. There is **exactly one copy** regardless of how many objects exist. Must be **defined outside the class** (in the `.cpp` file).

```cpp
class Counter {
public:
    static int count;    // declaration — no memory allocated yet
    int id;

    Counter() {
        ++count;
        id = count;
    }

    ~Counter() {
        --count;
    }
};

int Counter::count = 0;   // DEFINITION — required outside class

int main() {
    Counter c1;   // count=1, c1.id=1
    Counter c2;   // count=2, c2.id=2
    {
        Counter c3;   // count=3
    }             // c3 destroyed → count=2

    cout << Counter::count << "\n";  // access via class name: 2
}
```

> ⚠️ `static` members can be accessed as `ClassName::member` or via an object `obj.member`, but the latter is misleading (it looks like an instance member).

### `static` Member Functions

A static member function:
- Belongs to the class, not any object
- Has **no `this` pointer**
- Can only access other `static` members
- Called via `ClassName::function()`

```cpp
class MathUtils {
public:
    static double pi() { return 3.14159265; }

    static double circleArea(double r) {
        return pi() * r * r;   // can call other static methods
    }

    // static double foo() { return this->x; }  // ❌ no 'this' in static!
};

double area = MathUtils::circleArea(5.0);  // no object needed
```

### `static` Local Variables

A `static` local variable inside a function persists across calls and is initialized only once:

```cpp
void counter() {
    static int callCount = 0;  // initialized once, persists
    ++callCount;
    cout << "Called " << callCount << " times\n";
}

counter();  // Called 1 times
counter();  // Called 2 times
counter();  // Called 3 times
```

---

## 12. Friend Functions and Friend Classes

### Why `friend` Exists

Sometimes a function outside a class needs access to its private members — without making those members public (which would break encapsulation for everyone). `friend` gives **selective access** to a specific function or class.

### Friend Function

```cpp
class BankAccount {
private:
    double balance;
    string owner;

public:
    BankAccount(string o, double b) : owner(o), balance(b) {}

    // Grant friend access to this function
    friend void printAccountDetails(const BankAccount& acc);
    friend bool transfer(BankAccount& from, BankAccount& to, double amount);
};

// Friend function — NOT a member, but has access to private members
void printAccountDetails(const BankAccount& acc) {
    cout << "Owner: " << acc.owner           // ✅ private — accessible!
         << " | Balance: " << acc.balance << "\n";
}

bool transfer(BankAccount& from, BankAccount& to, double amount) {
    if (from.balance >= amount) {            // ✅ private — accessible!
        from.balance -= amount;
        to.balance   += amount;
        return true;
    }
    return false;
}
```

### Friend Class

```cpp
class Engine {                   // forward declaration may be needed
    friend class Car;            // Car gets full access to Engine's private members
private:
    int horsepower;
    double fuelLevel;
public:
    Engine(int hp) : horsepower(hp), fuelLevel(100.0) {}
};

class Car {
private:
    Engine engine;
    string model;
public:
    Car(string m, int hp) : model(m), engine(hp) {}

    void showEngineDetails() {
        // ✅ Car is friend of Engine — can access private members
        cout << model << " has " << engine.horsepower << "hp\n";
        cout << "Fuel: " << engine.fuelLevel << "%\n";
    }
};
```

### Important Properties of `friend`

| Property | Detail |
|---|---|
| **Not inherited** | If B is friend of A, B's subclass is NOT automatically friend of A |
| **Not transitive** | If A is friend of B, and B is friend of C, A is NOT friend of C |
| **Not mutual** | If A declares B as friend, A is not automatically friend of B |
| **Breaks encapsulation?** | Slightly — use sparingly, only when the coupling is natural |
| **Use cases** | Operator overloading (`<<`, `>>`), utility functions, tightly coupled classes |

> 🎯 **Interview insight:** `friend` is a deliberate design choice to give *specific* external functions access, rather than making members `public` (which gives *everyone* access). Use it when two classes are inherently coupled (like a container and its iterator).

---

## 13. Operator Overloading

### What It Is

C++ lets you redefine the behavior of built-in operators (`+`, `-`, `*`, `<<`, `>>`, `==`, `[]`, etc.) for user-defined types. This makes objects feel natural to use.

### Why It Matters

Without operator overloading:
```cpp
Vector v1(1, 2), v2(3, 4);
Vector v3 = v1.add(v2);            // verbose, unnatural
bool equal = v1.equals(v2);
```

With operator overloading:
```cpp
Vector v3 = v1 + v2;               // natural!
bool equal = (v1 == v2);           // natural!
```

### How to Overload Operators

```cpp
class Vector2D {
public:
    double x, y;

    Vector2D(double x = 0, double y = 0) : x(x), y(y) {}

    // Overload + as member function
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }

    // Overload - as member function
    Vector2D operator-(const Vector2D& other) const {
        return Vector2D(x - other.x, y - other.y);
    }

    // Overload * (scalar multiplication)
    Vector2D operator*(double scalar) const {
        return Vector2D(x * scalar, y * scalar);
    }

    // Overload == for comparison
    bool operator==(const Vector2D& other) const {
        return (x == other.x) && (y == other.y);
    }

    // Overload [] for element access
    double operator[](int index) const {
        if (index == 0) return x;
        if (index == 1) return y;
        throw out_of_range("Index out of range");
    }

    // Overload += (compound assignment)
    Vector2D& operator+=(const Vector2D& other) {
        x += other.x;
        y += other.y;
        return *this;   // return reference to self for chaining
    }

    // Overload << as FRIEND (needs access to private, not member)
    friend ostream& operator<<(ostream& os, const Vector2D& v) {
        os << "(" << v.x << ", " << v.y << ")";
        return os;   // return stream for chaining: cout << v1 << v2;
    }
};

int main() {
    Vector2D v1(1, 2), v2(3, 4);
    Vector2D v3 = v1 + v2;        // (4, 6)
    cout << v3 << "\n";           // uses overloaded <<
    v1 += v2;
    cout << (v1 == v2) << "\n";
    cout << v3[0] << "\n";        // 4
}
```

### Prefix vs Postfix Increment/Decrement

```cpp
class Counter {
    int value;
public:
    Counter(int v = 0) : value(v) {}

    // Prefix ++c — returns reference to modified object
    Counter& operator++() {
        ++value;
        return *this;
    }

    // Postfix c++ — takes dummy int parameter, returns old value
    Counter operator++(int) {
        Counter old = *this;  // save old state
        ++value;
        return old;           // return old state
    }

    int get() const { return value; }
};

Counter c(5);
++c;    // prefix:  value becomes 6, returns c
c++;    // postfix: value becomes 7, returns Counter(6)
```

### Operators That CAN and CANNOT Be Overloaded

| Can Overload | Cannot Overload |
|---|---|
| `+`, `-`, `*`, `/`, `%` | `::` (scope resolution) |
| `==`, `!=`, `<`, `>`, `<=`, `>=` | `.` (member access) |
| `[]` (subscript) | `.*` (pointer-to-member) |
| `()` (function call) | `?:` (ternary) |
| `<<`, `>>` (stream) | `sizeof` |
| `++`, `--` | `typeid` |
| `=`, `+=`, `-=`, etc. | |
| `new`, `delete` | |

### Rules of Operator Overloading

1. You cannot create new operators (no `**` for power)
2. You cannot change the precedence or associativity of operators
3. At least one operand must be a user-defined type
4. Cannot overload for built-in types only

---

## 14. Abstract Classes and Pure Virtual Functions

### Pure Virtual Function

A **pure virtual function** is declared with `= 0`. It has **no implementation** in the base class. Any class containing a pure virtual function becomes an **abstract class**.

```cpp
class Shape {
public:
    virtual double area()      const = 0;   // pure virtual
    virtual double perimeter() const = 0;   // pure virtual
    virtual void   draw()            = 0;   // pure virtual

    // Can also have non-pure virtual (default behavior, can override)
    virtual void printInfo() const {
        cout << "Area: " << area() << "\n";
    }

    // Can have concrete (non-virtual) methods too
    void describe() const {
        cout << "I am a shape.\n";
    }

    virtual ~Shape() {}    // always virtual destructor
};

// Shape s;       // ❌ ERROR — cannot instantiate abstract class

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area()      const override { return 3.14159 * r * r; }
    double perimeter() const override { return 2 * 3.14159 * r; }
    void   draw()            override { cout << "Drawing circle\n"; }
};
```

### Interface via Pure Abstract Class

C++ has no `interface` keyword (unlike Java). An **interface** is simulated by a class where **ALL methods are pure virtual** and there are **no data members**:

```cpp
// "Interface" in C++
class Printable {
public:
    virtual void print()     const = 0;
    virtual void printToFile(const string& path) const = 0;
    virtual ~Printable() {}   // always virtual destructor
};

class Serializable {
public:
    virtual string serialize()   const = 0;
    virtual void   deserialize(const string& data) = 0;
    virtual ~Serializable() {}
};

// Class implementing multiple "interfaces" (multiple inheritance)
class Report : public Printable, public Serializable {
    string content;
public:
    Report(string c) : content(c) {}
    void   print()     const override { cout << content << "\n"; }
    void   printToFile(const string& p) const override { /* write to file */ }
    string serialize() const override { return content; }
    void   deserialize(const string& d) override { content = d; }
};
```

### Abstract Class Facts

| Fact | Detail |
|---|---|
| **Cannot instantiate** | `Shape s;` → compile error |
| **Can have pointer/reference** | `Shape* s = new Circle(5);` → ✅ used for polymorphism |
| **Can have constructor** | Yes — called by derived class via initializer list |
| **Can have non-pure methods** | Yes — partial default implementation |
| **Derived must override all** | If derived doesn't override all pure virtuals, it's also abstract |

---

## 15. Method Overloading vs Overriding in C++

### Method Overloading (Compile-Time Polymorphism)

Multiple functions in the **same scope** with the **same name** but **different parameter lists**. The compiler selects the right one at compile time.

```cpp
class Calculator {
public:
    int    add(int a, int b)          { return a + b; }
    double add(double a, double b)    { return a + b; }
    int    add(int a, int b, int c)   { return a + b + c; }
    string add(string a, string b)    { return a + b; }
};

Calculator c;
c.add(2, 3);          // int version
c.add(2.5, 3.5);      // double version
c.add(1, 2, 3);       // three-arg version
c.add("Hi", " World");// string version
```

### Method Overriding (Run-Time Polymorphism)

A derived class provides a **specific implementation** of a `virtual` method from the base class.

```cpp
class Animal {
public:
    virtual void speak() const {   // virtual — enables override
        cout << "Animal speaks\n";
    }
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void speak() const override {  // override — explicitly marks this as overriding
        cout << "Woof!\n";
    }
};

class Cat : public Animal {
public:
    void speak() const override {
        cout << "Meow!\n";
    }
};

Animal* a = new Dog();
a->speak();   // Woof! — runtime dispatch
delete a;
```

### Key Comparison

| Feature | Overloading | Overriding |
|---|---|---|
| **Scope** | Same class (or namespace) | Base and derived class |
| **Parameters** | Must differ | Must be identical |
| **`virtual`** | Not required | Required for dynamic dispatch |
| **Binding** | Static (compile-time) | Dynamic (runtime) via vtable |
| **Return type** | Can differ | Must match (or covariant) |
| **`override` keyword** | Not applicable | Recommended — prevents silent bugs |
| **Works with `static`** | Yes — static methods can be overloaded | No — static methods cannot be overridden |

### Function Hiding — A C++ Trap

When a derived class defines a function with the same name as a base class function (but different signature, without `virtual`), it **hides** all base class overloads of that name:

```cpp
class Base {
public:
    void show()         { cout << "Base::show()\n"; }
    void show(int x)    { cout << "Base::show(int)\n"; }
};

class Derived : public Base {
public:
    void show(double d) { cout << "Derived::show(double)\n"; }
    // This HIDES both Base::show() and Base::show(int)!
};

Derived d;
d.show(3.14);  // ✅ Derived::show(double)
d.show();      // ❌ ERROR — Base::show() is hidden!
d.show(5);     // ❌ ERROR — Base::show(int) is hidden!

// Fix: use 'using' to bring base overloads into scope
class DerivedFixed : public Base {
public:
    using Base::show;   // bring all Base::show overloads into scope
    void show(double d) { cout << "DerivedFixed::show(double)\n"; }
};
```

> 🎯 **Interview trap:** Function hiding is a common C++ gotcha. Always use `using Base::method` if you want to extend overloads, not hide them.

---

## 16. The `this` Pointer in C++

### What `this` Is

`this` is a **hidden pointer** passed to every non-static member function, pointing to the object on which the function is called. Its type is `ClassName*` (or `const ClassName*` for const methods).

```cpp
class Rectangle {
    double width, height;

public:
    Rectangle(double w, double h) : width(w), height(h) {}

    // Use 1: Disambiguate member from parameter
    void setWidth(double width) {
        this->width = width;      // this->width = member field
                                  // width = parameter
    }

    // Use 2: Return *this for method chaining (fluent interface)
    Rectangle& setHeight(double h) {
        this->height = h;
        return *this;             // return reference to current object
    }

    Rectangle& scale(double factor) {
        width  *= factor;
        height *= factor;
        return *this;
    }

    double area() const { return width * height; }
};

// Method chaining using return *this
Rectangle r(4, 5);
double a = r.setHeight(6).scale(2).area();  // chain: sets h=6, scales, gets area
```

### Use 3: Pass Current Object to Another Function

```cpp
class Node {
public:
    int data;
    void registerWith(Registry& reg) {
        reg.add(this);    // pass pointer to current object
    }
};
```

### Use 4: Compare Objects

```cpp
bool isSame(const Rectangle& other) const {
    return this == &other;   // checks if same memory address (same object)
}
```

### `this` in `const` Member Functions

In a `const` member function, `this` has type `const ClassName*` — you can't modify members through it.

```cpp
class Foo {
    int x;
public:
    void modify() {       // this = Foo*
        this->x = 5;     // ✅ allowed
    }

    int get() const {     // this = const Foo*
        // this->x = 5;  // ❌ ERROR — const method, can't modify
        return this->x;   // ✅ reading is fine
    }
};
```

---

## 17. Memory Management — `new`, `delete`, and RAII

### Manual Memory Management — The C++ Way

Unlike Java (garbage collected), C++ requires **explicit memory management** for heap-allocated objects. This is both a power and a responsibility.

```cpp
// Allocating on heap
int* p = new int(42);           // allocate single int, initialized to 42
int* arr = new int[10];         // allocate array of 10 ints
int* arr2 = new int[10]();      // value-initialized to zero

// Deallocating — MUST be done manually
delete p;                       // free single object
delete[] arr;                   // free array — MUST use delete[]
delete[] arr2;

p   = nullptr;   // good practice — avoid dangling pointers
arr = nullptr;
```

### Common Memory Bugs

| Bug | Description | Example |
|---|---|---|
| **Memory Leak** | Allocated memory never freed | `new` without `delete` |
| **Dangling Pointer** | Pointer to freed memory | `delete p; *p = 5;` |
| **Double Delete** | Freeing same memory twice | `delete p; delete p;` — UB |
| **Use After Free** | Accessing freed memory | UB — crash or data corruption |
| **Array/scalar mismatch** | `delete` instead of `delete[]` | UB |

### RAII — Resource Acquisition Is Initialization

**RAII** is the C++ idiom that **ties resource lifetime to object lifetime**. Resources (memory, files, locks) are:
- **Acquired** in the constructor
- **Released** in the destructor

Since destructors run automatically (on scope exit), resources are never leaked — even if exceptions are thrown.

```cpp
class ManagedArray {
    int* data;
    size_t size;

public:
    ManagedArray(size_t n) : data(new int[n]()), size(n) {
        cout << "Allocated " << n << " ints\n";
    }

    ~ManagedArray() {
        delete[] data;              // guaranteed to run — no leak!
        cout << "Freed memory\n";
    }

    int& operator[](size_t i) { return data[i]; }

    // No raw pointer access — encapsulated safely
};

void process() {
    ManagedArray arr(100);  // allocated
    arr[0] = 42;
    // ... even if exception thrown here ...
}   // ← arr destroyed, memory freed automatically — RAII!
```

### Smart Pointers (C++11) — RAII Built-In

Modern C++ uses **smart pointers** from `<memory>`. You should almost never use raw `new`/`delete` in modern C++.

#### `unique_ptr` — Exclusive Ownership

```cpp
#include <memory>

auto p = make_unique<int>(42);       // allocates int(42)
cout << *p << "\n";                  // 42
// p goes out of scope → automatically deleted — no leak!

// Cannot copy unique_ptr (exclusive ownership)
// auto p2 = p;      // ❌ ERROR
auto p2 = move(p);   // ✅ transfer ownership — p becomes nullptr
```

#### `shared_ptr` — Shared Ownership (Reference Counted)

```cpp
auto sp1 = make_shared<string>("Hello");   // ref count = 1
{
    auto sp2 = sp1;                        // ref count = 2 (shallow copy!)
    cout << *sp2 << "\n";
}   // sp2 destroyed → ref count = 1 (not deleted yet)
// sp1 destroyed → ref count = 0 → string deleted
```

#### `weak_ptr` — Non-Owning Observer (Breaks Cycles)

```cpp
shared_ptr<int> sp = make_shared<int>(10);
weak_ptr<int>   wp = sp;    // doesn't increase ref count

if (auto locked = wp.lock()) {  // safely get shared_ptr if still alive
    cout << *locked << "\n";
}
```

### When to Use Which

| Smart Pointer | When | Ownership |
|---|---|---|
| `unique_ptr` | Single owner; most common case | Exclusive |
| `shared_ptr` | Multiple owners need same object | Shared (ref-counted) |
| `weak_ptr` | Observe without owning; break cycles | None |
| Raw pointer | Non-owning reference into existing memory | None |

---

## 18. Copy Constructor and Assignment Operator

### The Rule of Three (pre-C++11)

If a class manages a resource (heap memory, file handle), you must define all three:
1. **Copy constructor**
2. **Copy assignment operator**
3. **Destructor**

If you let the compiler generate any of these, it does a **shallow copy** — copying the pointer, not the data it points to. Two objects then own the same memory → double-delete.

```cpp
class DynamicArray {
    int* data;
    size_t size;

public:
    // Constructor
    DynamicArray(size_t n) : size(n), data(new int[n]()) {}

    // Destructor
    ~DynamicArray() { delete[] data; }

    // Copy Constructor — deep copy
    DynamicArray(const DynamicArray& other)
        : size(other.size), data(new int[other.size]) {
        copy(other.data, other.data + other.size, data);
        cout << "Copy constructor called\n";
    }

    // Copy Assignment Operator
    DynamicArray& operator=(const DynamicArray& other) {
        if (this == &other) return *this;   // self-assignment check

        delete[] data;                      // free existing resource
        size = other.size;
        data = new int[size];
        copy(other.data, other.data + size, data);
        cout << "Copy assignment called\n";
        return *this;
    }

    int& operator[](size_t i) { return data[i]; }
};

DynamicArray a(5);
a[0] = 10;

DynamicArray b = a;    // copy constructor — b has its own copy of data
DynamicArray c(3);
c = a;                  // copy assignment — c gets fresh copy of a's data
```

### The Rule of Five (C++11)

With move semantics (Section 19), if you define any of the three, also define:
4. **Move constructor**
5. **Move assignment operator**

### The Rule of Zero

**Best rule:** Design classes so they don't manage raw resources at all — use RAII types (`unique_ptr`, `vector`, `string`). Then the compiler-generated defaults work perfectly.

```cpp
class Employee {
    string name;                    // string manages its own memory
    vector<string> skills;          // vector manages its own memory
    unique_ptr<Address> address;    // unique_ptr manages the Address

public:
    Employee(string n) : name(move(n)) {}
    // No destructor, copy, or move needed — Rule of Zero!
    // All members correctly handle their own resources
};
```

---

## 19. Move Semantics and Rvalue References (C++11)

### The Problem: Unnecessary Copies

Before C++11, returning a large object from a function meant copying it — expensive!

```cpp
vector<int> createBigVector() {
    vector<int> v(1000000);
    // ... fill it ...
    return v;   // Pre-C++11: copies 1 million ints — expensive!
}

vector<int> result = createBigVector();  // another copy!
```

### Lvalue vs Rvalue

| Concept | Definition | Example |
|---|---|---|
| **Lvalue** | Has a persistent name and address — can appear on left of `=` | `int x = 5;` — `x` is lvalue |
| **Rvalue** | Temporary, no persistent address — can only appear on right of `=` | `5`, `x + y`, return value of function |
| **Lvalue reference** | `T&` — binds to lvalues | `int& ref = x;` |
| **Rvalue reference** | `T&&` — binds to rvalues (C++11) | `int&& rref = 5;` |

### Move Constructor and Move Assignment

Instead of copying a temporary's resources, **steal** them:

```cpp
class DynamicArray {
    int* data;
    size_t size;

public:
    DynamicArray(size_t n) : size(n), data(new int[n]()) {}
    ~DynamicArray() { delete[] data; }

    // Move Constructor — steals data from temporary
    DynamicArray(DynamicArray&& other) noexcept
        : size(other.size), data(other.data) {
        other.data = nullptr;   // leave source in valid (empty) state
        other.size = 0;
        cout << "Move constructor called\n";
    }

    // Move Assignment Operator
    DynamicArray& operator=(DynamicArray&& other) noexcept {
        if (this == &other) return *this;
        delete[] data;           // free current resource
        data = other.data;       // steal other's resource
        size = other.size;
        other.data = nullptr;    // leave source valid
        other.size = 0;
        cout << "Move assignment called\n";
        return *this;
    }
};

DynamicArray a(1000000);
DynamicArray b = move(a);   // move constructor — no copy, just pointer transfer!
// a.data is now nullptr, b.data has the data
```

### `std::move` — Casting to Rvalue

`std::move` doesn't move anything — it **casts** an lvalue to an rvalue reference, enabling the move constructor/assignment to be selected:

```cpp
string s1 = "Hello";
string s2 = move(s1);    // s1's internals transferred to s2
// s1 is now in a valid but unspecified state (likely empty)
cout << s2 << "\n";       // "Hello"
cout << s1.empty() << "\n"; // likely 1 (true)
```

### Return Value Optimization (RVO/NRVO)

The compiler often eliminates copies entirely when returning objects — constructs the return value directly in the caller's memory. C++17 guarantees this in many cases (mandatory copy elision).

```cpp
vector<int> makeVector() {
    return vector<int>(1000000);  // No copy — RVO constructs in caller's space
}
```

---

## 20. Multiple Inheritance and the Diamond Problem

### Multiple Inheritance — C++ Supports It

```cpp
class Flyable {
public:
    void fly()  { cout << "Flying\n"; }
    virtual ~Flyable() {}
};

class Swimmable {
public:
    void swim() { cout << "Swimming\n"; }
    virtual ~Swimmable() {}
};

class Duck : public Flyable, public Swimmable {
public:
    void quack() { cout << "Quack!\n"; }
};

Duck d;
d.fly();    // from Flyable
d.swim();   // from Swimmable
d.quack();  // Duck's own
```

### The Diamond Problem

When two base classes both inherit from the same grandparent, and a child class inherits from both bases, the grandparent's members appear **twice** in the child — causing ambiguity and wasted memory.

```
        Animal (has: name, eat())
       /       \
    Flyable   Swimmable         Both inherit Animal
       \       /
         Duck                  Duck has TWO copies of Animal!
```

```cpp
class Animal {
public:
    string name;
    void eat() { cout << name << " eats\n"; }
};

class Flyable  : public Animal { public: void fly()  {} };
class Swimmable: public Animal { public: void swim() {} };

class Duck : public Flyable, public Swimmable { };

Duck d;
d.name = "Donald";   // ❌ AMBIGUOUS — which name? Flyable::name or Swimmable::name?
d.eat();             // ❌ AMBIGUOUS — which eat()?
d.Flyable::eat();    // ✅ explicit disambiguation (ugly!)
```

### Solution: Virtual Inheritance

Declare the shared base as `virtual`. Virtual inheritance ensures **only one copy** of the base class exists in the hierarchy.

```cpp
class Animal {
public:
    string name;
    void eat() { cout << name << " eats\n"; }
};

class Flyable   : virtual public Animal { public: void fly()  {} };
class Swimmable : virtual public Animal { public: void swim() {} };

class Duck : public Flyable, public Swimmable { };

Duck d;
d.name = "Donald";   // ✅ No ambiguity — one shared Animal
d.eat();             // ✅ One eat()
d.fly();
d.swim();
```

### How Virtual Inheritance Works Internally

With virtual inheritance, each virtually-inherited base has a **virtual base pointer (vbptr)** — a pointer to the shared base subobject. The shared base is placed at the end of the object layout and accessed through the pointer.

```
Duck's memory layout (virtual inheritance):
┌──────────────┐
│ Flyable part │ (vbptr → shared Animal)
├──────────────┤
│ Swimmable part│ (vbptr → shared Animal)
├──────────────┤
│ Duck-specific │
├──────────────┤
│ Animal (shared)│ ← ONE copy at the end
└──────────────┘
```

### Constructor Call Order with Virtual Inheritance

```cpp
class Animal {
public:
    Animal(string n) { cout << "Animal(" << n << ")\n"; }
};

class Flyable   : virtual public Animal {
public:
    Flyable() : Animal("from Flyable") {}
};

class Swimmable : virtual public Animal {
public:
    Swimmable() : Animal("from Swimmable") {}
};

class Duck : public Flyable, public Swimmable {
public:
    Duck() : Animal("from Duck"), Flyable(), Swimmable() {}
    // The MOST DERIVED class (Duck) is responsible for calling Animal's constructor
};

Duck d;
// Output: Animal(from Duck) — only called once, by Duck directly!
```

> 🎯 **Key rule with virtual inheritance:** The **most-derived class** is responsible for constructing the virtual base. The intermediate classes' calls to the virtual base constructor are **ignored**.

---

## 21. Templates — Generic Programming in C++

### What Templates Are

Templates allow you to write **type-independent code**. Instead of writing `max(int, int)`, `max(double, double)`, `max(string, string)` separately, you write it once with a template parameter.

C++ templates are resolved entirely at **compile time** — no runtime overhead.

### Function Templates

```cpp
// Generic max function
template <typename T>
T maxOf(T a, T b) {
    return (a > b) ? a : b;
}

cout << maxOf(3, 5)        << "\n";  // int version generated
cout << maxOf(3.14, 2.72)  << "\n";  // double version generated
cout << maxOf('a', 'z')    << "\n";  // char version generated
// Compiler generates separate code for each type — called template instantiation
```

### Class Templates

```cpp
template <typename T>
class Stack {
    vector<T> data;

public:
    void push(const T& item) { data.push_back(item); }

    T pop() {
        if (data.empty()) throw underflow_error("Stack is empty");
        T top = data.back();
        data.pop_back();
        return top;
    }

    T& top() {
        if (data.empty()) throw underflow_error("Stack is empty");
        return data.back();
    }

    bool empty() const { return data.empty(); }
    size_t size() const { return data.size(); }
};

Stack<int>    intStack;
Stack<string> strStack;
Stack<double> dblStack;

intStack.push(10);
strStack.push("Hello");
```

### Template Specialization

Provide a specific implementation for a particular type:

```cpp
// Generic print
template <typename T>
void print(T val) {
    cout << val << "\n";
}

// Specialization for bool — print "true"/"false" instead of 1/0
template <>
void print<bool>(bool val) {
    cout << (val ? "true" : "false") << "\n";
}

print(42);       // generic: "42"
print(true);     // specialized: "true"
```

### Multiple Template Parameters

```cpp
template <typename K, typename V>
class Pair {
public:
    K key;
    V value;
    Pair(K k, V v) : key(k), value(v) {}
    void print() const { cout << key << " : " << value << "\n"; }
};

Pair<string, int> p("age", 25);
p.print();   // age : 25
```

### Non-Type Template Parameters

```cpp
template <typename T, size_t N>
class FixedArray {
    T data[N];   // compile-time size!
public:
    T& operator[](size_t i) { return data[i]; }
    size_t size() const { return N; }
};

FixedArray<int, 10> arr;    // 10-element int array, stack-allocated
FixedArray<double, 3> vec;  // 3-element double array
```

> 🎯 **Interview insight:** Templates in C++ are the foundation of the Standard Template Library (STL) — `vector<T>`, `map<K,V>`, `sort()`, etc. are all templates. Understanding templates shows you understand modern C++.

---

## 22. Object Relationships — Association, Aggregation, Composition

### Association — "Uses" Relationship

Objects interact but are **independently created and destroyed**. Neither owns the other.

```cpp
class Teacher {
    string name;
public:
    Teacher(string n) : name(n) {}
    void teach(const Student& s) const;  // uses Student but doesn't own
};

class Student {
    string name;
public:
    Student(string n) : name(n) {}
    string getName() const { return name; }
};

void Teacher::teach(const Student& s) const {
    cout << name << " teaches " << s.getName() << "\n";
}
```

### Aggregation — Weak HAS-A

Part can **exist independently** of the whole. Whole doesn't manage the part's lifecycle. Typically, the part is **passed in** (not created inside).

```cpp
class Department {
    string name;
    vector<Employee*> employees;   // pointers to external Employee objects

public:
    Department(string n) : name(n) {}

    void addEmployee(Employee* e) {
        employees.push_back(e);    // takes pointer — Employee created outside
    }

    // Department destroyed → employees NOT destroyed (they live on)
};

Employee e1("Alice"), e2("Bob");   // live independently
{
    Department dept("Engineering");
    dept.addEmployee(&e1);
    dept.addEmployee(&e2);
}   // dept destroyed — e1 and e2 still alive!
```

### Composition — Strong HAS-A

Part **cannot exist independently**. Whole creates and destroys the part.

```cpp
class Engine {
    int horsepower;
public:
    Engine(int hp) : horsepower(hp) {}
    void start() { cout << "Engine started (" << horsepower << "hp)\n"; }
};

class Car {
    Engine engine;     // Engine is a VALUE member — created and destroyed with Car
    string model;

public:
    Car(string m, int hp) : model(m), engine(hp) {}  // Engine constructed here

    void drive() {
        engine.start();
        cout << model << " is moving\n";
    }
    // Car destroyed → Engine destroyed (it's a member, not a pointer)
};

Car c("Toyota", 150);   // Engine(150) also created
c.drive();
// c goes out of scope → Car + Engine both destroyed
```

### Using Smart Pointers to Express Relationships

```cpp
// Composition with unique_ptr — Car owns Heart, Heart dies with Car
class Human {
    unique_ptr<Heart> heart;   // unique — only Human owns this Heart

public:
    Human() : heart(make_unique<Heart>()) {}
    // Heart destroyed when Human is destroyed — composition
};

// Aggregation with shared_ptr — shared ownership
class Team {
    vector<shared_ptr<Player>> players;   // team doesn't exclusively own players

public:
    void addPlayer(shared_ptr<Player> p) { players.push_back(p); }
    // Players can be shared with other teams, survive team destruction
};
```

### Comparison

| Feature | Association | Aggregation | Composition |
|---|---|---|---|
| **Relationship** | Uses/knows | Weak HAS-A | Strong HAS-A |
| **Lifecycle** | Independent | Independent | Dependent |
| **Ownership** | No | Partial (shared) | Full |
| **C++ representation** | Passes by reference/pointer | Stores pointer (raw or shared_ptr) | Stores by value or unique_ptr |
| **Example** | Teacher–Student | Department–Employee | Car–Engine, Human–Heart |

---

## 23. SOLID Principles in C++

### S — Single Responsibility Principle

```cpp
// ❌ Violates SRP — Order does too much
class Order {
public:
    void calculateTotal() { }
    void printOrder()     { }   // printing — different responsibility
    void saveToDatabase() { }   // persistence — different responsibility
};

// ✅ Follows SRP — each class does one thing
class Order         { public: void calculateTotal() { } };
class OrderPrinter  { public: void print(const Order& o) { } };
class OrderRepository { public: void save(const Order& o) { } };
```

### O — Open/Closed Principle

```cpp
// ❌ Violates OCP — must modify every time a new shape is added
double totalArea(const vector<void*>& shapes, const vector<string>& types) {
    double total = 0;
    for (size_t i = 0; i < shapes.size(); i++) {
        if (types[i] == "circle")    total += /* ... */;
        else if (types[i] == "rect") total += /* ... */;
        // Must add new else-if for every new shape!
    }
    return total;
}

// ✅ Follows OCP — add new shapes without touching this code
class Shape { public: virtual double area() const = 0; virtual ~Shape(){} };
class Circle    : public Shape { public: double area() const override { return 3.14*r*r; } double r; };
class Rectangle : public Shape { public: double area() const override { return w*h; } double w,h; };
class Triangle  : public Shape { public: double area() const override { return 0.5*b*h; } double b,h; };

double totalArea(const vector<unique_ptr<Shape>>& shapes) {
    double total = 0;
    for (const auto& s : shapes) total += s->area();  // works for any Shape
    return total;
}
```

### L — Liskov Substitution Principle

```cpp
// ❌ Violates LSP — Square breaks Rectangle's postconditions
class Rectangle {
protected:
    double w, h;
public:
    virtual void setWidth (double width)  { w = width; }
    virtual void setHeight(double height) { h = height; }
    double area() const { return w * h; }
};

class Square : public Rectangle {
public:
    void setWidth (double side) override { w = h = side; }
    void setHeight(double side) override { w = h = side; }
};

// Code that works for Rectangle but FAILS for Square
void testRectangle(Rectangle& r) {
    r.setWidth(4);
    r.setHeight(5);
    assert(r.area() == 20);  // ❌ Fails for Square — area is 25!
}

// ✅ Fix: don't inherit Square from Rectangle
class Shape       { public: virtual double area() const = 0; };
class Rectangle   : public Shape { double w,h; public: double area() const override{return w*h;} };
class Square      : public Shape { double s;   public: double area() const override{return s*s;} };
```

### I — Interface Segregation Principle

```cpp
// ❌ Fat interface forces irrelevant implementations
class Machine {
public:
    virtual void print()  = 0;
    virtual void scan()   = 0;
    virtual void fax()    = 0;   // old printer can't fax!
    virtual ~Machine() {}
};

class OldPrinter : public Machine {
public:
    void print() override { /* OK */ }
    void scan()  override { /* ??? can't scan */ }
    void fax()   override { /* ??? can't fax  */ }
};

// ✅ Segregated interfaces — implement only what's relevant
class Printable { public: virtual void print() = 0; virtual ~Printable(){} };
class Scannable { public: virtual void scan()  = 0; virtual ~Scannable(){} };
class Faxable   { public: virtual void fax()   = 0; virtual ~Faxable(){}  };

class OldPrinter   : public Printable { public: void print() override {} };
class MultiFunPrinter : public Printable, public Scannable, public Faxable {
    void print() override {} void scan() override {} void fax() override {}
};
```

### D — Dependency Inversion Principle

```cpp
// ❌ High-level depends on low-level concrete class
class MySQLDatabase {
public:
    void save(const string& data) { cout << "MySQL: " << data << "\n"; }
};

class UserRepository {
    MySQLDatabase db;   // tightly coupled to MySQL!
public:
    void save(const string& user) { db.save(user); }
};

// ✅ Both depend on abstraction
class IDatabase {
public:
    virtual void save(const string& data) = 0;
    virtual ~IDatabase() {}
};

class MySQLDatabase  : public IDatabase { void save(const string& d) override { /*MySQL*/ } };
class PostgresDB     : public IDatabase { void save(const string& d) override { /*PG*/   } };

class UserRepository {
    unique_ptr<IDatabase> db;   // depends on abstraction!

public:
    UserRepository(unique_ptr<IDatabase> database)
        : db(move(database)) {}

    void save(const string& user) { db->save(user); }
};

// Inject any database at runtime
auto repo = UserRepository(make_unique<PostgresDB>());
```

---

## 24. Design Patterns in C++

### Singleton

```cpp
class Logger {
    static Logger* instance;
    ofstream logFile;

    Logger() : logFile("app.log") {}   // private constructor

public:
    // Thread-safe C++11 version — static local variable
    static Logger& getInstance() {
        static Logger instance;        // initialized once, thread-safe in C++11
        return instance;
    }

    void log(const string& msg) {
        logFile << msg << "\n";
        cout << "[LOG] " << msg << "\n";
    }

    // Prevent copy and assignment
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;
};

Logger::getInstance().log("Application started");
```

### Factory Method

```cpp
class Animal {
public:
    virtual void speak() const = 0;
    virtual ~Animal() {}
};

class Dog : public Animal { public: void speak() const override { cout << "Woof\n"; } };
class Cat : public Animal { public: void speak() const override { cout << "Meow\n"; } };

class AnimalFactory {
public:
    static unique_ptr<Animal> create(const string& type) {
        if (type == "dog") return make_unique<Dog>();
        if (type == "cat") return make_unique<Cat>();
        throw invalid_argument("Unknown animal: " + type);
    }
};

auto a = AnimalFactory::create("dog");
a->speak();   // Woof
```

### Builder

```cpp
class Pizza {
    string size, crust, sauce;
    vector<string> toppings;

    Pizza() {}

public:
    class Builder {
        Pizza pizza;
    public:
        Builder& size(string s)    { pizza.size = s;            return *this; }
        Builder& crust(string c)   { pizza.crust = c;           return *this; }
        Builder& sauce(string s)   { pizza.sauce = s;           return *this; }
        Builder& topping(string t) { pizza.toppings.push_back(t); return *this; }

        Pizza build() { return pizza; }
    };

    void print() const {
        cout << size << " pizza, " << crust << " crust, "
             << sauce << " sauce, toppings: ";
        for (const auto& t : toppings) cout << t << " ";
        cout << "\n";
    }
};

Pizza p = Pizza::Builder()
    .size("Large")
    .crust("Thin")
    .sauce("Tomato")
    .topping("Cheese")
    .topping("Pepperoni")
    .build();
p.print();
```

### Observer

```cpp
#include <functional>
#include <vector>

class EventEmitter {
    vector<function<void(const string&)>> listeners;

public:
    void on(function<void(const string&)> callback) {
        listeners.push_back(callback);
    }

    void emit(const string& event) {
        for (const auto& cb : listeners) cb(event);
    }
};

EventEmitter emitter;
emitter.on([](const string& e) { cout << "Logger: " << e << "\n"; });
emitter.on([](const string& e) { cout << "Analytics: " << e << "\n"; });
emitter.emit("UserLoggedIn");
```

### Strategy

```cpp
class SortStrategy {
public:
    virtual void sort(vector<int>& data) = 0;
    virtual ~SortStrategy() {}
};

class BubbleSort : public SortStrategy {
public:
    void sort(vector<int>& data) override { /* bubble sort */ cout << "Bubble sort\n"; }
};

class QuickSort : public SortStrategy {
public:
    void sort(vector<int>& data) override { /* quick sort */  cout << "Quick sort\n"; }
};

class Sorter {
    unique_ptr<SortStrategy> strategy;
public:
    Sorter(unique_ptr<SortStrategy> s) : strategy(move(s)) {}
    void setStrategy(unique_ptr<SortStrategy> s) { strategy = move(s); }
    void sort(vector<int>& data) { strategy->sort(data); }
};

vector<int> data = {5,3,1,4,2};
Sorter s(make_unique<QuickSort>());
s.sort(data);
s.setStrategy(make_unique<BubbleSort>());  // swap strategy at runtime
s.sort(data);
```

---

## 25. Exception Handling in C++

### Basic Syntax

```cpp
#include <stdexcept>

double divide(double a, double b) {
    if (b == 0) throw invalid_argument("Division by zero");
    return a / b;
}

int main() {
    try {
        double r = divide(10.0, 0.0);
    }
    catch (const invalid_argument& e) {
        cerr << "Error: " << e.what() << "\n";
    }
    catch (const exception& e) {
        cerr << "General error: " << e.what() << "\n";
    }
    catch (...) {          // catches anything
        cerr << "Unknown error\n";
    }
}
```

### Exception Hierarchy in C++

```
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::out_of_range
│   ├── std::domain_error
│   └── std::length_error
└── std::runtime_error
    ├── std::overflow_error
    ├── std::underflow_error
    └── std::range_error
```

### Custom Exceptions

```cpp
class InsufficientFundsException : public runtime_error {
    double amount;
public:
    InsufficientFundsException(double amt)
        : runtime_error("Insufficient funds: need " + to_string(amt)),
          amount(amt) {}

    double getAmount() const { return amount; }
};

class BankAccount {
    double balance;
public:
    BankAccount(double b) : balance(b) {}

    void withdraw(double amount) {
        if (amount > balance)
            throw InsufficientFundsException(amount - balance);
        balance -= amount;
    }
};

try {
    BankAccount acc(100.0);
    acc.withdraw(150.0);
}
catch (const InsufficientFundsException& e) {
    cout << e.what() << "\n";
    cout << "Short by: " << e.getAmount() << "\n";
}
```

### `noexcept` Specifier

Declare that a function will NOT throw exceptions — enables compiler optimizations:

```cpp
void swap(int& a, int& b) noexcept {    // guaranteed no exceptions
    int temp = a; a = b; b = temp;
}

// Move constructors should almost always be noexcept
DynamicArray(DynamicArray&& other) noexcept : data(other.data), size(other.size) {
    other.data = nullptr;
}
```

> ⚠️ If a `noexcept` function throws, `std::terminate()` is called immediately.

### RAII and Exception Safety

The great power of RAII: resources are cleaned up **even when exceptions are thrown** — because destructors always run on scope exit, whether normal or exceptional.

```cpp
void processFile(const string& path) {
    ifstream file(path);    // RAII — file opened in constructor
    if (!file) throw runtime_error("Cannot open: " + path);

    // ... process file ...

    throw runtime_error("Something went wrong");

    // file destructor called automatically — file closed even after exception!
}
```

---

## 26. Common Interview Traps

---

### ❌ Trap 1: "Non-virtual functions are always the base class version"
✅ **Correct:** Yes — without `virtual`, calling through a base pointer always calls the **base class version** (static binding). This is the most important difference between C++ and Java. In Java, all methods are virtual by default. In C++, you must explicitly write `virtual`.

```cpp
Animal* a = new Dog();
a->sound();   // NON-virtual → Animal::sound() — NOT Dog's!
a->speak();   // virtual → Dog::speak() — correctly dispatched!
```

---

### ❌ Trap 2: "Destructor is only needed when you dynamically allocate"
✅ **Correct (partially):** You need a destructor for any resource (memory, file handles, mutex locks, sockets). More importantly: **if your class is a base class**, you **must** have a `virtual` destructor — even if your class doesn't manage any resource — because derived classes might.

---

### ❌ Trap 3: "Copy constructor is called for assignment"
✅ **Correct:** The **copy constructor** is called for initialization (`A b = a`). The **copy assignment operator** is called for assignment to an already-constructed object (`b = a` where `b` was already declared). These are two different operations and two different functions.

```cpp
MyClass a;
MyClass b = a;   // copy CONSTRUCTOR (b is being constructed)
MyClass c;
c = a;           // copy ASSIGNMENT OPERATOR (c already exists)
```

---

### ❌ Trap 4: "struct and class are totally different in C++"
✅ **Correct:** In C++, `struct` and `class` are **nearly identical** — the only differences are:
1. Default member access: `struct` → `public`, `class` → `private`
2. Default inheritance: `struct : Base` → `public`, `class : Base` → `private`

Both can have methods, constructors, destructors, inheritance, and virtual functions.

---

### ❌ Trap 5: "Virtual functions have the same performance as regular functions"
✅ **Correct:** Virtual function calls go through the **vtable** — an extra pointer dereference and memory lookup. In tight loops or on embedded systems, this can matter. Non-virtual calls are resolved at compile time — zero overhead. This is why C++ makes you opt-in to `virtual` rather than having it by default.

---

### ❌ Trap 6: "Multiple inheritance always causes the diamond problem"
✅ **Correct:** The diamond problem only occurs when two base classes both inherit from the **same grandparent**. Simple multiple inheritance (two unrelated base classes) is safe and common. The fix when diamond does occur is **virtual inheritance**.

---

### ❌ Trap 7: "Friend functions break encapsulation"
✅ **Correct (nuanced):** `friend` *selectively* breaks encapsulation — granting specific external code access to private members. This is intentional and controlled. It's better than making members `public` (which gives *all* code access). Used correctly for operator overloading (`<<`, `>>`) and tightly-coupled utility functions, `friend` is a legitimate tool.

---

### ❌ Trap 8: "Delete is the same as delete[]"
✅ **Correct:** `delete` frees a single object. `delete[]` frees an array. Using `delete` on an array (`new T[N]`) is **undefined behavior** — only the first element's destructor runs, and the memory deallocation may be wrong. Always match `new[]` with `delete[]`.

```cpp
int* p   = new int(5);    // single object
int* arr = new int[10];   // array

delete p;     // ✅ correct
delete[] arr; // ✅ correct
delete arr;   // ❌ UB — wrong operator
```

---

### ❌ Trap 9: "You can override a function without virtual"
✅ **Correct:** If you define a function with the same name in a derived class **without virtual** in the base, it **hides** the base function (not overrides). Through a base pointer, the base version is called. This is called **function hiding** or **name hiding**. The `override` keyword catches this at compile time.

```cpp
class Base  { void show() { cout << "Base\n"; } };
class Derived : public Base { void show() { cout << "Derived\n"; } }; // hides, not overrides

Base* b = new Derived();
b->show();   // "Base" — NOT "Derived"! No virtual, no dynamic dispatch.
```

---

### ❌ Trap 10: "`const` after a function name is the same as `const` before it"
✅ **Correct:**
- `const` **before** return type: the returned value is const
- `const` **after** function name: the function is a const member — promises not to modify `*this`

```cpp
const string  getName()  { return name; }  // returns a const string
string        getName() const { return name; }  // const member function — can't modify object
const string  getName() const { return name; }  // both
```

---

### ❌ Trap 11: "Abstract classes can't be used at all"
✅ **Correct:** Abstract classes **cannot be instantiated**, but:
- ✅ You CAN declare **pointers and references** to abstract classes (used for polymorphism)
- ✅ They CAN have **constructors** (called by derived class via initializer list)
- ✅ They CAN have **non-pure (concrete) methods**
- ✅ They CAN have **data members**

---

### ❌ Trap 12: "Operator overloading can change an operator's meaning completely"
✅ **Correct:** While you can technically make `+` do subtraction, it's terrible practice. Operator overloading should **preserve the intuitive meaning** of the operator for your type. `+` on a vector should add, `==` should test equality, `<<` should stream output.

---

## 27. Scenario and Coding Questions

### Scenario 1: Why does this print "Base" instead of "Derived"?

```cpp
class Base {
public:
    void display() { cout << "Base\n"; }   // NOT virtual!
    virtual ~Base() {}
};

class Derived : public Base {
public:
    void display() { cout << "Derived\n"; }
};

Base* b = new Derived();
b->display();    // prints "Base" — WHY?
delete b;
```

**Answer:** `display()` is not `virtual`. Without `virtual`, C++ uses **static binding** — the compiler resolves the call at compile time based on the **pointer type** (`Base*`), not the **object type** (`Derived`). Add `virtual` to `Base::display()` and you'll get "Derived".

---

### Scenario 2: What's wrong with this code?

```cpp
class Animal {
public:
    Animal() { cout << "Animal created\n"; }
    ~Animal() { cout << "Animal destroyed\n"; }   // NOT virtual!
};

class Dog : public Animal {
    int* data;
public:
    Dog() : data(new int[100]) { cout << "Dog created\n"; }
    ~Dog() { delete[] data; cout << "Dog destroyed\n"; }
};

Animal* a = new Dog();
delete a;
```

**Answer:** Non-virtual destructor in the base class. `delete a` calls `Animal::~Animal()` only — `Dog::~Dog()` is **never called**. The `int[100]` is never freed → **memory leak**. Fix: make `~Animal()` virtual.

---

### Scenario 3: Implement a thread-safe Singleton in C++

```cpp
class Singleton {
    Singleton() = default;

public:
    // Meyers Singleton — thread-safe in C++11 (guaranteed by standard)
    static Singleton& getInstance() {
        static Singleton instance;   // initialized once, thread-safe
        return instance;
    }

    Singleton(const Singleton&) = delete;            // no copy
    Singleton& operator=(const Singleton&) = delete; // no assignment

    void doWork() { cout << "Working\n"; }
};

Singleton::getInstance().doWork();
```

---

### Scenario 4: Implement a generic Stack

```cpp
template <typename T>
class Stack {
    vector<T> data;

public:
    void push(T item)  { data.push_back(move(item)); }  // move for efficiency
    void pop()         {
        if (empty()) throw underflow_error("Stack underflow");
        data.pop_back();
    }
    T&   top()         {
        if (empty()) throw underflow_error("Stack is empty");
        return data.back();
    }
    bool empty() const { return data.empty(); }
    size_t size() const { return data.size(); }
};

Stack<int>    s1;
s1.push(1); s1.push(2); s1.push(3);
cout << s1.top() << "\n";  // 3
s1.pop();
cout << s1.size() << "\n"; // 2
```

---

### Scenario 5: Overload `<<` for a custom class

```cpp
class Point {
    double x, y;
public:
    Point(double x, double y) : x(x), y(y) {}

    // friend because operator<< is not a member, needs private access
    friend ostream& operator<<(ostream& os, const Point& p) {
        os << "(" << p.x << ", " << p.y << ")";
        return os;   // return ostream for chaining
    }

    friend istream& operator>>(istream& is, Point& p) {
        is >> p.x >> p.y;
        return is;
    }
};

Point p(3.0, 4.0);
cout << "Point: " << p << "\n";         // Point: (3, 4)
cout << p << " is a point\n";           // (3, 4) is a point
```

---

### Scenario 6: Design a Shape Hierarchy

```cpp
class Shape {
protected:
    string color;
public:
    Shape(string c = "black") : color(c) {}
    virtual double area()      const = 0;
    virtual double perimeter() const = 0;
    virtual void   draw()      const = 0;
    virtual string type()      const = 0;
    virtual ~Shape() {}

    void printInfo() const {
        cout << type() << " [" << color << "] "
             << "Area=" << area()
             << " Perimeter=" << perimeter() << "\n";
    }
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r, string c = "black") : Shape(c), radius(r) {}
    double area()      const override { return 3.14159 * radius * radius; }
    double perimeter() const override { return 2 * 3.14159 * radius; }
    void   draw()      const override { cout << "Drawing circle r=" << radius << "\n"; }
    string type()      const override { return "Circle"; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double w, double h, string c="black") : Shape(c), w(w), h(h) {}
    double area()      const override { return w * h; }
    double perimeter() const override { return 2*(w+h); }
    void   draw()      const override { cout << "Drawing rectangle " << w << "x" << h << "\n"; }
    string type()      const override { return "Rectangle"; }
};

// Polymorphic usage
vector<unique_ptr<Shape>> shapes;
shapes.push_back(make_unique<Circle>(5.0, "red"));
shapes.push_back(make_unique<Rectangle>(4.0, 6.0, "blue"));

for (const auto& s : shapes) {
    s->draw();
    s->printInfo();
}
```

---

## 28. Must-Know Essentials — Final Checklist

### Fundamentals

- [ ] Difference between `class` and `struct` in C++ (only default access)
- [ ] Stack vs heap allocation; `.` vs `->` operator
- [ ] Why C++ OOP is different from Java — no GC, no default virtual, manual memory
- [ ] What is `const` correctness; what is a `const` member function

### The Four Pillars in C++

- [ ] Encapsulation: `private` fields + public getters/setters; `const` methods
- [ ] Abstraction: pure virtual functions (`= 0`); header/source file separation
- [ ] Inheritance: `: public Base`; base constructor via initializer list
- [ ] Polymorphism: `virtual` keyword REQUIRED; vtable and vptr mechanism

### Constructors & Destructors

- [ ] 6 types of constructors (default, parameterized, copy, move, delegating, converting)
- [ ] When is each constructor called?
- [ ] Initializer list vs body assignment — why initializer list is better
- [ ] Why virtual destructor is MANDATORY in base classes used polymorphically
- [ ] Construction order: base first, derived last. Destruction: reverse.
- [ ] Rule of Three / Rule of Five / Rule of Zero

### Access Specifiers

- [ ] `private` (class only), `protected` (class + derived), `public` (everywhere)
- [ ] Inheritance access specifiers: `public`, `protected`, `private` and their effect on members

### Virtual Functions & Polymorphism

- [ ] `virtual` — enables dynamic dispatch; not virtual by default in C++
- [ ] `override` — compile-time verification; catches typos in overrides
- [ ] `final` — prevent further overriding or subclassing
- [ ] Pure virtual `= 0` — makes class abstract; cannot instantiate
- [ ] vtable: what it is, when it exists, how dynamic dispatch uses it
- [ ] Function hiding (same name, no `virtual`) vs overriding

### C++-Specific Features

- [ ] `friend` function / class — selective encapsulation bypass
- [ ] Operator overloading — which can/cannot be overloaded; member vs friend
- [ ] `static` data members — one per class; must define outside class
- [ ] `static` member functions — no `this`; accessed via `ClassName::`
- [ ] `mutable` — allow modification in `const` methods
- [ ] Templates — function templates, class templates, specialization

### Memory Management

- [ ] `new` / `delete` vs `new[]` / `delete[]` — never mix!
- [ ] RAII — resource tied to object lifetime; destructor guarantees cleanup
- [ ] `unique_ptr` — exclusive ownership; `shared_ptr` — shared (ref-counted); `weak_ptr` — non-owning
- [ ] Common memory bugs: leak, dangling pointer, double delete

### Copy & Move Semantics

- [ ] Shallow copy vs deep copy — why the difference matters for pointers
- [ ] Copy constructor vs copy assignment operator — when each is called
- [ ] Move constructor / move assignment — steal instead of copy
- [ ] `std::move` — casts lvalue to rvalue; doesn't actually move
- [ ] RVO/NRVO — compiler optimization that eliminates copies on return

### Multiple Inheritance

- [ ] Diamond problem — two bases share same grandparent → ambiguity + duplicate data
- [ ] `virtual` inheritance — `class B : virtual public A` — one shared copy of A
- [ ] Most-derived class calls virtual base constructor

### SOLID Principles

- [ ] S — Single responsibility; one reason to change
- [ ] O — Open for extension, closed for modification
- [ ] L — Derived must be substitutable for base without breaking behavior
- [ ] I — Don't force irrelevant method implementations
- [ ] D — Depend on abstractions (interfaces), not concrete classes

### Design Patterns

- [ ] Singleton — one instance, private constructor, static local variable (Meyers)
- [ ] Factory — hide creation behind a method, return abstract pointer
- [ ] Builder — step-by-step construction with method chaining
- [ ] Observer — publish/subscribe, `function<>` callbacks
- [ ] Strategy — swap algorithm via interface + `unique_ptr`

### Exception Handling

- [ ] `throw`, `try`, `catch`, `catch(...)` (catch-all)
- [ ] Custom exceptions inherit from `std::exception` or its subclasses
- [ ] `noexcept` — function promises no exceptions; enables optimizations
- [ ] RAII ensures cleanup even when exceptions are thrown

---

## 📚 Quick Reference: C++ OOP Interview One-Liners

| Question | One-liner Answer |
|---|---|
| What is OOP? | Paradigm organizing code around objects — data + behavior — using Encapsulation, Abstraction, Inheritance, Polymorphism |
| Class vs struct in C++? | Identical except: class defaults to private access/inheritance; struct defaults to public |
| Why `virtual` in C++? | Methods are NOT virtual by default; `virtual` enables runtime polymorphism via vtable |
| What is a vtable? | Per-class array of function pointers to virtual method implementations; vptr in each object points to it |
| `override` keyword? | Tells compiler this overrides a base virtual method; compile error if no matching virtual exists |
| Virtual destructor needed? | Yes — always in base classes; without it, deleting via base pointer skips derived destructor → leak |
| Pure virtual function? | `= 0` — no implementation in base; makes class abstract; derived must implement |
| Difference: function hiding vs overriding? | Hiding: same name, no virtual → base version always called via base pointer. Overriding: virtual → derived version called |
| What is `friend`? | Grants specific external function/class access to private members — selective, not general |
| When is copy constructor called? | On initialization from object: `A b = a`, pass-by-value, return-by-value |
| Copy vs move constructor? | Copy: duplicates resource (new allocation). Move: steals resource (pointer transfer) from temporary |
| `std::move` does what? | Casts lvalue to rvalue reference — enables move constructor/assignment to be selected |
| Rule of Three? | If you define destructor, copy constructor, or copy assignment — define all three |
| Rule of Zero? | Use RAII members (vector, unique_ptr, string) so compiler-generated defaults just work |
| RAII? | Resource Acquisition Is Initialization — tie resource lifetime to object; destructor always frees |
| `unique_ptr` vs `shared_ptr`? | unique_ptr: one owner, zero overhead. shared_ptr: multiple owners, ref-counted overhead |
| Diamond problem? | Ambiguity when two base classes share a grandparent — solved by virtual inheritance |
| Virtual inheritance? | `class B : virtual public A` — ensures one shared A subobject in diamond hierarchy |
| Operator overloading rules? | Can't create new operators; can't change precedence; at least one user-defined type operand |
| Template? | Compile-time generic code; separate instantiation for each type used |
| `const` member function? | Promises not to modify `*this`; `this` has type `const Class*`; callable on const objects |
| `mutable`? | Member that can be modified even in a `const` method (e.g., cache) |
| `noexcept`? | Function promises no exceptions; enables move optimizations; terminate() if violated |
| Singleton in C++11? | Static local variable in `getInstance()` — zero-overhead, thread-safe, lazy |

---

*📌 Remember: In a C++ interview, knowing **why** C++ makes certain choices (no default virtual, manual memory, `const` correctness) is what demonstrates deep understanding. C++ gives you power — interviewers want to see you understand the responsibility that comes with it. Good luck with your placements!*
