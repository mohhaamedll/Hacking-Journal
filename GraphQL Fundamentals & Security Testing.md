# What is GraphQL

GraphQL is a query language for APIs that was developed to reduce the need for multiple API end points, by exposing application data through a single GraphQL endpoint.

Instead of making multiple endpoints like in the traditional REST APIs, GraphQL usually uses one endpoint containing multiple operations and data types. 

GraphQL acts as a layer between the Front-End and the backend, clients send queries to the GraphQL endpoint to fetch data, reducing unnecessary data transfer and making applications more flexible and efficient. 

GraphQL requests are commonly sent over HTTP using JSON request bodies, and responses are usually returned in JSON format.

GraphQL is built around a schema, inside the schema there is a list of types, each type has fields inside each field there optionally can be arguments or inputs.
- *Example:* *`{  Type: Query --> Inside the query type there are fields {Business, Review} --> inside each field there are inputs {ID , username, ..}   `*

There are also mutation that are responsible for making a changes in the application, such as deleting a user or change an email address, mutations are used for actions that change server-side data.

There are also aliases and variables, GraphQL queries and mutations contain fields, and those fields can accept arguments. Variables allow dynamic values to be passed into queries, making them reusable and easier to manage.
- *Example: We are fetching a user data so we just make one function that takes a dynamic argument that's a changeable variable so Instead of writing a separate query for every user, the same query can be reused with different variable values.*

Variables are important because they reduce unnecessary repetition and make queries reusable.

Aliases are also important because GraphQL does not allow the same field to be queried multiple times with different arguments unless aliases are used.
- *Example:  We want to fetch a product 1 { ID , Price } and product 2 { ID, Price, Detail, Discount } so both requests use the same field `GetProduct` but with different arguments, so aliases are required to avoid conflicts.*
- *`{ productOne: GetProduct(ID: "1"){ID, Price} ProductTwo: GetProduct(ID: "2"){ID, Price, Detail, Discount} }`*
So now both data can be fetched without any problem.

--------------------------------------------------------------------------
# How to Obtain info from GraphQL 

After knowing what GraphQL is, how it work and its structure you should know how to discover the schema of a GraphQL API.

And we do that using introspection which is a built-in GraphQL feature that allows clients to query the schema itself. Tools like Apollo or GraphiQL can automatically generate introspection queries to retrieve schema details.

*Introspection Query Example*:
```
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
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
  type { ...TypeRef }
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

When you find a GraphQL endpoint you can send the introspection query to it, If introspection is enabled, the server responds with schema metadata in JSON format. 

You should inspect it so you can understand the structure of the GraphQL API schema, but if the data is too large and you can't read it all you can use **GRAPHQL VOYAGER** which will give you an interactive graphical visualization of the entire structure of the GraphQL application **(Schema Visualization)**.

If introspection is disabled you change your approach like:
1) Sending malformed queries and analyzing GraphQL error messages to reveal:
- type names
- valid arguments
- mutations
2) Field suggestion leaks, Some GraphQL servers return messages like:
- “Did you mean `user`?”
3) GraphQL IDE exposure: Sometimes tools like: *GraphiQL, Apollo Sandbox, Playground* are still exposed even when introspection is disabled.
4) JavaScript file analysis: Frontend JS bundles often contain:
- query names
- mutation names
- fragments
- schema hints
5) Brute-force field enumeration: Using wordlists to test common fields like:
- `user`
- `admin`
- `email`
- `passwordReset`

*NOTE: In production environments, introspection is often disabled or restricted to prevent schema enumeration and reduce attack surface.*

-------------------------------------------------------------------------
# GraphQL Attack Vectors

Once you understand the schema and can interact with queries and mutations, you can begin security testing for vulnerabilities.

 - Test for **IDORs** (Insecure Direct Object References), Check whether the GraphQL application properly implements access controls..
 
- Test for SQL injection if the backend unsafely concatenates user-controlled input into database queries instead of using parameterized queries.

- Test for **XSS** (Cross Site Scripting) an application GraphQL application could be vulnerable to Stored XSS, which could lead to token theft..

- Test for Broken Access Controls You may be able to execute privileged mutation operations, such as deleting users, without proper authorization checks.

-------------------------------------------------------------------------
# Disclaimer  
  
This content is intended for educational purposes and authorized security testing only. Do not test systems without proper permission.
