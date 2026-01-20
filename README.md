# Design Patterns
Design patterns are reusable solutions to commonly occurring software design problems.

Design patterns are categorized into three main groups:
- Creational
- Structural
- Behavioral

## 1. Creational Design Patterns
Creational patterns focus on object creation mechanisms, aiming to create objects in a flexible and reusable way.

### 1.1 Abstract Factory Design Pattern

The Abstract Factory Pattern provides an interface for creating **families of related objects**
without specifying their concrete implementations.

```TS
interface Button {
  render(): void;
}

interface Modal {
  render(): void;
}

class LightButton implements Button {
  render() {
    console.log("Render light button");
  }
}

class LightModal implements Modal {
  render() {
    console.log("Render light modal");
  }
}

class DarkButton implements Button {
  render() {
    console.log("Render dark button");
  }
}

class DarkModal implements Modal {
  render() {
    console.log("Render dark modal");
  }
}

interface UIFactory {
  createButton(): Button;
  createModal(): Modal;
}

class LightThemeFactory implements UIFactory {
  createButton(): Button {
    return new LightButton();
  }

  createModal(): Modal {
    return new LightModal();
  }
}

class DarkThemeFactory implements UIFactory {
  createButton(): Button {
    return new DarkButton();
  }

  createModal(): Modal {
    return new DarkModal();
  }
}

function renderUI(factory: UIFactory) {
  const button = factory.createButton();
  const modal = factory.createModal();

  button.render();
  modal.render();
}

const themeFactory: UIFactory =
  Math.random() > 0.5
    ? new LightThemeFactory()
    : new DarkThemeFactory();

renderUI(themeFactory);
```

### Pros

✅ Ensures compatible products  
✅ Easy to swap families    
✅ Strong adherence to DIP  
✅ Scales well for complex systems  

### Cons

❌ More abstraction  
❌ Harder to add new product types  
❌ Overkill for small apps  

### 1.2 Factory Method Pattern
Defines an interface for creating an object but lets subclasses alter the type of objects that will be created.

```TS
// 1️⃣ Common interface
interface Payment {
  pay(amount: number): void;
}

// 2️⃣ Concrete implementations
class PaypalPayment implements Payment {
  pay(amount: number): void {
    console.log(`Paying ${amount} with PayPal`);
  }
}

class CreditCardPayment implements Payment {
  pay(amount: number): void {
    console.log(`Paying ${amount} with Credit Card`);
  }
}

// 3️⃣ Allowed payment types
type PaymentType = "paypal" | "credit";

// 4️⃣ Factory
class PaymentFactory {
  private static registry: Record<PaymentType, new () => Payment> = {
    paypal: PaypalPayment,
    credit: CreditCardPayment,
  };

  static create(type: PaymentType): Payment {
    const PaymentClass = this.registry[type];
    return new PaymentClass();
  }
}

// 5️⃣ Client code
const payment = PaymentFactory.create("paypal");
payment.pay(100);
```

### Pros
✅ Decouples creation from usage  
✅ Centralized creation logic  
✅ Easier to test (mock Payment)  
✅ Supports SOLID principles  

### Cons
❌ Extra abstraction  
❌ Factory can grow too large  
❌ Simple cases don’t need it  

### 1.3 Singleton Pattern
Ensures that a class has only one instance and provides a global access point to it.

```TS
class AppConfig {
  private static instance: AppConfig;

  private constructor() {}

  static getInstance(): AppConfig {
    if (!AppConfig.instance) {
      AppConfig.instance = new AppConfig();
    }
    return AppConfig.instance;
  }
}

const configA = AppConfig.getInstance();
const configB = AppConfig.getInstance();

console.log(configA === configB);      // true
```

### Pros
✅ Guarantees a single shared instance  
✅ Consistent global state  
✅ Useful for configuration, logging, caching  

### Cons
❌ Acts like global state  
❌ Harder to mock and test  
❌ Overuse leads to tight coupling  

### 1.4 Builder Design Pattern

The Builder Pattern is used to construct complex objects step by step.  
It allows creating different representations of an object using the same construction process.

```TS
class User {
  constructor(
    public name: string,
    public age: number,
    public email: string,
    public phone?: string,
    public address?: string
  ) {}
}

class UserBuilder {
  private name!: string;
  private age!: number;
  private email!: string;
  private phone?: string;
  private address?: string;

  setName(name: string): this {
    this.name = name;
    return this;
  }

  setAge(age: number): this {
    this.age = age;
    return this;
  }

  setEmail(email: string): this {
    this.email = email;
    return this;
  }

  setPhone(phone: string): this {
    this.phone = phone;
    return this;
  }

  setAddress(address: string): this {
    this.address = address;
    return this;
  }

  build(): User {
    return new User(this.name, this.age, this.email, this.phone, this.address);
  }
}

const user = new UserBuilder()
  .setName("John")
  .setAge(45)
  .setEmail("john.doe@example.com")
  .setPhone("123-456")
  .setAddress("Washington")
  .build();
```

### Pros
✅ Easy to add new rules  
✅ Highly readable  

### Cons
❌ More classes  
❌ Overkill for simple validation  

### 1.5 Prototype Design Pattern
The Prototype Pattern creates new objects by **cloning an existing object** instead of creating them from scratch.

```TS
interface Prototype<T> {
  clone(): T;
}

class UserProfile implements Prototype<UserProfile> {
  constructor(
    public role: string,
    public permissions: string[],
    public theme: string
  ) {}

  clone(): UserProfile {
    return new UserProfile(
      this.role,
      [...this.permissions], // deep copy
      this.theme
    );
  }
}

const adminTemplate = new UserProfile(
  "admin",
  ["read", "write", "delete"],
  "dark"
);

const admin1 = adminTemplate.clone();
admin1.theme = "light";

const admin2 = adminTemplate.clone();

console.log(adminTemplate.theme); // dark
console.log(admin1.theme);        // light
console.log(admin2.theme);        // dark
```

### Pros
✅ Fast object creation  
✅ Avoids complex constructors  
✅ Reduces duplication  
✅ Good for templates  

### Cons
❌ Cloning logic can get complex  
❌ Deep copies must be handled carefully  
❌ Not ideal for very dynamic objects  
❌ Circular references might be very tricky

## 2. Structural Design Patterns
Structural patterns focus on how classes and objects are composed to form larger, flexible structures.

### 2.1 Decorator Design Pattern
The Decorator Pattern allows you to **add new behavior to an object dynamically**
without modifying its original class.

It works by **wrapping** an object and extending its behavior.

```TS
interface Notifier {
  send(message: string): void;
}

class EmailNotifier implements Notifier {
  send(message: string): void {
    console.log(`Email: ${message}`);
  }
}

class SMSDecorator implements Notifier {
  constructor(private notifier: Notifier) {}

  send(message: string): void {
    this.notifier.send(message);
    console.log(`SMS: ${message}`);
  }
}

class PushDecorator implements Notifier {
  constructor(private notifier: Notifier) {}

  send(message: string): void {
    this.notifier.send(message);
    console.log(`Push: ${message}`);
  }
}

const notifier =
  new PushDecorator(
    new SMSDecorator(
      new EmailNotifier()
    )
  );

notifier.send("Order shipped");
```

### Pros
✅ Open/Closed Principle  
✅ Flexible combinations  
✅ Single Responsibility  
✅ Reusable behavior  

### Cons
❌ Many small classes  
❌ Debugging can be harder  
❌ Order of decorators matters  

### 2.2 Adapter Design Pattern
The Adapter Pattern allows **objects with incompatible interfaces** to work together
by converting one interface into another that the client expects.

```TS
class LegacyLogger {
  logMessage(msg: string) {
    console.log("LEGACY:", msg);
  }
}

interface Logger {
  log(message: string): void;
}

class LoggerAdapter implements Logger {
  constructor(private legacy: LegacyLogger) {}

  log(message: string): void {
    this.legacy.logMessage(message);
  }
}
```

### Pros
✅ Allows reuse of existing code  
✅ Decouples client from third-party APIs  
✅ Follows Open/Closed Principle  

### Cons
❌ Extra layer of abstraction  
❌ Can hide complexity  

## 3. Behavioral Design Patterns
Behavioral patterns focus on communication between objects and the assignment of responsibilities.
