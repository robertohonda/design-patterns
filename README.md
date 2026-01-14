# Design Patterns
Design patterns are reusable solutions to commonly occurring software design problems.

Design patterns are categorized into three main groups:
- Creational
- Structural
- Behavioral

## 1. Creational Design Patterns
Creational patterns focus on object creation mechanisms, aiming to create objects in a flexible and reusable way.

### 1.1 Factory Method Pattern
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

### 1.2 Singleton Pattern
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

### 1.3 Builder Design Pattern

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

## 2. Structural Design Patterns
Structural patterns focus on how classes and objects are composed to form larger, flexible structures.

## 3. Behavioral Design Patterns
Behavioral patterns focus on communication between objects and the assignment of responsibilities.
