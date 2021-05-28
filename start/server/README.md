REST CLIENT ---rest calls---> REST API
Front End                     Back End

Why GraphQL

```
                                GRAPHQL 
REST CLIENT   ->   | schema -> resolver -> datasource |   ->   REST API
```

```js
const { gql, ApolloServer } = require('apollo-server');
const { DataSource } = require('apollo-datasource');

const typeDefs = gql`
type User{
  id: ID!
  name: String!
  isHappy: Boolean!    
}

type Query {
  users: [User]!
  user(id: ID!): User
}

type Mutation {
  updateUserHappiness(id: ID!, happy: Boolean!): HappinessUpdateResponse!
}

type HappinessUpdateResponse {
  success: Boolean!
  message: String
}

`

class UserAPI extends DataSource {
    constructor() {
        super();
        this.users = [
            {id: 1, name: "Jasmine Nwanyi", isHappy: true},
            {id: 2, name: "Otobrise Kampe", isHappy: false}
        ];
    }

    /**
     * This is a function that gets called by ApolloServer when being setup.
     * This function gets called with the datasource config including things
     * like caches and context. We'll assign this.context to the request context
     * here, so we can know about the user making requests
     */
    initialize(config) {
        this.context = config.context;
    }

    async findOne({ id }) {
        return this.users[id];
    }
    
    async findAllUsers() {
        return this.users || [];
    }

    async findOrCreateUser({ name: nameArg } = {}) {
        const matching = this.users.filter(user => user.name === nameArg);
        return (matching && matching[0]) ? matching[0] : createUser({ name: nameArg });
    }
    
    async createUser({ name: nameArg } = {}) {
        const user = {id: (this.users.length + 1), name: nameArg, isHappy: true};
        this.users.push(user);
        return user;
    }

    async updateUserHappiness({ id, happy }) {
        for(user of this.users) {
            if(user.id === id) {
                user.happy = happy;
                break;
            }
        }
        return happy;
    }
}

const resolvers = {
    Query: {
        users: (_, __, { dataSources }) => dataSources.userAPI.findAllUsers(),
        user: (_, { id }, { dataSources }) => dataSources.userAPI.findOne({ id: id })
    }
};

const server = new ApolloServer({
    typeDefs,
    resolvers,
    dataSources: () => ({
        userAPI: new UserAPI()
    })
});

server.listen().then(() => {
    console.log(`
    Server is running!
    Listening on port 4000
    Explore at https://studio.apollographql.com/dev
  `);
});
```

- GraphQL schema definitions are enclosed in backticks.
- `Query` and `Mutation` are in built schema types.
- `Query` for read, `Mutation` for write operations.
- `User` is a custom type.   
- You can and should define custom types for data encapsulation.
- The `UserAPI` could fetch data from a REST API, SQL datasource or other data source.
- In simplified Example, the `UserAPI` contains its own data.

