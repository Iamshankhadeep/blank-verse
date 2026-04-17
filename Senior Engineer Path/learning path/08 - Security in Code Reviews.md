# 08 - Security in Code Reviews

As a senior engineer, you're a guardian against security vulnerabilities. Most security bugs are caught in code review, not by security teams.

## Critical Security Checklist

### 🔴 HIGH PRIORITY - Always Check

#### 1. Secrets & Credentials
**BLOCK if found:**
- [ ] API keys, tokens, passwords in code
- [ ] Hardcoded credentials
- [ ] Connection strings with passwords
- [ ] Private keys
- [ ] AWS/cloud credentials

```javascript
// ❌ BLOCK THIS
const API_KEY = "sk_live_abc123xyz789";
const password = "admin123";

// ✅ APPROVE THIS
const API_KEY = process.env.API_KEY;
const password = process.env.DB_PASSWORD;
```

**Look for:**
- `.env` files being committed (should be in `.gitignore`)
- Secrets in config files
- Hardcoded tokens in tests

#### 2. SQL Injection
**BLOCK if found:**
- [ ] String concatenation in SQL queries
- [ ] Unparameterized queries
- [ ] Raw user input in queries

```javascript
// ❌ BLOCK THIS - SQL Injection vulnerability
const query = `SELECT * FROM users WHERE id = ${userId}`;
db.query(query);

// userId = "1 OR 1=1; DROP TABLE users;--"

// ✅ APPROVE THIS - Parameterized query
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);

// OR use an ORM
const user = await User.findById(userId);
```

#### 3. Cross-Site Scripting (XSS)
**BLOCK if found:**
- [ ] Unescaped user input rendered to HTML
- [ ] `innerHTML` with user data
- [ ] Direct DOM manipulation with user input

```javascript
// ❌ BLOCK THIS
element.innerHTML = userInput; // XSS!
document.write(userData);

// ✅ APPROVE THIS
element.textContent = userInput; // Auto-escaped
element.innerText = userInput;

// Or use framework escaping
<div>{userInput}</div> // React auto-escapes
```

**Particularly dangerous in:**
- Comment sections
- User profiles
- Search results
- Any user-generated content

#### 4. Authentication & Authorization
**BLOCK if missing:**
- [ ] Authentication checks
- [ ] Authorization checks (can THIS user do THIS action?)
- [ ] Password hashing (never store plaintext!)
- [ ] Session management

```javascript
// ❌ BLOCK THIS - No authorization
app.delete('/api/users/:id', (req, res) => {
  deleteUser(req.params.id); // Any user can delete any user!
});

// ✅ APPROVE THIS
app.delete('/api/users/:id', requireAuth, (req, res) => {
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  deleteUser(req.params.id);
});

// ❌ BLOCK THIS - Plaintext password
user.password = req.body.password;

// ✅ APPROVE THIS
user.password = await bcrypt.hash(req.body.password, 10);
```

#### 5. Path Traversal
**BLOCK if found:**
- [ ] User input in file paths
- [ ] Unvalidated file names

```javascript
// ❌ BLOCK THIS
const filePath = `/uploads/${req.params.filename}`;
res.sendFile(filePath);

// Attack: filename = "../../etc/passwd"

// ✅ APPROVE THIS
const filename = path.basename(req.params.filename); // Remove paths
const filePath = path.join('/uploads', filename);
if (!filePath.startsWith('/uploads')) {
  return res.status(400).send('Invalid path');
}
res.sendFile(filePath);
```

#### 6. Command Injection
**BLOCK if found:**
- [ ] User input in shell commands
- [ ] Unvalidated input to `exec`, `system`, `eval`

```javascript
// ❌ BLOCK THIS - Command injection
const { exec } = require('child_process');
exec(`ping ${userInput}`); // userInput = "; rm -rf /"

// ✅ APPROVE THIS
const { execFile } = require('child_process');
execFile('ping', [userInput]); // Args are escaped

// OR validate input
if (!/^[a-zA-Z0-9.-]+$/.test(userInput)) {
  throw new Error('Invalid input');
}
```

### 🟡 MEDIUM PRIORITY - Should Check

#### 7. Sensitive Data Exposure
- [ ] Are we logging sensitive data?
- [ ] Are error messages exposing internal details?
- [ ] Is sensitive data cached?
- [ ] Are we returning too much data in APIs?

```javascript
// ❌ BAD
logger.info(`User login: ${email}, password: ${password}`);

// ❌ BAD - Exposing internal error
catch (error) {
  res.status(500).json({ error: error.stack });
}

// ✅ GOOD
logger.info(`User login: ${email}`);

// ✅ GOOD - Generic error to user
catch (error) {
  logger.error(error); // Log internally
  res.status(500).json({ error: 'Internal server error' });
}
```

#### 8. Insecure Deserialization
- [ ] Deserializing untrusted data
- [ ] Using `eval()` or `Function()` with user input

```javascript
// ❌ BLOCK THIS
const userData = eval(req.body.data); // NEVER!
const obj = JSON.parse(req.body.data);
someFunction(obj); // If someFunction uses obj properties unsafely

// ✅ APPROVE THIS
const userData = JSON.parse(req.body.data);
validate(userData); // Validate structure and values
```

#### 9. Broken Access Control
- [ ] Can users access resources they shouldn't?
- [ ] Are object IDs exposed in URLs without auth checks?
- [ ] Can users modify data they don't own?

```javascript
// ❌ BAD - Insecure Direct Object Reference (IDOR)
app.get('/api/orders/:id', async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order); // Anyone can view any order!
});

// ✅ GOOD
app.get('/api/orders/:id', requireAuth, async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (order.userId !== req.user.id) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  res.json(order);
});
```

#### 10. Cross-Site Request Forgery (CSRF)
- [ ] Are state-changing operations protected?
- [ ] Is CSRF token validation in place?

```javascript
// ❌ BAD - No CSRF protection
app.post('/api/transfer-money', (req, res) => {
  transferMoney(req.body.amount, req.body.to);
});

// ✅ GOOD - CSRF token required
app.use(csrfProtection);
app.post('/api/transfer-money', (req, res) => {
  // CSRF token automatically validated by middleware
  transferMoney(req.body.amount, req.body.to);
});
```

## Security Code Review Checklist

Use this for every PR:

### Input Validation
- [ ] All user input is validated
- [ ] Whitelist validation (allow known good, not block known bad)
- [ ] Type checking
- [ ] Length limits
- [ ] Format validation (email, phone, etc.)

### Output Encoding
- [ ] User data is escaped when displayed
- [ ] Proper encoding for context (HTML, URL, JS, SQL)
- [ ] No `innerHTML` with user data

### Authentication & Authorization
- [ ] Auth required for protected endpoints
- [ ] Authorization checks present
- [ ] Session management secure
- [ ] Passwords properly hashed

### Data Protection
- [ ] Sensitive data encrypted at rest
- [ ] HTTPS for data in transit
- [ ] No secrets in code
- [ ] Secure cookie flags (httpOnly, secure, sameSite)

### Error Handling
- [ ] Errors don't expose sensitive info
- [ ] Proper logging (no sensitive data)
- [ ] Generic error messages to users

## Red Flags in Code Reviews

🚩 **Immediate red flags:**
- `eval()`
- `exec()` with user input
- String concatenation in SQL
- `.innerHTML` with user data
- Hardcoded passwords/keys
- `require('crypto')` without proper usage
- `md5()` or `sha1()` for passwords (use bcrypt!)

## Common Security Mistakes by Level

### Junior Mistakes:
- Storing passwords in plaintext
- No input validation
- Using `==` instead of `===` (type coercion bugs)

### Mid-Level Mistakes:
- Missing authorization checks
- Logging sensitive data
- Improper error handling

### Senior Should Catch:
- IDOR vulnerabilities
- Race conditions in async code
- Timing attacks
- Improper use of crypto
- Business logic flaws

## This Week's Practice

### Day 1-2: Learn
- [ ] Read OWASP Top 10: https://owasp.org/www-project-top-ten/
- [ ] Study each vulnerability type
- [ ] Understand attack vectors

### Day 3-4: Review
- [ ] Review 3 PRs with security lens
- [ ] Look for injection vulnerabilities
- [ ] Check authentication/authorization

### Day 5: Test
- [ ] Try to find vulnerabilities in your own code
- [ ] Fix what you find
- [ ] Add security tests

### Day 6-7: Document
- [ ] Create security checklist for your team
- [ ] Document common vulnerabilities in your stack
- [ ] Share learnings with team

## Questions to Ask in Every Review

1. **Where does this data come from?** (User input = untrusted)
2. **Where does this data go?** (Database, display, file system?)
3. **Who can call this?** (Authentication)
4. **What can they do?** (Authorization)
5. **What happens if input is malicious?**
6. **What happens if this fails?**
7. **Are we logging anything sensitive?**

## Resources

- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **OWASP Cheat Sheets**: https://cheatsheetseries.owasp.org/
- **CWE Top 25**: https://cwe.mitre.org/top25/
- **Security Headers**: https://securityheaders.com/

## Remember

> "Security is not a product, it's a process." - Bruce Schneier

- Security is everyone's responsibility
- Better to block a PR than fix a breach
- When in doubt, ask a security expert
- Stay updated on new vulnerabilities

---

**Previous:** [[07 - Testing Strategies]]
**Next:** [[09 - System Design Basics]]
**Related:** [[PR Review Checklist]]
