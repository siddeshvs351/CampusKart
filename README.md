# CampusKart (Next.js + Express + MySQL + MongoDB)

A second-hand marketplace exclusively for PES University students to buy, sell, and exchange textbooks, electronics, cycles, and lab essentials inside the campus safely.

---

## 👥 Team Members

| Name | SRN | Department / Branch |
| :--- | :--- | :--- |
| **Siddesh V S** | `PES1UG24CS693` | CSE |
| **Shaizan Ulla Shariff** | `PES1UG24CS690` | CSE |

---

## 🛠️ Technology Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | Next.js 14 (App Router), React 18 | Modern, responsive app with Plus Jakarta Sans typography and custom radial controls |
| **Backend** | Node.js, Express.js | REST API with JWT authentication, cookie sessions & error handling |
| **Relational Database** | MySQL 8.0+ | Structured data: `users`, `listings`, `categories`, `listing_images`, `wishlist`, `email_otps` |
| **Document Database** | MongoDB | Fast, flexible collections: `conversations`, `messages`, `notifications` |
| **Mail System** | Nodemailer | Transactional HTML emails: Registration OTP verification, listing status & sale alerts |

---

## ✨ Key Features

1. **Public Catalog Browsing**: Visitors can explore the marketplace catalog, use 1-click category pills (Books, Electronics, Cycles, Furniture, etc.), search, and view listing details without needing an immediate login.
2. **PES Email Verification**: Registrations are restricted strictly to PES University student emails (`pes...@stu.pes.edu`). A 6-digit OTP code is sent via SMTP to verify mailbox ownership.
3. **Interactive Radial Arc Menu**: A bottom speed-dial fan-out menu offering instant access to:
   * ➕ **Create Listing** (`/sell`)
   * 🔔 **Notifications** (in-place modal with real-time updates & unread indicators)
   * 👤 **Student Profile** (in-place modal showing SRN, Branch, Batch, Phone, and quick links)
   * 🪟 **My Listings** (`/my-listings`)
   * 📦 **Closed Deals** (`/purchases`)
4. **Post an Item**: Students can list second-hand items with up to 4 photos, condition tags, description, and price.
5. **Admin Moderation**: Listings go through an admin review queue before going live. Admins can approve or reject with a feedback reason, triggering automated notifications and status emails.
6. **In-Campus Chat & Privacy**: Students can message sellers directly inside CampusKart. The seller's phone number remains protected until a student starts a chat about that listing.
7. **Mark as Sold**: Sellers can record which student purchased their item. That item is marked as sold and appears under the buyer's **Purchases** tab.

---

## 📐 Database Architecture & Documentation

CampusKart uses a **hybrid database model**, combining MySQL for strict relational data integrity and MongoDB for fast-growing flexible communication streams.

* Full architectural justification: [docs/DATABASE.md](docs/DATABASE.md)
* MySQL Schema source: [backend/sql/schema.sql](backend/sql/schema.sql)
* DBMS Project dataset script: [docs/campuskart_dbms_tables_and_data.sql](docs/campuskart_dbms_tables_and_data.sql)
* DBMS 42 Coursework queries: [docs/campuskart_dbms_queries.sql](docs/campuskart_dbms_queries.sql)

---

### 1. Entity-Relationship (ER) Diagram

The ER Diagram uses **Chen notation**, capturing all strong entities, weak entities, attributes, primary keys, and relationships.

![CampusKart ER Diagram](docs/CampusKart_ER_Diagram.png)

*High-resolution vector format:* [docs/CampusKart_ER_Diagram.svg](docs/CampusKart_ER_Diagram.svg)  
*Graphviz source:* [docs/CampusKart_ER_Diagram.dot](docs/CampusKart_ER_Diagram.dot)

---

### 2. Relational Schema Diagram

The Relational Schema illustrates the normalized relational structure (3NF/BCNF), table columns, primary keys (**PK**), and foreign key (**FK**) references.

![CampusKart Relational Schema](docs/CampusKart_Relational_Schema.png)

*High-resolution vector format:* [docs/CampusKart_Relational_Schema.svg](docs/CampusKart_Relational_Schema.svg)

---

### 3. Relational Tables (MySQL)

```
+----------------------------------------------------------------------------------------------------+
|                                      CAMPUSKART RELATIONAL SCHEMA                                   |
+----------------------------------------------------------------------------------------------------+
|  users (id [PK], email, srn, mobile, name, password_hash, role, branch, admission_year, ...)      |
|  categories (id [PK], name)                                                                        |
|  listings (id [PK], seller_id [FK], category_id [FK], title, description, price, condition, ...)   |
|  listing_images (listing_id [PK, FK], image_no [PK], image_path)  <-- Weak Entity                  |
|  wishlist (user_id [PK, FK], listing_id [PK, FK], created_at)     <-- M:N Relationship             |
|  email_otps (id [PK], email, otp_hash, attempts, expires_at, created_at)                           |
+----------------------------------------------------------------------------------------------------+
```

#### Detailed Table Specifications

#### `users`
Represents verified PES University students and platform administrators.
* **Primary Key:** `id` (INT AUTO_INCREMENT)
* **Columns:** `email` (UNIQUE), `srn` (UNIQUE), `mobile`, `name`, `password_hash`, `role` (`student` | `admin`), `branch`, `admission_year`, `is_verified`, `created_at`
* **Constraints:** `CONSTRAINT ck_users_mobile CHECK (mobile IS NULL OR mobile REGEXP '^[6-9][0-9]{9}$')`

#### `categories`
Fixed categories for organizing campus listings (Books, Electronics, Cycles & Vehicles, Furniture, Clothing, Lab Equipment, Stationery, Others).
* **Primary Key:** `id` (INT AUTO_INCREMENT)
* **Columns:** `name` (VARCHAR(60) UNIQUE)

#### `listings`
Items offered for sale by students, subject to administrative approval.
* **Primary Key:** `id` (INT AUTO_INCREMENT)
* **Foreign Keys:**
  * `seller_id` ➔ `users(id)` (`ON DELETE CASCADE`)
  * `category_id` ➔ `categories(id)`
  * `buyer_id` ➔ `users(id)` (`ON DELETE SET NULL`)
  * `reviewed_by` ➔ `users(id)` (`ON DELETE SET NULL`)
* **Columns:** `title`, `description`, `price`, `item_condition` (`new`, `like_new`, `good`, `fair`), `status` (`pending`, `approved`, `rejected`, `sold`), `reject_reason`, `sold_at`, `reviewed_at`, `created_at`, `updated_at`
* **Constraints:** `CONSTRAINT ck_listing_price CHECK (price >= 0)`

#### `listing_images` (Weak Entity)
An image only exists in relation to its parent listing.
* **Composite Primary Key:** (`listing_id`, `image_no`)
* **Foreign Key:** `listing_id` ➔ `listings(id)` (`ON DELETE CASCADE`)
* **Columns:** `image_path` (VARCHAR(255))

#### `wishlist` (Many-to-Many Relationship)
Tracks which students have saved items for later.
* **Composite Primary Key:** (`user_id`, `listing_id`)
* **Foreign Keys:**
  * `user_id` ➔ `users(id)` (`ON DELETE CASCADE`)
  * `listing_id` ➔ `listings(id)` (`ON DELETE CASCADE`)
* **Columns:** `created_at` (TIMESTAMP)

#### `email_otps`
Stores hashed 6-digit verification codes for student registrations.
* **Primary Key:** `id` (INT AUTO_INCREMENT)
* **Columns:** `email`, `otp_hash`, `attempts`, `expires_at`, `created_at`
* **Index:** `idx_otp_email (email)`

---

### 4. Document Collections (MongoDB)

Flexible, high-velocity communications are handled by MongoDB:

1. **`conversations`**:
   * Tracks one thread per `(listingId, buyerId)`.
   * Fields: `listingId`, `listingTitle`, `buyerId`, `buyerName`, `sellerId`, `sellerName`, `lastMessage`, `lastMessageAt`, `unreadBuyer`, `unreadSeller`.
2. **`messages`**:
   * Individual chat messages between buyers and sellers.
   * Fields: `conversationId`, `senderId`, `text`, `createdAt`.
3. **`notifications`**:
   * In-app bell alerts for messages, approval, rejection, and sales.
   * Fields: `userId`, `type` (`message`, `listing_approved`, `listing_rejected`, `item_sold`), `text`, `link`, `read`, `ref`, `createdAt`.

---

## 🚀 Getting Started (Windows PowerShell)

### Prerequisites
* **Node.js** 18+
* **MySQL 8** running locally
* **MongoDB** running locally

### 1) Backend Setup
```powershell
cd backend

# Copy environment template
copy .env.example .env

# Edit .env:
# - Set MYSQL_PASSWORD (your local MySQL root password)
# - Set JWT_SECRET (a secure random string)
# - Configure SMTP credentials (e.g. Gmail App Password) or leave blank for terminal dev mode

npm install

# Seed the admin user (admin@campuskart.local / Admin@12345)
npm run seed:admin

# Test your SMTP email configuration
npm run test:email

# Start backend API (http://localhost:5000)
npm run dev
```

### 2) Frontend Setup (Open a new terminal)
```powershell
cd frontend
npm install

# Start Next.js dev server (http://localhost:3000)
npm run dev
```

---

## 📧 Email Configuration

In `backend/.env`, configure your preferred mail provider:

```env
# Example: Gmail SMTP with App Password
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=xxxx xxxx xxxx xxxx
MAIL_FROM=CampusKart <your-email@gmail.com>
```

> **Note:** If `SMTP_HOST` is left empty, CampusKart runs in **Development Console Mode**, automatically printing the 6-digit OTP verification codes directly to the backend terminal for local testing.

---

## 📁 Project Directory Structure

```
campuskart/
├── backend/
│   ├── sql/schema.sql              # MySQL DDL table schemas, checks, and foreign keys
│   ├── src/
│   │   ├── config/                 # mysql.js & mongo.js database connectors
│   │   ├── middleware/             # auth.js (JWT verify + optionalAuth), upload.js (Multer)
│   │   ├── models/                 # Mongoose schemas (Chat.js, Notification.js)
│   │   ├── routes/                 # admin.js, auth.js, chats.js, listings.js, notifications.js
│   │   ├── utils/                  # college.js (PES email parser), email.js (Nodemailer), testEmail.js
│   │   ├── seedAdmin.js            # Admin creation script (npm run seed:admin)
│   │   └── server.js               # Express application entry point
│   ├── .env                        # Environment configuration (active)
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── app/                    # Next.js App Router: layout.jsx, globals.css, one folder per route (login, sell, listings/[id], ...)
│   │   ├── components/             # Navbar.jsx, Footer.jsx, RadialMenu.jsx, ListingCard.jsx, SoldButton.jsx
│   │   ├── lib/                    # api.js, auth.jsx (Route guards), format.js (Rupee, Categories)
│   │   └── views/                  # Page screens: Browse, Listing, Login, Register, Verify, Sell, MyListings, Wishlist, Purchases, Chats, Admin
│   ├── next.config.js              # Proxies /api and /uploads to the Express backend
│   ├── jsconfig.json               # "@/..." alias -> src/
│   └── package.json
└── docs/
    ├── CampusKart_ER_Diagram.png   # Full ER diagram (Chen notation)
    ├── CampusKart_ER_Diagram.svg   # High-resolution vector ER diagram
    ├── CampusKart_Relational_Schema.png # Relational schema diagram
    ├── CampusKart_Relational_Schema.svg # High-resolution vector relational schema
    ├── CampusKart_ER_Diagram.dot   # Graphviz ER diagram source
    ├── DATABASE.md                 # MySQL vs. MongoDB architectural split explanation
    ├── campuskart_dbms_tables_and_data.sql # Workbench script with mock dataset
    └── campuskart_dbms_queries.sql # Coursework SQL queries
```

---

## 🎓 DBMS Mini Project Scripts

A standalone database script is provided in `docs/campuskart_dbms_tables_and_data.sql`. It creates a database named `campuskart_dbms` with sample student records and listings. Test queries can be executed individually from `docs/campuskart_dbms_queries.sql`.
