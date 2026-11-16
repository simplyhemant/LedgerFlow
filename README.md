# 💸 LedgerFlow: Banking Transaction System

A backend banking system built with **Node.js, Express, and MongoDB**. It supports secure authentication, multiple accounts per user, ledger-based balance calculation, and idempotent fund transfers using MongoDB transactions.

---

## 📌 Features

- **User Authentication**: register, login, and logout with JWT (cookie or Bearer token)
- **Token Blacklisting**: logged-out tokens are invalidated and can't be reused
- **Account Management**: create accounts, list your accounts, check balances
- **Ledger-Based Balances**: balance is derived from DEBIT/CREDIT entries instead of a stored number
- **Idempotent Transfers**: an idempotency key prevents duplicate transactions on retries
- **ACID Transactions**: MongoDB sessions keep debit and credit entries atomic
- **Email Notifications**: emails on registration and on successful transfers
- **Initial Funds API**: a system user can seed funds into an account

---

## 🛠️ Tech Stack

| Layer          | Technology                      |
| -------------- | ------------------------------- |
| Runtime        | Node.js                         |
| Framework      | Express.js                      |
| Database       | MongoDB + Mongoose              |
| Auth           | JSON Web Tokens (JWT), cookies  |
| Email          | Nodemailer                      |
| Config         | dotenv                          |

---

## 🧠 How It Works

### Ledger-Based Balance

Balances are never stored as a mutable field. Every movement of money creates ledger entries:

```
Balance = SUM(CREDIT entries) - SUM(DEBIT entries)
```

This gives a full audit trail and avoids inconsistent balance updates.

### The 10-Step Transfer Flow

1. Validate the request
2. Validate the idempotency key
3. Check account status (must be `ACTIVE`)
4. Derive the sender's balance from the ledger
5. Create the transaction (`PENDING`)
6. Create the `DEBIT` ledger entry
7. Create the `CREDIT` ledger entry
8. Mark the transaction `COMPLETED`
9. Commit the MongoDB session
10. Send an email notification

### Idempotency

Each transfer requires a unique `idempotencyKey`. If the same key is sent again:

| Existing status | Response                              |
| --------------- | ------------------------------------- |
| `COMPLETED`     | Returns the original transaction      |
| `PENDING`       | Tells the client it's still processing |
| `FAILED`        | Asks the client to retry              |
| `REVERSED`      | Asks the client to retry              |

---

## 📁 Project Structure

```
├── src
│   ├── controllers
│   │   ├── auth.controller.js
│   │   ├── account.controller.js
│   │   └── transaction.controller.js
│   ├── models
│   │   ├── user.model.js
│   │   ├── account.model.js
│   │   ├── transaction.model.js
│   │   ├── ledger.model.js
│   │   └── blackList.model.js
│   ├── routes
│   ├── middleware
│   ├── services
│   │   └── email.service.js
│   └── app.js
├── server.js
├── .env
└── package.json
```

---

## 🚀 Getting Started

### Prerequisites

- Node.js v18+
- MongoDB running as a **replica set** (required for transactions). MongoDB Atlas works out of the box.

### Installation

```bash
# Clone the repository
git clone https://github.com/<your-username>/ledgerflow-api.git
cd ledgerflow-api

# Install dependencies
npm install
```

### Environment Variables

Create a `.env` file in the root directory:

```env
PORT=3000
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret

# Email (Nodemailer)
EMAIL_USER=your_email@example.com
EMAIL_PASS=your_email_app_password
```

### Run the Server

```bash
# Development
npm run dev

# Production
npm start
```

---

## 📡 API Endpoints

### Auth

| Method | Endpoint             | Description          | Auth |
| ------ | -------------------- | -------------------- | ---- |
| POST   | `/api/auth/register` | Register a new user  | ❌   |
| POST   | `/api/auth/login`    | Login a user         | ❌   |
| POST   | `/api/auth/logout`   | Logout and blacklist token | ✅ |

### Accounts

| Method | Endpoint                         | Description            | Auth |
| ------ | -------------------------------- | ---------------------- | ---- |
| POST   | `/api/accounts`                  | Create a new account   | ✅   |
| GET    | `/api/accounts`                  | Get all your accounts  | ✅   |
| GET    | `/api/accounts/balance/:accountId` | Get account balance  | ✅   |

### Transactions

| Method | Endpoint                             | Description                    | Auth        |
| ------ | ------------------------------------ | ------------------------------ | ----------- |
| POST   | `/api/transactions`                  | Transfer money between accounts | ✅         |
| POST   | `/api/transactions/system/initial-funds` | Seed initial funds         | ✅ (system user) |

---

## 📥 Sample Requests

### Register

```json
POST /api/auth/register
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securePassword123"
}
```

### Create Transaction

```json
POST /api/transactions
{
  "fromAccount": "<fromAccountId>",
  "toAccount": "<toAccountId>",
  "amount": 500,
  "idempotencyKey": "unique-key-12345"
}
```

### Sample Response

```json
{
  "message": "Transaction completed successfully",
  "transaction": {
    "_id": "...",
    "fromAccount": "...",
    "toAccount": "...",
    "amount": 500,
    "status": "COMPLETED"
  }
}
```

---

## 🔐 Security Highlights

- Passwords are hashed and excluded from queries by default (`select: false`)
- JWT expires in 3 days
- Logged-out tokens are stored in a blacklist
- Users can only view balances of their own accounts
- Transfers require both accounts to be `ACTIVE`

---## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

## 👤 Author

**Your Name**
- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [your-name](https://linkedin.com/in/your-name)

## 🔮 Future Improvements

- Role-based access control (admin / system user)
- Transaction history endpoint with pagination
- Rate limiting and request validation (Joi / Zod)
- Transaction reversal flow
- Unit and integration tests (Jest + Supertest)
- Docker support
- Swagger / OpenAPI documentation

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to open an issue or submit a pull request.

---

