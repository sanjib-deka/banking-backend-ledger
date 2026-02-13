# Backend Ledger System

A robust banking backend application implemented with Node.js, Express, and MongoDB. This system uses a **double-entry ledger** architecture to ensure data integrity and accurate financial tracking.

## 🚀 Features

-   **Double-Entry Ledger**: Every transaction records a Credit and a Debit entry. The ledger is immutable.
-   **ACID Transactions**: Uses MongoDB Multi-Document Transactions to ensure money transfers are atomic.
-   **Event Sourcing Style Balance**: Account balances are calculated dynamically by aggregating ledger entries, ensuring zero discrepancy.
-   **Idempotency**: Prevents duplicate transactions using idempotency keys.
-   **Authentication**: Secure JWT-based authentication with token blacklisting for logout.
-   **Account Management**: Users can create multiple accounts (e.g., Savings, Current) with different currencies.

## 🛠️ Tech Stack

-   **Runtime**: Node.js
-   **Framework**: Express.js
-   **Database**: MongoDB (Mongoose ODM)
-   **Authentication**: JSON Web Tokens (JWT), bcryptjs
-   **Validation**: Mongoose Schema Validation

## 📂 Project Structure

```
src/
├── config/         # Database configuration
├── controllers/    # Request handlers (Auth, Account, Transaction)
├── middleware/     # Auth middleware
├── models/         # Mongoose models (User, Account, Transaction, Ledger)
├── routes/         # API Route definitions
├── services/       # External services (Email, etc.)
└── app.js          # Express app setup
```

## ⚙️ Setup & Installation

1.  **Clone the repository**
    ```bash
    git clone <repository-url>
    cd backend-ledger
    ```

2.  **Install Dependencies**
    ```bash
    npm install
    ```

3.  **Environment Variables**
    Create a `.env` file in the root directory with the following variables:
    ```env
    PORT=3000
    MONGODB_URI=mongodb://localhost:27017/backend-ledger
    JWT_SECRET=your_super_secret_key
    # Add other necessary variables (e.g., EMAIL_SERVICE credentials)
    ```

4.  **Run the Server**
    ```bash
    npm run dev
    ```

## 🔌 API Endpoints

### Authentication
-   `POST /api/auth/register` - Register a new user.
-   `POST /api/auth/login` - Login and receive JWT.
-   `POST /api/auth/logout` - Logout (invalidate token).

### Accounts
-   `POST /api/accounts` - Create a new bank account.
-   `GET /api/accounts` - List all accounts for the logged-in user.
-   `GET /api/accounts/balance/:accountId` - Get the current calculated balance of an account.

### Transactions
-   `POST /api/transactions` - Transfer funds between accounts.
    -   **Body**: `{ "fromAccount": "ID", "toAccount": "ID", "amount": 100, "idempotencyKey": "unique-key" }`
-   `POST /api/transactions/system/initial-funds` - Inject initial funds (System Admin only).

## 📖 Architecture Highlights

### Double-Entry Ledger
The system does not store a simple "balance" field that is updated. Instead, every money movement creates two immutable `Ledger` entries:
1.  **DEBIT** from the source account.
2.  **CREDIT** to the destination account.

The balance is always derived by summing `Total Credits - Total Debits` for a given account. This prevents race conditions and data corruption common in simple balance-update systems.

### Atomic Transfers
Money transfers involve multiple operations:
1.  Check sender balance.
2.  Create Transaction record (Pending).
3.  Create Debit Ledger entry.
4.  Create Credit Ledger entry.
5.  Update Transaction status (Completed).

All these steps are wrapped in a **MongoDB Session**, ensuring that either ALL succeed or NONE succeed (Atomic).
