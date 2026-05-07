# Implementing-Security-Measures-OWASP-Juice-Shop-

## Objective of Task

- Apply input validation and sanitization to prevent injection attacks using `validator`.
- Add password hashing and authentication security using `bcrypt` and `JWT`.
- Secure the application against XSS (Cross-Site Scripting) using Angular sanitization and Helmet middleware.

---

# Backend Implementation (`login.ts`)

## Input Verification & Sanitization

User input is validated and sanitized before processing:

```typescript
email = validator.trim(email || '')
email = validator.escape(email)

if (!validator.isEmail(email)) {
  return res.status(400).json({ error: 'Invalid email format' })
}

if (!validator.isLength(password || '', { min: 5 })) {
  return res.status(400).json({ error: 'Password too short' })
}
```

### Security Benefits
- Removes unnecessary spaces from user input.
- Escapes dangerous characters to reduce injection attacks.
- Ensures email format is valid.
- Prevents weak passwords by enforcing minimum length.

---

## Database Query (Sequelize)

```typescript
const authenticatedUser = await models.sequelize.query(
  `SELECT * FROM Users
   WHERE email = :email
   AND deletedAt IS NULL`,
  {
    replacements: { email },
    model: UserModel,
    plain: true
  }
)
```

### Security Benefits
- Uses parameterized queries through Sequelize replacements.
- Helps prevent SQL Injection attacks.
- Ensures only active users are retrieved.

---

## Password Verification (`bcrypt`)

```typescript
const bcrypt = require('bcrypt')

const isPasswordValid = await bcrypt.compare(
  password,
  user.data.password
)

if (!isPasswordValid) {
  return res.status(401).send('Invalid email or password')
}
```

### Security Benefits
- Passwords are securely compared using bcrypt hashing.
- Plain-text passwords are never stored in the database.
- Protects against password theft and brute-force attacks.

---

## JWT Token Generation

```typescript
const jwt = require('jsonwebtoken')

const token = jwt.sign(
  {
    id: user.data.id,
    email: user.data.email
  },
  'your-secret-key',
  { expiresIn: '1h' }
)
```

### Security Benefits
- Generates secure authentication tokens.
- Token expires automatically after 1 hour.
- Enables secure session management.

---

# User Model Security (`User.ts`)

## Password Hashing Hook

```typescript
password: {
  type: DataTypes.STRING
},

User.addHook('beforeUpdate', async (user: User) => {
  if (user.changed('password')) {
    user.password = await bcrypt.hash(user.password, 10)
  }
})
```

### Security Benefits
- Automatically hashes passwords before updating.
- Prevents storing plain-text passwords.
- Uses bcrypt salt rounds for stronger encryption.

---

## Email Validation Hook

```typescript
User.addHook('afterValidate', async (user: User) => {
  if (
    user.email &&
    user.email.toLowerCase() ===
    `acc0unt4nt@${config.get<string>('application.domain')}`.toLowerCase()
  ) {
    await Promise.reject(
      new Error(
        'Nice try, but this is not how the "Ephemeral Accountant" challenge works!'
      )
    )
  }
})
```

### Security Benefits
- Prevents unauthorized or restricted email usage.
- Adds additional validation after model verification.

---

# Server Security (`server.ts` – Helmet Middleware)

Helmet is used to secure HTTP headers and prevent XSS attacks.

```typescript
const app = express()

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'", "'unsafe-eval'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:"]
      }
    }
  })
)
```

### Security Benefits
- Adds secure HTTP headers.
- Reduces risk of Cross-Site Scripting (XSS).
- Restricts untrusted scripts and external resources.

---

# Frontend Security (Angular – XSS Fixes)

## Fix for Inner HTML Injection

```typescript
tableData[i].description =
  this.sanitizer.sanitize(1, tableData[i].description)
```

### Security Benefits
- Sanitizes HTML content before rendering.
- Prevents malicious scripts from executing in the browser.

---

## Fix for Search Input XSS

### Vulnerable Code

```typescript
searchValue =
  this.sanitizer.bypassSecurityTrustHtml(queryParam)
```

### Secure Fix

```typescript
this.searchValue = queryParam
```

### Security Benefits
- Removes unsafe HTML bypassing.
- Prevents attackers from injecting malicious scripts through search parameters.

---

# Conclusion

This implementation improves application security by:
- Validating and sanitizing user input.
- Hashing passwords securely using bcrypt.
- Using JWT for secure authentication.
- Preventing SQL Injection attacks.
- Securing HTTP headers with Helmet.
- Protecting the frontend from XSS vulnerabilities using Angular sanitization.
