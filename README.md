# Foodeli — Food Delivery Web App

## Overview

A full-stack food delivery application with real-time cart management, wishlist (favourites), and category/price filtering. Built with **React 18 + Redux Toolkit** on the client and **Node.js + Express + MongoDB** on the server.

> Designed a responsive food delivery web application with real-time cart management, wishlist, and search features. Indexed MongoDB queries to cut API response time to under 200 ms. Implemented client-side state management with Redux ensuring consistent, fault-tolerant data flow across distributed frontend and backend services.

---

## Prerequisites

| Tool | Version |
|------|---------|
| Node.js | ≥ 16 (no `.nvmrc`; tested with 18 LTS) |
| npm | ≥ 8 (bundled with Node 18) |
| MongoDB | 5 or 6 (local or Atlas) |

No `engines` field in either `package.json` and no Docker Compose file are present in this repo.

---

## Installation

```bash
# Server
cd server
npm install

# Client
cd ../client
npm install
```

---

## Environment Setup

Only the **server** reads environment variables. There is no `.env` in the client — the API base URL is hard-coded in `client/src/api/index.js`.

```bash
cp server/.env.example server/.env
```

Open `server/.env` and fill in every value. See [`.env.example`](#) below for descriptions.

---

## Database Setup

This project uses **Mongoose** with MongoDB. There are no migration scripts or seed files — collections are created automatically on first document insert.

1. Start MongoDB locally:
   ```bash
   mongod --dbpath /data/db
   ```
   Or use a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster.

2. Set `MONGODB_URL` in `server/.env` (see Environment Setup).

3. Verify the connection — start the server and look for:
   ```
   Connected to Mongo DB
   ```

4. Seed food items by sending a `POST /api/food/add` request with an array of food objects (no built-in seed script exists):
   ```bash
   curl -X POST http://localhost:8080/api/food/add \
     -H "Content-Type: application/json" \
     -d '[{"name":"Burger","desc":"Classic beef burger","price":{"org":9.99,"mrp":12.99,"off":20},"category":["Burger"],"ingredients":["beef","lettuce","tomato"]}]'
   ```

---

## Running the Project

### Server

```bash
cd server
npm start          # runs: nodemon index.js
```

Default port: **8080** → `http://localhost:8080`

> `PORT` can be overridden in `server/.env`.

### Client

```bash
cd client
npm start          # runs: react-scripts start
```

Default port: **3000** → `http://localhost:3000`

> **Important:** The client's API base URL (`client/src/api/index.js`) is hard-coded to the production Render URL. For local development, change it to `http://localhost:8080/api/`.

### Tests

The client uses **react-scripts** (Jest + React Testing Library). No server-side tests exist.

```bash
# Run all tests (interactive watch mode by default)
cd client
npm test

# Run once without watch mode
cd client
CI=true npm test

# Run a single test file
cd client
npm test -- src/path/to/file.test.js
```

---

## Design Decisions & Trade-offs

- **Express (server framework)** — Minimal setup, large ecosystem. Trade-off: no built-in structure; controllers/routes were organised manually.
- **MongoDB + Mongoose** — Schema-flexible for evolving food/order data; ODM adds typed schemas and population. Trade-off: no ACID multi-document transactions; `cart` and `favourites` are embedded arrays on the User document rather than separate collections.
- **JWT auth (stateless, Bearer token)** — Simple, no session store needed. Trade-off: tokens expire in `9999 years` — there is no refresh flow and no token invalidation on logout.
- **Redux Toolkit + redux-persist** — Centralised, predictable state; `redux-persist` survives page refresh via `localStorage`. Trade-off: adds boilerplate; snackbar state is also persisted (unnecessary).
- **Styled-components + MUI** — Styled-components for custom layout/theme; MUI for ready-made interactive widgets (Slider, CircularProgress, Snackbar). Trade-off: two CSS-in-JS libraries in the same app increases bundle size.
- **Flat REST API design** — Cart, favourites, and orders are sub-resources of `/api/user/`. Simple to reason about. Trade-off: not fully RESTful (e.g. orders are retrieved via a GET that also mutates favourites — a known bug, see Notes).
- **No server-side pagination** — `Food.find(filter)` returns all matching documents. Trade-off: fast to implement; will not scale past a few thousand items.
- **Client-side filtering state** — Filter selections live in local component state; every change triggers a fresh API call. Trade-off: simple but causes redundant requests on each slider tick.

---

## Notes

### Assumptions
- Single-tenant, single-user-role app (no admin panel).
- Local development only; no Docker or CI/CD configuration.
- Food images are stored as external URLs, not uploaded files.
- Payment is UI-only (card fields are uncontrolled inputs; no payment gateway is integrated).

### Known Bugs
- **`getAllOrders` controller is broken** — it's a copy-paste of `addToFavorites`. It adds to favourites instead of fetching orders. Any call to `GET /api/user/order` will silently add a product to favourites.
- **Token key mismatch** — `UserSlice.js` writes `foodeli-app-token` to `localStorage` on login, but Cart, Favourites, FoodDetails, and ProductsCard all read `krist-app-token`. Authenticated requests to protected routes will always fail (token is `null`).
- **`/search`, `/orders`, and `/contact` routes** are linked in the Navbar but have no corresponding `<Route>` defined in `App.js`.
- The `search` query param in `getFoodItems` matches against `title` (a non-existent field) instead of `name`.

### What I'd Improve With More Time
- Fix the token key mismatch and the `getAllOrders` copy-paste bug.
- Add server-side pagination (`limit` + `skip` or cursor-based).
- Add rate limiting (e.g. `express-rate-limit`) on auth endpoints.
- Add input validation (e.g. `zod` or `express-validator`) — currently none exists server-side.
- Add a `.nvmrc` and `engines` field to lock Node version.
- Parameterise the client's API `baseURL` via `REACT_APP_API_URL` environment variable.
- Improve accessibility (missing `alt` on images, no ARIA labels on icon buttons).

