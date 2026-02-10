# API Search Utility

Create a function that searches an API with complex query parameters.

## Requirements

Implement a function `searchProducts(filters)` that:

1. Accepts a filters object with properties like: `category`, `minPrice`, `maxPrice`, `tags` (array), and `inStock` (boolean)
2. Constructs a GET request to `https://api.shop.com/products/search`
3. Adds all filter properties as query parameters
4. Supports chaining multiple query parameter additions
5. Returns the parsed response data

Example call:
```javascript
searchProducts({
  category: 'electronics',
  minPrice: 100,
  maxPrice: 500,
  tags: ['sale', 'featured'],
  inStock: true
})
```

Should generate a request like: `/products/search?category=electronics&minPrice=100&maxPrice=500&tags[]=sale&tags[]=featured&inStock=true`

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

Elegant and feature-rich HTTP client library with a fluent chainable API for making HTTP requests in Node.js and browsers.

## Test Cases

@test Input: {category: 'books', minPrice: 10}
Expected: Request includes query parameters ?category=books&minPrice=10

@test Input: {tags: ['new', 'popular'], inStock: true}
Expected: Request includes array and boolean in query string

@test Input: Multiple query objects added incrementally
Expected: All parameters merged into single query string
