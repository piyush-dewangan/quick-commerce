# Quick Commerce

A full-stack e-commerce web application with email-verified authentication, a persistent shopping cart, an admin product console, and image uploads. Built with **Node.js, Express, MongoDB and EJS**, organized in a clean **routes → controllers → services → models** architecture.

![Node.js](https://img.shields.io/badge/Node.js-18%2B-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose%207-47A248?logo=mongodb&logoColor=white)
![EJS](https://img.shields.io/badge/Views-EJS-B4CA65)
![License](https://img.shields.io/badge/License-ISC-blue)

<!--
Add 3 to 4 screenshots here (home page, product listing, cart, admin panel).
Example:
![Product listing](docs/listing.png)
-->

---

## Highlights

- **Complete auth lifecycle:** sign up, email verification, login, logout, forgot password and reset, all driven by tokenized email links.
- **Session-based access control:** protected routes sit behind a reusable `checklogin` middleware, and unverified accounts cannot reach the storefront.
- **Server-side pagination with "Load more":** the catalogue is served in pages of 5 using `skip` / `limit`, so the browser never downloads the whole product table.
- **Persistent per-user cart:** add, increment, decrement and remove items, select a subset with checkboxes, and see the live total at checkout.
- **Admin console:** separate admin login, product create / update / delete, and product image upload through Multer.
- **Data integrity on delete:** removing a product also pulls it from every user's cart in a single `updateMany` with `$pull`, so nobody is left with a dead item.
- **Transactional email:** verification and password-reset emails are sent through the Mailjet v3.1 API.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| Runtime | Node.js |
| Web framework | Express 4 |
| Database | MongoDB with Mongoose 7 |
| Templating | EJS (server-rendered views) |
| Sessions | express-session |
| File uploads | Multer (disk storage) |
| Email | Mailjet (`node-mailjet`) |
| Front end | HTML, CSS, vanilla JavaScript (XHR / fetch for cart and pagination) |

---

## Architecture

The code follows a layered design so that HTTP concerns, business logic and data access stay separate and individually testable.

```
Request
  │
  ▼
routes/          URL → handler mapping, middleware wiring
  │
  ▼
controllers/     request / response handling, flow control
  │
  ▼
services/        data-access helpers (searchUser, updateCartarray, getRangeProducts ...)
  │
  ▼
models           Mongoose schemas (users, products, mycarts, adminUsers)
```

```
quick-commerce/
├── main.js                  App bootstrap, middleware, upload endpoint
├── config.js                MongoDB connection
├── routes/
│   ├── login.js             /, /login, /createAccount, /forgot, /verifymail/:token ...
│   ├── mainpage.js          /loadmore/:page, /logout, /reset
│   ├── cart.js              /mycart, /addcart/:id, /checkout, /placeorder ...
│   └── admin.js             /checkadminlogin, /loginadmin, /updateproduct, /deleteP/:id
├── controllers/             Request handlers per feature area
├── services/                Reusable query / update functions per collection
├── middleware/checklogin.js Session guard for protected routes
├── methods/                 Mailjet email senders (verification, password reset)
├── views/                   EJS templates (home, mainpage, cart, checkout, admin ...)
├── public/                  Static assets and client-side scripts
├── uploads/                 Product images saved by Multer
├── users.js  products.js  mycart.js  adminusers.js    Mongoose models
└── package.json
```

---

## Data Model

| Collection | Fields |
| --- | --- |
| `users` | `username`, `email`, `phone`, `password`, `ev` (email verified), `mailToken` |
| `products` | `id`, `name`, `image`, `price`, `details`, `qty` |
| `mycarts` | `email`, `cart[]` (embedded product snapshot + `quantity` + `checkbox`) |
| `adminUsers` | `email`, `password` |

Each user gets exactly one cart document, created at sign up. Cart lines embed a snapshot of the product plus `quantity` and a `checkbox` flag, which makes the cart a single read and lets checkout total only the selected items.

---

## Key Flows

**Sign up and verification**
1. User registers. Email and phone are checked for duplicates.
2. The account is created unverified (`ev: false`) with a time-based `mailToken`, and an empty cart is provisioned.
3. Mailjet sends a verification link to `/verifymail/:token`, which flips `ev` to `true`.

**Forgot password**
1. User submits their email at `/forgot`.
2. A reset link to `/verifyforgot/:token` is emailed.
3. The link opens a session scoped to that user, who sets a new password at `/reset`.

**Catalogue and pagination**
1. `/` renders the first page for a logged-in, verified user.
2. "Load more" calls `/loadmore/:page`, which returns the next 5 products as JSON via `skip` and `limit` and stops at the last page.

**Cart and checkout**
1. `/cart/:id` adds a product or increments it if already present.
2. `/addcart/:id`, `/removecart/:id` and `/cartdelete/:id` adjust quantity or remove a line.
3. `/checkbox/:pid/:cbv` toggles selection, `/checkout` totals the selected lines, and `/placeorder` clears them from the cart.

**Admin**
1. Admin signs in at `/checkadminlogin`.
2. `/loginadmin` lists every product, with inline update and delete.
3. `/addnewproduct` posts a multipart form to `/upload`, where Multer stores the image and the product is created.

---

## Routes at a Glance

| Area | Method and path | Auth |
| --- | --- | --- |
| Auth | `GET /`, `POST /login`, `GET /newccount`, `POST /createAccount` | Public |
| Email | `GET /verifymail/:token`, `GET/POST /forgot`, `GET /verifyforgot/:token` | Public (token) |
| Catalogue | `GET /loadmore/:page`, `GET /productData`, `GET /logout` | Session |
| Account | `GET/POST /reset` | Session |
| Cart | `GET /cart/:id`, `/mycart`, `/addcart/:id`, `/removecart/:id`, `/cartdelete/:id`, `/checkbox/:pid/:cbv`, `/checkout`, `/placeorder` | `checklogin` |
| Admin | `GET/POST /checkadminlogin`, `GET /loginadmin`, `POST /updateproduct`, `GET /deleteP/:id`, `GET /addnewproduct`, `POST /upload` | Admin login |

---

## Getting Started

**Prerequisites:** Node.js 18+ and a MongoDB instance running locally on the default port.

```bash
# 1. Clone
git clone https://github.com/piyush-dewangan/quick-commerce.git
cd quick-commerce

# 2. Install dependencies
npm install

# 3. Start MongoDB (default: mongodb://0.0.0.0:27017), then run the app
node main.js
```

Open **http://localhost:5000**.

**Configuration**
- MongoDB connection string: `config.js`
- Mailjet credentials and sender address: `methods/sendEmail.js` and `methods/verifyforgot.js`
- Seed data to get started: `products/product.txt` contains sample products, and an admin account can be inserted into the `adminUsers` collection.

---

## Roadmap

I treat this project as a working foundation and have a clear list of what a production version needs:

- [ ] Hash passwords with bcrypt and replace plaintext comparison
- [ ] Move secrets (session secret, Mailjet keys, DB URI) into environment variables with `dotenv`
- [ ] Use cryptographically random, expiring tokens for email verification and password reset
- [ ] Change state-changing cart and admin actions from `GET` to `POST` / `PUT` / `DELETE`, and protect admin routes with a session guard
- [ ] Replace the embedded cart array with an `orders` collection and a real order history
- [ ] Payment gateway integration (Razorpay or Stripe)
- [ ] Input validation (Joi or express-validator), centralized error handling and request logging
- [ ] Automated tests with Jest and Supertest, plus a Dockerfile and CI pipeline
- [ ] Search, categories and filters on the catalogue

---

## What This Project Demonstrates

- Designing a layered Express application and keeping controllers thin
- Modeling relational-style data (users, carts, products) in MongoDB and keeping it consistent
- Building end-to-end auth flows with email verification and recovery
- Server-side pagination and incremental loading
- File upload handling and static asset serving
- Integrating a third-party transactional email API

---

## Author

**Piyush Dewangan**, Frontend Developer based in Noida, India.

- Portfolio: [piyushdewangan.vercel.app](https://piyushdewangan.vercel.app)
- GitHub: [@piyush-dewangan](https://github.com/piyush-dewangan)

---

## License

Released under the ISC License.
