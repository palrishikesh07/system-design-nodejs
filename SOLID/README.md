# SOLID Principles in TypeScript --- Banking Domain Interview Guide

**Audience:** Senior Software Engineer / Tech Lead\
**Language:** TypeScript\
**Examples:** Banking accounts, transfers, payments, repositories and
notifications

------------------------------------------------------------------------

## How to answer a SOLID question in an interview

Use this four-step structure:

1.  **Definition:** Explain the principle in one sentence.
2.  **Violation:** Describe what is wrong with the bad design.
3.  **Improvement:** Explain the design change and show code.
4.  **Impact:** Connect it to a banking use case, testing, changeability
    or reliability.

**Memory phrase:** SOLID helps us design software that is easier to
change, test and maintain.

# 1. What is SOLID?

SOLID is a group of five object-oriented design principles:

  -----------------------------------------------------------------------
  Letter            Principle         Easy phrase       Banking example
  ----------------- ----------------- ----------------- -----------------
  S                 Single            One               Separate account,
                    Responsibility    responsibility    statement and
                                                        notification work

  O                 Open/Closed       Extend without    Add UPI without
                                      repeatedly        rewriting
                                      modifying stable  transfer
                                      code              orchestration

  L                 Liskov            Subtypes honour   Do not make every
                    Substitution      their contracts   account support
                                                        overdraft

  I                 Interface         Small, focused    Statement reader
                    Segregation       interfaces        need not
                                                        implement
                                                        withdrawal

  D                 Dependency        Depend on         Transfer service
                    Inversion         abstractions      depends on
                                                        repository
                                                        contracts
  -----------------------------------------------------------------------

**Why SOLID matters:** - Reduces tight coupling. - Makes code easier to
unit test. - Helps isolate changes. - Supports adding new business
capabilities. - Makes responsibilities and contracts clearer.

**Important:** SOLID is guidance, not a rule to create an interface or
class for every method. Too many abstractions can make a simple
application harder to understand.

------------------------------------------------------------------------

# 2. S --- Single Responsibility Principle (SRP)

## Theory

**Easy definition to remember: One module should have one clear
responsibility --- one main reason to change.**

A class should focus on one cohesive job. It does not mean a class can
have only one method.

### Banking scenario

A bank application has to: - Open an account. - Send an SMS to the
customer. - Generate an account statement.

These responsibilities change for different reasons. For example, the
SMS provider may change without changing account-opening rules.

## Bad practice --- one class does everything

``` ts
class BankService {
  createAccount(customerName: string): void {
    console.log("Creating account for", customerName);
    // Database save logic
  }

  sendSms(message: string): void {
    console.log("Sending SMS:", message);
    // SMS provider integration
  }

  generateStatement(accountId: string): string {
    console.log("Generating statement for", accountId);
    // PDF generation logic
    return "statement.pdf";
  }
}
```

### Problems / drawbacks

1.  The class has multiple reasons to change.
2.  A change to the SMS provider can affect account-related code.
3.  Unit testing becomes harder because unrelated behaviours live
    together.
4.  The class can grow into a God class.
5.  Reusing statement generation in another workflow becomes difficult.

## Good practice --- separate responsibilities

``` ts
class AccountService {
  createAccount(customerName: string): void {
    console.log("Creating account for", customerName);
  }
}

class SmsNotificationService {
  send(message: string): void {
    console.log("Sending SMS:", message);
  }
}

class StatementService {
  generate(accountId: string): string {
    console.log("Generating statement for", accountId);
    return "statement.pdf";
  }
}
```

### Advantages

-   Each class has a clear purpose.
-   Changes are more isolated.
-   Unit tests can focus on one responsibility.
-   Services can be reused independently.

### Tradeoff

There are more classes and files. This is useful when responsibilities
genuinely change independently; do not split every tiny operation into
its own class.

### Interview answer

> SRP means a module should have one cohesive responsibility and one
> main reason to change. In a banking application, I would separate
> account creation, SMS notifications and statement generation because
> each has different business rules and different reasons to change.
> This improves maintainability and makes unit testing easier.

### Follow-up questions

**Q: Does SRP mean one method per class?**\
A: No. A class can have multiple methods as long as they support the
same cohesive responsibility.

**Q: What is a God class?**\
A: A class that takes on too many responsibilities and becomes a central
place for unrelated business logic. It is often a sign of an SRP
violation.

**Q: Can splitting classes too much be bad?**\
A: Yes. Excessive fragmentation adds navigation and wiring overhead.
Split around meaningful responsibilities and reasons to change.

------------------------------------------------------------------------

# 3. O --- Open/Closed Principle (OCP)

## Theory

**Easy definition to remember: Software should be open for extension but
closed for modification.**

In practice, design stable parts of the system so new variations can be
added with minimal changes to existing tested code.

### Banking scenario

A transfer service initially supports UPI and IMPS. Later, the bank
needs to support NEFT and RTGS.

## Bad practice --- growing if/else conditions

``` ts
type TransferType = "UPI" | "IMPS";

class TransferService {
  transfer(type: TransferType, amount: number): void {
    if (type === "UPI") {
      console.log("Processing UPI:", amount);
    } else if (type === "IMPS") {
      console.log("Processing IMPS:", amount);
    }
  }
}
```

Suppose NEFT is added:

``` ts
// We must edit TransferService and add another condition.
// More payment types mean more branches and regression risk.
```

### Problems / drawbacks

1.  Every new transfer type modifies the same class.
2.  Conditional logic grows.
3.  Existing code becomes more likely to be affected by new features.
4.  Testing all combinations becomes harder.

## Good practice --- use a common contract

``` ts
interface TransferMethod {
  transfer(amount: number): Promise<string>;
}

class UpiTransfer implements TransferMethod {
  async transfer(amount: number): Promise<string> {
    console.log("Processing UPI:", amount);
    return "UPI-REF-001";
  }
}

class ImpsTransfer implements TransferMethod {
  async transfer(amount: number): Promise<string> {
    console.log("Processing IMPS:", amount);
    return "IMPS-REF-001";
  }
}

class BankTransferService {
  execute(
    method: TransferMethod,
    amount: number
  ): Promise<string> {
    return method.transfer(amount);
  }
}
```

Add a new implementation without changing `BankTransferService`:

``` ts
class NeftTransfer implements TransferMethod {
  async transfer(amount: number): Promise<string> {
    console.log("Processing NEFT:", amount);
    return "NEFT-REF-001";
  }
}

const service = new BankTransferService();
const reference = await service.execute(new NeftTransfer(), 5000);
```

### Advantages

-   New transfer methods can be added independently.
-   The orchestration class stays stable.
-   Each implementation can be unit tested separately.
-   Reduces the risk of changing existing payment flows.

### Tradeoff

-   Adds interfaces and implementation classes.
-   A common contract must represent genuinely common behaviour.
-   Some changes to existing code are still necessary when requirements
    or defects demand them.

### Interview answer

> OCP means we should be able to extend a system with new behaviour
> without repeatedly modifying stable, tested code. For banking
> transfers, I define a TransferMethod interface and implement UPI, IMPS
> and NEFT separately. The transfer service depends on the contract, so
> adding a new rail does not require adding another conditional branch
> to the orchestration class.

### Follow-up questions

**Q: Does OCP mean existing code must never change?**\
A: No. It means likely variations should have extension points. Bug
fixes and changed requirements can still require existing code changes.

**Q: Is an interface always required for OCP?**\
A: No. Interfaces are one technique. Composition, callbacks, strategy
objects and other extension mechanisms can also help.

**Q: What design pattern is commonly used with OCP here?**\
A: Strategy pattern. Each transfer implementation is a strategy selected
by the caller or a factory.

------------------------------------------------------------------------

# 4. L --- Liskov Substitution Principle (LSP)

## Theory

**Easy definition to remember: A subtype must honour the promises of its
parent type, so it can be substituted without breaking correct client
behaviour.**

A child class should not strengthen the requirements, weaken the
guarantees, or unexpectedly reject behaviour promised by its parent
contract.

### Banking scenario

Savings accounts do not support overdrafts, while some current accounts
do. Do not force every account to implement an overdraft operation.

## Bad practice --- parent promises something every child cannot do

``` ts
class BankAccount {
  withdraw(amount: number): void {
    console.log("Withdrawing", amount);
  }

  useOverdraft(amount: number): void {
    throw new Error("Overdraft not supported");
  }
}

class SavingsAccount extends BankAccount {}

function chargeEmergencyExpense(account: BankAccount): void {
  account.useOverdraft(1000);
}

const savings = new SavingsAccount();
chargeEmergencyExpense(savings); // Unexpected runtime failure
```

### Problems / drawbacks

1.  The subtype cannot honour the parent contract.
2.  Code accepting `BankAccount` assumes an operation that some accounts
    cannot support.
3.  Runtime exceptions replace reliable type-level design.
4.  Consumers become coupled to implementation-specific limitations.

## Good practice --- model capabilities separately

``` ts
interface WithdrawableAccount {
  withdraw(amount: number): void;
}

interface OverdraftEnabledAccount extends WithdrawableAccount {
  useOverdraft(amount: number): void;
}

class SavingsAccount implements WithdrawableAccount {
  withdraw(amount: number): void {
    console.log("Savings withdrawal:", amount);
  }
}

class CurrentAccount implements OverdraftEnabledAccount {
  withdraw(amount: number): void {
    console.log("Current account withdrawal:", amount);
  }

  useOverdraft(amount: number): void {
    console.log("Using overdraft:", amount);
  }
}
```

Now a function requiring overdraft explicitly asks for that capability:

``` ts
function chargeEmergencyExpense(
  account: OverdraftEnabledAccount
): void {
  account.useOverdraft(1000);
}
```

### Advantages

-   Contracts match actual business capabilities.
-   Consumers do not assume unsupported operations.
-   Implementations can be substituted safely.
-   Fewer surprising runtime failures.

### Tradeoff

More capability interfaces may be needed. Keep them cohesive and based
on real domain behaviour.

### Interview answer

> LSP says a subtype must be usable wherever its parent type is expected
> without breaking the parent's behavioural contract. In banking, a
> savings account should not inherit an overdraft operation if it cannot
> support it. I would model overdraft as a separate capability
> implemented only by eligible account types.

### Follow-up questions

**Q: Is method overriding always LSP?**\
A: No. A method can be syntactically overridden but still violate the
parent's behavioural expectations.

**Q: What is a practical sign of an LSP violation?**\
A: Subclasses throwing "not supported" exceptions for normal parent
operations, or callers needing type checks to avoid broken behaviour.

**Q: How do you fix an LSP violation?**\
A: Revisit the parent contract, split capabilities, or use composition
so types promise only behaviour they can honour.

------------------------------------------------------------------------

# 5. I --- Interface Segregation Principle (ISP)

## Theory

**Easy definition to remember: Clients should not be forced to depend on
methods they do not use.**

Prefer small, focused interfaces over one large interface that contains
unrelated operations.

### Banking scenario

A read-only statement viewer needs to read statements. It does not need
to deposit money, withdraw money or close accounts.

## Bad practice --- a large interface

``` ts
interface BankingOperations {
  deposit(amount: number): void;
  withdraw(amount: number): void;
  generateStatement(): string;
  closeAccount(): void;
}

class StatementViewer implements BankingOperations {
  deposit(amount: number): void {
    throw new Error("Not supported");
  }

  withdraw(amount: number): void {
    throw new Error("Not supported");
  }

  generateStatement(): string {
    return "Statement content";
  }

  closeAccount(): void {
    throw new Error("Not supported");
  }
}
```

### Problems / drawbacks

1.  Implementations are forced to provide irrelevant methods.
2.  Empty methods or "not supported" exceptions appear.
3.  A change to an unrelated method can affect many implementers.
4.  Interfaces become difficult to maintain.

## Good practice --- smaller interfaces

``` ts
interface Depositable {
  deposit(amount: number): void;
}

interface Withdrawable {
  withdraw(amount: number): void;
}

interface StatementReadable {
  generateStatement(): string;
}

class SavingsAccount
  implements Depositable, Withdrawable, StatementReadable {
  deposit(amount: number): void {
    console.log("Deposit:", amount);
  }

  withdraw(amount: number): void {
    console.log("Withdraw:", amount);
  }

  generateStatement(): string {
    return "Savings statement";
  }
}

class StatementViewer implements StatementReadable {
  generateStatement(): string {
    return "Read-only statement";
  }
}
```

### Advantages

-   Implementations only depend on relevant operations.
-   Contracts are easier to understand.
-   Changes have a smaller impact.
-   Tests and mocks need fewer methods.

### Tradeoff

Too many tiny interfaces can make navigation difficult. Group operations
when they represent one cohesive capability for a client.

### Interview answer

> ISP says clients should not be forced to depend on methods they do not
> use. In banking, a read-only statement viewer should implement only
> statement-reading capability, not deposit, withdrawal or
> account-closure methods. I split large interfaces into focused
> capabilities.

### Follow-up questions

**Q: Is ISP only about reducing the number of methods?**\
A: No. It is about designing interfaces around the needs of their
consumers, not simply making every interface small.

**Q: What is a fat interface?**\
A: An interface containing many unrelated operations that force
implementers or consumers to depend on methods they do not need.

------------------------------------------------------------------------

# 6. D --- Dependency Inversion Principle (DIP)

## Theory

**Easy definition to remember: High-level business logic should depend
on abstractions, not directly on low-level implementation details.**

The business use case should not need to know whether data is stored in
MySQL, MongoDB or an in-memory test repository.

### Banking scenario

AccountService saves account data. It should not construct a specific
MySQL repository internally.

## Bad practice --- tight coupling

``` ts
class MySqlAccountRepository {
  save(accountNumber: string): void {
    console.log("Saving to MySQL:", accountNumber);
  }
}

class AccountService {
  private repository = new MySqlAccountRepository();

  openAccount(accountNumber: string): void {
    this.repository.save(accountNumber);
  }
}
```

### Problems / drawbacks

1.  AccountService is tied to MySQL.
2.  Replacing the database requires changing business logic.
3.  Unit tests may need a real database or awkward workarounds.
4.  Business logic knows too much about infrastructure.

## Good practice --- depend on an abstraction and inject it

``` ts
interface AccountRepository {
  save(accountNumber: string): Promise<void>;
}

class MySqlAccountRepository implements AccountRepository {
  async save(accountNumber: string): Promise<void> {
    console.log("Saving to MySQL:", accountNumber);
  }
}

class MongoAccountRepository implements AccountRepository {
  async save(accountNumber: string): Promise<void> {
    console.log("Saving to MongoDB:", accountNumber);
  }
}

class AccountService {
  constructor(
    private readonly repository: AccountRepository
  ) {}

  openAccount(accountNumber: string): Promise<void> {
    return this.repository.save(accountNumber);
  }
}

const repository = new MySqlAccountRepository();
const accountService = new AccountService(repository);

await accountService.openAccount("ACC1001");
```

### Advantages

-   Business logic is independent of database vendor.
-   Easy to inject fake repositories in unit tests.
-   Infrastructure can be replaced without rewriting the use case.
-   Dependencies are explicit.

### Tradeoff

Dependency wiring is required. A DI container can help in a larger
application, but constructor injection is often sufficient.

### Interview answer

> DIP says high-level modules should depend on abstractions rather than
> concrete low-level implementations. In banking, AccountService depends
> on an AccountRepository contract rather than constructing a MySQL
> repository. I inject the implementation through the constructor, which
> makes the service easier to test and allows infrastructure to change
> independently.

### Follow-up questions

**Q: Is DIP the same as dependency injection?**\
A: No. DIP is a design principle; dependency injection is a technique
for supplying dependencies from outside an object.

**Q: What is the benefit of constructor injection?**\
A: Dependencies are explicit, required dependencies can be enforced at
construction time, and tests can provide fakes or mocks.

**Q: Can DIP be used without a DI framework?**\
A: Yes. Passing a dependency into a constructor manually is dependency
injection. A framework is optional.

------------------------------------------------------------------------

# 7. DIP vs DI vs IoC

These terms are related but different.

  ------------------------------------------------------------------------
  Term                    Easy definition         Banking example
  ----------------------- ----------------------- ------------------------
  DIP                     Design principle        AccountService depends
                                                  on AccountRepository

  DI                      Technique               Pass
                                                  MySqlAccountRepository
                                                  into AccountService

  IoC                     Control over object     NestJS container creates
                          creation/wiring is      and injects providers
                          delegated               
  ------------------------------------------------------------------------

## Simple constructor injection

``` ts
interface Notifier {
  send(message: string): Promise<void>;
}

class SmsNotifier implements Notifier {
  async send(message: string): Promise<void> {
    console.log("SMS:", message);
  }
}

class TransferService {
  constructor(private readonly notifier: Notifier) {}

  async completeTransfer(): Promise<void> {
    await this.notifier.send("Transfer completed");
  }
}

const notifier = new SmsNotifier();
const transferService = new TransferService(notifier);

await transferService.completeTransfer();
```

**Memory trick:** DIP is the rule; DI is one way to follow the rule; IoC
is the broader transfer of control.

------------------------------------------------------------------------

# 8. All five principles together --- banking transfer design

## Requirement

A banking application should: - Transfer money between accounts. -
Support different transfer methods. - Save transfer records. - Send a
notification. - Allow infrastructure implementations to be replaced and
tested.

## High-level design

``` text
TransferController
       |
       v
TransferService (business use case)
       |
       +---- TransferMethod interface
       |          +---- UPI implementation
       |          +---- IMPS implementation
       |
       +---- TransferRepository interface
       |          +---- MySQL implementation
       |
       +---- Notifier interface
                  +---- SMS implementation
                  +---- Email implementation
```

## Principle mapping

  -----------------------------------------------------------------------
  Principle                           How it appears
  ----------------------------------- -----------------------------------
  SRP                                 Transfer, persistence and
                                      notification have distinct
                                      responsibilities

  OCP                                 New transfer method can be added as
                                      another implementation

  LSP                                 Each transfer implementation
                                      honours the transfer contract

  ISP                                 Focused repository, transfer and
                                      notifier contracts

  DIP                                 TransferService depends on
                                      abstractions
  -----------------------------------------------------------------------

## Illustrative TypeScript orchestration

``` ts
interface TransferMethod {
  execute(amountInPaise: number): Promise<string>;
}

interface TransferRepository {
  save(reference: string, amountInPaise: number): Promise<void>;
}

interface Notifier {
  send(message: string): Promise<void>;
}

class TransferService {
  constructor(
    private readonly method: TransferMethod,
    private readonly repository: TransferRepository,
    private readonly notifier: Notifier
  ) {}

  async transfer(amountInPaise: number): Promise<string> {
    if (!Number.isInteger(amountInPaise) || amountInPaise <= 0) {
      throw new Error("Invalid amount");
    }

    const reference = await this.method.execute(amountInPaise);

    await this.repository.save(reference, amountInPaise);

    await this.notifier.send(`Transfer processed: ${reference}`);

    return reference;
  }
}
```

### Important production warning

This is an architectural illustration, not a complete production banking
transfer. A real transfer requires more than SOLID:

1.  Database transaction boundaries for ledger and account updates.
2.  Concurrency control such as row-level locking or safe conditional
    updates.
3.  Idempotency keys to prevent duplicate debits on retries.
4.  Durable, balanced debit and credit ledger entries.
5.  Transactional outbox or equivalent reliable notification delivery.
6.  Audit logs, authorisation, limits, fraud checks and reconciliation.
7.  Clear handling of pending, successful, failed and unknown provider
    outcomes.

Do not allow a notification failure to make a successfully committed
transfer appear rolled back. Persist the business result and deliver
notifications reliably after commit.

------------------------------------------------------------------------

# 9. Common interview questions and answers

## Basic

### Q1. What is SOLID?

SOLID is a set of five object-oriented design principles that help make
code easier to maintain, extend and test: SRP, OCP, LSP, ISP and DIP.

### Q2. What is SRP?

A module should have one cohesive responsibility and one main reason to
change. For example, account creation and SMS delivery should not be
mixed into one service.

### Q3. What is OCP?

Software should allow new behaviour to be added with minimal changes to
stable existing code. For example, add NEFT as a new transfer
implementation instead of adding another branch to a large conditional.

### Q4. What is LSP?

A subtype should honour the behavioural contract of its parent type. A
savings account should not be treated as an overdraft-enabled account if
it cannot perform that operation.

### Q5. What is ISP?

Clients should not be forced to depend on methods they do not use. A
statement viewer should not need to implement withdrawal.

### Q6. What is DIP?

High-level business logic should depend on abstractions, not concrete
infrastructure classes. AccountService should depend on
AccountRepository rather than directly constructing
MySQLAccountRepository.

## Intermediate

### Q7. Can we implement SOLID without inheritance?

Yes. SOLID can be applied with interfaces, composition, functions and
dependency injection. Inheritance is not required.

### Q8. Does SRP mean one method per class?

No. It means one cohesive responsibility, not one method.

### Q9. Is OCP always achieved by interfaces?

No. Interfaces are one technique. Composition, strategy objects,
callbacks and plugin mechanisms can also support extension.

### Q10. How are LSP and ISP related?

ISP helps create focused contracts; LSP requires implementations to
honour those contracts. Together they help prevent consumers from
relying on unsupported behaviour.

### Q11. How does DIP improve unit testing?

A service depending on an interface can receive a fake or mock
implementation. Tests can exercise business logic without a real
database or external provider.

### Q12. Can too much SOLID be harmful?

Yes. Excessive abstractions, factories and tiny interfaces create
indirection and maintenance overhead. Apply the principles where they
solve real coupling, changeability or testing problems.

## Senior-level / scenario-based

### Q13. A new payment rail must be added. What do you change?

I would add a new implementation of the existing transfer contract and
register it with the selection mechanism. I would avoid modifying stable
orchestration code unless the new rail has genuinely different business
requirements. I would add unit, contract and integration tests for the
new implementation.

### Q14. A service has 12 injected dependencies. Is that good?

Not necessarily. It may indicate the service has too many
responsibilities or coordinates too many unrelated workflows. I would
inspect the use cases and split cohesive responsibilities where
appropriate, without blindly reducing the dependency count.

### Q15. How would you design a notification service supporting SMS, email and push?

Define a focused notification contract and implement each channel
separately. Inject the selected channel or use a dispatcher based on
notification type. Handle retries, provider timeouts, delivery status
and idempotency separately from the core banking transaction.

### Q16. A child class throws "operation not supported" for a parent method. What does it suggest?

It may be an LSP violation. I would revisit the parent contract and
model the operation as a separate capability if not all subclasses
support it.

### Q17. How would you test a transfer service?

Unit test it with fake TransferMethod, TransferRepository and Notifier
implementations. Verify success, invalid amount, provider failure,
repository failure and notification failure. Integration tests should
verify real database transaction and provider-adapter behaviour.

### Q18. What if the database changes from MySQL to MongoDB?

If the use case depends on a repository abstraction, implement a new
repository adapter and wire it in. The business use case should not need
to know the database vendor. Data modelling and transaction semantics
still need careful redesign; changing the adapter alone does not
guarantee equivalent behaviour.

### Q19. How do you handle a transfer provider timeout?

Do not assume timeout means failure; the provider may have processed the
request but the response may have been lost. Use an
idempotency/reference strategy, query or reconcile provider status, and
represent the outcome as pending/unknown until confirmed. Avoid blindly
retrying a potentially successful debit.

### Q20. What is the difference between SOLID and design patterns?

SOLID gives design principles and goals. Design patterns are reusable
approaches to recurring design problems. For example, Strategy can help
implement OCP for multiple payment methods.

------------------------------------------------------------------------

# 10. SOLID quick revision sheet

  --------------------------------------------------------------------------
  Principle               Memorise this           Bad smell
  ----------------------- ----------------------- --------------------------
  SRP                     One responsibility      God class

  OCP                     Extend without          Growing if/else or switch
                          repeatedly modifying    
                          stable code             

  LSP                     Subtypes honour their   Unsupported methods in
                          contracts               child

  ISP                     Focused interfaces      Fat interface

  DIP                     Depend on abstractions  `new ConcreteDatabase()`
                                                  inside business service
  --------------------------------------------------------------------------

## 30-second interview answer

> SOLID is a set of five object-oriented design principles. SRP says one
> module should have one cohesive responsibility. OCP says we should
> extend behaviour without repeatedly modifying stable code. LSP says
> subtypes must honour their contracts. ISP says clients should not
> depend on methods they do not use. DIP says high-level business logic
> should depend on abstractions rather than concrete implementations. In
> banking, these principles help separate account, transfer, payment,
> repository and notification concerns so the system is easier to test
> and evolve.

------------------------------------------------------------------------

# 11. Practice exercise --- test yourself

Try these without looking at the answers.

1.  A `CustomerService` creates customers, sends SMS and generates PDF
    statements. Which principle is violated?
2.  Every new payment method requires editing a large switch statement.
    Which principle should guide the redesign?
3.  A `SavingsAccount` inherits `useOverdraft()` but throws an
    exception. Which principle is at risk?
4.  A read-only statement component must implement `withdraw()` and
    `deposit()`. Which principle is violated?
5.  `TransferService` directly creates `new MySqlRepository()`. Which
    principle is being violated?
6.  Explain DIP vs DI in your own words.
7.  Explain how SOLID helps unit testing.
8.  Give one example where too much abstraction would be harmful.

## Answers

1.  SRP --- unrelated responsibilities are mixed.
2.  OCP --- add payment strategies behind a stable contract.
3.  LSP --- the subtype cannot honour the inherited contract.
4.  ISP --- the interface forces unused methods.
5.  DIP --- business logic depends directly on a concrete infrastructure
    class.
6.  DIP is a design principle; DI is a technique for supplying
    dependencies.
7.  Focused modules and injectable abstractions allow isolated tests
    with fakes and mocks.
8.  Creating interfaces and factories for stable, trivial logic that has
    no meaningful variation.

------------------------------------------------------------------------

# 12. Seven-day learning plan

  Day   Topic                Practice
  ----- -------------------- ----------------------------------------------------
  1     SRP                  Split account, notification and statement services
  2     OCP                  Add UPI, IMPS and NEFT using a transfer contract
  3     LSP                  Model savings and overdraft-enabled accounts
  4     ISP                  Split banking operations into focused capabilities
  5     DIP and DI           Inject MySQL and in-memory repositories
  6     All five together    Build a small transfer orchestration service
  7     Interview revision   Explain each principle aloud without notes

## Retention technique

For each principle, remember:

-   One-line definition.
-   One bad code smell.
-   One improved design.
-   One banking example.
-   One drawback or tradeoff.

**Final advice:** In a senior interview, do not just say "I use SOLID."
Explain a real design problem, the principle involved, the tradeoff, and
how you would test the improved design. That demonstrates engineering
judgement rather than memorisation.
