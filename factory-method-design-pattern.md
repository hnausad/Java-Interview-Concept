The **Factory Method Design Pattern** is a foundational creational pattern. Instead of using the `new` operator directly to instantiate objects within your primary business logic, it defines an interface or abstract class for object creation, but defers the actual instantiation to specialized subclasses.

---

## 1. Detailed Description

### The Problem It Solves

In complex financial applications, business processing workflows must remain incredibly stable, secure, and audited. However, the concrete financial instruments or data structures they process can vary wildly.

If your core processing engine contains hardcoded `new SavingsAccount()` or `new CreditCardAccount()` statements, your core business layer becomes tightly coupled to specific product types. Every time the bank introduces a new product variant (like a Wealth Management Account or a Corporate Loan Account), you have to rip open, modify, and re-test your core ledger or transaction processing engine. This violates the **Open/Closed Principle** (code should be open for extension but closed to modification) and introduces severe regression risks to core banking functions.

### How It Works

The Factory Method pattern decouples the **execution logic** from the **creation logic**:

1. **Product Interface:** Defines the common operational operations that all bank account variants must support (e.g., `.calculateInterest()` or `.processWithdrawal()`).
2. **Concrete Products:** The actual, specific implementations of the account types (e.g., `SavingsAccount`, `CurrentAccount`).
3. **Creator (Factory) Class:** Contains the high-level banking business logic (like standard onboarding, auditing, or compliance workflows). It declares an abstract method—the **Factory Method**—for creating an account.
4. **Concrete Creators:** Subclasses that override the factory method to instantiate and return the exact concrete financial product required.

### The Core Benefit

The primary transaction processing engine only interacts with the generic **Product Interface** and the **Creator class**. It remains completely oblivious to which specific type of account it is executing.

---

## 2. Functional Banking Example

Think of an automated **Bank Account Onboarding and Issuance System**.

When a customer walks into a branch or uses the mobile app to open a new account, the bank must execute a strict, standardized operational pipeline:

1. Run a Know Your Customer (KYC) identity check.
2. Verify credit bureau scores.
3. **Instantiate the specific account type.**
4. Issue a welcome kit and generate an account number.

Steps 1, 2, and 4 are absolutely identical whether the customer is opening a basic retail savings account or a high-volume corporate current account.

Instead of writing a messy conditional block (`if accountType == "SAVINGS"`) in the middle of this critical onboarding pipeline, the system utilizes a Factory Method:

* The core compliance engine calls an abstract method `.createAccount()`.
* If a retail customer applies, the `RetailOnboardingEngine` factory spawns a `SavingsAccount`.
* If a business applies, the `CorporateOnboardingEngine` factory spawns a `CurrentAccount`.

The core pipeline processes the compliance steps flawlessly without ever needing to know the technical structural differences between retail and corporate databases.

---

## 3. Code Example

Here is a clean, production-ready implementation of this Banking Onboarding Engine in **Java**.

### Step 1: The Product Interface and Concrete Products

```java
// The common interface all banking products must implement
interface BankAccount {
    void configureAccountRules();
    void IssueWelcomeKit();
}

// Concrete Product A: Retail Savings Account
class SavingsAccount implements BankAccount {
    @Override
    public void configureAccountRules() { 
        System.out.println("Applying 4.0% interest rate. Enforcing minimum balance of INR 10,000."); 
    }
    @Override
    public void IssueWelcomeKit() { 
        System.out.println("Dispatching Standard Debit Card and Checkbook."); 
    }
}

// Concrete Product B: Corporate Current Account
class CurrentAccount implements BankAccount {
    @Override
    public void configureAccountRules() { 
        System.out.println("Applying 0% interest rate. Enabling multi-currency transaction processing and Overdraft limits."); 
    }
    @Override
    public void IssueWelcomeKit() { 
        System.out.println("Configuring Corporate Payment Gateway Access and Corporate Credit Cards."); 
    }
}

```

### Step 2: The Creator Class (The Factory Definition)

```java
// The central core engine defining the onboarding pipeline
abstract class AccountOnboardingEngine {

    // This is the actual Factory Method
    protected abstract BankAccount createAccount();

    // The standardized, immutable workflow for account opening
    public void onboardCustomer() {
        System.out.println("--- Starting Compliance Pipeline ---");
        System.out.println("1. Running mandatory KYC check against national identity registry...");
        System.out.println("2. Verifying credit bureau metrics...");

        // Deferring the object creation to the specific subclass factory
        BankAccount account = createAccount(); 
        
        // Processing business logic purely through the abstract interface
        account.configureAccountRules();
        account.IssueWelcomeKit();
        
        System.out.println("SUCCESS: Account activation complete.\n");
    }
}

```

### Step 3: Concrete Creators (Subclassed Factories)

```java
// Concrete Factory for Retail Banking divisions
class RetailOnboardingBranch extends AccountOnboardingEngine {
    @Override
    protected BankAccount createAccount() {
        return new SavingsAccount();
    }
}

// Concrete Factory for Corporate Banking divisions
class CorporateOnboardingBranch extends AccountOnboardingEngine {
    @Override
    protected BankAccount createAccount() {
        return new CurrentAccount();
    }
}

```

---

### Verification: Running the Banking Engine

```java
public class BankingFactoryDemo {
    public static void main(String[] args) {
        // A retail branch handles a standard walk-in customer
        AccountOnboardingEngine retailBranch = new RetailOnboardingBranch();
        retailBranch.onboardCustomer();

        // A commercial banking executive onboard a newly registered tech business
        AccountOnboardingEngine corporateBranch = new CorporateOnboardingBranch();
        corporateBranch.onboardCustomer();
    }
}

```

### Execution Output

```text
--- Starting Compliance Pipeline ---
1. Running mandatory KYC check against national identity registry...
2. Verifying credit bureau metrics...
Applying 4.0% interest rate. Enforcing minimum balance of INR 10,000.
Dispatching Standard Debit Card and Checkbook.
SUCCESS: Account activation complete.

--- Starting Compliance Pipeline ---
1. Running mandatory KYC check against national identity registry...
2. Verifying credit bureau metrics...
Applying 0% interest rate. Enabling multi-currency transaction processing and Overdraft limits.
Configuring Corporate Payment Gateway Access and Corporate Credit Cards.
SUCCESS: Account activation complete.

```

### Why this is a powerful asset in interviews:

If the bank decides to launch a **"SalaryAccount"** or a **"LoanAccount"** next quarter, you don't touch a single line of your core KYC/Compliance workflow engine. You simply implement the `BankAccount` interface for the new product, write its matching `OnboardingEngine` factory subclass, and plug it in seamlessly. This guarantees that your core, heavily audited pipeline remains completely safe from code changes.
