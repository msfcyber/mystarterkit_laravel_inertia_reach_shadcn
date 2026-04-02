# PRD (Product Requirement Document)
## WebApp Billing Hoster (WHMCS-like)

---

## 1. Overview
WebApp ini adalah sistem billing hosting yang terintegrasi dengan berbagai panel (cPanel, Plesk, DirectAdmin, Proxmox, dll) untuk mengelola penjualan produk hosting, domain, VPS, dan layanan terkait.

Target user:
- Customer
- Admin
- Customer Service (CS)
- Technical Support

Tujuan:
- Otomatisasi provisioning layanan
- Manajemen billing & invoice
- Integrasi panel hosting & payment gateway
- Self-service untuk customer

---

## 2. Tech Stack (Rekomendasi)
- Backend: Laravel
- Frontend: Inertia + Vue / React
- Database: MySQL / PostgreSQL
- Queue: Redis
- Payment Gateway: Midtrans / Mayar / Xendit
- Email: SMTP / API (Sendgrid, Mailgun)

---

## 3. Role & Permission
### Role:
- Admin
- CS
- Technical Support
- Customer

### Akses:
| Module            | Admin | CS | Tech | Customer |
|------------------|------|----|------|----------|
| Dashboard        | ✔    | ✔  | ✔    | ✔        |
| Product          | ✔    | ✖  | ✖    | View     |
| Order            | ✔    | ✔  | ✖    | ✔        |
| Ticket           | ✔    | ✔  | ✔    | ✔        |
| Server           | ✔    | ✖  | ✔    | ✖        |
| Billing          | ✔    | ✔  | ✖    | ✔        |

---

## 4. Modules & Features

### 4.1 Dashboard
- Admin: statistik revenue, user, order, ticket
- CS: ticket & order monitoring
- Tech: server status, provisioning
- Customer: layanan aktif, invoice, ticket

---

### 4.2 Authentication
- Login / Register
- Social login (Google)
- Forgot password
- Email verification

---

### 4.3 Product Management
- Kategori produk
- Produk:
  - Shared Hosting
  - VPS
  - Dedicated Server
  - Reseller Hosting
- Pricing (monthly, yearly)
- Addon (SSL, backup, dll)

---

### 4.4 Domain Management
- Register domain
- Transfer domain
- Renew domain
- WHOIS check
- Integrasi registrar API

---

### 4.5 Server Management
- Integrasi:
  - cPanel
  - Plesk
  - DirectAdmin
  - Proxmox (VPS)
- Auto provisioning:
  - Create account
  - Suspend
  - Terminate
- Monitoring server

---

### 4.6 Service Management
- List layanan customer
- Status:
  - Active
  - Suspended
  - Cancelled
- Upgrade / downgrade
- Auto suspend jika overdue

---

### 4.7 Order / Shop
- Home shop page
- Custom menu
- Product listing
- Cart system
- Checkout

---

### 4.8 Invoice & Billing
- Generate invoice otomatis
- Status:
  - Unpaid
  - Paid
  - Overdue
- Reminder email
- Pajak (PPN optional)

---

### 4.9 Payment Gateway
- Integrasi:
  - Midtrans
  - Mayar
  - Xendit
- Payment method:
  - VA
  - E-wallet
  - QRIS
- Callback handling

---

### 4.10 Transaction
- History transaksi
- Payment log
- Refund handling

---

### 4.11 Ticket System
- Create ticket
- Reply ticket
- Status:
  - Open
  - Answered
  - Closed
- Assignment ke CS / Tech

---

### 4.12 Email System (SMTP)
- SMTP config
- Template email:
  - Invoice
  - Payment success
  - Ticket reply
- Queue email

---

### 4.13 General Settings
- Company profile
- Currency
- Tax
- Invoice prefix
- Branding

---

## 5. Workflow

### 5.1 User Registration
1. User register (manual / Google)
2. Email verification
3. Login ke dashboard

---

### 5.2 Order Flow
1. User buka shop
2. Pilih produk
3. Tambah ke cart
4. Checkout
5. Generate invoice
6. User melakukan pembayaran
7. Payment callback diterima
8. Status invoice = Paid
9. Sistem auto provisioning service

---

### 5.3 Provisioning Flow
1. Order paid
2. Sistem kirim API ke panel
3. Panel create account
4. Sistem simpan credential
5. Email dikirim ke user

---

### 5.4 Billing Flow
1. Sistem generate invoice periodik
2. Kirim email reminder
3. Jika overdue:
   - Suspend service
4. Jika paid:
   - Unsuspend service

---

### 5.5 Ticket Flow
1. User buat ticket
2. Masuk ke CS
3. Jika teknikal → assign ke Tech
4. Reply sampai selesai
5. Close ticket

---

### 5.6 Domain Flow
1. User cek domain
2. Pilih domain
3. Checkout
4. Payment
5. Register ke registrar API

---

## 6. Database (High Level)

### Tables:
- users
- roles
- products
- categories
- orders
- order_items
- services
- invoices
- transactions
- tickets
- ticket_replies
- servers
- domains
- settings

---

## 7. API Integration
- Panel API (cPanel, Plesk, DirectAdmin, Proxmox)
- Domain Registrar API
- Payment Gateway API

---

## 8. Security
- RBAC (Role Based Access Control)
- CSRF protection
- Rate limiting
- Audit log

---

## 9. Future Enhancement
- Multi-currency
- Affiliate system
- Promo code
- Usage-based billing (VPS)
- Monitoring (Grafana integration)

---

## 10. Notes
- Gunakan queue untuk provisioning & email
- Gunakan webhook untuk payment
- Modular architecture agar mudah tambah panel baru

---

END PRD

