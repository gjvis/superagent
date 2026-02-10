# Search Products with Filters

Build a function that searches for products using various filter criteria passed as query parameters.

## Requirements

Create a function `searchProducts(filters)` that:

- Accepts an object containing search filters (category, minPrice, maxPrice, sortBy)
- Makes an HTTP GET request to `https://api.example.com/products/search`
- Appends the filters as query string parameters
- Returns a promise that resolves with the array of matching products

## Test Cases

- Calling `searchProducts({category: 'electronics', minPrice: 100, maxPrice: 500})` should append `?category=electronics&minPrice=100&maxPrice=500` to the URL @test
- Calling `searchProducts({sortBy: 'price', category: 'books'})` should properly encode both parameters in the query string @test
- The function should handle empty filter objects gracefully @test

## Dependencies { .dependencies }

### superagent 4.0.0 { .dependency }

A progressive HTTP client library with a fluent API for making HTTP requests in Node.js and browsers.
