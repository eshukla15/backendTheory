# Backend Engineering - Comprehensive Guide

> *"It's all about maintaining the system that deals with data"*

## Table of Contents

- [Core Principles](#core-principles)
- [Infrastructure & Networking](#infrastructure--networking)
- [HTTP Protocol](#http-protocol)
- [Routing](#routing)
- [Serialization & Deserialization](#serialization--deserialization)
- [Authentication & Authorization](#authentication--authorization)
- [Request Handling](#request-handling)
- [API Design (REST)](#api-design-rest)
- [Databases](#databases)
- [Business Logic Layer](#business-logic-layer)
- [Caching](#caching)
- [Additional Backend Concepts](#additional-backend-concepts)
- [Best Practices](#best-practices)

---

## Core Principles

**Why Backend Engineering Matters**

Backend engineering is fundamentally about building and maintaining systems that manage data efficiently, securely, and at scale. Understanding the core principles rather than limiting yourself to a specific tech stack enables you to adapt to any technology and build robust systems.

**Key Philosophy**: Not limiting yourself to a tech stack is the result of understanding basic principles of backend engineering.

---

## Infrastructure & Networking

### DNS (Domain Name System)

DNS translates human-readable domain names into IP addresses that computers use to communicate.

**DNS Record Types**:
- **A Record**: Points directly to a specific IP address
- **CNAME Record**: Points to another domain or server name

**DNS Resolution Flow**:
1. Browser checks its local DNS cache
2. Query reaches Recursive Resolver (provided by ISP or third-party like Google DNS)
3. Recursive Resolver connects to Root Servers (only 13 across the world, highest in hierarchy)
4. Root servers provide IP address of TLD servers (.com, .in, etc.)
5. Query goes to the appropriate TLD name server
6. TLD server returns the authoritative name server for the domain
7. Final IP address is returned and cached at multiple levels

**Caching Layers**:
- Browser DNS cache
- Operating system DNS cache
- Recursive resolver cache (maintained by ISP)

### AWS Infrastructure

**EC2 & Security Groups**:
- Security groups are assigned to EC2 instances
- They define which ports are allowed for incoming/outgoing traffic
- SSH (port 22) is commonly used to login from command line
- Security groups act as virtual firewalls at the instance level

### NGINX - The Reverse Proxy

**What is NGINX?**

NGINX is a high-performance, open-source software used primarily as:
- **Web Server**: Serves static files efficiently
- **Reverse Proxy**: Sits in front of backend servers
- **Load Balancer**: Distributes traffic across multiple servers
- **HTTP Cache**: Caches responses to reduce backend load

**Why Use a Reverse Proxy?**

A reverse proxy is a server that sits in front of your application servers and manages:
- Request routing and redirection
- SSL/TLS termination
- Load balancing
- Centralized configuration management
- Security (hiding backend server details)

**Key Advantage**: Instead of configuring redirects and routes on each backend server individually, you manage everything in one place.

**Design for Maximum Concurrency**:
- Handles thousands of simultaneous connections
- Low memory footprint
- Event-driven, asynchronous architecture
- Ideal for high-traffic applications

**Request Flow**:
```
Browser → DNS Server → AWS Firewalls → AWS Instance → NGINX → Node.js Server (localhost)
```

---

## Why Backend Can't Be Written in Frontend

### 1. Security Measures

**Browser Sandbox Environment**:
- Browsers run in isolated environments (sandboxes)
- Code can only access limited resources:
  - DOM APIs
  - Cookie storage
  - Local storage
  - Limited file system access

**Why Sandboxing Exists**:
If browsers weren't sandboxed, malicious code could:
- Access your computer's file system
- Read sensitive data
- Execute harmful operations
- Compromise your entire system

### 2. CORS (Cross-Origin Resource Sharing)

**What is CORS?**

CORS is a browser-level security mechanism that restricts web applications from making requests to a different domain than the one that served the application.

**How it Works**:
- Browsers block cross-origin requests by default
- Server must explicitly allow requests from specific origins
- Configured through HTTP headers:
  - `Access-Control-Allow-Origin`
  - `Access-Control-Allow-Methods`
  - `Access-Control-Allow-Headers`

**Purpose**:
- Prevents malicious websites from making unauthorized requests
- Protects user data and privacy
- Enables controlled resource sharing between domains

### 3. Database Connections

**Backend Advantages**:
- Maintains connection pools for efficient database access
- Optimizes queries centrally
- Manages transactions properly

**Frontend Limitations**:
- Browsers aren't designed for persistent database connections
- Each user would require separate connections
- No query optimization possible
- Security risk of exposing database credentials
- Overwhelming database with individual user connections

### 4. Computing Power

- Backend servers have dedicated resources
- Can handle heavy computations, data processing
- Run complex algorithms without impacting user experience
- Scale horizontally by adding more servers

---

## HTTP Protocol

### Why HTTP is the Heart of Backend

HTTP (Hypertext Transfer Protocol) is fundamental to backend engineering for several reasons:

#### 1. Statelessness

**What it Means**:
- HTTP has no memory of past interactions
- Each request carries all information needed to process it
- Server doesn't maintain session state between requests

**Benefits**:
- **Simplicity**: Clean, predictable design
- **Scalability**: Any server can handle any request
- **No session overhead**: Don't need to store session information on server

**Developer Responsibility**:
Developers must include necessary state information in:
- Cookies
- Headers
- Request body
- URL parameters

#### 2. Client-Server Model

**Architecture**:
- Clear separation between client and server
- Connection always initiated by client
- Server responds to requests

**Protocols**:
- **HTTP**: Standard protocol
- **HTTPS**: HTTP + Security (SSL/TLS encryption)

### TCP vs UDP

**TCP (Transmission Control Protocol)**:
- Reliable data transmission
- Guarantees packet delivery and order
- Connection-oriented
- Used by HTTP

**UDP (User Datagram Protocol)**:
- Faster but unreliable
- No guaranteed delivery
- Connectionless
- Used for streaming, gaming (HTTP/3 uses QUIC over UDP)

### HTTP Evolution

#### HTTP/1.0
- Each request = new TCP connection
- Very slow performance
- Connection closed after each response

#### HTTP/1.1
- **Persistent connections**: Multiple requests over same TCP connection
- **Pipelining**: Send multiple requests without waiting for responses
- **Better caching**: Improved cache control mechanisms
- **Reduced latency**: Reusing connections saves time

#### HTTP/2.0
- **Multiplexing**: Multiple requests/responses over single connection simultaneously
- **Header compression (HPACK)**: Reduces overhead
- **Binary framing**: More efficient than text-based HTTP/1.1
- **Server push**: Server can send resources before client requests them

#### HTTP/3.0
- **Uses QUIC protocol**: Built on UDP instead of TCP
- **Designed for speed and reliability**
- **Better performance on unreliable networks**
- **Reduced connection establishment time**

**Current State**: HTTP/1.1, 2.0, and 3.0 are all in active use depending on server/client support.

### HTTP Request/Response Structure

**Request Components**:
```
GET /api/users/123 HTTP/1.1
Host: example.com
Authorization: Bearer token123
Content-Type: application/json
User-Agent: Mozilla/5.0

{
  "name": "John Doe"
}
```

**Response Components**:
```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9"

{
  "id": 123,
  "name": "John Doe"
}
```

### HTTP Headers

**What are Headers?**
- Key-value pairs in HTTP messages
- Extensible and modifiable
- Act as "remote control" for server behavior

**Common Header Types**:

**Request Headers**:
- `Authorization`: Authentication credentials
- `User-Agent`: Client information
- `Accept`: Preferred response format
- `Content-Type`: Body format being sent

**Response Headers**:
- `Content-Type`: Response body format
- `Cache-Control`: Caching directives
- `Set-Cookie`: Send cookies to client
- `ETag`: Resource version identifier

**Resource**: [HTTP Headers - GeeksforGeeks](https://www.geeksforgeeks.org/web-tech/http-headers/)

### HTTP Methods

**Purpose**: Define the **intent of the interaction** and give clear semantic meaning.

| Method | Purpose | Idempotent |
|--------|---------|------------|
| GET | Fetch data | Yes |
| POST | Create/Send data | No |
| PUT | Update (complete replacement) | Yes |
| PATCH | Update (partial modification) | No* |
| DELETE | Delete data | Yes |

**Idempotent vs Non-Idempotent**:
- **Idempotent**: Multiple identical requests have same effect as one request
- **Non-idempotent**: Multiple requests may have different effects

*PATCH can be idempotent depending on implementation

### HTTP Status Codes

Status codes provide consistent, language-agnostic communication between client and server.

**Categories**:

**1xx - Informational**:
- `100 Continue`: Server received headers, client can continue
- `101 Switching Protocols`: Server switching to different protocol

**2xx - Success**:
- `200 OK`: Request successful
- `201 Created`: Resource created successfully
- `204 No Content`: Success but no response body

**3xx - Redirection**:
- `301 Moved Permanently`: Resource permanently moved
- `302 Found`: Temporary redirect
- `304 Not Modified`: Cached version is still valid

**4xx - Client Errors**:
- `400 Bad Request`: Invalid request syntax
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Server understands but refuses
- `404 Not Found`: Resource doesn't exist
- `429 Too Many Requests`: Rate limit exceeded

**5xx - Server Errors**:
- `500 Internal Server Error`: Generic server error
- `502 Bad Gateway`: Invalid response from upstream server
- `503 Service Unavailable`: Server temporarily unavailable
- `504 Gateway Timeout`: Upstream server timeout

**Resource**: [HTTP Status Codes - GeeksforGeeks](https://www.geeksforgeeks.org/computer-networks/what-are-http-status-codes/)

### HTTP Caching

**Purpose**: Store copies of responses for reuse, avoiding redundant data downloads.

**How it Works**:
1. Client makes request
2. Server responds with cache headers (`Cache-Control`, `ETag`, `Last-Modified`)
3. Client caches response
4. Subsequent requests check cache validity:
   - If valid → use cached version
   - If expired → request fresh copy

**Cache Headers**:
- `Cache-Control: max-age=3600`: Cache for 1 hour
- `ETag`: Unique identifier for resource version
- `Last-Modified`: When resource was last changed

**Benefits**:
- Reduced bandwidth usage
- Faster page loads
- Lower server load
- Better user experience

### Content Negotiation

**What is it?**

A mechanism where client and server agree on the best format to exchange data.

**Negotiation Types**:

**1. Media Type** (Accept header):
```
Accept: application/json, application/xml
```

**2. Language** (Accept-Language):
```
Accept-Language: en-US, en;q=0.9, es;q=0.8
```

**3. Encoding** (Accept-Encoding):
```
Accept-Encoding: gzip, deflate, br
```

**Fallback Strategy**:
If server can't provide preferred format, it sends the best available alternative.

### Handling Large Requests/Responses

**Multipart Content**:
- Data sent/received in chunks
- Efficient for large files
- Used in file uploads, streaming

**Example Use Cases**:
- File uploads
- Video streaming
- Large dataset transfers

### SSL/TLS and HTTPS

**SSL (Secure Sockets Layer)**:
- Legacy protocol for securing data transmission
- Uses encryption to protect sensitive information
- Now deprecated in favor of TLS

**TLS (Transport Layer Security)**:
- Modern, more secure version of SSL
- Uses certificates to authenticate servers
- Encrypts connection between client and server

**HTTPS**:
- HTTP + TLS/SSL
- Encrypts all communication
- Prevents man-in-the-middle attacks
- Essential for sensitive data (passwords, payment info)

**How TLS Works**:
1. Client connects to server
2. Server sends TLS certificate
3. Client verifies certificate
4. Encrypted connection established
5. All data transmitted is encrypted

---

## Routing

### What is Routing?

Routing is the process of determining how an application responds to a client request at a specific endpoint (URI/path) and HTTP method.

**Basic Concept**:
```
HTTP Method + API Route + Handler = Response
```

### Route Types

#### 1. Static Routes
Fixed paths with no variables:
```
GET /api/books
POST /api/books
DELETE /api/books
```

#### 2. Dynamic Routes (Path Parameters)
Include variable segments:
```
GET /api/users/:id
GET /api/products/:productId
DELETE /api/orders/:orderId
```

The `:id`, `:productId`, etc. are placeholders that accept any value.

#### 3. Query Parameters
Key-value pairs appended to URL:
```
GET /api/search?query=backend&category=engineering&page=2
GET /api/products?sort=price&order=desc
```

**Structure**: `/path?key1=value1&key2=value2`

#### 4. Nested Routes
Hierarchical resource relationships:
```
GET /api/users/123/posts/678
GET /api/organizations/:orgId/teams/:teamId/members
```

**Use Case**: Accessing resources that belong to parent resources.

### Route Design Principles

1. **RESTful naming**: Use nouns, not verbs (`/users` not `/getUsers`)
2. **Consistent structure**: Follow patterns across your API
3. **Hierarchy**: Use nesting for related resources
4. **Versioning**: Include version in URL (`/api/v1/users`)

---

## Serialization & Deserialization

### The Problem

Two machines at different locations need to communicate:
- Client (JavaScript)
- Server (Rust/Python/Java)

How do they exchange data when they use different languages?

### The Solution

**Serialization & Deserialization**: Converting data to/from a language-agnostic format.

**Serialization**: 
```
Language Object → Standard Format (JSON/XML)
```

**Deserialization**:
```
Standard Format (JSON/XML) → Language Object
```

### Standards

#### Text-Based Formats

**JSON (JavaScript Object Notation)**:
```json
{
  "name": "John Doe",
  "age": 30,
  "email": "john@example.com"
}
```
- Human-readable
- Language-agnostic
- Most popular for web APIs

**XML (eXtensible Markup Language)**:
```xml
<user>
  <name>John Doe</name>
  <age>30</age>
  <email>john@example.com</email>
</user>
```
- Verbose but structured
- Good for complex data
- Used in enterprise systems

**YAML (YAML Ain't Markup Language)**:
```yaml
name: John Doe
age: 30
email: john@example.com
```
- Human-friendly
- Used in configuration files
- Whitespace-sensitive

#### Binary Formats

- **Protocol Buffers (protobuf)**
- **MessagePack**
- **BSON**

**Advantages**:
- More compact
- Faster parsing
- Efficient for high-volume data

**Disadvantages**:
- Not human-readable
- Require schema definitions

### Why It Matters

Serialization/deserialization enables:
- **Interoperability**: Different systems can communicate
- **Data persistence**: Save and load application state
- **Network transmission**: Send data over HTTP
- **Platform independence**: Works across operating systems

---

## Authentication & Authorization

### Core Concepts

**Authentication**: 
- Verifying identity
- Answers: "Who are you?"
- Example: Login with username/password

**Authorization**:
- Verifying permissions
- Answers: "What can you do?"
- Example: Admin vs. regular user roles

### Modern Authentication Types

- **OAuth 2.0**: Delegation-based authorization
- **JWT (JSON Web Tokens)**: Stateless authentication
- **Zero Trust Architecture**: Never trust, always verify

---

## Sessions vs JWT

### The Statelessness Problem

HTTP is stateless, meaning each request is independent. But we needed:
- User login persistence across pages
- Shopping cart retention
- User preferences

This led to **sessions** and later **JWT tokens**.

---

### Session-Based Authentication (Stateful)

**How It Works**:

1. User sends credentials (email, password)
2. Server validates credentials
3. Server creates unique session ID
4. Session ID + user data stored in persistent storage (e.g., Redis)
5. Session ID sent to client as a cookie
6. Client includes cookie in all subsequent requests
7. Server looks up session ID in storage to verify user

**Flow Diagram**:
```
Client → Login (credentials)
Server → Validate → Create Session ID → Store in Redis
Server → Send Cookie (Session ID)
Client → Request + Cookie
Server → Lookup Session ID in Redis → Authorize
```

**Advantages**:
✅ Real-time information about all users
✅ Can revoke/logout users immediately
✅ More secure (server controls everything)
✅ Suitable for web applications where cookies work well

**Disadvantages**:
❌ Server must store session data for every user
❌ Memory overhead increases with users
❌ Scaling requires session replication
❌ Latency and consistency challenges
❌ Not ideal for mobile apps

**Best For**: SaaS applications, traditional web apps

---

### JWT-Based Authentication (Stateless)

**How It Works**:

1. User sends credentials (email, password)
2. Server validates credentials
3. Server creates signed JWT containing user info and roles
4. JWT sent back to client
5. Client stores JWT (localStorage, sessionStorage)
6. Client sends JWT in Authorization header
7. Server verifies JWT signature using secret key
8. No database lookup needed - all info in token

**JWT Structure**:
```
Header.Payload.Signature
```

**Parts**:
- **Header**: Algorithm and token type
- **Payload**: User data (claims) - user ID, roles, expiry
- **Signature**: Encrypted hash (Header + Payload + Secret)

**Example JWT**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VySWQiOjEyMywicm9sZSI6ImFkbWluIiwiZXhwIjoxNjQwOTk1MjAwfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Flow Diagram**:
```
Client → Login (credentials)
Server → Validate → Generate JWT (sign with secret)
Server → Send JWT
Client → Store JWT
Client → Request + JWT in header
Server → Verify signature → Decode → Authorize
```

**Advantages**:
✅ Highly scalable (no server-side storage)
✅ Perfect for distributed systems
✅ Works great for mobile applications
✅ Reduced database queries
✅ Stateless - any server can verify

**Disadvantages**:
❌ Cannot revoke tokens before expiry
❌ Compromised token = full access until expiry
❌ Larger payload size (sent with every request)
❌ Difficult to invalidate individual sessions

**Best For**: Microservices, mobile apps, distributed systems

**Security Note**: JWT tokens use Base64 encoding, making them interoperable across systems, but they're not encrypted by default - avoid storing sensitive data in payload.

---

### Token Theft & Security

**JWT Token Theft Risk**:
If someone steals your JWT token, they can impersonate you until the token expires.

**Mitigation Strategies**:
- Short expiration times (15-60 minutes)
- Refresh token rotation
- HTTPS only
- Secure storage (httpOnly cookies)
- Token blacklisting for critical operations

**Best Practice**: Use third-party authentication providers (Auth0, Firebase Auth, AWS Cognito) in production systems.

---

### Hybrid Approach

**Strategy**: Combine session-based and JWT-based authentication.

**Implementation**:
- **Web applications**: Use stateful sessions (better control)
- **Mobile applications**: Use stateless JWT (better scaling)
- **Microservices**: JWT for service-to-service communication

---

### Other Authentication Methods

#### API Key-Based Authentication

**How It Works**:
- Client gets a unique API key
- Key sent with every request
- Server validates key

**Use Cases**:
- Machine-to-machine communication
- Third-party integrations
- Simple authentication needs

**Pros**:
✅ Easy to implement
✅ Good for programmatic access

**Cons**:
❌ Less secure than OAuth
❌ Difficult to manage permissions
❌ No user delegation

---

### OAuth 2.0 - Delegation & Authorization

**The Problem OAuth Solves**:

Imagine you want to use a third-party app to access your Google Drive files. How do you grant access without sharing your Google password?

**Solution**: OAuth 2.0 allows **delegated access** through tokens with limited capabilities.

#### OAuth Evolution

**OAuth 1.0**:
- Complex cryptographic signatures
- Difficult to implement
- Good security but cumbersome

**OAuth 2.0**:
- Introduced bearer tokens (simpler)
- Multiple authorization flows
- Easier implementation
- Industry standard

#### OAuth 2.0 Roles

1. **Resource Owner**: The user (person who owns the data)
2. **Client**: The application requesting access
3. **Resource Server**: The server hosting the protected resources
4. **Authorization Server**: Issues access tokens

#### OAuth 2.0 Flow (Authorization Code)

```
1. Client redirects user to Authorization Server
2. User logs into their account
3. User grants permissions to Client
4. Authorization Server sends authorization code to Client
5. Client exchanges code for access token
6. Client uses access token to access Resource Server
```

**Example**:
```
App (Client) wants to access your Google Drive (Resource Server)
→ You (Resource Owner) login to Google (Auth Server)
→ You grant permission
→ App receives access token
→ App uses token to read your files
```

#### Important Distinction

**OAuth 2.0 is for Authorization, NOT Authentication**.

It answers: "What can this app do?" not "Who is this user?"

---

### OpenID Connect (OIDC)

**What is OIDC?**

OpenID Connect builds on OAuth 2.0 to add **authentication** capabilities.

**Key Addition**: **ID Token** (a JWT containing user identity information)

**ID Token Contains**:
- User ID
- Email
- Name
- Token issuer
- Expiration time

**OIDC Flow**:
```
1. Client redirects to Authorization Server
2. User logs in and grants permissions
3. Authorization Server sends:
   - Authorization code
   - ID Token (JWT with user info)
4. Client exchanges code for:
   - Access token (for API access)
   - ID Token (for user identity)
```

**Use Cases**:
- "Sign in with Google"
- "Login with Facebook"
- Single Sign-On (SSO)

---

### Role-Based Access Control (RBAC)

**Purpose**: Distribute different roles and permissions to users.

**Example Roles**:
- Admin: Full access
- Editor: Read + Write
- Viewer: Read only

**Implementation**:
```javascript
// User object with role
{
  userId: 123,
  role: "editor",
  permissions: ["read", "write"]
}

// Authorization check
if (user.permissions.includes("write")) {
  // Allow edit operation
}
```

---

### Security: Timing Attacks

**The Vulnerability**:

Servers often check username first, then password:
```python
if username != stored_username:
    return error  # Returns immediately

if password != stored_password:
    return error  # Takes longer (hashing comparison)
```

**Attack**:
Attackers measure response time differences to determine:
- If username exists (faster response)
- If password is wrong (slower response)

**Mitigation**:
```python
# Add consistent delay
time.sleep(random.uniform(0.1, 0.3))
return error
```

Always introduce a fake, consistent delay to prevent timing-based attacks.

---

### Cookies

**What is a Cookie?**

A cookie is a small piece of data stored in the user's browser, sent by the server.

**Key Property**: Cookies are accessible only by the server that created them.

**Cookie Attributes**:
- **HttpOnly**: JavaScript cannot access (prevents XSS attacks)
- **Secure**: Only sent over HTTPS
- **SameSite**: Controls cross-site request behavior
- **Expiration**: When cookie becomes invalid

**Use Cases**:
- Session management
- User preferences
- Tracking (analytics)

**Example**:
```
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
```

---

## Request Handling

### Application Layers

Modern backend applications typically follow a layered architecture:

```
Request → Middleware → Controller → Service → Repository → Database
```

#### 1. Controller Layer (Handler)

**Responsibilities**:
- Receive HTTP requests
- Extract data from request (query params, body, headers)
- Call service layer methods
- Format and return HTTP responses
- Handle HTTP-specific concerns

**Example**:
```javascript
async function getUserController(req, res) {
  const userId = req.params.id;
  const user = await userService.getUser(userId);
  res.status(200).json(user);
}
```

#### 2. Service Layer (Business Logic)

**Responsibilities**:
- Implement business rules
- Coordinate between multiple repositories
- Handle complex operations
- Make external API calls
- Send notifications
- Trigger webhooks

**Example**:
```javascript
async function createOrderService(orderData) {
  // Business logic
  const user = await userRepository.findById(orderData.userId);
  const product = await productRepository.findById(orderData.productId);
  
  if (product.stock < orderData.quantity) {
    throw new Error('Insufficient stock');
  }
  
  const order = await orderRepository.create(orderData);
  await emailService.sendOrderConfirmation(user.email, order);
  
  return order;
}
```

#### 3. Repository Layer (Data Access)

**Responsibilities**:
- Direct database interactions
- Execute queries
- Return raw data
- Single responsibility per method

**Principle**: Each repository method should perform only one type of database operation.

**Example**:
```javascript
async function findUserById(id) {
  return await db.query('SELECT * FROM users WHERE id = ?', [id]);
}

async function createUser(userData) {
  return await db.query('INSERT INTO users SET ?', [userData]);
}
```

---

### Request Flow

#### 1. Extract Data from Request

**GET requests** - Extract from:
- Query parameters: `req.query`
- Path parameters: `req.params`

**POST/PUT/PATCH/DELETE** - Extract from:
- Request body: `req.body`
- Headers: `req.headers`

#### 2. Deserialization (Binding)

Convert JSON to language-specific format:
```
JSON → JavaScript Object / Python Dict / Java Object
```

**Example**:
```javascript
// Incoming JSON
{
  "name": "John",
  "age": "30"
}

// Deserialized to JavaScript object
const user = {
  name: "John",
  age: 30  // String "30" → Number 30
};
```

#### 3. Validation & Transformation

**Validation Types**:

**Syntactic Validation**: Structure check
```javascript
// Email format
if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
  throw new Error('Invalid email format');
}
```

**Semantic Validation**: Meaning check
```javascript
// Age must be positive
if (age < 0) {
  throw new Error('Age cannot be negative');
}
```

**Type Validation**: Data type check
```javascript
// Ensure age is a number
if (typeof age !== 'number') {
  throw new Error('Age must be a number');
}
```

**Transformation Example**:
```javascript
// JSON sends strings by default
// Transform to correct type
const userId = parseInt(req.body.userId); // "123" → 123
const price = parseFloat(req.body.price); // "19.99" → 19.99
```

**Why Both Frontend and Backend Validation?**

- **Frontend validation**: Better user experience (immediate feedback)
- **Backend validation**: Security and data integrity (never trust client)

---

## Middleware

### What is Middleware?

Middleware are functions that execute between receiving a request and sending a response. They can:
- Process the request
- Modify request/response objects
- End request-response cycle
- Call next middleware in stack

**Signature**:
```javascript
function middleware(req, res, next) {
  // Process request
  // Either send response or call next()
  next(); // Pass to next middleware
}
```

### Request Pipeline

```
Client Request
  ↓
Middleware 1 (CORS)
  ↓
Middleware 2 (Logging)
  ↓
Middleware 3 (Authentication)
  ↓
Router
  ↓
Middleware 4 (Validation)
  ↓
Handler/Controller
  ↓
Service Layer
  ↓
Repository Layer
  ↓
...
Response to Client
```

### Common Middleware Types

#### 1. Security Middleware

**CORS (Cross-Origin Resource Sharing)**:
```javascript
app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  next();
});
```

**Authentication Verification**:
```javascript
function authenticateToken(req, res, next) {
  const token = req.headers['authorization'];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' });
    }
    req.user = user;
    next();
  });
}
```

**Rate Limiting**:
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per windowMs
  message: 'Too many requests, please try again later',
  statusCode: 429
});

app.use('/api/', limiter);
```

#### 2. Logging & Monitoring

```javascript
function requestLogger(req, res, next) {
  console.log(`${new Date().toISOString()} - ${req.method} ${req.url}`);
  console.log('Headers:', req.headers);
  console.log('Body:', req.body);
  next();
}
```

#### 3. Error Handling

```javascript
function errorHandler(err, req, res, next) {
  console.error('Error:', err);
  
  res.status(err.status || 500).json({
    error: {
      message: err.message || 'Internal server error',
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
}

// Use as last middleware
app.use(errorHandler);
```

#### 4. Compression

```javascript
const compression = require('compression');

// Compress responses
app.use(compression());
```

### Middleware Ordering

**Order matters!** Middleware executes in the order it's defined.

**Typical Order**:
1. CORS (must be first)
2. Compression
3. Body parsing
4. Logging
5. Authentication
6. Rate limiting
7. Routes
8. Error handling (must be last)

---

### Request Context

**What is Request Context?**

A storage mechanism accessible across middleware and services for the current request.

**Use Cases**:
- Pass user information through request pipeline
- Store request-specific metadata
- Logging correlation IDs
- Performance metrics

**Example**:
```javascript
// Middleware adds user to request context
function authenticate(req, res, next) {
  const user = verifyToken(req.headers.authorization);
  req.context = { user, requestId: generateId() };
  next();
}

// Controller accesses context
function getUserProfile(req, res) {
  const userId = req.context.user.id;
  // ...
}
```

---

## API Design (REST)

### History of REST

**Creator**: Roy Fielding (2000)
- Co-author of HTTP specification
- Introduced REST in his doctoral dissertation

**Early Web Challenges**:
- Tim Berners-Lee created HTTP, HTML, URI, web servers (1990)
- WWW grew rapidly
- Scalability became a major issue
- Needed architectural principles for distributed systems

**Roy Fielding's Solution**: REST architectural style

**Resource**: [Fielding's Dissertation](https://roy.gbiv.com/pubs/dissertation/fielding_dissertation.pdf)

---

### REST Principles

REST stands for **Representational State Transfer**.

#### Breaking Down REST

**Representational**:
- Resources represented in various formats (JSON, XML, HTML)
- Same resource can have multiple representations
- Client specifies preferred format via `Accept` header

**State**:
- Current condition of a resource
- Can be transferred between client and server
- Example: Cart state in e-commerce

**Transfer**:
- Movement of resource representations using HTTP methods
- State changes through well-defined operations

---

### Core REST Constraints

#### 1. Client-Server Architecture
- Separation of concerns
- Client and server evolve independently
- Clear interface between them

#### 2. Stateless
- No session state on server
- Each request contains all necessary information
- Enables better scalability

#### 3. Cacheable
- Responses must define if they're cacheable
- Improves performance and scalability
- Reduces server load

#### 4. Uniform Interface
- Consistent resource identification (URIs)
- Standardized operations (HTTP methods)
- Self-descriptive messages
- HATEOAS (Hypermedia as the Engine of Application State)

#### 5. Layered System
- Client can't tell if connected directly to end server
- Intermediary servers (load balancers, caches) transparent
- Improves scalability and security

#### 6. Code on Demand (Optional)
- Server can extend client functionality
- Send executable code (JavaScript) to client
- Not commonly used

---

### REST API Design Process

#### 1. Schema-First Design

**Approach**:
1. Design API schema/blueprint first
2. Define resources and endpoints
3. Specify request/response formats
4. Document before implementation

**Tools**:
- **OpenAPI (Swagger)**: API specification standard
- **Insomnia/Postman**: API design and testing
- **API Blueprint**: Documentation format

#### 2. Identify Resources

**Questions to Ask**:
- What entities exist in your domain?
- What relationships exist between entities?
- What operations do users need?

**Example - Project Management System**:
- Resources: Users, Projects, Tasks, Comments
- Relationships: Users own Projects, Projects contain Tasks

#### 3. Design Endpoints

**RESTful Naming Conventions**:

✅ **Good**:
```
GET    /users          # List users
GET    /users/123      # Get specific user
POST   /users          # Create user
PUT    /users/123      # Update user
DELETE /users/123      # Delete user

GET    /users/123/projects      # User's projects
POST   /projects/456/tasks      # Add task to project
```

❌ **Bad**:
```
GET    /getUsers
POST   /createUser
GET    /user/delete/123
POST   /updateUserInfo
```

**Principles**:
- Use nouns, not verbs
- Use plural names for collections
- Use HTTP methods for actions
- Keep URLs hierarchical and logical

---

### Idempotency in REST

**Definition**: An operation is idempotent if calling it multiple times has the same effect as calling it once.

**Idempotent Methods**:
- **GET**: Reading data doesn't change it
- **PUT**: Updating to same value multiple times = same result
- **DELETE**: Deleting already-deleted resource = same result

**Non-Idempotent Methods**:
- **POST**: Creating multiple times = multiple resources

**Example**:
```
POST /orders         # Non-idempotent (creates new order each time)
PUT /orders/123      # Idempotent (same state after multiple updates)
DELETE /orders/123   # Idempotent (deleted is deleted)
```

**Why It Matters**:
- Safe to retry idempotent operations
- Network failures can be handled gracefully
- Client retries don't cause duplicate effects

---

### Pagination

**Problem**: Returning large lists of resources is inefficient.

**Solution**: Return data in chunks (pages).

**Common Approaches**:

#### Offset-Based Pagination
```
GET /api/users?page=2&limit=20
```

**Response**:
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

#### Cursor-Based Pagination
```
GET /api/users?cursor=eyJpZCI6MTIzfQ==&limit=20
```

**Benefits**:
- More efficient for real-time data
- Handles insertions/deletions better
- Used by Facebook, Twitter

**Why Pagination Matters**:
- Reduces response size
- Faster serialization/deserialization
- Better user experience
- Lower memory usage

---

## Databases

### What is a Database?

A database is a structured storage system that allows CRUD (Create, Read, Update, Delete) operations with:
- Data persistence (survives app restarts)
- Structure and organization
- Data integrity
- Concurrency control

### Why Not Text Files?

**Problems with Text Files**:
- Difficult parsing
- No enforced structure
- No data integrity constraints
- Poor concurrency handling
- No query optimization
- No transaction support
- Difficult to scale

---

### PostgreSQL

**Why PostgreSQL?**

- **Open Source**: Free and community-driven
- **SQL Standards Compliance**: Follows SQL standards closely
- **Extensible**: Rich feature set, custom functions, data types
- **Scalable**: Handles large datasets efficiently
- **JSON Support**: Excellent support for JSON data (better than many NoSQL databases)
- **ACID Compliant**: Reliable transactions
- **Active Development**: Regular updates and improvements

---

### PostgreSQL Features

#### Custom Data Types - ENUM

Create a data type that accepts only specific values:

```sql
-- Define custom type
CREATE TYPE project_status AS ENUM ('active', 'completed', 'on_hold', 'cancelled');

-- Use in table
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  status project_status DEFAULT 'active'
);

-- Insert
INSERT INTO projects (name, status) VALUES ('API Project', 'active');

-- Only accepts defined values
INSERT INTO projects (name, status) VALUES ('Bad Project', 'invalid'); -- ERROR
```

**Benefits**:
- Data validation at database level
- Prevents invalid values
- Clear documentation of allowed states
- Better than CHECK constraints

#### Advanced PostgreSQL Features

- **Full-text search**: Built-in search capabilities
- **JSONB**: Binary JSON with indexing
- **Array types**: Store arrays directly
- **Range types**: Date ranges, integer ranges
- **Window functions**: Advanced analytics
- **CTEs (Common Table Expressions)**: Complex queries
- **Triggers**: Automatic actions on data changes
- **Stored Procedures**: Server-side logic

---

### Database Migrations

**What are Migrations?**

Migrations are a sequence of changes to your database schema tracked over time.

**Purpose**:
- Version control for database schema
- Track all changes in order
- Easy rollback to previous state
- Consistent schema across environments
- Team collaboration

**Example Tools**:
- **Flyway**: Java-based migration tool
- **Liquibase**: Database-independent migrations
- **Knex.js**: JavaScript migrations
- **Alembic**: Python migrations
- **Rails Migrations**: Ruby on Rails

**Migration File Example**:
```sql
-- Migration: 001_create_users_table.sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Migration: 002_add_username_to_users.sql
ALTER TABLE users ADD COLUMN username VARCHAR(100);

-- Rollback for 002
ALTER TABLE users DROP COLUMN username;
```

**Benefits**:
- Never lose track of database changes
- Easy to replicate schema across environments
- Safe rollbacks if something goes wrong
- Clear history of schema evolution

---

## Business Logic Layer

### Separation of Concerns

**Architecture**:
```
Controller → Service (Business Logic) → Repository → Database
```

**Service Layer Responsibilities**:
- Implement business rules
- Coordinate multiple operations
- Handle complex workflows
- Integrate external services
- Enforce business constraints

**Example - E-commerce Order Processing**:
```javascript
async function processOrder(orderData) {
  // 1. Validate business rules
  if (orderData.total < MINIMUM_ORDER_VALUE) {
    throw new Error('Order below minimum value');
  }
  
  // 2. Check inventory
  const product = await productRepository.findById(orderData.productId);
  if (product.stock < orderData.quantity) {
    throw new Error('Insufficient stock');
  }
  
  // 3. Calculate final price with discounts
  const finalPrice = calculateDiscount(orderData);
  
  // 4. Process payment
  const payment = await paymentGateway.charge(finalPrice);
  
  // 5. Create order
  const order = await orderRepository.create({
    ...orderData,
    finalPrice,
    paymentId: payment.id
  });
  
  // 6. Update inventory
  await productRepository.updateStock(product.id, product.stock - orderData.quantity);
  
  // 7. Send notifications
  await emailService.sendOrderConfirmation(orderData.email, order);
  await webhookService.notifyPartner(order);
  
  return order;
}
```

**Why Business Logic in Service Layer?**
- Reusability across different controllers
- Easier testing
- Clear separation from HTTP concerns
- Single source of truth for business rules

---

## Caching

### What is Caching?

Caching is storing a subset of frequently accessed data in a location that's faster and easier to access than the original source.

**Goal**: Reduce latency and server load by avoiding expensive operations (database queries, API calls, computations).

---

### Types of Caching

#### 1. Network-Level Caching - CDN

**CDN (Content Delivery Network)**:
- Caches static assets (images, CSS, JS, videos)
- Distributes content to edge servers worldwide
- Serves content from geographically closest server

**Components**:
- **PoP (Point of Presence)**: Geographic location with multiple edge servers
- **Edge Servers**: Servers that cache and serve content
- **Origin Server**: Original source of content

**How It Works**:
1. User requests resource (e.g., image)
2. DNS points to nearest PoP
3. Edge server checks cache
4. If cached → serve immediately
5. If not cached → fetch from origin, cache, then serve
6. TTL (Time To Live) determines how long to cache

**Benefits**:
- Reduced latency (closer to user)
- Reduced origin server load
- Better availability
- DDoS protection

**Example Providers**: CloudFlare, AWS CloudFront, Akamai

---

#### 2. DNS Caching

**Multiple Cache Levels**:

1. **Browser DNS Cache**: 
   - Stores recent DNS lookups
   - Fastest lookup

2. **Operating System DNS Cache**: 
   - System-level cache
   - Shared across applications

3. **Recursive Resolver Cache**: 
   - ISP or third-party DNS cache (e.g., 8.8.8.8)
   - Reduces queries to authoritative servers

**DNS Resolution with Caching**:
```
1. Browser checks its cache
2. If not found, OS cache checked
3. If not found, Recursive Resolver checks its cache
4. If not found, query Root Servers
5. Query TLD servers (.com, .org)
6. Query authoritative name server
7. Cache result at all levels
```

**TTL**: Determines how long DNS records are cached

---

#### 3. Application-Level Caching

**In-Memory Caching** (RAM):
- Extremely fast (nanoseconds)
- Volatile (data lost on restart)
- Limited by available memory

**Technologies**:

##### Redis (Remote Dictionary Server)

**What is Redis?**
- In-memory, key-value store
- NoSQL database
- Sub-millisecond latency

**Features**:
- Data structures: Strings, Lists, Sets, Sorted Sets, Hashes
- Persistence options (snapshots, append-only file)
- Pub/Sub messaging
- Transactions
- Lua scripting

**Common Use Cases**:

**1. Database Query Caching**:
```javascript
async function getUser(userId) {
  // Check cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Cache miss - query database
  const user = await db.query('SELECT * FROM users WHERE id = ?', [userId]);
  
  // Store in cache
  await redis.setex(`user:${userId}`, 3600, JSON.stringify(user));
  
  return user;
}
```

**2. Session Storage**:
```javascript
// Store session
await redis.setex(`session:${sessionId}`, 86400, JSON.stringify(sessionData));

// Retrieve session
const session = await redis.get(`session:${sessionId}`);
```

**3. Rate Limiting**:
```javascript
async function checkRateLimit(userId) {
  const key = `rate:${userId}`;
  const count = await redis.incr(key);
  
  if (count === 1) {
    await redis.expire(key, 3600); // 1 hour window
  }
  
  return count <= 100; // Max 100 requests per hour
}
```

**4. Leaderboards** (Sorted Sets):
```javascript
// Update score
await redis.zadd('leaderboard', score, userId);

// Get top 10
const top10 = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');
```

##### Memcached

**Simpler alternative to Redis**:
- Pure key-value cache
- No data structures
- Multi-threaded
- Good for simple caching needs

---

#### 4. CPU Cache (Hardware-Level)

**CPU Cache Hierarchy**:
- **L1 Cache**: Fastest, smallest (per core)
- **L2 Cache**: Moderate speed, moderate size (per core)
- **L3 Cache**: Slower, larger (shared across cores)

**Why It Matters**:
- Modern CPUs use cache to avoid RAM access
- Cache-friendly code runs significantly faster
- Array iteration benefits from cache locality

---

### Caching Strategies

#### 1. Lazy Loading (Cache-Aside)

**How It Works**:
```
1. Check cache
2. If hit → return cached data
3. If miss → query database, cache result, return data
```

**Pros**:
- Only cache what's actually requested
- Simple to implement

**Cons**:
- Cache misses cause latency
- Potential for stale data

#### 2. Write-Through

**How It Works**:
```
1. Write to database
2. Write to cache
3. Return success
```

**Pros**:
- Cache always consistent with database
- No cache misses on reads

**Cons**:
- Higher write latency
- Cache might store unused data

#### 3. Write-Behind (Write-Back)

**How It Works**:
```
1. Write to cache
2. Return success immediately
3. Asynchronously write to database
```

**Pros**:
- Very fast writes
- Reduced database load

**Cons**:
- Risk of data loss
- Complexity in failure handling

---

### Cache Eviction Policies

**When cache is full, what do you remove?**

#### 1. LRU (Least Recently Used)
- Remove items not accessed for longest time
- Most common policy
- Good for general use

#### 2. LFU (Least Frequently Used)
- Remove items accessed least often
- Good for identifying hot data

#### 3. TTL (Time To Live)
- Set expiration time when caching
- Automatic removal after expiry
- Good for time-sensitive data

#### 4. FIFO (First In, First Out)
- Remove oldest items first
- Simple but less effective

**Redis Example**:
```javascript
// Set with TTL
await redis.setex('key', 3600, 'value'); // Expires in 1 hour

// Configure eviction policy
// redis.conf: maxmemory-policy allkeys-lru
```

---

### Caching Best Practices

1. **Cache frequently accessed, rarely changing data**
2. **Set appropriate TTL values**
3. **Monitor cache hit rates**
4. **Handle cache failures gracefully**
5. **Invalidate cache on updates**
6. **Use namespaced keys** (`user:123`, `product:456`)
7. **Consider cache warming** (pre-populate cache)
8. **Monitor memory usage**

---

## Additional Backend Concepts

### Transaction Emails

**What are Transaction Emails?**

Automated emails triggered by user actions, containing important information.

**Examples**:
- Order confirmations
- Password reset links
- Welcome emails
- Invoice receipts
- Shipping notifications

**Key Characteristics**:
- Triggered by events (not marketing)
- Time-sensitive
- Personalized
- High open rates

**Implementation**:
```javascript
async function sendOrderConfirmation(order) {
  await emailService.send({
    to: order.customerEmail,
    subject: `Order Confirmation #${order.id}`,
    template: 'order-confirmation',
    data: {
      orderNumber: order.id,
      items: order.items,
      total: order.total
    }
  });
}
```

**Tools**: SendGrid, Mailgun, AWS SES, Postmark

---

### Task Queuing

**What is Task Queuing?**

Asynchronous job processing - handling time-consuming tasks outside the request-response cycle.

**Use Cases**:
- Email sending
- Image processing
- Report generation
- Data imports
- Video encoding
- Webhook deliveries

**Benefits**:
- Faster API responses
- Better resource utilization
- Retry mechanisms
- Priority handling
- Scalability

**How It Works**:
```
Client Request → API → Queue Job → Return immediately
                        ↓
                   Background Worker processes job
```

**Example - Image Processing**:
```javascript
// Controller
async function uploadImage(req, res) {
  const imageUrl = await storage.upload(req.file);
  
  // Queue background job
  await queue.add('process-image', {
    imageUrl,
    userId: req.user.id,
    operations: ['resize', 'watermark', 'optimize']
  });
  
  // Return immediately
  res.json({ message: 'Image uploaded, processing started' });
}

// Worker
queue.process('process-image', async (job) => {
  const { imageUrl, operations } = job.data;
  
  for (const op of operations) {
    await imageProcessor[op](imageUrl);
  }
  
  await db.updateImageStatus(imageUrl, 'processed');
});
```

**Popular Tools**:
- **Bull** (Node.js): Redis-based queue
- **Celery** (Python): Distributed task queue
- **Sidekiq** (Ruby): Background jobs
- **AWS SQS**: Managed queue service
- **RabbitMQ**: Message broker

---

### Elasticsearch

**What is Elasticsearch?**

A distributed, RESTful search and analytics engine built on Apache Lucene.

**Primary Use Case**: Full-text search

**When to Use**:
- Search functionality (products, documents, logs)
- Real-time analytics
- Log aggregation
- Application performance monitoring

**Key Features**:
- Fast full-text search
- Fuzzy matching
- Faceted search (filters)
- Geospatial queries
- Aggregations

**Example Use Case - E-commerce Search**:
```javascript
// Index products
await elasticsearch.index({
  index: 'products',
  body: {
    name: 'iPhone 14 Pro',
    description: 'Latest iPhone with advanced camera',
    price: 999,
    category: 'smartphones'
  }
});

// Search products
const results = await elasticsearch.search({
  index: 'products',
  body: {
    query: {
      multi_match: {
        query: 'iphone camera',
        fields: ['name', 'description']
      }
    }
  }
});
```

**Not a Database Replacement**:
- Use alongside relational database
- Database = source of truth
- Elasticsearch = search index

---

### Error Handling

**Global Error Handler**:
```javascript
function errorHandler(err, req, res, next) {
  // Log error
  logger.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });
  
  // Determine status code
  const statusCode = err.statusCode || 500;
  
  // Send response
  res.status(statusCode).json({
    error: {
      message: process.env.NODE_ENV === 'production' 
        ? 'An error occurred' 
        : err.message,
      ...(process.env.NODE_ENV === 'development' && { 
        stack: err.stack 
      })
    }
  });
}
```

**Best Practices**:
- Catch all errors
- Log with context
- Don't expose internal details in production
- Use error monitoring (Sentry, Rollbar)
- Return appropriate status codes

---

### Configuration Management

**Environment-Based Config**:

```javascript
// config/index.js
module.exports = {
  port: process.env.PORT || 3000,
  database: {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    name: process.env.DB_NAME
  },
  redis: {
    host: process.env.REDIS_HOST,
    port: process.env.REDIS_PORT
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '1h'
  }
};
```

**Best Practices**:
- Never commit secrets to version control
- Use environment variables
- Different configs for dev/staging/production
- Validate configuration on startup
- Use tools like dotenv (Node.js), python-decouple (Python)

---

### Logging, Monitoring, and Observability

#### Logging

**What to Log**:
- Request/Response details
- Errors and exceptions
- Business events
- Performance metrics

**Log Levels**:
- **ERROR**: Something failed
- **WARN**: Potential issue
- **INFO**: Important events
- **DEBUG**: Detailed information (dev only)

**Structured Logging**:
```javascript
logger.info({
  event: 'user_login',
  userId: 123,
  ip: '192.168.1.1',
  timestamp: new Date().toISOString()
});
```

#### Monitoring

**Metrics to Track**:
- Response times
- Error rates
- Request volume
- CPU/Memory usage
- Database query times

**Tools**: Prometheus, Grafana, Datadog, New Relic

#### Observability

**Three Pillars**:
1. **Logs**: What happened
2. **Metrics**: How much/how many
3. **Traces**: Request flow through system

**Distributed Tracing**:
- Track requests across microservices
- Identify bottlenecks
- Debug complex issues

**Tools**: Jaeger, Zipkin, OpenTelemetry

---

### Graceful Shutdown

**Why It Matters**:
- Finish processing current requests
- Close database connections properly
- Prevent data corruption
- Clean up resources

**Implementation**:
```javascript
const server = app.listen(3000);

process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');
  });
  
  // Close database connections
  await db.close();
  
  // Close Redis connections
  await redis.quit();
  
  process.exit(0);
});
```

---

### Security Best Practices

1. **Input Validation**: Never trust user input
2. **SQL Injection Prevention**: Use parameterized queries
3. **XSS Prevention**: Sanitize output, use CSP headers
4. **CSRF Protection**: Use CSRF tokens
5. **Rate Limiting**: Prevent abuse
6. **HTTPS**: Encrypt all traffic
7. **Secure Headers**: Use Helmet.js or equivalent
8. **Dependency Management**: Keep dependencies updated
9. **Secrets Management**: Use vaults (AWS Secrets Manager, HashiCorp Vault)
10. **Principle of Least Privilege**: Minimal permissions

---

### Scaling and Performance

#### Vertical Scaling (Scale Up)
- Add more resources to single server
- Easier to implement
- Has physical limits

#### Horizontal Scaling (Scale Out)
- Add more servers
- Better for high availability
- Requires load balancing

**Load Balancing Strategies**:
- Round Robin
- Least Connections
- IP Hash
- Weighted Distribution

**Performance Optimization**:
- Database indexing
- Query optimization
- Caching strategies
- CDN for static assets
- Compression (gzip, brotli)
- Connection pooling

---

### Concurrency and Parallelism

**Concurrency**: Managing multiple tasks at once (not necessarily simultaneously)

**Parallelism**: Executing multiple tasks simultaneously

**Node.js Model**:
- Single-threaded event loop
- Concurrent (event-driven)
- Worker threads for CPU-intensive tasks

**Python Model**:
- GIL (Global Interpreter Lock)
- Threading for I/O-bound tasks
- Multiprocessing for CPU-bound tasks

---

### Object Storage and Large Files

**Object Storage Services**:
- AWS S3
- Google Cloud Storage
- Azure Blob Storage

**Use Cases**:
- User uploads (images, documents)
- Backups
- Static website hosting
- Large file storage

**Best Practices**:
- Generate pre-signed URLs for uploads
- Use CDN for delivery
- Implement lifecycle policies
- Set appropriate access controls

**Example - Direct Upload to S3**:
```javascript
// Generate pre-signed URL
const presignedUrl = s3.getSignedUrl('putObject', {
  Bucket: 'my-bucket',
  Key: `uploads/${userId}/${filename}`,
  Expires: 60,
  ContentType: 'image/jpeg'
});

// Client uploads directly to S3 using URL
// No file passes through backend server
```

---

### Real-time Backend Systems

**Technologies**:
- **WebSockets**: Full-duplex communication
- **Server-Sent Events (SSE)**: Server-to-client streaming
- **WebRTC**: Peer-to-peer communication

**Use Cases**:
- Chat applications
- Live notifications
- Collaborative editing
- Live dashboards
- Gaming

**Example - WebSocket Chat**:
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    // Broadcast to all clients
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  });
});
```

---

### Testing and Code Quality

**Testing Pyramid**:
```
        /\
       /E2E\
      /------\
     /Integration\
    /--------------\
   /  Unit Tests    \
  /------------------\
```

**Unit Tests**:
- Test individual functions
- Fast and isolated
- Mock external dependencies

**Integration Tests**:
- Test components together
- Database interactions
- API endpoints

**End-to-End Tests**:
- Test complete user flows
- Slower but comprehensive

**Code Quality Tools**:
- Linters (ESLint, Pylint)
- Formatters (Prettier, Black)
- Static analysis (SonarQube)
- Code coverage (Jest, pytest-cov)

---

### 12-Factor App Principles

Methodology for building modern, scalable applications:

1. **Codebase**: One codebase, many deploys
2. **Dependencies**: Explicitly declare dependencies
3. **Config**: Store config in environment
4. **Backing Services**: Treat as attached resources
5. **Build, Release, Run**: Separate stages
6. **Processes**: Stateless processes
7. **Port Binding**: Self-contained services
8. **Concurrency**: Scale via process model
9. **Disposability**: Fast startup and shutdown
10. **Dev/Prod Parity**: Keep environments similar
11. **Logs**: Treat as event streams
12. **Admin Processes**: Run as one-off processes

**Resource**: [12factor.net](https://12factor.net/)

---

### OpenAPI Standard (Swagger)

**What is OpenAPI?**

A specification for describing RESTful APIs in a standard, machine-readable format.

**Benefits**:
- Auto-generated documentation
- API client generation
- Validation
- Testing tools

**Example OpenAPI Spec**:
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /users:
    get:
      summary: List users
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

**Tools**: Swagger UI, Swagger Editor, ReDoc

---

### Webhooks

**What are Webhooks?**

HTTP callbacks that notify your application when events occur in external systems.

**How They Work**:
```
External Service → Event occurs → HTTP POST to your endpoint
```

**Example - Payment Gateway Webhook**:
```javascript
app.post('/webhooks/payment', async (req, res) => {
  const event = req.body;
  
  // Verify webhook signature
  if (!verifySignature(req.headers['x-signature'], event)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process event
  if (event.type === 'payment.success') {
    await orderService.markAsPaid(event.orderId);
  }
  
  // Acknowledge receipt
  res.status(200).send('OK');
});
```

**Best Practices**:
- Verify webhook signatures
- Respond quickly (200 OK)
- Process asynchronously
- Handle retries
- Log all webhook events

---

### DevOps for Backend Engineers

**CI/CD (Continuous Integration/Continuous Deployment)**:
- Automated testing
- Automated deployments
- Version control

**Infrastructure as Code**:
- Terraform
- AWS CloudFormation
- Ansible

**Containerization**:
- Docker
- Container orchestration (Kubernetes)

**Monitoring and Alerts**:
- Set up alerts for errors
- Monitor resource usage
- Track SLOs (Service Level Objectives)

**Deployment Strategies**:
- Blue-Green deployment
- Canary releases
- Rolling updates

---

## Best Practices

### 1. API Design
- Follow REST principles
- Use proper HTTP methods and status codes
- Version your APIs
- Document thoroughly

### 2. Security
- Validate all inputs
- Use HTTPS everywhere
- Implement rate limiting
- Store secrets securely
- Regular security audits

### 3. Performance
- Index database columns
- Implement caching
- Use CDN for static assets
- Optimize queries
- Monitor performance metrics

### 4. Code Quality
- Write tests
- Use linters and formatters
- Code reviews
- Follow SOLID principles
- Document complex logic

### 5. Error Handling
- Centralized error handling
- Meaningful error messages
- Proper logging
- Don't expose sensitive info

### 6. Scalability
- Design for horizontal scaling
- Use message queues for async tasks
- Implement caching strategies
- Database connection pooling

### 7. Monitoring
- Log everything important
- Set up alerts
- Track key metrics
- Use APM tools

### 8. Documentation
- API documentation (OpenAPI)
- Code comments
- README files
- Architecture diagrams

---

## Deep Dive: Background Tasks & Task Queuing

### What are Background Tasks?

**Definition**: Background tasks are any type of code that runs outside of the request-response lifecycle.

**Key Characteristic**: The API responds to the client immediately without waiting for the task to complete.

**Why Background Tasks?**

When operations take too long to complete within an HTTP request timeout:
- Sending emails
- Processing images
- Generating reports
- Sending push notifications
- Data imports/exports
- Video transcoding
- Batch operations

**Without Background Tasks**:
```
Client → API → Process (5 seconds) → Response
Client waits 5 seconds ❌
```

**With Background Tasks**:
```
Client → API → Queue Task → Immediate Response
                     ↓
              Background Worker processes task
Client waits < 100ms ✅
```

---

### Background Task Architecture

**Common Flow**:

1. **API receives request**
2. **API validates and queues the task**
3. **API returns immediate response** (usually 202 Accepted)
4. **Background worker picks up task from queue**
5. **Worker processes task**
6. **Worker updates status/notifies user**

**Example - Email Sending**:
```javascript
// API Endpoint
app.post('/api/send-email', async (req, res) => {
  const { to, subject, body } = req.body;
  
  // Validate input
  if (!to || !subject || !body) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  // Queue the email task
  const jobId = await emailQueue.add('send-email', {
    to,
    subject,
    body,
    userId: req.user.id
  });
  
  // Respond immediately
  res.status(202).json({ 
    message: 'Email queued for sending',
    jobId: jobId
  });
});

// Background Worker
emailQueue.process('send-email', async (job) => {
  const { to, subject, body } = job.data;
  
  try {
    await emailService.send({ to, subject, body });
    console.log(`Email sent successfully to ${to}`);
  } catch (error) {
    console.error('Failed to send email:', error);
    throw error; // Queue will retry
  }
});
```

---

### Common Background Task Use Cases

#### 1. Sending Emails
```javascript
// Order confirmation
orderQueue.add('send-order-confirmation', {
  orderId: order.id,
  customerEmail: order.email,
  items: order.items
});
```

#### 2. Image Processing
```javascript
// Resize, optimize, generate thumbnails
imageQueue.add('process-image', {
  imageUrl: uploadedImage.url,
  operations: [
    { type: 'resize', width: 800, height: 600 },
    { type: 'thumbnail', width: 150, height: 150 },
    { type: 'optimize', quality: 85 }
  ]
});
```

#### 3. Report Generation
```javascript
// Generate monthly sales report
reportQueue.add('generate-report', {
  type: 'monthly-sales',
  month: '2024-01',
  userId: req.user.id,
  format: 'pdf'
}, {
  priority: 2, // Lower priority
  delay: 60000 // Wait 1 minute before starting
});
```

#### 4. Push Notifications
```javascript
// Send notification to all users
notificationQueue.add('broadcast-notification', {
  title: 'New Feature Released!',
  body: 'Check out our latest update',
  targetUsers: 'all-active'
});
```

#### 5. Data Import/Export
```javascript
// Import CSV with 100k rows
importQueue.add('import-csv', {
  fileUrl: csvFile.url,
  model: 'products',
  userId: req.user.id
}, {
  timeout: 300000 // 5 minute timeout
});
```

---

### Third-Party Task Queue Services

**Most teams use managed services** rather than building their own:

#### Popular Options:

**1. AWS SQS (Simple Queue Service)**
```javascript
// Push to queue
await sqs.sendMessage({
  QueueUrl: QUEUE_URL,
  MessageBody: JSON.stringify({
    type: 'send-email',
    data: { to, subject, body }
  })
});

// Worker polls queue
const messages = await sqs.receiveMessage({
  QueueUrl: QUEUE_URL,
  MaxNumberOfMessages: 10
});
```

**2. Redis + Bull (Node.js)**
```javascript
const Queue = require('bull');
const emailQueue = new Queue('emails', REDIS_URL);

// Add job
await emailQueue.add({ to, subject, body });

// Process jobs
emailQueue.process(async (job) => {
  await sendEmail(job.data);
});
```

**3. RabbitMQ**
- Full-featured message broker
- Supports multiple messaging patterns
- Enterprise-grade reliability

**4. Google Cloud Tasks**
- Managed task queue service
- HTTP-based task execution
- Automatic retry and rate limiting

**5. Celery (Python)**
```python
# Define task
@app.task
def send_email(to, subject, body):
    email_service.send(to, subject, body)

# Queue task
send_email.delay('user@example.com', 'Hello', 'World')
```

---

### Task Queue Features

#### 1. Retry Logic
```javascript
emailQueue.add('send-email', data, {
  attempts: 3,           // Retry up to 3 times
  backoff: {
    type: 'exponential', // Wait longer between retries
    delay: 5000          // Start with 5 second delay
  }
});
```

#### 2. Priority Queues
```javascript
// High priority (process first)
queue.add('urgent-task', data, { priority: 1 });

// Normal priority
queue.add('normal-task', data, { priority: 5 });

// Low priority (process last)
queue.add('bulk-task', data, { priority: 10 });
```

#### 3. Scheduled Tasks
```javascript
// Run task in 1 hour
queue.add('reminder', data, {
  delay: 3600000 // 1 hour in milliseconds
});

// Recurring tasks (cron-like)
queue.add('daily-cleanup', data, {
  repeat: {
    cron: '0 2 * * *' // Every day at 2 AM
  }
});
```

#### 4. Rate Limiting
```javascript
queue.add('api-call', data, {
  rateLimit: {
    max: 100,      // Maximum 100 jobs
    duration: 60000 // Per minute
  }
});
```

#### 5. Job Status Tracking
```javascript
// Add job and get ID
const job = await queue.add('process-order', data);

// Check status later
const jobStatus = await queue.getJob(job.id);
console.log(jobStatus.progress); // 0-100
console.log(jobStatus.returnvalue); // Result after completion
```

---

### Best Practices for Background Tasks

**1. Make Tasks Idempotent**

Tasks might run multiple times due to retries:
```javascript
// BAD - increments every time
async function processPayment(orderId) {
  await db.query('UPDATE orders SET attempts = attempts + 1 WHERE id = ?', [orderId]);
}

// GOOD - check if already processed
async function processPayment(orderId) {
  const order = await db.query('SELECT * FROM orders WHERE id = ?', [orderId]);
  
  if (order.paymentStatus === 'completed') {
    return; // Already processed
  }
  
  // Process payment...
}
```

**2. Keep Tasks Small and Focused**

```javascript
// BAD - one big task
queue.add('process-order', { orderId });

// GOOD - break into smaller tasks
queue.add('charge-payment', { orderId });
queue.add('send-confirmation', { orderId });
queue.add('update-inventory', { orderId });
queue.add('notify-warehouse', { orderId });
```

**3. Set Appropriate Timeouts**

```javascript
queue.add('quick-task', data, { timeout: 30000 }); // 30 seconds
queue.add('long-task', data, { timeout: 600000 }); // 10 minutes
```

**4. Monitor Queue Health**

```javascript
// Check queue metrics
const waiting = await queue.getWaitingCount();
const active = await queue.getActiveCount();
const failed = await queue.getFailedCount();

if (waiting > 10000) {
  alert('Queue backlog is too high!');
}
```

**5. Handle Failures Gracefully**

```javascript
queue.on('failed', (job, error) => {
  logger.error({
    jobId: job.id,
    jobType: job.name,
    error: error.message,
    data: job.data
  });
  
  // Notify admin for critical tasks
  if (job.name === 'process-payment') {
    alertAdmin(`Payment processing failed: ${job.id}`);
  }
});
```

---

## Elasticsearch - Deep Dive

### What is Elasticsearch?

Elasticsearch is a distributed, RESTful search and analytics engine built on **Apache Lucene**.

**Core Purpose**: Fast, scalable full-text search

**Key Features**:
- Full-text search with relevance scoring
- Real-time indexing and search
- Distributed architecture (horizontal scaling)
- RESTful API
- Schema-free (flexible JSON documents)

---

### How Elasticsearch Works

#### Inverted Index

**Traditional Database**:
```
Document 1: "The quick brown fox"
Document 2: "The lazy dog"
Document 3: "Quick brown dogs"
```

Search for "quick" → Scan every document ❌ Slow

**Inverted Index** (Elasticsearch's approach):
```
Term        | Document IDs
------------|-------------
the         | [1, 2]
quick       | [1, 3]
brown       | [1, 3]
fox         | [1]
lazy        | [2]
dog         | [2]
dogs        | [3]
```

Search for "quick" → Immediate lookup: [1, 3] ✅ Fast

**How It Works**:
1. Documents are analyzed and broken into terms (words)
2. Each term maps to document IDs where it appears
3. Searches look up terms in the index
4. Results are ranked by relevance

---

### Apache Lucene

**What is Apache Lucene?**

- Core library that powers Elasticsearch
- Written in Java
- Provides indexing and search capabilities
- Elasticsearch is essentially a distributed wrapper around Lucene

**Elasticsearch's Addition**:
- Distributed architecture
- RESTful API
- JSON-based
- Cluster management
- Replication and sharding

---

### Relevance Scoring

Elasticsearch ranks search results by relevance using several factors:

#### 1. Term Frequency (TF)

How many times does the search term appear in a document?

```
Document 1: "elasticsearch elasticsearch elasticsearch"
Document 2: "elasticsearch is powerful"

Search for "elasticsearch"
→ Document 1 scores higher (appears 3 times vs 1 time)
```

#### 2. Inverse Document Frequency (IDF)

How rare is the term across all documents?

```
Common term: "the" (appears in 90% of documents) → Lower score
Rare term: "elasticsearch" (appears in 5% of documents) → Higher score
```

**Why?** Rare terms are more meaningful for distinguishing relevant documents.

#### 3. Document Length Normalization

Shorter documents that contain the term rank higher than longer documents:

```
Document 1 (10 words): "elasticsearch tutorial"
Document 2 (1000 words): "...elasticsearch..." (buried in text)

Search for "elasticsearch"
→ Document 1 scores higher (more focused on the topic)
```

#### 4. Field Boosting

Give more weight to certain fields:

```javascript
{
  "query": {
    "multi_match": {
      "query": "elasticsearch",
      "fields": [
        "title^3",        // 3x boost
        "description^2",  // 2x boost
        "content"         // 1x (default)
      ]
    }
  }
}
```

Match in title = 3x more important than match in content

---

### Elasticsearch in the ELK Stack

**ELK = Elasticsearch + Logstash + Kibana**

#### Complete Logging Solution:

**1. Logstash** (Data Collection & Processing):
```
Application Logs → Logstash → Parse, Transform, Enrich → Elasticsearch
```

**Example Configuration**:
```
input {
  file {
    path => "/var/log/application.log"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "application-logs-%{+YYYY.MM.dd}"
  }
}
```

**2. Elasticsearch** (Storage & Search):
- Stores logs
- Provides search capabilities
- Aggregates data

**3. Kibana** (Visualization):
- Web interface for Elasticsearch
- Create dashboards
- Visualize metrics
- Search and analyze logs

**Complete Flow**:
```
Application → Logs → Logstash → Elasticsearch → Kibana Dashboard
```

---

### Elasticsearch Use Cases

#### 1. Application Search
```javascript
// Index products
await elasticsearch.index({
  index: 'products',
  body: {
    name: 'iPhone 14 Pro',
    description: 'Latest iPhone with A16 chip',
    category: 'smartphones',
    price: 999
  }
});

// Fuzzy search (handles typos)
const results = await elasticsearch.search({
  index: 'products',
  body: {
    query: {
      fuzzy: {
        name: {
          value: 'iPhon',  // Typo
          fuzziness: 'AUTO'
        }
      }
    }
  }
});
// Returns: iPhone 14 Pro
```

#### 2. Log Analysis
```javascript
// Search error logs from last hour
const errors = await elasticsearch.search({
  index: 'application-logs-*',
  body: {
    query: {
      bool: {
        must: [
          { match: { level: 'ERROR' } },
          { range: { timestamp: { gte: 'now-1h' } } }
        ]
      }
    }
  }
});
```

#### 3. Real-time Analytics
```javascript
// Aggregate sales by region
const analytics = await elasticsearch.search({
  index: 'sales',
  body: {
    size: 0,
    aggs: {
      sales_by_region: {
        terms: { field: 'region' },
        aggs: {
          total_revenue: { sum: { field: 'amount' } }
        }
      }
    }
  }
});
```

---

### Elasticsearch Best Practices

**1. Not a Primary Database**

```javascript
// WRONG - Elasticsearch as primary database
await elasticsearch.index({
  index: 'users',
  id: userId,
  body: userData
});

// CORRECT - Database first, then index
await db.createUser(userData);
await elasticsearch.index({
  index: 'users',
  id: userId,
  body: userData
});
```

**2. Use Appropriate Analyzers**

```javascript
// Configure text analysis
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "snowball"]
        }
      }
    }
  }
}
```

**3. Implement Proper Index Management**

```javascript
// Use index aliases
PUT /products-v1
POST /_aliases
{
  "actions": [
    { "add": { "index": "products-v1", "alias": "products" } }
  ]
}

// Zero-downtime reindexing
POST /_reindex
{
  "source": { "index": "products-v1" },
  "dest": { "index": "products-v2" }
}
```

---

## Error Handling - A Fault-Tolerant Mindset

### Why Fault Tolerance Matters

**Goal**: Provide a seamless experience to users even when things go wrong.

**Reality**: Systems fail. The question is how gracefully you handle failures.

**Mindset Shift**:
- Assume everything will fail eventually
- Design for failure, not just success
- Graceful degradation over complete failure

---

### Types of Errors in Backend Systems

#### 1. Logic Errors

**Characteristic**: Code runs without crashing, but produces unexpected results.

**Examples**:
```javascript
// Wrong calculation
function calculateDiscount(price, discountPercent) {
  return price * discountPercent; // Should be price * (discountPercent / 100)
}

// Off-by-one error
for (let i = 0; i <= array.length; i++) { // Should be i < array.length
  console.log(array[i]); // Last iteration: undefined
}

// Wrong condition
if (user.age > 18) { // Should be >=
  allowAccess();
}
```

**Prevention**:
- Comprehensive testing
- Code reviews
- Input validation
- Edge case testing

---

#### 2. Database Errors

**Common Database Error Types**:

**Connection Errors**:
```javascript
// Connection timeout
Error: connect ETIMEDOUT
Error: Too many connections
Error: Connection lost

// Handling
const pool = mysql.createPool({
  connectionLimit: 10,
  connectTimeout: 10000,
  acquireTimeout: 10000
});

pool.on('error', (err) => {
  if (err.code === 'PROTOCOL_CONNECTION_LOST') {
    // Reconnect
  }
});
```

**Constraint Violations**:
```javascript
// Unique constraint
Error: duplicate key value violates unique constraint "users_email_key"

// Foreign key constraint
Error: insert or update on table violates foreign key constraint

// Not null constraint
Error: null value in column "username" violates not-null constraint

// Handling
try {
  await db.createUser({ email: 'user@example.com' });
} catch (error) {
  if (error.code === '23505') { // PostgreSQL unique violation
    return res.status(409).json({ 
      error: 'Email already exists' 
    });
  }
  throw error;
}
```

**Query Errors**:
```javascript
// Syntax error
Error: syntax error at or near "SELCT"

// Invalid column
Error: column "naem" does not exist

// Handling
async function safeQuery(query, params) {
  try {
    return await db.query(query, params);
  } catch (error) {
    logger.error({
      query,
      params,
      error: error.message
    });
    
    throw new DatabaseError('Query failed', { 
      cause: error 
    });
  }
}
```

---

#### 3. Network/Connection Errors

**Types**:
- Timeout errors
- Connection refused
- DNS resolution failures
- Network unreachable

**Exponential Backoff Strategy**:

```javascript
async function retryWithBackoff(fn, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Calculate exponential backoff: 1s, 2s, 4s, 8s, 16s
      const delay = Math.pow(2, i) * 1000;
      
      // Add jitter to prevent thundering herd
      const jitter = Math.random() * 1000;
      
      await sleep(delay + jitter);
      
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
    }
  }
}

// Usage
const data = await retryWithBackoff(async () => {
  return await externalApi.fetchData();
});
```

**Why Exponential Backoff?**
- Gives failing service time to recover
- Prevents overwhelming the service with retries
- Jitter prevents synchronized retry storms

---

#### 4. Service Outage / Third-Party API Failures

**Scenario**: External service you depend on goes down.

**Circuit Breaker Pattern**:

```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log('Circuit breaker opened');
    }
  }
}

// Usage
const paymentCircuit = new CircuitBreaker();

app.post('/api/payment', async (req, res) => {
  try {
    const result = await paymentCircuit.execute(async () => {
      return await paymentGateway.charge(req.body);
    });
    res.json(result);
  } catch (error) {
    // Fallback: queue for later processing
    await paymentQueue.add('retry-payment', req.body);
    res.status(503).json({ 
      error: 'Payment service temporarily unavailable',
      message: 'Your payment will be processed shortly'
    });
  }
});
```

**States**:
- **CLOSED**: Normal operation, requests go through
- **OPEN**: Service is down, requests fail immediately
- **HALF_OPEN**: Testing if service recovered

---

#### 5. Input Validation Errors

**Never trust user input**:

```javascript
// Comprehensive validation
const { body, validationResult } = require('express-validator');

app.post('/api/users',
  // Validation middleware
  body('email').isEmail().normalizeEmail(),
  body('age').isInt({ min: 0, max: 150 }),
  body('username').isLength({ min: 3, max: 30 }).matches(/^[a-zA-Z0-9_]+$/),
  
  async (req, res) => {
    // Check validation results
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ 
        errors: errors.array() 
      });
    }
    
    // Process valid data
    const user = await createUser(req.body);
    res.status(201).json(user);
  }
);
```

**Validation Checklist**:
- ✅ Type validation
- ✅ Range validation
- ✅ Format validation (email, phone, etc.)
- ✅ Length validation
- ✅ Sanitization (remove dangerous characters)
- ✅ Business rule validation

---

#### 6. Configuration Errors

**Common Issues**:
- Missing environment variables
- Invalid configuration values
- Type mismatches

**Solution - Validate on Startup**:

```javascript
const config = {
  port: parseInt(process.env.PORT),
  database: {
    host: process.env.DB_HOST,
    port: parseInt(process.env.DB_PORT),
    name: process.env.DB_NAME
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1h'
  }
};

// Validate configuration
function validateConfig() {
  const required = [
    'PORT',
    'DB_HOST',
    'DB_PORT',
    'DB_NAME',
    'JWT_SECRET'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    console.error(`Missing required environment variables: ${missing.join(', ')}`);
    process.exit(1);
  }

  if (isNaN(config.port) || config.port < 1 || config.port > 65535) {
    console.error('Invalid PORT configuration');
    process.exit(1);
  }

  console.log('✅ Configuration validated successfully');
}

// Run validation before starting server
validateConfig();
app.listen(config.port);
```

---

### Error Handling Strategy

**Layered Error Handling**:

```javascript
// Custom error classes
class ApplicationError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
  }
}

class ValidationError extends ApplicationError {
  constructor(message, errors = []) {
    super(message, 400);
    this.errors = errors;
  }
}

class DatabaseError extends ApplicationError {
  constructor(message) {
    super(message, 500);
  }
}

class NotFoundError extends ApplicationError {
  constructor(resource) {
    super(`${resource} not found`, 404);
  }
}

// Global error handler
app.use((error, req, res, next) => {
  // Log error
  logger.error({
    error: error.message,
    stack: error.stack,
    url: req.url,
    method: req.method,
    userId: req.user?.id
  });

  // Operational vs Programming errors
  if (!error.isOperational) {
    // Programming error - something is seriously wrong
    logger.fatal('Unhandled programming error:', error);
    
    // In production, might want to restart the process
    if (process.env.NODE_ENV === 'production') {
      process.exit(1);
    }
  }

  // Send response
  res.status(error.statusCode || 500).json({
    error: {
      message: error.message,
      ...(error.errors && { errors: error.errors }),
      ...(process.env.NODE_ENV === 'development' && { 
        stack: error.stack 
      })
    }
  });
});
```

---

## Logging vs Monitoring vs Observability

These three concepts are often confused but serve different purposes:

---

### Logging

**Definition**: A **record of discrete events** that occurred in your system.

**Purpose**: Understand **what happened** in the past.

**What to Log**:
- Application events (user login, order placed)
- Errors and exceptions
- Request/response details
- Business-critical operations
- Security events

**Log Levels**:
```
ERROR   - Something failed
WARN    - Potential issue
INFO    - Important events
DEBUG   - Detailed information (development only)
TRACE   - Very detailed (rarely used)
```

**Structured Logging Example**:
```javascript
// BAD - Unstructured logging
console.log('User logged in');

// GOOD - Structured logging
logger.info({
  event: 'user_login',
  userId: user.id,
  email: user.email,
  ip: req.ip,
  userAgent: req.headers['user-agent'],
  timestamp: new Date().toISOString(),
  duration: loginDuration
});
```

**Benefits of Structured Logs**:
- Easy to search and filter
- Can be analyzed programmatically
- Consistent format
- Better for log aggregation tools

**Metadata to Include**:
- Timestamp
- Log level
- Event type
- User ID (if applicable)
- Request ID (for tracing)
- IP address
- Error details (message, stack trace)

**Example with Winston (Node.js)**:
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Usage
logger.info({
  event: 'order_created',
  orderId: order.id,
  userId: user.id,
  amount: order.total,
  items: order.items.length
});

logger.error({
  event: 'payment_failed',
  orderId: order.id,
  error: error.message,
  stack: error.stack
});
```

---

### Monitoring

**Definition**: **Real-time or near-real-time tracking** of system health and performance.

**Purpose**: Understand the **current state** of your system.

**What to Monitor**:

**System Metrics**:
- CPU usage
- Memory usage
- Disk I/O
- Network traffic

**Application Metrics**:
- Request rate (requests per second)
- Response time (latency)
- Error rate (percentage of failed requests)
- Throughput (data processed)

**Business Metrics**:
- Active users
- Orders per minute
- Revenue
- Conversion rate

**Example Metrics Collection**:
```javascript
const promClient = require('prom-client');

// Register default metrics
promClient.collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

const activeUsers = new promClient.Gauge({
  name: 'active_users',
  help: 'Number of currently active users'
});

// Middleware to track request duration
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || 'unknown', res.statusCode)
      .observe(duration);
  });
  
  next();
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

**Alerting**:
```javascript
// Alert rules (typically in Prometheus)
groups:
  - name: api_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        annotations:
          summary: "High error rate detected"
          
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 2
        for: 10m
        annotations:
          summary: "95th percentile response time above 2 seconds"
```

**Key Monitoring Tools**:
- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **Datadog**: All-in-one monitoring platform
- **New Relic**: Application performance monitoring

---

### Observability

**Definition**: The ability to understand the **internal state** of your system by examining its **outputs** (logs, metrics, traces).

**Purpose**: Answer **why** something happened and **how** it happened.

**Three Pillars of Observability**:

#### 1. Logs
What events occurred?

#### 2. Metrics
How much/how many? (quantitative data)

#### 3. Traces
What was the path of a request through the system?

---

### Distributed Tracing

**The Problem**:

In microservices, a single user request might touch 5-10 different services:
```
User Request → API Gateway → Auth Service → User Service → Order Service → Payment Service → Email Service
```

When something goes slow or fails, **where** is the bottleneck?

**The Solution: Distributed Tracing**

Track the entire journey of a request across all services.

**How It Works**:

1. Generate a unique **Trace ID** for each request
2. Each service creates a **Span** (segment of work)
3. Spans are linked by the Trace ID
4. Send spans to a tracing backend

**Example with OpenTelemetry**:
```javascript
const { trace } = require('@opentelemetry/api');
const tracer = trace.getTracer('my-service');

app.get('/api/orders/:id', async (req, res) => {
  // Create span for this operation
  const span = tracer.startSpan('get_order');
  
  try {
    span.setAttribute('order.id', req.params.id);
    span.setAttribute('user.id', req.user.id);
    
    // Database query (creates child span)
    const order = await getOrder(req.params.id);
    
    // External API call (creates child span)
    const userDetails = await userService.getUser(order.userId);
    
    span.setStatus({ code: SpanStatusCode.OK });
    res.json({ order, user: userDetails });
    
  } catch (error) {
    span.recordException(error);
    span.setStatus({ code: SpanStatusCode.ERROR });
    throw error;
  } finally {
    span.end();
  }
});
```

**Trace Visualization**:
```
Trace ID: abc123
Total Duration: 245ms

|-- get_order (245ms)
    |-- db_query (50ms)
    |-- get_user (180ms)
        |-- http_request (170ms)
        |-- parse_response (10ms)
    |-- serialize_response (15ms)
```

This shows:
- Total request took 245ms
- The `get_user` call took 180ms (bottleneck!)
- Database query was fast (50ms)

**Traces Help Answer**:
- Which service is slow?
- Where are requests failing?
- What's the request flow?
- How do services depend on each other?

---

### Observability Best Practices

**1. Correlation IDs**

Generate a unique ID for each request and include it in all logs:
```javascript
app.use((req, res, next) => {
  req.correlationId = uuid();
  res.setHeader('X-Correlation-ID', req.correlationId);
  next();
});

logger.info({
  correlationId: req.correlationId,
  event: 'request_received',
  method: req.method,
  url: req.url
});
```

**2. Context Propagation**

Pass context (trace ID, user ID) through the entire request chain:
```javascript
// Service A
const response = await axios.get('http://service-b/api/data', {
  headers: {
    'X-Trace-ID': req.traceId,
    'X-User-ID': req.user.id
  }
});

// Service B
app.use((req, res, next) => {
  req.traceId = req.headers['x-trace-id'];
  req.userId = req.headers['x-user-id'];
  next();
});
```

**3. Standard Log Format**

Use consistent format across all services:
```javascript
{
  "timestamp": "2024-02-15T10:30:00Z",
  "level": "INFO",
  "service": "order-service",
  "traceId": "abc123",
  "correlationId": "xyz789",
  "userId": "user-456",
  "event": "order_created",
  "orderId": "order-789",
  "duration": 150,
  "metadata": {
    "items": 3,
    "total": 99.99
  }
}
```

**4. Sampling for High-Traffic Systems**

Don't trace every request in production (too expensive):
```javascript
// Sample 10% of requests
const shouldTrace = Math.random() < 0.1;

if (shouldTrace) {
  const span = tracer.startSpan('operation');
  // ... trace this request
}
```

---

### Observability Tools

#### Tracing
- **Jaeger**: Open-source distributed tracing
- **Zipkin**: Alternative open-source tracer
- **OpenTelemetry**: Vendor-neutral standard

#### Complete Observability Platforms
- **New Relic**: APM with logs, metrics, traces
- **Datadog**: Full-stack observability
- **Grafana Stack**: Grafana + Loki (logs) + Tempo (traces) + Prometheus (metrics)

#### Log Aggregation
- **ELK Stack**: Elasticsearch + Logstash + Kibana
- **Splunk**: Enterprise log management
- **Loki**: Lightweight log aggregation (by Grafana)

---

### Putting It All Together

**Example: Debugging a Slow API Endpoint**

**Step 1: Check Metrics** (Monitoring)
```
Dashboard shows:
- 95th percentile response time: 3.5 seconds (usually 200ms)
- Error rate: 0.5% (usually 0.1%)
```

**Step 2: Check Logs** (Logging)
```
Search for errors in last hour:
level:ERROR AND timestamp:[now-1h TO now]

Found: Multiple "Database timeout" errors
```

**Step 3: Check Traces** (Observability)
```
Find slow requests in tracing UI
Trace shows:
- API call: 50ms
- Database query: 3000ms ← Bottleneck!
- Response formatting: 10ms
```

**Step 4: Dive Deeper**
```
Check specific query in logs:
Query: SELECT * FROM orders WHERE user_id = 123

Check database metrics:
- Query count spiked 10x
- Missing index on user_id column!
```

**Solution**: Add database index, response time drops to 200ms.

---

## Conclusion

Backend engineering is about understanding core principles and applying them to build robust, scalable, and maintainable systems. While specific technologies and frameworks change, these fundamental concepts remain constant.

**Key Takeaway**: "Not limiting myself to a tech stack will be the result of understanding basic principles of backend engineering."

Master these concepts, and you'll be able to adapt to any backend technology or framework.

---

## Additional Resources

- **HTTP**: [MDN HTTP Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- **REST**: [Roy Fielding's Dissertation](https://roy.gbiv.com/pubs/dissertation/fielding_dissertation.pdf)
- **PostgreSQL**: [Official Documentation](https://www.postgresql.org/docs/)
- **Redis**: [Redis Documentation](https://redis.io/documentation)
- **12-Factor App**: [12factor.net](https://12factor.net/)
- **OpenAPI**: [OpenAPI Specification](https://swagger.io/specification/)
