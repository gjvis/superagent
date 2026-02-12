# Promises and Async/Await

Use promises or async/await instead of callbacks for cleaner asynchronous code.

## Capabilities

### Promise Interface

SuperAgent requests implement the Promise interface for modern async patterns.

```javascript { .api }
interface Request {
  /**
   * Execute request and handle success
   * @param onFulfilled - Success handler
   * @param onRejected - Error handler
   * @returns Promise resolving to response
   */
  then<T>(
    onFulfilled: (res: Response) => T,
    onRejected?: (err: Error) => any
  ): Promise<T>;

  /**
   * Handle request errors
   * @param onRejected - Error handler
   * @returns Promise
   */
  catch(onRejected: (err: Error) => any): Promise<any>;

  /**
   * Send request with callback (traditional)
   * @param callback - Callback function(err, res)
   */
  end(callback?: (err: Error | null, res: Response) => void): void;
}
```

### Using Promises

Handle requests with promise chains instead of callbacks.

**Usage Examples:**

```javascript
const request = require('superagent');

// Basic promise usage
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
    return res.body;
  })
  .catch(err => {
    console.error('Error:', err.message);
  });

// Promise chaining
request
  .get('/api/users/123')
  .then(res => {
    console.log('User:', res.body);
    // Make another request based on first response
    return request.post('/api/activity').send({
      userId: res.body.id,
      action: 'viewed_profile'
    });
  })
  .then(res => {
    console.log('Activity logged:', res.body);
  })
  .catch(err => {
    console.error('Error in chain:', err.message);
  });

// Handle both success and error in .then()
request
  .get('/api/data')
  .then(
    res => {
      console.log('Success:', res.body);
    },
    err => {
      console.error('Error:', err.message);
    }
  );

// Transform response data
request
  .get('/api/users')
  .then(res => res.body)
  .then(users => users.filter(u => u.active))
  .then(activeUsers => {
    console.log('Active users:', activeUsers);
    return activeUsers.length;
  })
  .then(count => {
    console.log('Count:', count);
  })
  .catch(err => {
    console.error('Error:', err);
  });
```

### Using Async/Await

Use modern async/await syntax for even cleaner code.

**Usage Examples:**

```javascript
// Basic async/await
async function getUsers() {
  try {
    const res = await request.get('/api/users');
    console.log('Users:', res.body);
    return res.body;
  } catch (err) {
    console.error('Error:', err.message);
    throw err;
  }
}

// Multiple sequential requests
async function getUserProfile(userId) {
  try {
    // Get user
    const userRes = await request.get(`/api/users/${userId}`);
    const user = userRes.body;

    // Get user's posts
    const postsRes = await request.get(`/api/users/${userId}/posts`);
    const posts = postsRes.body;

    // Get user's comments
    const commentsRes = await request.get(`/api/users/${userId}/comments`);
    const comments = commentsRes.body;

    return { user, posts, comments };
  } catch (err) {
    console.error('Error fetching profile:', err.message);
    throw err;
  }
}

// Parallel requests with Promise.all
async function fetchDashboard() {
  try {
    const [usersRes, postsRes, commentsRes] = await Promise.all([
      request.get('/api/users'),
      request.get('/api/posts'),
      request.get('/api/comments')
    ]);

    return {
      users: usersRes.body,
      posts: postsRes.body,
      comments: commentsRes.body
    };
  } catch (err) {
    console.error('Error fetching dashboard:', err.message);
    throw err;
  }
}

// POST with async/await
async function createUser(userData) {
  try {
    const res = await request
      .post('/api/users')
      .send(userData)
      .set('Content-Type', 'application/json');

    console.log('User created:', res.body);
    return res.body;
  } catch (err) {
    if (err.status === 400) {
      console.error('Validation error:', err.response.body);
    } else {
      console.error('Error creating user:', err.message);
    }
    throw err;
  }
}

// Update with async/await
async function updateUser(userId, updates) {
  const res = await request
    .patch(`/api/users/${userId}`)
    .send(updates);

  return res.body;
}

// Delete with async/await
async function deleteUser(userId) {
  const res = await request.delete(`/api/users/${userId}`);
  return res.status === 204; // No Content
}
```

### Error Handling with Promises

Handle different types of errors in promise chains.

**Usage Examples:**

```javascript
// Basic error handling
request
  .get('/api/users')
  .catch(err => {
    if (err.response) {
      // Server responded with error status
      console.error('Status:', err.status);
      console.error('Body:', err.response.body);
    } else if (err.code === 'ECONNABORTED') {
      // Request timeout
      console.error('Request timed out');
    } else {
      // Network error or other issue
      console.error('Network error:', err.message);
    }
  });

// Handle specific status codes
request
  .get('/api/users/123')
  .catch(err => {
    if (err.status === 404) {
      console.log('User not found');
      return null; // Return default value
    } else if (err.status === 401) {
      console.log('Not authorized');
      // Redirect to login
      window.location = '/login';
    } else {
      throw err; // Re-throw other errors
    }
  })
  .then(res => {
    if (res) {
      console.log('User:', res.body);
    }
  });

// Try/catch with async/await
async function fetchData() {
  try {
    const res = await request.get('/api/data');
    return res.body;
  } catch (err) {
    if (err.status === 404) {
      return null; // Return default for not found
    }
    if (err.status >= 500) {
      console.error('Server error');
      return { error: 'Server unavailable' };
    }
    throw err; // Re-throw unexpected errors
  }
}

// Multiple catch blocks
async function complexOperation() {
  let user;

  try {
    const res = await request.get('/api/user');
    user = res.body;
  } catch (err) {
    console.error('Failed to fetch user:', err.message);
    user = null;
  }

  if (user) {
    try {
      await request.post('/api/activity').send({
        userId: user.id,
        action: 'login'
      });
    } catch (err) {
      // Log error but don't fail the whole operation
      console.error('Failed to log activity:', err.message);
    }
  }

  return user;
}
```

### Callback vs Promise vs Async/Await

Comparison of the three approaches.

**Callback style:**

```javascript
request
  .get('/api/users')
  .end((err, res) => {
    if (err) {
      console.error(err);
      return;
    }
    console.log(res.body);
  });
```

**Promise style:**

```javascript
request
  .get('/api/users')
  .then(res => {
    console.log(res.body);
  })
  .catch(err => {
    console.error(err);
  });
```

**Async/await style:**

```javascript
async function getUsers() {
  try {
    const res = await request.get('/api/users');
    console.log(res.body);
  } catch (err) {
    console.error(err);
  }
}
```

### Advanced Patterns

Complex async patterns with SuperAgent.

**Retry with exponential backoff:**

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const res = await request.get(url);
      return res.body;
    } catch (err) {
      if (i === maxRetries - 1) {
        throw err; // Last attempt failed
      }

      const delay = Math.pow(2, i) * 1000; // exponential backoff
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

**Parallel requests with error handling:**

```javascript
async function fetchMultiple(urls) {
  const results = await Promise.allSettled(
    urls.map(url => request.get(url))
  );

  return results.map((result, index) => {
    if (result.status === 'fulfilled') {
      return result.value.body;
    } else {
      console.error(`Failed to fetch ${urls[index]}:`, result.reason);
      return null;
    }
  });
}
```

**Race condition (use first response):**

```javascript
async function fetchFromMultipleSources() {
  const sources = [
    request.get('https://api1.example.com/data'),
    request.get('https://api2.example.com/data'),
    request.get('https://api3.example.com/data')
  ];

  try {
    const res = await Promise.race(sources);
    console.log('First response:', res.body);
    return res.body;
  } catch (err) {
    console.error('All sources failed:', err);
    throw err;
  }
}
```

**Sequential processing:**

```javascript
async function processUsersSequentially(userIds) {
  const results = [];

  for (const id of userIds) {
    try {
      const res = await request.get(`/api/users/${id}`);
      results.push(res.body);
    } catch (err) {
      console.error(`Failed to fetch user ${id}:`, err.message);
      results.push(null);
    }
  }

  return results;
}
```

**Batched parallel processing:**

```javascript
async function processBatches(items, batchSize = 5) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);

    const batchResults = await Promise.all(
      batch.map(item =>
        request.post('/api/process').send(item).catch(err => ({
          error: err.message
        }))
      )
    );

    results.push(...batchResults.map(r => r.body || r));
  }

  return results;
}
```
