```
REST CLIENT ---rest calls---> REST API
Front End                     Back End
```

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
        // Start with some default users
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

    findOne({ id }) {
        return (id >= 0 && this.users.length > id) ? this.users[id] : null;
    }
    
    findAll() {
        return this.users || [];
    }

    updateHappiness({ id, happy }) {
        let success = false;
        const user = this.findOne({id: id});
        if(user) {
            user.happy = happy;
            success = true;
        }
        return success;
    }
}

const resolvers = {
    Query: {
        users: (_, __, { dataSources }) => dataSources.userAPI.findAll(),
        user: (_, { id }, { dataSources }) => dataSources.userAPI.findOne({ id: id })
    },
    Mutation: {
        updateUserHappiness: (_, { id, happy }, { dataSources }) => {
            const success = dataSources.userAPI.updateHappiness({ id: id, happy: happy });
            return {success: success, message: success === true ? "Success" : "Failure"};
        }
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

### Schema

- GraphQL schema definitions are enclosed in backticks.
- `Query` and `Mutation` are in built schema types.
- `Query` for read, `Mutation` for write operations.
- `User` is a custom type.   
- You can and should define custom types for data encapsulation.
- Fields suffixed with an exclamation mark (`!`) may not be null, but may be empty arrays etc.
  
### DataSource

- The `UserAPI` could fetch data from a REST API, SQL datasource or other data source.
- Apollo-graphql defines some `DataSource` implementations e.g `RESTDataSource`.
- In this simplified Example, the `UserAPI` contains its own data.
- `DataSource` methods may be async. For example: `async findOne({ id }) {...}` 
  
### Resolver

- Resolvers were only created for those types that cannot be automatically inferred.
- We created a resolver for `Query.user` but not for `User.name` as `User.name` is already a field in the data returned by the `DataSource`
- Resolver methods may be async. For example: `async updateUserHappiness: (_, __, ___, ____) => {...}`
- Resolver function signature is: `fieldName: (parent, args, context, info) => data;` 
  
  ARGUMENT      |	DESCRIPTION        
  --------------|--------------
  parent	    |This is the return value of the resolver for this field's parent (the resolver for a parent field always executes before the resolvers for that field's children).
  args	        |This object contains all GraphQL arguments provided for this field.
  context	    |This object is shared across all resolvers that execute for a particular operation. Use this to share per-operation state, such as authentication information and access to data sources.
  info	        |This contains information about the execution state of the operation (used only in advanced cases).

  _Source: [Apollo GraphQL Tutorial](https://www.apollographql.com/docs/tutorial/resolvers/)_


