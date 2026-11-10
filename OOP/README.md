# TypeScript OOP Interview Guide --- Banking Domain

**Audience:** Senior Software Engineer / Tech Lead\
**Language:** TypeScript\
**Examples:** Customers, accounts, transfers, payments and
notifications.

## How to answer OOP questions

Use this pattern: **Definition → Problem → Code → Banking example.**
Explain why the design is useful, not just what the keyword means.

# 1. What is OOP?

**Easy definition:** Object-Oriented Programming organises software
around objects that contain data (properties) and behaviour (methods).

A bank account has data such as account number and balance, and
behaviour such as deposit, withdraw and balance enquiry.

``` ts
class BankAccount {
  constructor(
    public readonly accountNumber: string,
    private balance: number
  ) {}

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount("ACC1001", 5000);
account.deposit(1000);
console.log(account.getBalance()); // 6000
```

**Advantages:** Organises business logic, supports reuse, protects
state, and can improve testability.

**Drawbacks:** Too many classes or deep inheritance hierarchies can make
a simple system unnecessarily complex.

**Interview answer:** OOP models a system using objects that combine
state and behaviour. In banking, Customer, Account and Transaction can
be represented as objects with clear responsibilities.

# 2. Class and Object

-   **Class:** A blueprint used to create objects.
-   **Object:** An instance of a class with its own state.

``` ts
class Customer {
  constructor(
    public readonly customerId: string,
    public name: string
  ) {}

  updateName(newName: string): void {
    this.name = newName;
  }
}

const customer1 = new Customer("C101", "Rahul");
const customer2 = new Customer("C102", "Amit");
```

**Banking example:** `Customer` is the class; Rahul's and Amit's
customer records are separate objects.

**Interview question: Can we create multiple objects from one class?**\
Yes. A class is a blueprint, and each instance maintains its own state.

# 3. Four pillars of OOP

Memory trick:

-   **Encapsulation = Protect data**
-   **Abstraction = Hide complexity**
-   **Inheritance = Reuse and specialise**
-   **Polymorphism = One contract, many behaviours**

## 3.1 Encapsulation

**Easy definition:** Keep data and related methods together, restrict
direct access to internal data, and allow changes through controlled
operations.

### Bad code

``` ts
class BankAccount {
  public balance = 10000;
}

const account = new BankAccount();
account.balance = -50000; // Invalid state is possible
```

**Problem:** Any caller can bypass validation and put the account into
an invalid state.

### Improved code

``` ts
class BankAccount {
  private balance: number;

  constructor(openingBalance: number) {
    if (openingBalance < 0) throw new Error("Invalid opening balance");
    this.balance = openingBalance;
  }

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Amount must be positive");
    this.balance += amount;
  }

  withdraw(amount: number): void {
    if (amount <= 0) throw new Error("Amount must be positive");
    if (amount > this.balance) throw new Error("Insufficient funds");
    this.balance -= amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

**Advantages:** Protects state, centralises validation, prevents
accidental changes.

**Drawbacks:** Requires intentional methods for valid operations;
excessive getters/setters can create boilerplate.

**Banking example:** A customer cannot directly edit the balance;
deposits and withdrawals go through validated operations.

**Interview answer:** Encapsulation protects an object's internal state
by restricting direct access and exposing controlled methods.

**Follow-up: Does TypeScript `private` provide runtime security?**\
Ordinary TypeScript `private` is primarily a compile-time restriction.
JavaScript `#privateField` enforces private-field access at runtime.
Neither replaces API authorisation or database security.

## 3.2 Abstraction

**Easy definition:** Expose what an object can do and hide how it does
it.

**Banking example:** A transfer service uses a payment gateway without
knowing the provider's low-level API details.

### Bad design

``` ts
class PaymentService {
  pay(amount: number): void {
    // Connect to provider
    // Build provider request
    // Handle provider response
    // Update payment status
  }
}
```

**Problem:** Business workflow and provider-specific details are tightly
mixed.

### Improved design

``` ts
interface PaymentGateway {
  pay(amount: number): Promise<string>;
}

class UpiGateway implements PaymentGateway {
  async pay(amount: number): Promise<string> {
    return `UPI-REF-${amount}`;
  }
}

class CardGateway implements PaymentGateway {
  async pay(amount: number): Promise<string> {
    return `CARD-REF-${amount}`;
  }
}

class PaymentService {
  constructor(private readonly gateway: PaymentGateway) {}

  makePayment(amount: number): Promise<string> {
    return this.gateway.pay(amount);
  }
}

const service = new PaymentService(new UpiGateway());
await service.makePayment(5000);
```

**Advantages:** Reduces coupling, hides implementation details, supports
replacement and testing.

**Drawbacks:** Unnecessary abstractions add indirection and can make
debugging harder.

**Interview answer:** Abstraction hides implementation complexity and
exposes essential functionality. A banking transfer service can use a
PaymentGateway contract without knowing whether UPI or card processing
is behind it.

## 3.3 Inheritance

**Easy definition:** A child class acquires and specialises properties
or behaviour from a parent class.

**Banking example:** Savings and current accounts share account identity
and balance enquiry but may have different fee rules.

``` ts
abstract class BankAccount {
  constructor(
    public readonly accountNumber: string,
    protected balance: number
  ) {}

  getBalance(): number {
    return this.balance;
  }

  abstract calculateMonthlyFee(): number;
}

class SavingsAccount extends BankAccount {
  calculateMonthlyFee(): number {
    return 0;
  }
}

class CurrentAccount extends BankAccount {
  calculateMonthlyFee(): number {
    return 500;
  }
}
```

**Important keywords:** - `extends`: inherit from a class. - `super()`:
call the parent constructor. - `protected`: accessible inside the class
and subclasses. - `override`: explicitly mark an overridden method.

**Advantages:** Reuses common behaviour and supports specialisation.

**Drawbacks:** Parent-child coupling, fragile deep hierarchies, and
children may inherit behaviour that does not make sense.

**When to avoid it:** If classes only share a few methods but do not
have a genuine "is-a" relationship, consider composition.

**Interview answer:** Inheritance lets a child reuse and specialise
parent behaviour. SavingsAccount can extend a common BankAccount base
class.

## 3.4 Polymorphism

**Easy definition:** Different objects can be used through the same
contract while providing different behaviour.

**Banking example:** UPI, IMPS and NEFT implement a common transfer
operation.

``` ts
interface TransferMethod {
  transfer(amount: number): Promise<string>;
}

class ImpsTransfer implements TransferMethod {
  async transfer(amount: number): Promise<string> {
    return `IMPS processed: ${amount}`;
  }
}

class NeftTransfer implements TransferMethod {
  async transfer(amount: number): Promise<string> {
    return `NEFT processed: ${amount}`;
  }
}

class TransferService {
  execute(method: TransferMethod, amount: number): Promise<string> {
    return method.transfer(amount);
  }
}

const transferService = new TransferService();
await transferService.execute(new ImpsTransfer(), 5000);
await transferService.execute(new NeftTransfer(), 10000);
```

**Advantages:** Reduces large conditional blocks, makes extensions
easier, and supports interchangeable implementations.

**Drawbacks:** Too many implementations can be harder to navigate; a
poorly designed common contract may not fit every implementation.

**Interview answer:** Polymorphism allows different implementations to
be called through a common contract. UPI and NEFT can both implement
TransferMethod, so the service does not need to know their internal
details.

# 4. Types of polymorphism

## Compile-time signatures (TypeScript overloads)

TypeScript supports overload signatures, but there is one implementation
at runtime.

``` ts
class TransactionService {
  getTransaction(id: string): string;
  getTransaction(id: number): string;

  getTransaction(id: string | number): string {
    return `Transaction: ${id}`;
  }
}
```

## Runtime polymorphism (overriding)

``` ts
class Notification {
  send(): void {
    console.log("Generic notification");
  }
}

class EmailNotification extends Notification {
  override send(): void {
    console.log("Sending email");
  }
}

class SmsNotification extends Notification {
  override send(): void {
    console.log("Sending SMS");
  }
}

const notifications: Notification[] = [
  new EmailNotification(),
  new SmsNotification()
];

notifications.forEach(item => item.send());
```

**Interview distinction:** Overloading describes multiple call
signatures; overriding supplies a subclass implementation. In
TypeScript, overload signatures are a compile-time feature, not
Java-style runtime method overloading.

# 5. Interface vs Abstract Class

**Interface definition:** A contract that a class or compatible object
must satisfy.

``` ts
interface AccountRepository {
  save(accountNumber: string): Promise<void>;
}

class MySqlAccountRepository implements AccountRepository {
  async save(accountNumber: string): Promise<void> {
    console.log("Saving to MySQL:", accountNumber);
  }
}
```

**Abstract class definition:** A base class that cannot be instantiated
directly and can provide shared state, implemented methods and abstract
methods.

``` ts
abstract class Account {
  constructor(protected balance: number) {}

  getBalance(): number {
    return this.balance;
  }

  abstract calculateFee(): number;
}

class SavingsAccount extends Account {
  calculateFee(): number {
    return 0;
  }
}
```

  -----------------------------------------------------------------------
  Interface                           Abstract class
  ----------------------------------- -----------------------------------
  Defines a contract                  Defines a base class

  No instance implementation state    Can hold instance state

  A class can implement multiple      A class can extend only one class
  interfaces                          

  Useful for capabilities and         Useful for shared state and
  dependencies                        implementation

  Erased from emitted JavaScript      Has runtime class representation
  -----------------------------------------------------------------------

**Interview answer:** I use interfaces for contracts such as
repositories and payment providers. I use abstract classes when related
subclasses genuinely share state or common implementation.

# 6. Composition vs Inheritance

**Easy definition:** - Inheritance = IS-A. - Composition = HAS-A.

SavingsAccount IS-A BankAccount. A transfer service HAS-A payment
provider dependency.

``` ts
interface Notifier {
  send(message: string): void;
}

class SmsNotifier implements Notifier {
  send(message: string): void {
    console.log("SMS:", message);
  }
}

class TransferService {
  constructor(private readonly notifier: Notifier) {}

  completed(): void {
    this.notifier.send("Transfer completed");
  }
}

const transfer = new TransferService(new SmsNotifier());
transfer.completed();
```

**Why composition is often preferred:** It reduces rigid parent-child
coupling and allows behaviours to be replaced independently.

**Drawback:** Requires dependency wiring and may create more components.
Use it where behaviour needs to vary or be replaceable.

# 7. TypeScript access modifiers

  Modifier      Meaning
  ------------- ----------------------------------------------------
  `public`      Accessible from anywhere
  `private`     Accessible only inside the declaring class
  `protected`   Accessible inside the class and subclasses
  `readonly`    Property cannot be reassigned after initialisation

``` ts
class Account {
  public readonly accountNumber: string;
  private balance: number;
  protected accountType = "SAVINGS";

  constructor(accountNumber: string, balance: number) {
    this.accountNumber = accountNumber;
    this.balance = balance;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

Note: `readonly` prevents reassignment, not necessarily mutation of an
object stored in a readonly property.

# 8. Constructor, static and `super`

## Constructor

**Definition:** A special method called when an object is created; it
initialises object state.

``` ts
class Customer {
  constructor(
    public readonly id: string,
    public name: string
  ) {}
}

const customer = new Customer("C101", "Rahul");
```

TypeScript permits only one constructor implementation per class.
Overload signatures can describe different constructor inputs, but all
use one implementation.

## Static

**Definition:** A static member belongs to the class rather than an
individual object.

``` ts
class AccountUtils {
  static isValidAccountNumber(value: string): boolean {
    return /^ACC\\d{6}$/.test(value);
  }
}

AccountUtils.isValidAccountNumber("ACC123456");
```

Use static for stateless utilities or class-level values. Avoid using
static as a substitute for injected services because it can make testing
and replacement harder.

## `super`

`super()` calls the parent constructor. `super.method()` calls a parent
method implementation. In a derived constructor, call `super()` before
accessing `this`.

# 9. Complete banking example: account types

``` ts
abstract class BankAccount {
  constructor(
    public readonly accountNumber: string,
    protected balanceInPaise: number
  ) {}

  deposit(amount: number): void {
    this.validateAmount(amount);
    this.balanceInPaise += amount;
  }

  withdraw(amount: number): void {
    this.validateAmount(amount);

    if (amount > this.balanceInPaise) {
      throw new Error("Insufficient balance");
    }

    this.balanceInPaise -= amount;
  }

  getBalance(): number {
    return this.balanceInPaise;
  }

  abstract calculateMonthlyFee(): number;

  protected validateAmount(amount: number): void {
    if (!Number.isInteger(amount) || amount <= 0) {
      throw new Error("Amount must be positive integer paise");
    }
  }
}

class SavingsAccount extends BankAccount {
  calculateMonthlyFee(): number {
    return 0;
  }
}

class CurrentAccount extends BankAccount {
  calculateMonthlyFee(): number {
    return 50000; // ₹500 in paise
  }
}

const accounts: BankAccount[] = [
  new SavingsAccount("SAV101", 1000000),
  new CurrentAccount("CUR101", 5000000)
];

for (const account of accounts) {
  console.log(account.getBalance());
  console.log(account.calculateMonthlyFee());
}
```

**Concept mapping:**

  OOP concept        Example
  ------------------ ---------------------------------------
  Class and object   BankAccount and its instances
  Encapsulation      Controlled balance changes
  Abstraction        Abstract BankAccount
  Inheritance        SavingsAccount extends BankAccount
  Polymorphism       Different monthly fee implementations

**Production warning:** This is a learning model, not a production
banking ledger. Real transfers need transactional persistence,
concurrency control, idempotency, authorisation, auditability and
reconciliation. Use integer minor currency units or a suitable decimal
representation, not floating-point arithmetic.

# 10. Interview questions and answers --- Basic

## Q1. What is OOP?

OOP is a programming paradigm that organises code around objects
containing data and behaviour. In banking, Customer, Account and
Transaction can be represented as objects.

## Q2. What are the four pillars?

Encapsulation, Abstraction, Inheritance and Polymorphism.

## Q3. Class vs object?

A class is a blueprint; an object is an instance of that blueprint with
its own state.

## Q4. What is encapsulation?

Protecting internal state and exposing controlled operations. For
example, balance is private and changed through deposit and withdrawal
methods.

## Q5. What is abstraction?

Hiding implementation details and exposing essential functionality. For
example, a PaymentGateway contract hides provider-specific processing.

## Q6. What is inheritance?

A child class reuses and specialises parent behaviour. SavingsAccount
can extend BankAccount.

## Q7. What is polymorphism?

Different implementations can be used through a common contract, such as
UPI and NEFT implementing TransferMethod.

## Q8. Overloading vs overriding?

Overloading provides multiple call signatures. Overriding replaces
inherited behaviour in a subclass.

## Q9. What is an abstract class?

A base class that cannot be instantiated directly and can contain shared
implementation and abstract methods.

## Q10. What is an interface?

A contract describing the properties or operations a compatible class or
object must provide.

# 11. Intermediate interview questions

## Q11. Abstraction vs encapsulation?

Encapsulation protects internal state and controls access. Abstraction
hides implementation details and exposes essential operations. Private
balance demonstrates encapsulation; PaymentGateway demonstrates
abstraction.

## Q12. Abstract class vs interface?

An interface defines a contract. An abstract class can provide a
contract plus shared implementation and state. A class can implement
multiple interfaces but extend only one class.

## Q13. Can polymorphism exist without inheritance?

Yes. TypeScript's structural typing allows unrelated objects to satisfy
the same interface and be passed to the same function.

## Q14. Why prefer composition over inheritance?

Composition combines independent behaviours and avoids rigid
parent-child hierarchies. It is useful when a service must switch
between SMS and email providers.

## Q15. Can an abstract class have a constructor?

Yes. It can initialise shared state for subclasses, even though the
abstract class cannot be instantiated directly.

## Q16. Can a constructor be private?

Yes. A private constructor restricts external object creation. It can be
used for controlled creation patterns, such as a singleton, but should
not be used unnecessarily.

## Q17. What is method overriding?

A subclass provides its own implementation of an inherited method while
respecting its contract.

## Q18. What is `super`?

`super()` invokes the parent constructor; `super.method()` invokes a
parent implementation.

## Q19. Private vs protected?

Private is accessible only inside the declaring class. Protected is also
accessible inside subclasses.

## Q20. Static vs instance method?

A static method belongs to the class; an instance method belongs to an
object and can access instance state.

# 12. Senior-level and scenario-based questions

## Q21. How would you support UPI, IMPS, NEFT and RTGS?

Define a focused transfer contract and separate implementations. Inject
the selected implementation into the transfer use case. This uses
abstraction, polymorphism, OCP and DIP. Also handle provider-specific
validation, timeouts, idempotency and reconciliation.

## Q22. One class handles account creation, SMS, PDF statements and transfers. What is wrong?

It has multiple responsibilities and reasons to change. Separate
account, notification, statement and transfer responsibilities, then
coordinate them through application services.

## Q23. SavingsAccount inherits an overdraft method but cannot support it. What would you do?

Avoid forcing an unsupported capability into the base contract. Define a
separate overdraft capability and implement it only for eligible account
types.

## Q24. How do you unit test a service using MySQL?

Depend on a repository interface and inject a fake or mock repository in
unit tests. Use separate integration tests for the real SQL
implementation.

## Q25. Two ₹8,000 withdrawals arrive simultaneously when balance is ₹10,000. What do you do?

Make the balance check and debit atomic using a database transaction and
row-level locking or a conditional update. Only one withdrawal should
succeed. Use idempotency keys to make retries safe.

## Q26. How do you prevent duplicate transfers after a timeout?

Use an idempotency key scoped to the customer/account and operation.
Persist the key and transfer result transactionally so retries return
the original result instead of performing another debit.

## Q27. Why should business logic not live in controllers?

Controllers should handle HTTP concerns such as request validation and
response mapping. Business logic belongs in application services or
domain objects so it can be reused and tested independently.

## Q28. Is OOP always better than functional programming?

No. OOP is useful for stateful domain models and interchangeable
behaviours. Functional programming is useful for stateless
transformations. TypeScript applications can combine both.

## Q29. Can JavaScript implement OOP?

Yes. JavaScript supports objects, classes, inheritance, encapsulation
with private fields and polymorphism. TypeScript adds static typing,
interfaces and compile-time checks.

## Q30. What OOP mistakes should a senior developer avoid?

Avoid classes for every function, deep inheritance, exposing mutable
state, giant classes, unnecessary interfaces, and treating in-memory
validation as a substitute for database transaction safety.

# 13. Quick revision sheet

  -------------------------------------------------------------------------------
  Concept                 One-line memory phrase  Banking example
  ----------------------- ----------------------- -------------------------------
  OOP                     Organise around objects Customer and Account

  Class                   Blueprint               BankAccount

  Object                  Instance                ACC101

  Encapsulation           Protect data            Private balance

  Abstraction             Hide complexity         PaymentGateway

  Inheritance             Reuse and specialise    SavingsAccount extends
                                                  BankAccount

  Polymorphism            Same contract,          UPI and NEFT
                          different behaviour     

  Overloading             Multiple call           getTransaction(string/number)
                          signatures              

  Overriding              Subclass changes        Different account fees
                          inherited behaviour     

  Interface               Contract                AccountRepository

  Abstract class          Shared base with        BankAccount
                          incomplete behaviour    

  Composition             HAS-A                   TransferService has a Notifier

  Static                  Belongs to class        AccountUtils

  Constructor             Initialises object      Create Customer
  -------------------------------------------------------------------------------

## 30-second OOP interview answer

> OOP is a programming paradigm where we model a system using objects
> that contain data and behaviour. It has four pillars. Encapsulation
> protects data, abstraction hides implementation details, inheritance
> allows reuse and specialisation, and polymorphism allows multiple
> implementations through a common contract. In banking, I use these
> concepts to model accounts, transactions and payment methods while
> keeping business logic maintainable and testable.

# 14. Seven-day practice plan

  -----------------------------------------------------------------------
  Day                     Topic                   Coding task
  ----------------------- ----------------------- -----------------------
  1                       Class and object        Create Customer and
                                                  BankAccount

  2                       Encapsulation           Implement deposit and
                                                  withdrawal validation

  3                       Abstraction             Create PaymentGateway
                                                  interface

  4                       Inheritance             Create SavingsAccount
                                                  and CurrentAccount

  5                       Polymorphism            Implement UPI, IMPS and
                                                  NEFT

  6                       Interface vs abstract   Design
                          class                   AccountRepository and
                                                  BankAccount

  7                       Full revision           Explain all four
                                                  pillars without notes
  -----------------------------------------------------------------------

## Retention technique

For each concept:

1.  Read the one-line definition.
2.  Close the notes.
3.  Explain it aloud in your own words.
4.  Write the code without copying.
5.  Give one banking example.
6.  Answer one follow-up question.

**Final advice:** Do not memorise the entire code. Understand why the
bad design fails and why the improved design is better. Interviewers
value your reasoning more than memorised syntax.
