# University Organization Fund Management System

Production-style starter framework for a simulated university finance workflow with:
- **Backend**: Node.js, Express.js, PostgreSQL, Prisma ORM, JWT, RBAC
- **Frontend**: React (Vite), TailwindCSS, Axios, React Router

---

## 0) Install Required Apps First

Before you can run anything, you need three programs installed on your computer.

### Node.js (runs the backend and frontend tooling)

| Platform | Steps |
|----------|-------|
| **Windows** | Download the LTS installer from https://nodejs.org → run it → keep all defaults → restart your terminal |
| **macOS** | `brew install node` (requires Homebrew) **or** download from https://nodejs.org |
| **Linux (Ubuntu/Debian)** | `sudo apt update && sudo apt install -y nodejs npm` |

Verify: `node -v` should print something like `v20.x.x`

### PostgreSQL (the database)

| Platform | Steps |
|----------|-------|
| **Windows** | Download the installer from https://www.postgresql.org/download/windows/ → install with default options → **remember the password you set for the `postgres` user** |
| **macOS** | `brew install postgresql@16 && brew services start postgresql@16` |
| **Linux (Ubuntu/Debian)** | `sudo apt install -y postgresql postgresql-contrib && sudo systemctl start postgresql` |

Verify: `psql --version`

### Git (version control)

| Platform | Steps |
|----------|-------|
| **Windows** | Download from https://git-scm.com/download/win → install with defaults |
| **macOS** | `brew install git` or it ships with Xcode Command Line Tools (`xcode-select --install`) |
| **Linux** | `sudo apt install -y git` |

Verify: `git --version`

---

## 1) Project Structure — Annotated Guide

Every file and folder in the project, with a plain-English explanation of what it does.

```text
.
├── client/                          ← React frontend (the web page you open in a browser)
│   ├── .env.example                 ← Template for frontend environment variables (copy to .env)
│   ├── index.html                   ← Single HTML entry-point; React mounts into <div id="root">
│   ├── package.json                 ← Lists frontend npm packages and scripts (npm run dev, build…)
│   ├── postcss.config.js            ← PostCSS config required by TailwindCSS
│   ├── tailwind.config.js           ← TailwindCSS settings (colors, fonts, custom classes)
│   ├── vite.config.js               ← Vite bundler config: dev server port, proxy, build output
│   └── src/
│       ├── components/              ← Reusable UI building blocks shared across pages
│       │   ├── AppLayout.jsx        ← Wrapper that adds the Navbar to every authenticated page
│       │   ├── Navbar.jsx           ← Top navigation bar with links and logout button
│       │   ├── OrganizationCard.jsx ← Card UI showing one organization's name and balance
│       │   ├── ProtectedRoute.jsx   ← Redirects unauthenticated users away from private pages
│       │   ├── TransactionForm.jsx  ← Form to submit a new expense request (amount, description, receipt)
│       │   └── TransactionTable.jsx ← Table listing transactions with approve/reject buttons
│       ├── context/
│       │   └── AuthContext.jsx      ← React Context: stores logged-in user + token, exposes login/logout helpers
│       ├── pages/                   ← One file per "screen" the user can navigate to
│       │   ├── DashboardPage.jsx    ← Home screen after login: shows balance summary
│       │   ├── LoginPage.jsx        ← Email + password login form
│       │   ├── NotFoundPage.jsx     ← 404 page shown for unknown URLs
│       │   ├── OrganizationsPage.jsx← SUPER_ADMIN: list/create orgs and allocate budgets
│       │   ├── RegisterPage.jsx     ← OTP-based registration form
│       │   └── TransactionsPage.jsx ← Submit expenses and view transaction history
│       ├── routes/
│       │   └── AppRouter.jsx        ← Defines all URL routes and wraps them with ProtectedRoute
│       ├── services/
│       │   └── api.js               ← Axios instance pre-configured with the backend base URL and auth header
│       ├── index.css                ← Global CSS: imports Tailwind base/components/utilities
│       └── main.jsx                 ← React entry point: renders <App> into index.html's root div
│
├── server/                          ← Node.js/Express backend (the API)
│   ├── .env.example                 ← Template for backend environment variables (copy to .env)
│   ├── package.json                 ← Lists backend npm packages and scripts (dev, prisma:migrate…)
│   ├── uploads/                     ← Stores uploaded receipt image/PDF files
│   │   └── .gitkeep                 ← Empty file so Git tracks the otherwise-empty folder
│   └── src/
│       ├── app.js                   ← Creates the Express app: attaches middleware + mounts all routers
│       ├── server.js                ← Entry point: starts the HTTP server on the configured PORT
│       ├── controllers/             ← Handle HTTP request/response; call the matching service function
│       │   ├── authController.js    ← OTP request, OTP verify, resend OTP, login, get-me endpoints
│       │   ├── organizationController.js ← Create org, list orgs, allocate funds, delete, view balance
│       │   └── transactionController.js  ← Submit expense, history, approve, reject
│       ├── middleware/              ← Functions that run before a controller on every matching request
│       │   ├── authMiddleware.js    ← Checks for a valid JWT token; blocks unauthenticated requests
│       │   ├── errorMiddleware.js   ← Catches any thrown error and returns a JSON error response
│       │   ├── roleMiddleware.js    ← Checks the user's role; blocks requests from unauthorized roles
│       │   └── uploadMiddleware.js  ← Multer config: handles multipart/form-data file uploads
│       ├── models/
│       │   └── schema.prisma        ← Prisma schema: defines DB tables (User, Organization, Transaction)
│       ├── routes/                  ← Map URL paths to the correct controller functions
│       │   ├── authRoutes.js        ← /api/auth/* routes
│       │   ├── organizationRoutes.js← /api/organizations/* routes
│       │   └── transactionRoutes.js ← /api/transactions/* routes
│       ├── services/                ← Pure business logic; no HTTP objects, only data in/out
│       │   ├── authService.js       ← OTP generation/validation, user creation, password hashing, JWT issue
│       │   ├── organizationService.js ← Create org, allocate/deduct balance, fetch org data
│       │   └── transactionService.js  ← Create transaction, approve (deduct balance), reject, scoped history
│       └── utils/                   ← Shared helper modules used across the app
│           ├── ApiError.js          ← Custom error class with HTTP status code (throw new ApiError(404, "Not found"))
│           ├── asyncHandler.js      ← Wraps async controller functions so errors auto-forward to errorMiddleware
│           ├── jwt.js               ← Helpers: signToken(payload) and verifyToken(token)
│           └── prisma.js            ← Creates and exports a single shared Prisma client instance
│
└── README.md                        ← This file
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

## 5) Step-by-Step Setup

### 5.1 Get the code

Clone the repository (if you haven't already):

```bash
git clone https://github.com/K-dot22/BANK_ing.git
cd BANK_ing
```

Then switch to the branch that contains the full application code:

```bash
git fetch --all
git checkout cursor/university-fund-system-framework-a3f3
git pull
```

Verify the backend schema file is present:

```bash
# Mac / Linux / Git Bash on Windows
ls server/src/models/schema.prisma
```

```powershell
# Windows PowerShell
Test-Path .\server\src\models\schema.prisma
```

---

### 5.2 Create the PostgreSQL database

**Windows** – open the "SQL Shell (psql)" program that was installed with PostgreSQL, log in as `postgres`, then run:

```sql
CREATE DATABASE university_fund_db;
\q
```

**Mac / Linux** – open a terminal:

```bash
psql -U postgres -c "CREATE DATABASE university_fund_db;"
```

> If you get "role postgres does not exist" on Mac, replace `postgres` with your macOS username.

---

### 5.3 Backend setup

```bash
# Mac / Linux / Git Bash
cd server
cp .env.example .env
```

```powershell
# Windows PowerShell
cd .\server
Copy-Item .env.example .env
```

Open `server/.env` in any text editor and fill in your values:

```env
PORT=5000
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/university_fund_db?schema=public"
JWT_SECRET="replace_with_a_long_random_string"
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

> **Special characters in your DB password?** URL-encode them before pasting:
> - `@` → `%40` &nbsp; `#` → `%23` &nbsp; `%` → `%25` &nbsp; `:` → `%3A` &nbsp; `/` → `%2F` &nbsp; `?` → `%3F` &nbsp; `&` → `%26` &nbsp; `=` → `%3D`
>
> Full reference: https://www.w3schools.com/tags/ref_urlencode.ASP
>
> Example — password `Kathmandu112@`:
> ```
> DATABASE_URL="postgresql://postgres:Kathmandu112%40@localhost:5432/university_fund_db?schema=public"
> ```

Install packages, run database migrations, and start the server:

```bash
npm install
npm run prisma:migrate   # creates DB tables
npm run prisma:generate  # generates the Prisma client
npm run dev              # starts the backend with auto-reload
```

If you see `Missing script: prisma:migrate`, use the direct commands instead:

```bash
npx prisma migrate dev --name init --schema src/models/schema.prisma
npx prisma generate --schema src/models/schema.prisma
node src/server.js
```

Backend is running at **http://localhost:5000**. Quick check:

```bash
curl http://localhost:5000/api/health
```

---

### 5.4 Frontend setup

Open a **second** terminal (keep the backend running in the first):

```bash
# Mac / Linux / Git Bash
cd ../client
cp .env.example .env
npm install
npm run dev
```

```powershell
# Windows PowerShell
cd ..\client
Copy-Item .env.example .env
npm install
npm run dev
```

`client/.env` should contain:

```env
VITE_API_BASE_URL="http://localhost:5000/api"
```

Frontend is running at **http://localhost:5173** — open it in your browser.

---

### 5.5 First-run walkthrough

1. Open **http://localhost:5173** and click **Register**
2. Register the first account — choose role **SUPER_ADMIN**
3. Log in as SUPER_ADMIN
4. Go to **Organizations** → create an organization → allocate a budget
5. Note the `organizationCode` shown in the Organizations tab
6. Register a second account as **ORG_ADMIN** or **MEMBER** using that code
7. Log in as that user, go to **Transactions** → submit an expense request (optionally attach a receipt)
8. Log back in as SUPER_ADMIN → approve or reject the request
9. Confirm the organization balance decreases only when approved

---

## 6) Environment Variables Reference

### Backend (`server/.env`)
```env
PORT=5000                          # Port the Express server listens on
DATABASE_URL="postgresql://..."    # Full Prisma connection string to your PostgreSQL DB
JWT_SECRET="..."                   # Secret used to sign/verify JWT tokens — keep this private
JWT_EXPIRES_IN="1d"                # How long a login token stays valid (1d = 1 day)
CLIENT_URL="http://localhost:5173" # Frontend origin allowed by CORS
OTP_EXPIRES_MINUTES=10             # How many minutes an email OTP is valid
SMTP_HOST="smtp.gmail.com"         # Email server hostname
SMTP_PORT=587                      # Email server port (587 = STARTTLS)
SMTP_SECURE=false                  # true only if using port 465 (SSL)
SMTP_USER="your@gmail.com"         # Gmail address used to send OTP emails
SMTP_PASS="your_app_password"      # Gmail App Password (not your real password — see Google docs)
SMTP_FROM="..."                    # Display name + address shown in the From field
```

### Frontend (`client/.env`)
```env
VITE_API_BASE_URL="http://localhost:5000/api"  # URL of the running backend API
```

---

## 7) Common Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Missing script: prisma:migrate` | You are on the wrong Git branch | `git checkout cursor/university-fund-system-framework-a3f3 && git pull` |
| `Could not load --schema … file not found` | `server/src/models/schema.prisma` is missing | Same as above; verify the file exists after switching branch |
| `P1000 Authentication failed` | Wrong DB credentials in `DATABASE_URL` | Update `server/.env`; URL-encode special characters |
| `ECONNREFUSED` connecting to DB | PostgreSQL is not running | Windows: start "PostgreSQL" service in Services panel · Mac: `brew services start postgresql@16` · Linux: `sudo systemctl start postgresql` |
| Frontend: `"undefined" is not valid JSON` | Stale data in browser storage | Open browser DevTools console and run: `localStorage.removeItem("ufms_user"); localStorage.removeItem("ufms_token");` then reload |
| `npm: command not found` | Node.js not installed or not on PATH | Re-install Node.js and restart your terminal |
| `psql: command not found` | PostgreSQL not on PATH | **Windows**: add `C:\Program Files\PostgreSQL\<version>\bin` to your system PATH · **Mac**: run `brew link postgresql@16` or use `/opt/homebrew/opt/postgresql@16/bin/psql` · **Linux**: run `sudo apt install -y postgresql-client` |

---

## 8) Core Business Logic Notes

- Expense requests are created as `PENDING`.
- Organization balance is **deducted only when a SUPER_ADMIN approves** a request.
- Rejections can include an optional comment.
- Transaction history is automatically scoped by user role.
- Receipts are uploaded to `server/uploads/` and served at `/uploads/<filename>`.

---

## 9) Quick-Reference Command Cheat Sheet

```bash
# ── Start everything ──────────────────────────────────────────
cd server && npm run dev          # terminal 1: backend  → http://localhost:5000
cd client && npm run dev          # terminal 2: frontend → http://localhost:5173

# ── Database ──────────────────────────────────────────────────
npm run prisma:migrate            # apply schema changes to DB
npm run prisma:generate           # regenerate Prisma client after schema edits
npx prisma studio                 # open a visual DB browser at http://localhost:5555

# ── Useful checks ─────────────────────────────────────────────
curl http://localhost:5000/api/health          # confirm backend is running
node -v                                        # check Node version
psql -U postgres -c "\l"                       # list PostgreSQL databases
```
