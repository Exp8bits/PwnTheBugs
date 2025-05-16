
# GraphQL Mutation

Mutations are used to perform operations that modify, delete, or create data. They are a type of query that allows you to interact with data by adding, modifying, or deleting it on the server.

```graphql
query {
  __schema {
    mutationType {
      name
      fields {
        name
        args {
          name
          defaultValue
          type {
            ...TypeRef
          }
        }
        type {
          ...TypeRef
        }
      }
    }
  }
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Example of a mutation to create a new user account

```graphql
mutation {
  createUser(input: { name: "test test", email: "test@example.com", password: "securePassword" }) {
    user {
      id
      name
      email
    }
  }
}
```

- Here, `createUser` is the mutation used to create a new user.
- The `input` is the data sent to create the new account, such as the name, email, and password.
- The code returns the details of the newly created user, like the ID, name, and email.

