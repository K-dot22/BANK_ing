# University Organization Fund Management System

Production-style starter framework for a simulated university finance workflow with:
- **Backend**: Node.js, Express.js, PostgreSQL, Prisma ORM, JWT, RBAC
- **Frontend**: React (Vite), TailwindCSS, Axios, React Router

## 1) Project Structure

```text
.
├── client
│   ├── .env.example
│   ├── index.html
│   ├── package.json
│   ├── postcss.config.js
│   ├── tailwind.config.js
│   ├── vite.config.js
│   └── src
│       ├── components
│       │   ├── AppLayout.jsx
│       │   ├── Navbar.jsx
│       │   ├── OrganizationCard.jsx
│       │   ├── ProtectedRoute.jsx
│       │   ├── TransactionForm.jsx
│       │   └── TransactionTable.jsx
│       ├── context
│       │   └── AuthContext.jsx
│       ├── pages
│       │   ├── DashboardPage.jsx
│       │   ├── LoginPage.jsx
│       │   ├── NotFoundPage.jsx
│       │   ├── OrganizationsPage.jsx
│       │   ├── RegisterPage.jsx
│       │   └── TransactionsPage.jsx
│       ├── routes
│       │   └── AppRouter.jsx
│       ├── services
│       │   └── api.js
│       ├── index.css
│       └── main.jsx
├── server
│   ├── .env.example
│   ├── package.json
│   ├── uploads
│   │   └── .gitkeep
│   └── src
│       ├── app.js
│       ├── server.js
│       ├── controllers
│       │   ├── authController.js
│       │   ├── organizationController.js
│       │   └── transactionController.js
│       ├── middleware
│       │   ├── authMiddleware.js
│       │   ├── errorMiddleware.js
│       │   ├── roleMiddleware.js
│       │   └── uploadMiddleware.js
│       ├── models
│       │   └── schema.prisma
│       ├── routes
│       │   ├── authRoutes.js
│       │   ├── organizationRoutes.js
│       │   └── transactionRoutes.js
│       ├── services
│       │   ├── authService.js
│       │   ├── organizationService.js
│       │   └── transactionService.js
│       └── utils
│           ├── ApiError.js
│           ├── asyncHandler.js
│           ├── jwt.js
│           └── prisma.js
└── README.md
```

---

## 2) Prisma Schema

Located at: `server/src/models/schema.prisma`

- Includes required models and fields:
  - **Users**: `id`, `name`, `email`, `password`, `role`, `organizationId`
  - **Organizations**: `id`, `name`, `totalAllocated`, `currentBalance`
  - **Transactions**: `id`, `amount`, `description`, `status`, `submittedBy`, `organizationId`, `createdAt`
- Adds practical extensions:
  - `receiptUrl` for uploaded receipts
  - `reviewedBy` and `reviewComment` for approval/rejection audit trail

---

## 3) REST API Endpoints

Base URL: `http://localhost:5000/api`

### Auth
- `POST /auth/request-registration-otp` - Request OTP to begin registration
- `POST /auth/verify-registration-otp` - Verify OTP and create user account
- `POST /auth/resend-registration-otp` - Resend OTP for pending registration
- `POST /auth/login` - Login user
- `GET /auth/me` - Get current user profile (authenticated)

### Organizations
- `POST /organizations` - Create organization (**SUPER_ADMIN**)
- `GET /organizations` - List organizations (**SUPER_ADMIN**)
- `POST /organizations/:id/allocate` - Allocate funds (**SUPER_ADMIN**)
- `DELETE /organizations/:id` - Delete organization if no linked users/transactions (**SUPER_ADMIN**)
- `GET /organizations/:id/balance` - View balance (super admin or same-org user)
- `GET /organizations/me/balance` - View own organization balance (**ORG_ADMIN**, **MEMBER**)

### Transactions
- `POST /transactions` - Submit expense request (multipart, optional `receipt` file) (**ORG_ADMIN**, **MEMBER**)
- `GET /transactions/history` - Transaction history (role-scoped)
- `PATCH /transactions/:id/approve` - Approve pending expense (**SUPER_ADMIN**)
- `PATCH /transactions/:id/reject` - Reject pending expense with optional `comment` (**SUPER_ADMIN**)

### Health
- `GET /api/health` - API health check

---

## 4) Role-Based Access Rules

- **SUPER_ADMIN (University Finance Office)**
  - Create organizations
  - Allocate budgets
  - Approve/reject expenses
  - View all transactions

- **ORG_ADMIN (Treasurer)**
  - View own organization balance
  - Submit expense requests
  - View organization transaction history

- **MEMBER**
  - Submit expense requests
  - View only own submitted transactions

---

## 5) Step-by-Step Setup (Fresh Install from Scratch)

### 5.1 Prerequisites
- [Node.js LTS](https://nodejs.org/) (v18 or higher recommended)
- [PostgreSQL](https://www.postgresql.org/download/) running locally (v14 or higher)
- [Git](https://git-scm.com/downloads)

---

### 5.2 Clone the repository

```bash
git clone https://github.com/K-dot22/BANK_ing.git
cd BANK_ing
```

On PowerShell (Windows):

```powershell
git clone https://github.com/K-dot22/BANK_ing.git
cd BANK_ing
```

Then verify required backend files are present:

```bash
ls server/src/models/schema.prisma
```

On PowerShell:

```powershell
Test-Path .\server\src\models\schema.prisma
```

---

### 5.3 PostgreSQL setup

Create the database:

```sql
CREATE DATABASE university_fund_db;
```

If your PostgreSQL user is `postgres`, ensure you know its password.

---

### 5.4 Backend setup and run

```bash
cd server
cp .env.example .env
```

On PowerShell:

```powershell
cd .\server
copy .env.example .env
```

Edit `server/.env`:

```env
PORT=5000
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/university_fund_db?schema=public"
JWT_SECRET="replace_with_a_strong_secret"
JWT_EXPIRES_IN="1d"
CLIENT_URL="http://localhost:5173"
```

If your DB password has special URL characters, encode them:
- `@` -> `%40`
- `#` -> `%23`
- `%` -> `%25`

Example password `Kathmandu112@`:

```env
DATABASE_URL="postgresql://postgres:Kathmandu112%40@localhost:5432/university_fund_db?schema=public"
```

Install dependencies and run migrations:

```bash
npm install
npm run prisma:migrate
npm run prisma:generate
npm run dev
```

If scripts are missing in your local copy, you are on the wrong branch/version.  
Fallback direct commands:

```bash
npx prisma migrate dev --name init --schema src/models/schema.prisma
npx prisma generate --schema src/models/schema.prisma
node src/server.js
```

Backend runs at `http://localhost:5000`.

Health check:

```bash
curl http://localhost:5000/api/health
```

---

### 5.5 Frontend setup and run

```bash
cd ../client
cp .env.example .env
npm install
npm run dev
```

On PowerShell:

```powershell
cd ..\client
copy .env.example .env
npm install
npm run dev
```

Frontend runs at `http://localhost:5173`.

---

### 5.6 First manual verification flow

1. Register first account as `SUPER_ADMIN`
2. Login as super admin
3. Create organization
4. Allocate organization funds
5. Register `ORG_ADMIN` or `MEMBER` with `organizationCode` (shown in Organizations tab)
6. Submit expense request (optional receipt)
7. Approve/reject as `SUPER_ADMIN`
8. Confirm balance decreases only on approve

---

## 6) Environment Variables

### Backend (`server/.env`)
```env
PORT=5000
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/university_fund_db?schema=public"
JWT_SECRET="replace_with_a_strong_secret"
JWT_EXPIRES_IN="1d"
CLIENT_URL="http://localhost:5173"
OTP_EXPIRES_MINUTES=10
SMTP_HOST="smtp.gmail.com"
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER="your_email@gmail.com"
SMTP_PASS="your_gmail_app_password"
SMTP_FROM="University Fund System <your_email@gmail.com>"
```

If your password includes special characters (e.g., `@`), URL-encode it:
- `Kathmandu112@` -> `Kathmandu112%40`

### Frontend (`client/.env`)
```env
VITE_API_BASE_URL="http://localhost:5000/api"
```

---

## 7) Common Troubleshooting

- **`Missing script: prisma:migrate`**
  - Run `git pull` to make sure you have the latest version of the repository.
  - Then run `npm run` in `server` to verify scripts exist.

- **`Could not load --schema ... file not found`**
  - Ensure `server/src/models/schema.prisma` exists.
  - Verify with `Test-Path .\src\models\schema.prisma` (PowerShell) or `ls src/models/schema.prisma`.

- **`P1000 Authentication failed`**
  - Your `DATABASE_URL` credentials are incorrect.
  - Update `server/.env` with the correct password and URL-encode special characters.

- **Frontend error: `"undefined" is not valid JSON`**
  - Clear browser storage once:
    - `localStorage.removeItem("ufms_user")`
    - `localStorage.removeItem("ufms_token")`
    - reload page

---

## 8) Core Business Logic Notes

- Expense requests are created as `PENDING`.
- Organization balance is **deducted only when a SUPER_ADMIN approves** a request.
- Rejections can include an optional comment.
- Transaction history is automatically scoped by user role.
- Receipts are uploaded to `server/uploads` and served from `/uploads/...`.
