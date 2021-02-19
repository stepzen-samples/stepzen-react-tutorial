The client will query a GraphQL API created from the JSONPlaceholder API. We are able to use the [`@rest` directive](https://stepzen.com/blog/how-to-connect-any-rest-backend) to easily translate the JSONPlaceholder API into a GraphQL schema.

![04-fetch-users-from-jsonplaceholder](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xw3dvkovbdbikjxrride.png)

## Project Setup

Clone the repository and install dependencies.

```
git clone https://github.com/stepzen-samples/stepzen-react-tutorial
cd stepzen-react-tutorial && npm i
```

Create a `.env.local` file for local environment variables.

```
touch .env.local
```

You will need to include your StepZen API key and API endpoint into `.env.local`.

```
REACT_APP_STEPZEN_API_KEY=YOUR_KEY_HERE
REACT_APP_STEPZEN_ENDPOINT=YOUR_ENDPOINT_HERE
```

Start the development server on `localhost:3000`.

```
npm start
```

## GraphQL API

### users.graphql

The `User` interface includes an `id` for each `User` and information about the `User` such as their `name` and `email`. For our `Query` we just have a single query called `getUsers` that returns an array of `User` objects. The `@rest` directive accepts the `endpoint` from JSONPlaceholder.

```graphql
# stepzen/schema/users.graphql

interface User {
  id: ID!
  name: String!
  username: String!
  email: String!
  phone: String!
  website: String!
}

type Query {
  getUsers: [User]
}

type UserBackend implements User {}

type Query {
  getUsersBackend(unused: String!): [UserBackend]
    @supplies(query:"getUsers")
    @rest(endpoint:"https://jsonplaceholder.typicode.com/users")
}
```

### index.graphql

Our `schema` in `index.graphql` ties together all of our other schemas. For this example we just have the `users.graphql` file included in our `@sdl` directive.

```graphql
# stepzen/schema/index.graphql

schema
  @sdl(
    files: [
      "users.graphql"
    ]
  ) {
  query: Query
}
```

### Deploy API

The `stepzen start` command uploads and deploys your API automatically.

```bash
stepzen start stepzen-react-tutorial/users
```

A browser window with a GraphiQL query editor can be used to query your new endpoint.

![03-graphql-api-explorer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mr9wywu4doovb3h8j162.png)

This also deployed our API to `https://username.stepzen.net/stepzen-react-tutorial/users/__graphql`. Fill in your username and set the URL to the `REACT_APP_STEPZEN_ENDPOINT` environment variable.

## Apollo Client

```javascript
// src/index.js

import ApolloClient from "apollo-boost";
import { ApolloProvider } from "@apollo/react-hooks";

const {
  REACT_APP_STEPZEN_API_KEY,
  REACT_APP_STEPZEN_ENDPOINT
} = process.env;

const client = new ApolloClient({
  headers: {
    Authorization: `Apikey ${REACT_APP_STEPZEN_API_KEY}`,
  },
  uri: REACT_APP_STEPZEN_ENDPOINT,
});

ReactDOM.render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <h1>StepZen React Tutorial</h1>
      <Users />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
);
```

### GET_USERS_QUERY

```javascript
// src/Users.js

import { useQuery } from "@apollo/react-hooks"
import { gql } from "apollo-boost"

export const GET_USERS_QUERY = gql`
  query getUsers {
    getUsers {
      id
      name
    }
  }
`
```

### Users

```javascript
// src/Users.js

const Users = () => {
  const { data, loading } = useQuery(GET_USERS_QUERY)
  const users = data?.getUsers

  return (
    <>
      <h2>Users</h2>
      {loading ? (<div>Loading</div>) : (
        <pre>
          {JSON.stringify(users, null, "  ")}
        </pre>
      )}
    </>
  )
}
```