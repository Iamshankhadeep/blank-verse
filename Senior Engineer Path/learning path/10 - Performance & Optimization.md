# 10 - Performance & Optimization

Senior engineers know when to optimize and, more importantly, when NOT to. Premature optimization wastes time; measured optimization creates value.

## The Golden Rule

> "Premature optimization is the root of all evil." - Donald Knuth

**But also:**
> "Premature pessimization is equally bad." - Don't write obviously slow code

## The Performance Workflow

### 1. Measure First

❌ **Don't optimize without measuring**
```javascript
// "I think this might be slow..."
// Proceeds to spend 2 days optimizing
```

✅ **Do measure before optimizing**
```javascript
// Profile the application
// Identify actual bottlenecks
// Optimize the biggest bottleneck
// Measure improvement
```

### 2. Set Performance Goals

**Not specific enough:**
- "Make it faster"
- "Improve performance"

**Specific and measurable:**
- "Page load time < 2 seconds"
- "API response < 200ms for p95"
- "Support 10,000 concurrent users"
- "Database queries < 100ms"

### 3. Profile and Identify Bottlenecks

**Tools by platform:**
- **Browser:** Chrome DevTools Performance tab
- **Node.js:** `node --prof`, clinic.js
- **Python:** cProfile, py-spy
- **Java:** JProfiler, VisualVM
- **Database:** EXPLAIN ANALYZE, slow query log

## Common Performance Issues

### 1. N+1 Query Problem

**❌ The Problem (1 + N queries):**
```javascript
// Get all users
const users = await db.query('SELECT * FROM users');

// For each user, get their orders (N queries!)
for (const user of users) {
  user.orders = await db.query(
    'SELECT * FROM orders WHERE user_id = ?',
    [user.id]
  );
}
// Total: 1 + 100 = 101 queries for 100 users
```

**✅ Solution 1: JOIN (2 queries):**
```javascript
// Single query with JOIN
const usersWithOrders = await db.query(`
  SELECT
    u.*,
    o.id as order_id,
    o.total as order_total
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
`);

// Group results
const users = groupByUser(usersWithOrders);
// Total: 1 query
```

**✅ Solution 2: Eager Loading (2 queries):**
```javascript
// Get all users
const users = await db.query('SELECT * FROM users');
const userIds = users.map(u => u.id);

// Get all orders in one query
const orders = await db.query(
  'SELECT * FROM orders WHERE user_id IN (?)',
  [userIds]
);

// Map orders to users
const ordersByUser = groupBy(orders, 'user_id');
users.forEach(user => {
  user.orders = ordersByUser[user.id] || [];
});
// Total: 2 queries
```

### 2. Missing Database Indexes

**❌ Without Index (Slow):**
```sql
-- Full table scan - checks every row
SELECT * FROM users WHERE email = 'john@example.com';
-- Time: 2000ms for 1M rows
```

**✅ With Index (Fast):**
```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Same query now uses index
SELECT * FROM users WHERE email = 'john@example.com';
-- Time: 5ms
```

**When to add indexes:**
- ✅ Columns used in WHERE clauses
- ✅ Columns used in JOIN conditions
- ✅ Columns used in ORDER BY
- ⚠️ But indexes have overhead - don't index everything

### 3. Inefficient Algorithms

**❌ O(n²) - Nested loops:**
```javascript
function findDuplicates(arr) {
  const duplicates = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}
// Time complexity: O(n³) actually!
// For 10,000 items: ~1,000,000,000 operations
```

**✅ O(n) - Hash map:**
```javascript
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();

  for (const item of arr) {
    if (seen.has(item)) {
      duplicates.add(item);
    } else {
      seen.add(item);
    }
  }

  return Array.from(duplicates);
}
// Time complexity: O(n)
// For 10,000 items: ~10,000 operations
```

### 4. Unnecessary Re-renders (Frontend)

**❌ React: Creating new objects in render:**
```javascript
function UserList() {
  const users = getUsers();

  return users.map(user => (
    <UserCard
      key={user.id}
      user={user}
      style={{ padding: 10 }} // New object every render!
      onDelete={() => deleteUser(user.id)} // New function every render!
    />
  ));
}
```

**✅ Optimized:**
```javascript
const cardStyle = { padding: 10 }; // Outside component

function UserList() {
  const users = getUsers();
  const handleDelete = useCallback((userId) => {
    deleteUser(userId);
  }, []);

  return users.map(user => (
    <UserCard
      key={user.id}
      user={user}
      style={cardStyle}
      onDelete={handleDelete}
    />
  ));
}
```

### 5. Not Caching Expensive Operations

**❌ Recalculating every time:**
```javascript
app.get('/api/stats', async (req, res) => {
  // Expensive query run every request
  const stats = await db.query(`
    SELECT
      COUNT(*) as total_users,
      AVG(age) as avg_age,
      SUM(orders.total) as total_revenue
    FROM users
    JOIN orders ON users.id = orders.user_id
  `);
  res.json(stats);
});
// Takes 2 seconds every request
```

**✅ With caching:**
```javascript
let statsCache = null;
let cacheTime = null;
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

app.get('/api/stats', async (req, res) => {
  const now = Date.now();

  if (statsCache && (now - cacheTime) < CACHE_TTL) {
    return res.json(statsCache);
  }

  const stats = await db.query(`
    SELECT
      COUNT(*) as total_users,
      AVG(age) as avg_age,
      SUM(orders.total) as total_revenue
    FROM users
    JOIN orders ON users.id = orders.user_id
  `);

  statsCache = stats;
  cacheTime = now;

  res.json(stats);
});
// First request: 2 seconds
// Subsequent requests (within 5 min): <1ms
```

### 6. Synchronous Operations

**❌ Blocking operations:**
```javascript
app.post('/api/users', async (req, res) => {
  const user = await createUser(req.body);

  // These block the response
  await sendWelcomeEmail(user.email); // 2 seconds
  await generateAnalytics(user); // 3 seconds
  await updateSearchIndex(user); // 1 second

  res.json(user); // User waits 6 seconds!
});
```

**✅ Background processing:**
```javascript
app.post('/api/users', async (req, res) => {
  const user = await createUser(req.body);

  // Queue background tasks
  await queue.add('send-welcome-email', { userId: user.id });
  await queue.add('generate-analytics', { userId: user.id });
  await queue.add('update-search-index', { userId: user.id });

  res.json(user); // Response in <100ms
});

// Worker processes queue asynchronously
worker.process('send-welcome-email', async (job) => {
  await sendWelcomeEmail(job.data.userId);
});
```

## Optimization Techniques

### 1. Lazy Loading

**Load only what's needed:**
```javascript
// ❌ Load everything upfront
const allUsers = await fetchAllUsers(); // 10,000 users
displayFirst10(allUsers);

// ✅ Load in chunks
const firstPage = await fetchUsers({ page: 1, limit: 10 });
displayUsers(firstPage);

// Load more when needed
button.onclick = async () => {
  const nextPage = await fetchUsers({ page: 2, limit: 10 });
  displayUsers(nextPage);
};
```

### 2. Debouncing

**Limit function calls:**
```javascript
// ❌ Call API on every keystroke
input.addEventListener('input', async (e) => {
  const results = await searchAPI(e.target.value);
  displayResults(results);
});
// User types "hello" = 5 API calls

// ✅ Debounce - wait for user to stop typing
const debouncedSearch = debounce(async (query) => {
  const results = await searchAPI(query);
  displayResults(results);
}, 300);

input.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});
// User types "hello" = 1 API call

function debounce(fn, delay) {
  let timeoutId;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}
```

### 3. Memoization

**Cache function results:**
```javascript
// ❌ Recalculate every time
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
fibonacci(40); // Takes seconds

// ✅ Memoize results
function fibonacci(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n]) return memo[n];

  memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
  return memo[n];
}
fibonacci(40); // Milliseconds
```

### 4. Pagination

**Don't load all data:**
```javascript
// ❌ Return everything
app.get('/api/products', async (req, res) => {
  const products = await db.query('SELECT * FROM products');
  res.json(products); // 10,000 products
});

// ✅ Paginate
app.get('/api/products', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  const products = await db.query(
    'SELECT * FROM products LIMIT ? OFFSET ?',
    [limit, offset]
  );

  const total = await db.query('SELECT COUNT(*) as count FROM products');

  res.json({
    products,
    page,
    limit,
    total: total[0].count,
    totalPages: Math.ceil(total[0].count / limit)
  });
});
```

### 5. Connection Pooling

**Reuse database connections:**
```javascript
// ❌ Create new connection every query
async function getUser(id) {
  const connection = await mysql.createConnection(config);
  const user = await connection.query('SELECT * FROM users WHERE id = ?', [id]);
  await connection.end();
  return user;
}

// ✅ Use connection pool
const pool = mysql.createPool({
  ...config,
  connectionLimit: 10
});

async function getUser(id) {
  const connection = await pool.getConnection();
  try {
    const user = await connection.query('SELECT * FROM users WHERE id = ?', [id]);
    return user;
  } finally {
    connection.release(); // Return to pool
  }
}
```

## Performance Budgets

Set limits and enforce them:

**Web Performance:**
- Time to First Byte (TTFB): < 200ms
- First Contentful Paint (FCP): < 1.8s
- Time to Interactive (TTI): < 3.9s
- Total bundle size: < 200KB gzipped

**API Performance:**
- P50 (median): < 100ms
- P95 (95th percentile): < 500ms
- P99 (99th percentile): < 1000ms

**Database:**
- Query time: < 100ms
- Connection time: < 50ms

## Monitoring Performance

### Key Metrics to Track

**Backend:**
```javascript
const start = Date.now();

// Your operation
const result = await processOrder(order);

const duration = Date.now() - start;

// Log slow operations
if (duration > 1000) {
  logger.warn(`Slow operation: ${duration}ms`, {
    operation: 'processOrder',
    orderId: order.id
  });
}

// Send to monitoring
metrics.histogram('order.processing.duration', duration);
```

**Frontend:**
```javascript
// Performance API
const perfData = performance.getEntriesByType('navigation')[0];

console.log('Page load time:', perfData.loadEventEnd - perfData.fetchStart);
console.log('DOM ready:', perfData.domContentLoadedEventEnd - perfData.fetchStart);
console.log('Time to first byte:', perfData.responseStart - perfData.requestStart);
```

## Code Review: Performance Checklist

- [ ] No obvious N+1 queries
- [ ] Database queries have indexes
- [ ] Expensive operations are cached
- [ ] Long-running tasks are async/queued
- [ ] Pagination for large datasets
- [ ] No nested loops with large datasets
- [ ] Efficient algorithms (O(n) over O(n²))
- [ ] Connection pooling used
- [ ] No unnecessary re-renders (frontend)

## This Week's Practice

### Day 1-2: Measure
- [ ] Profile your application
- [ ] Identify 3 slowest operations
- [ ] Document baseline performance

### Day 3-4: Optimize
- [ ] Fix one bottleneck
- [ ] Measure improvement
- [ ] Document changes

### Day 5-7: Review
- [ ] Review PRs for performance issues
- [ ] Ask about N+1 queries
- [ ] Suggest caching opportunities

## Remember

> "Make it work, make it right, make it fast - in that order."

**Optimization checklist:**
1. ✅ Does it work correctly?
2. ✅ Is the code clean and maintainable?
3. ✅ Is it fast enough for users?
4. ⚠️ Only then: optimize further if needed

**Don't optimize:**
- Before profiling
- Code that runs once
- Micro-optimizations that don't matter
- At the expense of readability

**Do optimize:**
- After measuring
- User-facing slowness
- Proven bottlenecks
- With performance tests

---

**Previous:** [[09 - System Design Basics]]
**Next:** [[11 - Architecture Patterns]]
**Related:** [[Code Quality Checklist]]
