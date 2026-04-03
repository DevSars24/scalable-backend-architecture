# JavaScript Concepts Applied to the Backend (Node.js)

> **Every JS concept from the 8-part guide — shown with real backend/Node.js code.**

---

## Index

| JS Concept | Backend Application |
|-----------|---------------------|
| [Data Types & Hoisting](#1-data-types--hoisting-in-backend) | Schema design, environment configs |
| [Functions, Scope & Closures](#2-functions-scope--closures-in-backend) | Middleware factories, private config |
| [Prototypes & OOP](#3-prototypes--oop-in-backend) | Service classes, model inheritance |
| [Async / Promises / Event Loop](#4-async--event-loop-in-backend) | DB queries, HTTP calls, queues |
| [ES6+ Modern Syntax](#5-es6-modern-syntax-in-backend) | Config parsing, destructuring requests |
| [Advanced Patterns](#6-advanced-patterns-in-backend) | Caching, rate limiting, generators |
| [Event-Driven Architecture](#7-event-driven-architecture-in-backend) | EventEmitter, pub/sub, queues |
| [Error Handling & Storage](#8-error-handling--storage-in-backend) | Global error handlers, session stores |

---

## 1. Data Types & Hoisting in Backend

### Environment Config with Correct Types

```js
// config.js — type safety matters in production!
const config = {
  port: Number(process.env.PORT) || 3000,        // Number
  debug: process.env.DEBUG === "true",            // Boolean (not string!)
  dbUrl: process.env.DATABASE_URL ?? "",          // String (nullish fallback)
  maxConnections: BigInt(process.env.MAX_CONN),   // BigInt for large numbers
  allowedOrigins: process.env.ORIGINS?.split(",") // Array or undefined
};

// ❌ Common bug: forgetting env vars are STRINGS
if (process.env.DEBUG == true) { /* won't work — "true" != true */ }

// ✅ Correct
if (process.env.DEBUG === "true") { /* works */ }
```

### Hoisting gotcha in Express routes

```js
// ❌ BUG: trying to use middleware before it's declared
app.use(checkAuth); // undefined at this point!

var checkAuth = (req, res, next) => {
  // validate token
  next();
};

// ✅ Use function declaration (fully hoisted) or define before use
app.use(checkAuth);

function checkAuth(req, res, next) {
  // hoisted! works fine
  next();
}
```

### NaN in request validation

```js
// Express route handler
app.get("/users/:id", (req, res) => {
  const userId = parseInt(req.params.id, 10);

  if (isNaN(userId)) {
    return res.status(400).json({ error: "Invalid user ID" });
  }

  // safe to use userId now
});
```

---

## 2. Functions, Scope & Closures in Backend

### Middleware Factory with Closures

Closures power Express middleware factories — a function that creates customized middleware.

```js
// rateLimit.js — factory function using closure
function createRateLimiter(maxRequests, windowMs) {
  const requestCounts = new Map(); // private to this closure

  return function rateLimitMiddleware(req, res, next) {
    const ip = req.ip;
    const now = Date.now();

    if (!requestCounts.has(ip)) {
      requestCounts.set(ip, []);
    }

    // Remove requests outside the window
    const requests = requestCounts.get(ip).filter(
      time => now - time < windowMs
    );

    if (requests.length >= maxRequests) {
      return res.status(429).json({ error: "Too many requests" });
    }

    requests.push(now);
    requestCounts.set(ip, requests);
    next();
  };
}

// Usage in Express
const apiLimiter = createRateLimiter(100, 15 * 60 * 1000); // 100 req / 15 min
app.use("/api", apiLimiter);
```

### Private Config with Closure

```js
function createDbConfig() {
  const password = process.env.DB_PASS; // private — never exposed

  return {
    getConnectionString() {
      return `mongodb://admin:${password}@localhost:27017/mydb`;
    },
    getHost() { return "localhost"; }
    // password is never accessible from outside!
  };
}

const dbConfig = createDbConfig();
dbConfig.getConnectionString(); // uses password internally
dbConfig.password; // undefined — protected!
```

### `this` in Route Handlers

```js
// ❌ Arrow function loses `this` context in class
class UserController {
  constructor() {
    this.service = new UserService();
  }

  // Bind in constructor OR use arrow function property
  getUser = async (req, res) => {    // ✅ arrow function property
    const user = await this.service.findById(req.params.id);
    res.json(user);
  }
}

// Express route binding
const controller = new UserController();
router.get("/users/:id", controller.getUser);
// ✅ `this` is preserved because of arrow function
```

### call/apply in Backend

```js
// Reusing validation logic across different models
function validateRequired(fields) {
  const missing = fields.filter(field => !this[field]);
  if (missing.length > 0) {
    throw new Error(`Missing required fields: ${missing.join(", ")}`);
  }
}

const userPayload = { name: "Alice", email: "", role: "user" };
const orderPayload = { product: "Laptop", quantity: 2, userId: "" };

validateRequired.call(userPayload, ["name", "email"]);  // throws: "Missing: email"
validateRequired.call(orderPayload, ["product", "userId"]); // throws: "Missing: userId"
```

---

## 3. Prototypes & OOP in Backend

### Service Layer with Classes

```js
// base.service.js
class BaseService {
  constructor(model) {
    this.model = model;
  }

  async findById(id) {
    return this.model.findById(id);
  }

  async findAll(filter = {}) {
    return this.model.find(filter);
  }

  async create(data) {
    return this.model.create(data);
  }

  async update(id, data) {
    return this.model.findByIdAndUpdate(id, data, { new: true });
  }

  async delete(id) {
    return this.model.findByIdAndDelete(id);
  }
}

// user.service.js — extends base, adds user-specific logic
class UserService extends BaseService {
  constructor() {
    super(UserModel); // pass model to parent
  }

  async findByEmail(email) {
    return this.model.findOne({ email });
  }

  async hashAndCreate(userData) {
    const hashed = await bcrypt.hash(userData.password, 10);
    return this.create({ ...userData, password: hashed });
  }
}

// product.service.js
class ProductService extends BaseService {
  constructor() {
    super(ProductModel);
  }

  async findByCategory(category) {
    return this.findAll({ category });
  }
}
```

### Repository Pattern using Prototypes

```js
// Shared behavior through prototype
function Repository(model) {
  this.model = model;
}

Repository.prototype.save = function(data) {
  return new this.model(data).save();
};

Repository.prototype.findOne = function(query) {
  return this.model.findOne(query);
};

// Extend for specific models
function UserRepository() {
  Repository.call(this, User);
}
UserRepository.prototype = Object.create(Repository.prototype);
UserRepository.prototype.findByEmail = function(email) {
  return this.findOne({ email });
};
```

---

## 4. Async & Event Loop in Backend

### Database Query — Parallel vs Sequential

```js
// ❌ Sequential — 3x slower
async function getUserWithDetails(userId) {
  const user = await User.findById(userId);          // wait...
  const orders = await Order.find({ userId });       // wait...
  const wishlist = await Wishlist.find({ userId });  // wait...
  return { user, orders, wishlist };
}

// ✅ Parallel — all run simultaneously!
async function getUserWithDetails(userId) {
  const [user, orders, wishlist] = await Promise.all([
    User.findById(userId),
    Order.find({ userId }),
    Wishlist.find({ userId })
  ]);
  return { user, orders, wishlist };
}
```

### Handling Partial Failures with `Promise.allSettled`

```js
async function sendNotifications(userIds) {
  const notifications = userIds.map(id =>
    sendEmail(id).catch(err => ({ failed: true, id, reason: err.message }))
  );

  const results = await Promise.allSettled(notifications);

  const failed = results
    .filter(r => r.status === "rejected")
    .map(r => r.reason);

  console.log(`Sent: ${results.length - failed.length}, Failed: ${failed.length}`);
}
```

### Async Middleware in Express

```js
// Wrap async handlers to catch errors automatically
function asyncHandler(fn) {
  return function(req, res, next) {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Usage
router.get("/users/:id", asyncHandler(async (req, res) => {
  const user = await UserService.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "Not found" });
  res.json(user);
}));
```

### Callbacks (still used in Node.js core APIs)

```js
const fs = require("fs");

// Callback style (legacy)
fs.readFile("./config.json", "utf8", (err, data) => {
  if (err) return console.error("Error:", err);
  const config = JSON.parse(data);
});

// Promisified version (modern)
const fs = require("fs/promises");

async function readConfig() {
  try {
    const data = await fs.readFile("./config.json", "utf8");
    return JSON.parse(data);
  } catch (err) {
    console.error("Config read failed:", err.message);
  }
}
```

### Microtask vs Macrotask in Server Context

```js
// Understanding execution order matters for server correctness
async function processRequest(req) {
  console.log("1. Request received");

  setImmediate(() => {
    console.log("4. setImmediate — macrotask");
  });

  await Promise.resolve(); // microtask

  console.log("3. After microtask");
}

console.log("2. Synchronous code");
// Output: 1, 2, 3, 4
```

---

## 5. ES6 Modern Syntax in Backend

### Destructuring HTTP Requests

```js
// Route handler — clean destructuring of req
app.post("/register", async (req, res) => {
  const { name, email, password, role = "user" } = req.body; // with default!
  const { authorization } = req.headers;
  const { redirect } = req.query;

  if (!name || !email || !password) {
    return res.status(400).json({ error: "Missing required fields" });
  }
  // ...
});

// Nested destructuring for config
const { database: { host, port, name: dbName } } = require("./config");
```

### Rest/Spread for API Response Shaping

```js
// Remove sensitive fields — spread + destructuring
function sanitizeUser(user) {
  const { password, resetToken, __v, ...safeUser } = user.toObject();
  return safeUser;
  // returns everything EXCEPT password, resetToken, __v
}

// Merge defaults with incoming data
function updateProfile(userId, updates) {
  const defaults = { updatedAt: new Date(), updatedBy: "system" };
  const merged = { ...defaults, ...updates }; // updates override defaults
  return User.findByIdAndUpdate(userId, merged, { new: true });
}
```

### Optional Chaining with Mongoose documents

```js
async function getUserCity(userId) {
  const user = await User.findById(userId);

  // ✅ Safe access — no error if user or address is null
  return user?.profile?.address?.city ?? "Unknown";
}

// Conditional method calling
async function sendWelcomeEmail(userId) {
  const user = await User.findById(userId);
  await notification?.sendEmail?.(user?.email); // only if methods exist
}
```

### ES Modules in Node.js

```js
// package.json
// { "type": "module" }

// auth.service.js
export class AuthService {
  static async login(email, password) { /* ... */ }
  static async logout(token) { /* ... */ }
}

export const generateToken = (userId) => {
  return jwt.sign({ userId }, process.env.JWT_SECRET, { expiresIn: "7d" });
};

// app.js
import { AuthService, generateToken } from "./auth.service.js";
import express from "express";
```

---

## 6. Advanced Patterns in Backend

### Memoization for Expensive DB Queries

```js
// Simple in-memory cache for repeated queries
function memoize(fn, ttlMs = 60_000) {
  const cache = new Map();

  return async function(...args) {
    const key = JSON.stringify(args);
    const cached = cache.get(key);

    if (cached && Date.now() - cached.timestamp < ttlMs) {
      console.log("Cache hit:", key);
      return cached.value;
    }

    const result = await fn.apply(this, args);
    cache.set(key, { value: result, timestamp: Date.now() });
    return result;
  };
}

// Cache heavy category listing for 5 minutes
const getCategories = memoize(async () => {
  return Category.find({}).lean();
}, 5 * 60_000);

app.get("/categories", async (req, res) => {
  const categories = await getCategories(); // served from cache after 1st call
  res.json(categories);
});
```

### Currying for Reusable Query Builders

```js
// Curried logger — partial application
const log = level => message => data =>
  console.log(JSON.stringify({ level, message, data, ts: new Date() }));

const infoLog = log("INFO");
const errorLog = log("ERROR");

// Use throughout the app
infoLog("User logged in")({ userId: "123", ip: req.ip });
errorLog("Payment failed")({ orderId: "456", reason: "card_declined" });

// Curried database query helper
const findBy = field => value => model =>
  model.findOne({ [field]: value });

const findByEmail = findBy("email");
const findByUsername = findBy("username");

await findByEmail("alice@example.com")(User);
await findByUsername("alice123")(User);
```

### Generator for Paginated DB Queries

```js
// Stream large datasets without loading all into memory
async function* getUsersBatch(batchSize = 100) {
  let skip = 0;
  while (true) {
    const batch = await User.find({}).skip(skip).limit(batchSize).lean();
    if (batch.length === 0) break;

    yield batch;
    skip += batchSize;
  }
}

// Process 1 million users in batches of 100
async function sendNewsletterToAll() {
  for await (const batch of getUsersBatch(100)) {
    await Promise.all(batch.map(user => sendEmail(user.email)));
    console.log(`Sent to ${batch.length} users`);
  }
}
```

### WeakMap for Request-Scoped Data

```js
// Attach data to request objects without modifying them
const requestCache = new WeakMap();

function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  const decoded = jwt.verify(token, process.env.JWT_SECRET);

  // Store decoded user on the request (GC'd when request ends)
  requestCache.set(req, { user: decoded });
  next();
}

function getRequestUser(req) {
  return requestCache.get(req)?.user;
}
```

---

## 7. Event-Driven Architecture in Backend

### Node.js EventEmitter — Pub/Sub Pattern

```js
const { EventEmitter } = require("events");

class OrderService extends EventEmitter {
  async createOrder(orderData) {
    const order = await Order.create(orderData);

    // Emit events — decoupled from side effects
    this.emit("order:created", order);
    return order;
  }

  async shipOrder(orderId) {
    const order = await Order.findByIdAndUpdate(orderId, { status: "shipped" });
    this.emit("order:shipped", order);
    return order;
  }
}

// Subscribe to order events
const orderService = new OrderService();

orderService.on("order:created", async (order) => {
  await sendEmail(order.userId, "Order Confirmed", order);
});

orderService.on("order:created", async (order) => {
  await updateInventory(order.items);
});

orderService.on("order:shipped", async (order) => {
  await notifySMS(order.userId, `Your order ${order._id} has shipped!`);
});
```

### Event Delegation Pattern for Express Routers

```js
// Single error handler for all routes (like event delegation)
app.use((err, req, res, next) => {
  if (err instanceof ValidationError) {
    return res.status(400).json({ error: err.message, field: err.field });
  }
  if (err instanceof AuthenticationError) {
    return res.status(401).json({ error: err.message });
  }
  if (err.name === "CastError") {
    return res.status(400).json({ error: "Invalid ID format" });
  }
  res.status(500).json({ error: "Internal server error" });
});
```

### Debounce Pattern — Batch DB Writes

```js
// Debounce analytics writes — batch multiple events into one DB write
function createDebouncedWriter(collection, delay = 5000) {
  let buffer = [];
  let timeoutId;

  return function write(event) {
    buffer.push(event);
    clearTimeout(timeoutId);

    timeoutId = setTimeout(async () => {
      if (buffer.length === 0) return;
      const toFlush = [...buffer];
      buffer = [];
      await collection.insertMany(toFlush);
      console.log(`Flushed ${toFlush.length} events to DB`);
    }, delay);
  };
}

const writeAnalytics = createDebouncedWriter(AnalyticsEvent, 3000);

// In route handlers:
app.post("/track", (req, res) => {
  writeAnalytics(req.body); // batched, not written immediately
  res.status(202).send();
});
```

---

## 8. Error Handling & Storage in Backend

### Global Error Handling Architecture

```js
// errors/index.js — custom error hierarchy
class AppError extends Error {
  constructor(message, statusCode = 500, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational; // vs programmer errors
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(field, message) {
    super(message, 400);
    this.field = field;
  }
}

class UnauthorizedError extends AppError {
  constructor(message = "Unauthorized") {
    super(message, 401);
  }
}

// Global Express error handler
app.use((err, req, res, next) => {
  const isOperational = err instanceof AppError;

  if (!isOperational) {
    // Programmer error — log and crash (let PM2 restart)
    console.error("FATAL ERROR:", err);
    process.exit(1);
  }

  res.status(err.statusCode).json({
    status: "error",
    message: err.message,
    ...(err.field && { field: err.field })
  });
});

// Catch unhandled rejections
process.on("unhandledRejection", (reason, promise) => {
  console.error("Unhandled Rejection:", reason);
  process.exit(1);
});
```

### Deep Clone for Immutable Config

```js
// Prevent accidental mutation of shared config
const baseConfig = {
  database: { host: "localhost", port: 27017 },
  cache: { ttl: 3600, maxSize: 1000 }
};

function getConfig(overrides = {}) {
  const config = structuredClone(baseConfig); // deep clone!
  return mergeDeep(config, overrides);
}

// Each service gets its own config copy — no shared state bugs
const userServiceConfig = getConfig({ database: { port: 27018 } });
const orderServiceConfig = getConfig({ cache: { ttl: 300 } });
```

### Session Storage in Backend (Redis)

```js
// Using Redis as session storage (relates to localStorage/sessionStorage concept)
const session = require("express-session");
const RedisStore = require("connect-redis")(session);
const redis = require("redis");

const client = redis.createClient({ url: process.env.REDIS_URL });

app.use(session({
  store: new RedisStore({ client }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,      // not accessible via JS (like httpOnly cookie)
    secure: true,        // only over HTTPS
    sameSite: "strict",  // CSRF protection
    maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
  }
}));

// Session usage
app.post("/login", async (req, res) => {
  const user = await AuthService.login(req.body.email, req.body.password);
  req.session.userId = user._id; // stored in Redis
  res.json({ message: "Logged in" });
});
```

### map/filter/reduce for Data Transformation (API responses)

```js
// Product listing endpoint — transform raw DB data
app.get("/products", async (req, res) => {
  const { category, minPrice, maxPrice } = req.query;
  const rawProducts = await Product.find({}).lean();

  const products = rawProducts
    .filter(p => !category || p.category === category)
    .filter(p => !minPrice || p.price >= Number(minPrice))
    .filter(p => !maxPrice || p.price <= Number(maxPrice))
    .map(({ _id, name, price, category, image }) => ({
      id: _id,       // rename _id to id
      name,
      price,
      category,
      image,
      // computed field
      priceLabel: `$${price.toFixed(2)}`
    }))
    .sort((a, b) => a.price - b.price);

  const summary = rawProducts.reduce((acc, p) => {
    acc.totalCount++;
    acc.totalValue += p.price;
    acc.byCategory[p.category] = (acc.byCategory[p.category] || 0) + 1;
    return acc;
  }, { totalCount: 0, totalValue: 0, byCategory: {} });

  res.json({ products, summary });
});
```

---

## 🧠 Concept → Backend Pattern Quick Reference

| JS Concept | Backend Pattern |
|------------|----------------|
| Closures | Middleware factories, private config, rate limiters |
| Currying | Query builders, logger factories, validators |
| Memoization | Response caching, expensive query results |
| Generators | Batch processing, data streaming, pagination |
| Promises / async-await | DB queries, HTTP calls, file operations |
| `Promise.all` | Parallel DB queries, bulk operations |
| Event Loop (microtasks) | Middleware execution order |
| EventEmitter | Pub/sub, webhooks, queue triggers |
| Debounce | Batch DB writes, analytics buffering |
| Throttle | Rate limiting (per IP / per user) |
| Destructuring | Extracting req.body / req.headers cleanly |
| Rest/Spread | Response sanitization, config merging |
| Optional chaining | Safe Mongoose document access |
| Nullish coalescing | Config defaults, fallback values |
| Custom Errors | Domain-specific HTTP errors (404, 401, 400) |
| WeakMap | Request-scoped data storage |
| structuredClone | Immutable config copies |
| ES Modules | Tree-shaken, organized service imports |
