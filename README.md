# Graph API vs REST API

## Drawbacks of REST API

1. **Overfetching** - Getting back more data than we need  
   _Example:_ `mysites.com/api/courses`

2. **Underfetching** - Getting back less data than we need  
   _Example:_ `mysites.com/api/courses/1`

## GraphQL

1. The ability to nest any related data into a single query instead of making multiple queries as we might do in REST API.  
   This overcomes the drawbacks of REST API.

---

### Making Apollo Server

---

Refer to the official documentation:  
https://www.apollographql.com/docs/apollo-server/getting-started

**Perform the following steps only:**

**Step 1:** Create a new project  
**Step 2:** Install dependencies

Further steps are as follows:  
https://github.com/adityac1305/graphql-playground/tree/feature/create-apollo-server

---

### Schema and types

---

We also use the extension of Graphql syntax highlighter in VS Code for:
- Highlighting the schema syntax in template literals.
- Validating queries against your schema.
- Enabling autocomplete and linting.

**Example:**

<details>
<summary>schema.js <i>without</i> the extension</summary>

```javascript
export const typeDefs = `
  type Query {
    games: [Game]
  }
`;
```
</details>

<details>
<summary>schema.js <i>with</i> the extension</summary>

```javascript
export const typeDefs = `#graphql
  type Query {
    games: [Game]
  }
`;
```
</details>

- **typeDefs** → definitions of types of data
- **schema** describes the shape of the graph and data available on it.

In GraphQL, typeDefs (type definitions) describe the shape of your data and the operations you can perform.

They are written using the GraphQL Schema Definition Language (SDL), and they define:
1. What data types exist (User, Post, etc.)
2. What queries clients can run
3. What mutations (writes) are allowed
4. What relationships exist between types

In Apollo Server, you usually pass typeDefs when creating your server.

#### Standard Scalar Types in GraphQL

There are 5 standard scalar types:
- **Int** → whole numbers
- **Float** → decimals
- **String** → text
- **Boolean** → true/false
- **ID** → unique identifier (string-like, often used as primary key)

#### Making Parameters Required

If we want the parameters to be not null (required), we can update the code to add `!`.

For `platform: [String]`, if we want the platform to accept an array which is not null and also the values in the array should not be null, we add `!` outside the array and also inside it.

**Before:**
```graphql
type Game {
    id: ID,
    title: String,
    platform: [String]
}
```

**After:**
```graphql
type Game {
    id: ID!,
    title: String!,
    platform: [String!]!
}
```

`type Query` is the root "read menu" of your GraphQL API — it defines all the ways clients can fetch data.


---

### Resolver functions ###

---

**typeDefs**  
- define the schema/blueprint of the GraphQL API (what data and operations exist).
- ensure clients know exactly what they can query.

**resolvers**  
- provide the logic/implementation that actually returns the data.
- connect those queries/mutations to real data sources (DB, APIs, etc.).



### Installing nodemon

```shell
npm install -g nodemon
```

Example output:
```
C:\Users\adity>npm install -g nodemon
added 29 packages, and audited 30 packages in 2s

4 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

#### Checking version of nodemon

```shell
nodemon -v
```
Example output:
```
C:\Users\adity>nodemon -v
3.1.10
```

#### Checking where the global npm packages are installed

```shell
npm config get prefix
```
Example output:
```
C:\Users\adity>npm config get prefix
C:\Users\adity\AppData\Roaming\npm
```

#### Adding the following in environment variables "Path"
```
C:\Users\adity\AppData\Roaming\npm
```

Using nodemon restarts the server every time you make changes to the server.  
If you just use node, you will have to cancel the server manually and then rerun the server to pick up the new change.

#### Issue - Running nodemon commands gave us the following error

Has Execution Policies that block running scripts that are not digitally signed.  
`nodemon.ps1` (installed by npm globally) is treated as a script, and PowerShell refuses to run it because it’s unsigned.

#### Solution

Run the following command in PowerShell:
```shell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Example:
```
PS D:\Projects\Git_Repositories\GraphQL_Projects\graphql-playground> Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

_RemoteSigned_ → Allows running local scripts freely, requires downloaded scripts to be signed.  
This is safe for development on your machine.

---

### Query Variables ###

---

How do we get single game, single review or single author?

We add the following in `type Query` for returning single game with the passed id in argument in `schema.js`:
```graphql
game(id: ID!): Game
```

In the `index.js` we add the resolver for this as follows:
```javascript
game(_, args) {
    return db.games.find((game) => game.id === args.id);
}
```

Resolvers in GraphQL always receive four parameters (not mandatory):  
`resolver(parent, args, context, info)`

When we need to use only args:  
eg - `game(_, args)`

- **parent** ( _ here ) → result of the previous resolver (not needed in this case, so _ is used as a placeholder).
- **args** → an object containing the arguments passed to the query.

`db.games` is your in-memory database (the array of game objects).  
`.find()` iterates through the array of game objects (`(game) =>`) and returns the first game object where `game.id === args.id`.

Also, we can either use `db.games.find((game) => game.id === args.id);` or `db.games.find(game => game.id === args.id);`  
JavaScript lets you omit the parentheses when the arrow function has exactly one parameter.

**When parentheses are required:**
- There are zero parameters: eg - `() => console.log("Hello");`
- There are multiple parameters: eg - `(a, b) => a + b;`
- You want to destructure parameters: eg - `({ id }) => id === args.id;`

`===` is strict equality checker to see if the data type and value are same.

---

### Related Data ###

---

Adding in `schema.js`:

For Game:
```graphql
reviews: [Review!] # Every Game would have multiple reviews
```

For Review:
```graphql
author: Author!   # For every review we would have single author
game: Game!       # Every review is based off single game
```

For Author:
```graphql
reviews: [Review!] # Every author would write multiple reviews
```

The server needs resolvers for each nested field to know how to fetch that data.  
Adding in `index.js` in resolvers:
```javascript
Game: {
    reviews(parent) {
        return db.reviews.filter((review) => review.game_id === parent.id);
    }
},

Review: {
    author(parent) {
        return db.authors.find((author) => author.id === parent.author_id);
    },
    game(parent) {
        return db.games.find((game) => game.id === parent.game_id);
    }
},

Author: {
    reviews(parent) {
        return db.reviews.filter((review) => review.author_id === parent.id);
    }
}
```

---

### Mutations-adding-deleting-data ###

---

**Mutation:**  
Special root type in GraphQL for operations that change data (insert, update, delete).

#### Example Mutations

**addGame(game: AddGameInput!): Game**  
Takes an input object `game` (of type `AddGameInput`).

The datatype `AddGameInput` is defined as follows in `schema.js`:
```graphql
input AddGameInput {
  title: String!
  platform: [String!]!
}
```
`addGame` returns the newly created game.

**deleteGame(id: ID!): [Game]**  
Takes an `id` and returns the list of remaining games after deletion.


#### Using the spread operator

`...args.game`  
This is object spread syntax in JavaScript.  
It takes all the key-value pairs from `args.game` and copies them into the new object.

**Example:**

When  
`args.game = { title: "Zelda", platform: ["Switch"] }`  
Then  
```javascript
{
  ...args.game
}
```
Becomes  
```javascript
{
  title: "Zelda",
  platform: ["Switch"]
}
```

If you didn’t use the spread operator (`...args.game`), you’d need to manually do:
```javascript
let game = {
  title: args.game.title,
  platform: args.game.platform,
  id: Math.floor(Math.random() * 10000).toString(),
};
```

In JavaScript, `.push()` is an array method that adds a new element to the end of the array.  
As our db is a fake database, `db` is just an in-memory array, so `.push()` works as your “database insert”:
```javascript
db.games.push(game);
```

---

### Mutations-updating-data ###

---

You can also update data using mutations.  
For example, an `updateGame` mutation might take an `id` and an `edits` object, then update the matching game and return the updated game.

---

### Additional

---

**Object and Functions**

The following are present in the schema.js file

- **Query** → object  
  - `games()` → function (no args, returns all games)
  - `game(_, args)` → function (takes id, returns a single game)

- **Mutation** → object  
  - `addGame(_, args)` → function (creates and returns a new game)
  - `deleteGame(_, args)` → function (deletes game, returns updated list)
