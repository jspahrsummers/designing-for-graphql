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
- Shift from a **server-driven** model to a **client-driven** model
- **Powerful and flexible**, but introduces new challenges

---

# The Before Times

![Dog looking scared/confused](assets/michelle-tresemer-MjKUUaYQQ6U-unsplash.jpg)

1. [CORBA](https://chelseatroy.com/2018/08/01/api-design-part-1-before-there-was-rest/)
1. [SOAP](https://chelseatroy.com/2018/08/01/api-design-part-1-before-there-was-rest/)
1. [REST](https://chelseatroy.com/2018/08/10/api-design-part-2-the-arrival-of-rest/)
1. [HATEOAS](https://chelseatroy.com/2018/08/10/api-design-part-2-the-arrival-of-rest/) _(hypermedia)_

^ There are pros/cons for each of these, but I would say that they're all designed _from a server implementor's perspective_. What are the problems with that? Well…

---

# **Problem:** over-fetching

- Server can't know which fields the client needs
- Different clients need different data
- **Bad solution:** include _all data_ in responses

^ Workarounds like "detailed" resources or `?detail=full` start inching toward GraphQL anyway, but less precisely!

---

# **Problem:** under-fetching

- More app features = more data required
- Server implementation needs to be updated
- **Bad solution:** N+1 query patterns

^ Workarounds like "detailed" resources or `?detail=full` start inching toward GraphQL anyway, but less precisely!

---

![inline GraphQL logo](assets/GraphQL Logo.svg)

# [fit] **Introducing** GraphQL

---

[.quote: text-scale(0.5)]

> GraphQL is unapologetically driven by the requirements of views and the front‐end engineers that write them.
-- [GraphQL specification](https://spec.graphql.org/June2018/#sec-Overview)

---

# Client-driven

- Client gets **exactly** what it wants
- Fetch only the data needed for presentation
- Requires the server to be more flexible!

^ Other architectures are friendly to server implementors but not clients

---

# Strongly-typed & introspective

- Queries are **type checked** before execution
- Server can guarantee the shape of the response
- Tools can check query validity, inspect the schema

^ https://graphql.org/learn/introspection/
https://github.com/graphql/graphiql

---

![Plastic toy blocks](assets/ryan-quintal-US9Tc9pKNBU-unsplash.jpg)

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

[.column]
- Fetch similar fields in **multiple places**
- **Compose** many different fetches into one
- **Pattern match** on types within a query

^ https://spec.graphql.org/June2018/#sec-Language.Fragments

---

![Thumbs up from inside a plant](assets/katya-austin-4Vg6ez9jaec-unsplash.jpg)

## **(some)**
# [fit] Best practices

---

# Server should design **types**

- Schema should be clean, **graph-based**
- Don't assume anything about client fetch patterns
- Abstract away implementation details

^ e.g., data fulfilled by different backends

---

# Client should design **fragments**

- GraphQL queries are **hierarchical**
- **UI views** are also hierarchical!
- Define **one fragment per view**
- Compose all fragments on screen into one query

---

# **Unidirectional** data flow

- **Client data store** should be the source of truth
- Fetch initial state in a **query**
- Observe changes over time with a **subscription**
- Make changes with a **mutation**
- Refresh the UI whenever the data store changes
- Be careful about race conditions!

^ [Live queries](https://www.youtube.com/watch?v=BSw05rJaCpA) solve the race condition problem, but aren't widely implemented

---

[.build-lists: false]

# Pagination

[.column]
- Use **opaque** cursors—avoid page numbers or offsets
- Each page should have an **object graph**

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

- GraphQL is **unversioned**
- **Removing** fields, types will break clients
- **Changing** field types will break clients
- Use deprecation warnings!

^ Includes input fields. These rules only apply when clients have actually shipped using the fields.

---

# Nullability

- Changing a field to **nullable** is a breaking change
- Errors within a non-null field will fail the parent
- Better to make fields **nullable by default**

---

# Demand control

[.column]
- **Flexibility** makes possible many **unreasonable or malicious queries**
- The **GraphQL server** must manage this complexity

[.column]
1. [Estimate query complexity](https://github.com/slicknode/graphql-query-complexity)
1. [Preregister supported queries](https://principledgraphql.com/operations#8-access-and-demand-control)
1. Apply backend rate limits

^ e.g., maximum database QPS/QPM per client. If hit, issue HTTP 429 Too Many Requests or HTTP 503 Service Unavailable and require client to back off.

---

![Pencil on an open notebook](asets/jan-kahanek-g3O5ZtRk2E4-unsplash.jpg)

# [fit] Remember…

---

## GraphQL is
# [fit] client-driven
##  **_Design accordingly!_**

---

[.build-lists: false]

# Resources

- [GraphQL website](https://graphql.org)
- [GraphQL specification](https://spec.graphql.org/June2018/)
- [Principled GraphQL](https://principledgraphql.com)
