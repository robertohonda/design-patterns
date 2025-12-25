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

## 2. Structural Design Patterns
Structural patterns focus on how classes and objects are composed to form larger, flexible structures.

## 3. Behavioral Design Patterns
Behavioral patterns focus on communication between objects and the assignment of responsibilities.
