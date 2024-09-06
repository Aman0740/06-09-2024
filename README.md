## Q-1. Token Expire in MongoDB with example ?

In MongoDB, you can set an expiration time for documents by using **TTL (Time-To-Live) indexes**. These indexes automatically delete documents after a certain amount of time, which is useful for managing temporary data like session tokens or logs.

### How it Works:
- MongoDB uses a special field with a date or timestamp to determine when a document should expire.
- You create a TTL index on this field, and MongoDB will automatically remove the document once the specified time has passed.
  
### Example:
Let’s say you are storing tokens in MongoDB that expire after 1 hour.

1. **Create a collection and insert a document with a timestamp field:**
   ```js
   db.tokens.insertOne({
     token: "abc123",
     createdAt: new Date() // The current time
   });
   ```

2. **Create a TTL index on the `createdAt` field:**
   This index will automatically delete documents after 1 hour (3600 seconds).
   ```js
   db.tokens.createIndex(
     { "createdAt": 1 },
     { expireAfterSeconds: 3600 }
   );
   ```

### Behavior:
- When the document is inserted, MongoDB keeps it for 1 hour.
- After 1 hour, the TTL index automatically deletes the document.

### Key Points:
- The TTL index is non-blocking, meaning it doesn't immediately remove expired documents. Instead, MongoDB periodically checks and deletes them.
- The expiration is based on the system time of the MongoDB server.

This is useful for expiring tokens, session data, or any time-sensitive records.


## Q-2. mongoose req.headers.authorization .

In a Node.js app using **Mongoose** and **Express**, `req.headers.authorization` typically refers to the **Authorization** header in an HTTP request. This header is commonly used to send tokens (like JWTs) for authenticating API requests.

Here's a brief overview of how to access and use the `req.headers.authorization` header in your app:

### Example: Authenticating Requests Using JWT

1. **Client-Side**: When the client makes a request to a protected route, it sends a token in the `Authorization` header:
   ```http
   GET /protected-route HTTP/1.1
   Host: example.com
   Authorization: Bearer <your_token_here>
   ```

2. **Server-Side (Node.js + Express)**:
   - You can access the `Authorization` header using `req.headers.authorization` in your route handler or middleware.

3. **Extracting and Verifying the Token**:
   - First, you typically split the `Authorization` header to extract the token.
   - Then, you use a package like `jsonwebtoken` to verify the token.

### Code Example:
```js
const jwt = require('jsonwebtoken');

// Middleware to verify the token
const authenticateToken = (req, res, next) => {
  // Extract token from 'Authorization' header
  const authHeader = req.headers.authorization;
  const token = authHeader && authHeader.split(' ')[1]; // Split 'Bearer <token>'

  if (!token) {
    return res.status(401).json({ message: "Unauthorized: No token provided" });
  }

  // Verify the token
  jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ message: "Forbidden: Invalid token" });
    }
    req.user = user; // Store the user data in the request object
    next(); // Proceed to the next middleware or route handler
  });
};

// Protecting a route
app.get('/protected-route', authenticateToken, (req, res) => {
  res.send(`Hello, ${req.user.name}! You are authenticated.`);
});
```

### Breakdown:
1. **Access the token**: `req.headers.authorization` contains the `Authorization` header. You split it to get the token part (`Bearer <token>`).
2. **Verify the token**: The `jsonwebtoken` package verifies the token using a secret.
3. **Proceed with the request**: If the token is valid, the request continues; otherwise, an error is returned.

This is commonly used to protect routes in APIs that require user authentication.
