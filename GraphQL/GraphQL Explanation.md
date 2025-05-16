
# What Is GraphQL?

GraphQL is a query language for APIs and a runtime for executing those queries. It's considered a modern alternative to REST APIs.

## Advantages of GraphQL:

1. Flexibility: You request exactly the data you need.  
2. Better performance: Less over-fetching and fewer requests.  
3. Combine data from multiple sources in a single request.  
4. Self-documenting: The schema defines and explains what's available.

---

## GraphQL vs. REST

### REST:
Works with multiple endpoints, like:  
GET /users | GET /users/1/posts

**Simple Example:**  
Request:
```
GET /user/1
```
Response:
```json
{
  "id": 1,
  "name": "Ali",
  "email": "ali@example.com",
  "phone": "123456789"
}
```

- Even if you only need the name, you get all the data.

### GraphQL:
Works through a single endpoint (usually `/graphql` or `/api/graphql`).  
You define exactly what data you want in your query.

**Simple Example:**  
Request:
```graphql
{
  user(id: 1) {
    name
    email
  }
}
```
Response:
```json
{
  "data": {
    "user": {
      "name": "Ali",
      "email": "ali@example.com"
    }
  }
}
```

- See the difference? You control the structure of the data you receive.
- REST can use GET requests, while GraphQL typically uses POST requests — especially when sending queries with a body.

---

## Comparison Table

| Feature                     | REST API                                | GraphQL                                 |
|----------------------------|------------------------------------------|------------------------------------------|
| Number of endpoints         | Many endpoints (e.g. /users, /posts)     | One endpoint (usually /graphql)          |
| Request method              | Uses GET, POST, PUT, DELETE              | Mostly uses POST                         |
| Data fetching               | Returns fixed data                       | You choose exactly what you need         |
| Too much / too little data  | Sometimes gives extra or missing data    | Only gives what you ask for              |
| Versioning                 | Uses versions like /v1, /v2              | No need for versioning                   |
| Multiple data in one call   | Needs multiple requests                  | One request can get everything           |
| Documentation               | Not always clear                         | Built-in and easy to explore             |
| Response size               | Might be large                           | Smaller, based on your query             |
| Easy to learn               | Easier for beginners                     | A bit harder at first                    |
| Security                    | Standard security methods                | Needs extra care (like query limits)     |

---

## GraphQL Query Examples

### Get All Users' Info:
```graphql
{
  users {
    id
    name
    email
  }
}
```
- This query asks the server to return a list of all users with just their id, name, and email.

### Get Info About a Specific User:
```graphql
{
  user(username: "admin") {
    email
    password
  }
}
```
- This query asks the server to return the email and password of the admin.
