# Promises

Native Promise support for modern asynchronous JavaScript patterns with SuperAgent.

## Capabilities

### Promise Interface

SuperAgent requests implement the Promise interface.

```javascript { .api }
/**
 * Execute request and return Promise
 * @param {Function} resolve - Success handler (res) => any
 * @param {Function} [reject] - Error handler (err) => any
 * @returns {Promise} Promise that resolves with Response or rejects with Error
 */
Request.prototype.then = function(resolve, reject);

/**
 * Handle request errors
 * @param {Function} reject - Error handler (err) => any
 * @returns {Promise} Promise for error handling
 */
Request.prototype.catch = function(reject);
```

**Usage Examples:**

```javascript
const request = require('superagent');

// Using .then()
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
  });

// Using .then() with error handler
request
  .get('/api/users')
  .then(
    res => {
      console.log('Success:', res.body);
    },
    err => {
      console.error('Error:', err.message);
    }
  );

// Using .catch()
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });

// Chaining promises
request
  .get('/api/users')
  .then(res => {
    console.log('Users:', res.body);
    return request.get(`/api/users/${res.body[0].id}`);
  })
  .then(res => {
    console.log('First user:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```

### Async/Await

Use async/await syntax for cleaner asynchronous code.

**Usage Examples:**

```javascript
// Basic async/await
async function getUsers() {
  const res = await request.get('/api/users');
  return res.body;
}

// With error handling
async function getUsersWithErrorHandling() {
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
async function getUserAndProfile(userId) {
  try {
    const userRes = await request.get(`/api/users/${userId}`);
    const user = userRes.body;

    const profileRes = await request.get(`/api/profiles/${user.profileId}`);
    const profile = profileRes.body;

    return { user, profile };
  } catch (err) {
    console.error('Failed to fetch user data:', err.message);
    throw err;
  }
}

// Parallel requests with Promise.all
async function getAllData() {
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
    console.error('Failed to fetch data:', err.message);
    throw err;
  }
}

// Race between requests
async function getFirstResponse() {
  try {
    const res = await Promise.race([
      request.get('/api/server1/data'),
      request.get('/api/server2/data')
    ]);

    console.log('First response:', res.body);
    return res.body;
  } catch (err) {
    console.error('All requests failed:', err.message);
    throw err;
  }
}
```

### Promise Chaining

Chain multiple requests with proper error handling.

**Usage Examples:**

```javascript
// Sequential operations
request
  .post('/api/users')
  .send({ name: 'John', email: 'john@example.com' })
  .then(res => {
    console.log('User created:', res.body.id);
    return request
      .post('/api/profiles')
      .send({ userId: res.body.id, bio: 'Developer' });
  })
  .then(res => {
    console.log('Profile created:', res.body.id);
    return request.get(`/api/users/${res.body.userId}`);
  })
  .then(res => {
    console.log('Complete user data:', res.body);
  })
  .catch(err => {
    console.error('Operation failed:', err.message);
  });

// Transformation pipeline
request
  .get('/api/users')
  .then(res => res.body)
  .then(users => users.filter(u => u.active))
  .then(activeUsers => activeUsers.map(u => u.id))
  .then(userIds => {
    console.log('Active user IDs:', userIds);
    return userIds;
  })
  .catch(err => {
    console.error('Error:', err.message);
  });

// Conditional chaining
request
  .get('/api/users/123')
  .then(res => {
    if (res.body.needsUpdate) {
      return request
        .put('/api/users/123')
        .send({ lastLogin: Date.now() });
    }
    return res;
  })
  .then(res => {
    console.log('User data:', res.body);
  })
  .catch(err => {
    console.error('Error:', err.message);
  });
```

### Error Handling

Comprehensive error handling with promises.

**Usage Examples:**

```javascript
// Basic error handling
request
  .get('/api/users')
  .then(res => {
    console.log('Success:', res.body);
  })
  .catch(err => {
    console.error('Request failed:', err.message);

    if (err.response) {
      // Server responded with error status
      console.error('Status:', err.response.status);
      console.error('Body:', err.response.body);
    } else {
      // Network error or request setup error
      console.error('Network error');
    }
  });

// Async/await error handling
async function fetchWithErrorHandling() {
  try {
    const res = await request.get('/api/users');
    return res.body;
  } catch (err) {
    if (err.response) {
      // Server responded with error status
      const status = err.response.status;

      if (status === 404) {
        console.error('Not found');
      } else if (status === 401) {
        console.error('Unauthorized');
      } else if (status >= 500) {
        console.error('Server error');
      }
    } else if (err.timeout) {
      console.error('Request timeout');
    } else if (err.code) {
      console.error('Network error:', err.code);
    }

    throw err;
  }
}

// Handle specific errors differently
async function fetchWithRecovery() {
  try {
    const res = await request.get('/api/users');
    return res.body;
  } catch (err) {
    if (err.response && err.response.status === 404) {
      // Return empty array for 404
      return [];
    }

    if (err.timeout) {
      // Retry on timeout
      console.log('Timeout, retrying...');
      const res = await request.get('/api/users').timeout(10000);
      return res.body;
    }

    // Re-throw other errors
    throw err;
  }
}

// Multiple catch handlers
request
  .get('/api/users')
  .then(res => res.body)
  .then(data => processData(data))
  .catch(err => {
    // Handle request errors
    console.error('Request error:', err.message);
    return []; // Return default value
  })
  .then(data => displayData(data))
  .catch(err => {
    // Handle display errors
    console.error('Display error:', err.message);
  });
```

### Promise Utilities

Use Promise utilities with SuperAgent requests.

**Usage Examples:**

```javascript
// Promise.all - Wait for all requests
async function fetchAllData() {
  const [users, posts, comments] = await Promise.all([
    request.get('/api/users').then(r => r.body),
    request.get('/api/posts').then(r => r.body),
    request.get('/api/comments').then(r => r.body)
  ]);

  return { users, posts, comments };
}

// Promise.all with error handling
async function fetchAllWithErrors() {
  try {
    const results = await Promise.all([
      request.get('/api/users'),
      request.get('/api/posts'),
      request.get('/api/comments')
    ]);

    return results.map(r => r.body);
  } catch (err) {
    // If any request fails, all fail
    console.error('At least one request failed:', err.message);
    throw err;
  }
}

// Promise.allSettled - Wait for all, handle errors individually
async function fetchAllSettled() {
  const results = await Promise.allSettled([
    request.get('/api/users'),
    request.get('/api/posts'),
    request.get('/api/comments')
  ]);

  const users = results[0].status === 'fulfilled'
    ? results[0].value.body
    : [];

  const posts = results[1].status === 'fulfilled'
    ? results[1].value.body
    : [];

  const comments = results[2].status === 'fulfilled'
    ? results[2].value.body
    : [];

  return { users, posts, comments };
}

// Promise.race - First to complete
async function fetchFromFastestServer() {
  try {
    const res = await Promise.race([
      request.get('https://server1.example.com/api/data'),
      request.get('https://server2.example.com/api/data'),
      request.get('https://server3.example.com/api/data')
    ]);

    console.log('Fastest server responded:', res.body);
    return res.body;
  } catch (err) {
    console.error('First request failed:', err.message);
    throw err;
  }
}

// Promise.any - First successful response
async function fetchFromAnyServer() {
  try {
    const res = await Promise.any([
      request.get('https://server1.example.com/api/data'),
      request.get('https://server2.example.com/api/data'),
      request.get('https://server3.example.com/api/data')
    ]);

    console.log('First successful response:', res.body);
    return res.body;
  } catch (err) {
    console.error('All requests failed:', err.message);
    throw err;
  }
}
```

### Complex Async Workflows

Real-world examples of complex asynchronous workflows.

**Usage Examples:**

```javascript
// Paginated data fetching
async function fetchAllPages() {
  const allData = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const res = await request
      .get('/api/users')
      .query({ page, limit: 100 });

    allData.push(...res.body.users);

    hasMore = res.body.hasMore;
    page++;
  }

  return allData;
}

// Batch processing with concurrency limit
async function processBatch(items, concurrency = 3) {
  const results = [];

  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);

    const batchResults = await Promise.all(
      batch.map(item =>
        request
          .post('/api/process')
          .send(item)
          .then(r => r.body)
      )
    );

    results.push(...batchResults);
  }

  return results;
}

// Retry with exponential backoff
async function fetchWithRetry(url, maxRetries = 3) {
  let lastError;

  for (let i = 0; i < maxRetries; i++) {
    try {
      const res = await request.get(url);
      return res.body;
    } catch (err) {
      lastError = err;
      const delay = Math.pow(2, i) * 1000;

      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);

      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// Dependent requests with caching
const cache = new Map();

async function fetchUserWithProfile(userId) {
  // Check cache
  const cacheKey = `user:${userId}`;
  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  // Fetch user
  const userRes = await request.get(`/api/users/${userId}`);
  const user = userRes.body;

  // Fetch profile
  const profileRes = await request.get(`/api/profiles/${user.profileId}`);
  const profile = profileRes.body;

  // Combine and cache
  const result = { ...user, profile };
  cache.set(cacheKey, result);

  return result;
}

// Waterfall with error recovery
async function createUserWorkflow(userData) {
  try {
    // Step 1: Create user
    const userRes = await request
      .post('/api/users')
      .send(userData);

    const userId = userRes.body.id;

    try {
      // Step 2: Create profile
      const profileRes = await request
        .post('/api/profiles')
        .send({ userId, bio: userData.bio });

      try {
        // Step 3: Send welcome email
        await request
          .post('/api/emails/welcome')
          .send({ userId });

        return { success: true, userId };

      } catch (emailErr) {
        // Email failed but user/profile created
        console.error('Welcome email failed:', emailErr.message);
        return { success: true, userId, emailFailed: true };
      }

    } catch (profileErr) {
      // Profile creation failed, rollback user
      console.error('Profile creation failed:', profileErr.message);

      await request.delete(`/api/users/${userId}`);

      throw new Error('User creation rolled back');
    }

  } catch (userErr) {
    console.error('User creation failed:', userErr.message);
    throw userErr;
  }
}
```

### Complete Promise Example

Comprehensive example showing promise patterns.

**Usage Examples:**

```javascript
const request = require('superagent');

class UserService {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  // Simple promise
  async getUser(id) {
    const res = await request.get(`${this.baseURL}/users/${id}`);
    return res.body;
  }

  // With error handling
  async createUser(userData) {
    try {
      const res = await request
        .post(`${this.baseURL}/users`)
        .send(userData);

      console.log('User created:', res.body.id);
      return res.body;

    } catch (err) {
      if (err.response && err.response.status === 400) {
        throw new Error('Invalid user data');
      }
      throw err;
    }
  }

  // Sequential operations
  async createUserWithProfile(userData, profileData) {
    // Create user first
    const user = await this.createUser(userData);

    try {
      // Then create profile
      const profileRes = await request
        .post(`${this.baseURL}/profiles`)
        .send({ ...profileData, userId: user.id });

      return {
        user,
        profile: profileRes.body
      };

    } catch (err) {
      // Rollback user creation on profile failure
      await request.delete(`${this.baseURL}/users/${user.id}`);
      throw err;
    }
  }

  // Parallel operations
  async getUserData(userId) {
    const [user, posts, comments] = await Promise.all([
      request.get(`${this.baseURL}/users/${userId}`),
      request.get(`${this.baseURL}/posts?userId=${userId}`),
      request.get(`${this.baseURL}/comments?userId=${userId}`)
    ]);

    return {
      user: user.body,
      posts: posts.body,
      comments: comments.body
    };
  }

  // Promise chaining
  getUserAndUpdate(userId, updates) {
    return request
      .get(`${this.baseURL}/users/${userId}`)
      .then(res => {
        const user = res.body;
        return request
          .put(`${this.baseURL}/users/${userId}`)
          .send({ ...user, ...updates });
      })
      .then(res => res.body)
      .catch(err => {
        console.error('Update failed:', err.message);
        throw err;
      });
  }
}

// Usage
const service = new UserService('https://api.example.com');

// Async/await
async function main() {
  try {
    // Simple request
    const user = await service.getUser(123);
    console.log('User:', user);

    // Create with profile
    const newUser = await service.createUserWithProfile(
      { name: 'John', email: 'john@example.com' },
      { bio: 'Developer', location: 'USA' }
    );
    console.log('Created:', newUser);

    // Parallel requests
    const userData = await service.getUserData(123);
    console.log('User data:', userData);

    // Promise chaining
    const updated = await service.getUserAndUpdate(123, { name: 'Jane' });
    console.log('Updated:', updated);

  } catch (err) {
    console.error('Error:', err.message);
  }
}

main();
```

### Important Notes

- **Do Not Mix**: Do not mix `.then()`/`.catch()` with `.end()` callback. Choose one pattern and stick with it.
- **Error Handling**: Always use `.catch()` or try/catch with async/await to handle errors. Unhandled promise rejections can crash Node.js applications.
- **Response vs Body**: Promises resolve with the full Response object, not just the body. Access data with `res.body`.
- **Chaining**: Promise chains allow for sequential operations. Use `Promise.all()` for parallel operations.
- **Async/Await**: Modern async/await syntax provides cleaner code than promise chains for complex workflows.
- **No Callback**: When using promises, don't call `.end()` - the promise is automatically triggered.
- **Return Values**: Always return promises from `.then()` handlers to maintain the chain.
