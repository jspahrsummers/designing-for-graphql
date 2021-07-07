theme: Fira, 3
autoscale: true
build-lists: true
list: alignment(left)

# Designing
## for
# [fit] **GraphQL**

---

# Why this talk?

- GraphQL invalidates existing assumptions about client–server APIs
- Shift from a _server-driven_ model to a _client-driven_ model
- Powerful and flexible, but introduces new challenges

---

# Why GraphQL?

---

# The Before Times

1. [CORBA](https://chelseatroy.com/2018/08/01/api-design-part-1-before-there-was-rest/)
1. [SOAP](https://chelseatroy.com/2018/08/01/api-design-part-1-before-there-was-rest/)
1. [REST](https://chelseatroy.com/2018/08/10/api-design-part-2-the-arrival-of-rest/)
1. [HATEOAS](https://chelseatroy.com/2018/08/10/api-design-part-2-the-arrival-of-rest/) _(hypermedia)_

^ There are pros/cons for each of these, but I would say that they're all designed _from a server implementor's perspective_. What are the problems with that? Well…

---

# **Problem:** over-fetching

- Server can't know which fields the client needs
- Different clients need different data (e.g., mobile vs. desktop)
- Wasteful to include unnecessary data in responses

^ Workarounds like "detailed" resources or `?detail=full` start inching toward GraphQL anyway, but less precisely!

---

# **Problem:** under-fetching

- New app features means additional data required
- Either the server code has to change or the client has to make more queries
- End up with N+1 patterns if data is too limited

^ Workarounds like "detailed" resources or `?detail=full` start inching toward GraphQL anyway, but less precisely!

---

> GraphQL is unapologetically driven by the requirements of views and the front‐end engineers that write them.
-- [GraphQL specification](https://spec.graphql.org/June2018/#sec-Overview)

---

# Client-driven

- Client gets exactly what it wants
- Other architectures are friendly to server implementors but not clients
- Requires the server to be more defensive!

---

# Strongly-typed & introspective

- Queries are type-checked before execution
- The server can guarantee the shape of the response
- Tools can check query validity, inspect the schema

^ https://graphql.org/learn/introspection/
https://github.com/graphql/graphiql

---

# GraphQL
# primitives

---

[.build-lists: false]

# Queries

- Equivalent to REST `GET`, `HEAD`

[.column]
```
query {
  hero {
    name
    friends {
      name
    }
  }
}
```

[.column]
```json
{
  "hero": {
    "name": "R2-D2",
    "friends": [
      { "name": "Luke Skywalker" },
      { "name": "Han Solo" },
      { "name": "Leia Organa" }
    ]
  }
}
```

^ This omits the top-level "data" field that results are actually returned under, for simplified formatting on a slide. See: https://graphql.org/learn/queries/

---

[.build-lists: false]

# Mutations

- Optional operation type
- Equivalent to REST `POST`, `PUT`, `DELETE`

[.column]
```
mutation {
  likeStory(storyID: "12345") {
    story {
      likeCount
    }
  }
}
```

[.column]
```json
{
  "likeStory": {
    "story": {
      "likeCount": 21
    }
  }
}
```

^ https://graphql.org/learn/queries/

---

[.build-lists: false]

# Subscriptions

- Optional operation type
- Streaming, pub/sub as a first-class concept

```
subscription {
  newMessage(roomID: "123") {
    sender
    text
  }
}
```

^ https://spec.graphql.org/June2018/#sec-Subscription

---

# Subscriptions

```json
[]
```

^ https://spec.graphql.org/June2018/#sec-Subscription

---

# Subscriptions

```json
[{
  "newMessage": {
    "sender": "Alice",
    "text": "Hello world!"
  }
}]
```

^ https://spec.graphql.org/June2018/#sec-Subscription

---

# Subscriptions

```json
[{
  "newMessage": {
    "sender": "Alice",
    "text": "Hello world!"
  }
},
{
  "newMessage": {
    "sender": "Bob",
    "text": "yo"
  }
}]
```

^ https://spec.graphql.org/June2018/#sec-Subscription

---

# Subscriptions

```json
[{
  "newMessage": {
    "sender": "Alice",
    "text": "Hello world!"
  }
},
{
  "newMessage": {
    "sender": "Bob",
    "text": "yo"
  }
},
{
  "newMessage": {
    "sender": "Taylor",
    "text": "👋"
  }
}]
```

^ https://spec.graphql.org/June2018/#sec-Subscription

---

# Fragments

[.column]
- Fetch the same fields in multiple places
- Compose many different fetches into one
- Pattern-match on type within a query

[.column]
```
{
  user(id: 4) {
    friends(first: 10) {
      ...friendFields
    }
    mutualFriends(first: 10) {
      ...friendFields
    }
  }
}

fragment friendFields on User {
  id
  name
  profilePic(size: 50)
}
```

^ https://spec.graphql.org/June2018/#sec-Language.Fragments

---

## **(some)**
# [fit] Best practices

---

# Server design should focus on types

- GraphQL schema should be designed around types that make sense
- Don't make assumptions about client fetch patterns
- Abstract away ugly details where possible

^ e.g., data fulfilled by different backends

---

# Client design should focus on fragments

- GraphQL queries are hierarchical
- UI views are also hierarchical!
- Define one fragment per view
- Compose all the fragments on screen together into one single query

---

# Fetch-and-subscribe

- Client fetches initial state in a query, observes changes with a subscription
- No polling, repeat queries necessary
- Local data store can be kept fully consistent
- Be careful about race conditions!

^ [Live queries](https://www.youtube.com/watch?v=BSw05rJaCpA) solve the race condition problem, but aren't widely implemented

---

# Pagination

[.column]
- Use opaque cursors, not page numbers or offsets
- Each page should yield a fully hierarchical object type

[.column]
```
{
  friends(first: 10, after: "opaqueCursor") {
    edges {
      cursor
      node {
        id
        name
      }
    }
    pageInfo {
      hasNextPage
    }
  }
}
```

^ https://relay.dev/graphql/connections.htm

---

# Backwards compatibility

- GraphQL is unversioned
- Fields, types should not be removed
- Fields should not _change_ type
- _Can_ replace list results with empty lists
- Use deprecation warnings!

^ Includes input fields. These rules only apply when clients have actually shipped using the fields, of course.

---

# Nullability

- Changing a field to nullable is a breaking change
- Errors within a non-null field will fail the parent
- Better to make fields nullable by default

---

# Demand control

[.column]
- GraphQL requires the server to assume more complexity
- The possible space of unreasonable or malicious queries is _huge_

[.column]
- [Estimate query complexity](https://github.com/slicknode/graphql-query-complexity)
- [Preregister supported queries](https://principledgraphql.com/operations#8-access-and-demand-control)
- Apply backend rate limits

^ e.g., maximum database QPS/QPM per client. If hit, issue HTTP 429 Too Many Requests or HTTP 503 Service Unavailable and require client to back off.

---

# [fit] Remember…

---

## GraphQL is
# [fit] client-driven
##  **_Design accordingly!_**

---

# Resources

- [GraphQL website](https://graphql.org)
- [GraphQL specification](https://spec.graphql.org/June2018/)
- [Principled GraphQL](https://principledgraphql.com)
