    # lspe-graphql-demo
    
    
    
    Pre Requisites
    ==============
    
    1. Install Node JS and NPM from https://nodejs.org/en/
    2. Validate using npm -v and node -v
    3. Install MongoDB from https://www.mongodb.com/download-center#community
    4. Validate Mongo is up and running on default port 27017
    5. Add bin folder to PATH variable so that you can run Mongo commands from shell
    
    Set up
    ======
    
    1. Create new directory : "LSPE-GraphQL-Demo"
    2. cd LSPE-GraphQL-Demo
    3. mkdir server
    4. cd server
    5. npm init
    6. npm install express express-graphql graphql --save
    7. npm install bluebird cors mongoose --save
    
    
    Code Base
    ==========
    
    1. Create Directories
        config
        models
        graphql
        graphql/types
        graphql/queries
        graphql/mutations
    
    2. Create Files
    
        config/config
    
    module.exports = {
        //MongoDB configuration
        development: {
            db: 'mongodb://127.0.0.1/graphql',
            app: {
                name: 'graphql'
            }
        }
    };
    
    
        config/mongoose.js
    
    var env = process.env.NODE_ENV || 'development',
        config = require('./config')[env],
        mongoose = require('mongoose');
    
    module.exports = function () {
        mongoose.Promise = global.Promise;
        var db = mongoose.connect(config.db, { useNewUrlParser : true });
        mongoose.connection.on('error', function (err) {
            console.log('Error: Could not connect to MongoDB. Did you forget to run `mongod`?'.red);
        }).on('open', function () {
            console.log('Connection extablised with MongoDB')
        })
        return db;
    };
    
        models/user.js
    
    var mongoose = require('mongoose');
    var Schema   = mongoose.Schema;
    
    var userSchema = new Schema({
        name : {
            type : String,
            required : true
        }
    });
    
    var Model = mongoose.model('User', userSchema);
    module.exports = Model;
    
        graphql/types/user.js
    
    var GraphQLObjectType = require('graphql').GraphQLObjectType;
    var GraphQLNonNull = require('graphql').GraphQLNonNull;
    var GraphQLID = require('graphql').GraphQLID;
    var GraphQLString = require('graphql').GraphQLString;
    
    // User Type
    exports.userType = new GraphQLObjectType({
      name: 'user',
      fields: function () {
        return {
          id: {
            type: new GraphQLNonNull(GraphQLID)
          },
          name: {
            type: GraphQLString
          }
        }
      }
    });
    
    
        graphql/queries/user.js
    
    var GraphQLObjectType = require('graphql').GraphQLObjectType;
    var GraphQLList = require('graphql').GraphQLList;
    var UserModel = require('../../models/user');
    var userType = require('../types/user').userType;
    
    // Query
    exports.queryType = new GraphQLObjectType({
      name: 'Query',
      fields: function () {
        return {
          users: {
            type: new GraphQLList(userType),
            resolve: function () {
              const users = UserModel.find().exec()
              if (!users) {
                throw new Error('Error')
              }
              return users
            }
          }
        }
      }
    });
    
    
        graphql/mutations/add.js
    
    var GraphQLNonNull = require('graphql').GraphQLNonNull;
    var GraphQLString = require('graphql').GraphQLString;
    var UserType = require('../types/user');
    var UserModel = require('../../models/user');
    
    exports.add = {
      type: UserType.userType,
      args: {
        name: {
          type: new GraphQLNonNull(GraphQLString),
        }
      },
      resolve(root, params) {
        const uModel = new UserModel(params);
        const newUser = uModel.save();
        if (!newUser) {
          throw new Error('Error');
        }
        return newUser
      }
    }
    
        graphql/mutations/update.js
    
    var GraphQLNonNull = require('graphql').GraphQLNonNull;
    var GraphQLString = require('graphql').GraphQLString;
    var UserType = require('../types/user');
    var UserModel = require('../../models/user');
    
    exports.update = {
      type: UserType.userType,
      args: {
        id: {
          name: 'id',
          type: new GraphQLNonNull(GraphQLString)
        },
        name: {
          type: new GraphQLNonNull(GraphQLString),
        }
      },
      resolve(root, params) {
        return UserModel.findByIdAndUpdate(
          params.id,
          { $set: { name: params.name } },
          { new: true }
        )
          .catch(err => new Error(err));
      }
    }
    
        graphql/mutations/remove.js
    
    var GraphQLNonNull = require('graphql').GraphQLNonNull;
    var GraphQLString = require('graphql').GraphQLString;
    var UserType = require('../types/user');
    var UserModel = require('../../models/user');
    
    exports.remove = {
      type: UserType.userType,
      args: {
        id: {
          type: new GraphQLNonNull(GraphQLString)
        }
      },
      resolve(root, params) {
        const removeduser = UserModel.findByIdAndRemove(params.id).exec();
        if (!removeduser) {
          throw new Error('Error')
        }
        return removeduser;
      }
    }
    
        graphql/mutations/index.js
    
    var addUser = require('./add').add;
    var removeUser = require('./remove').remove;
    var updateUser = require('./update').update;
    
    module.exports = {
      addUser,
      removeUser,
      updateUser
    }
    
        graphql/index.js
    
    var GraphQLSchema = require('graphql').GraphQLSchema;
    var GraphQLObjectType = require('graphql').GraphQLObjectType;
    var queryType = require('./queries/user').queryType;
    var mutation = require('./mutations/index');
    
    exports.userSchema = new GraphQLSchema({
      query: queryType,
      mutation: new GraphQLObjectType({
        name: 'Mutation',
        fields: mutation
      })
    })
    
        server.js
    
    const express = require("express");
    const mongoose = require('./config/mongoose');
    const graphqlHTTP = require("express-graphql");
    const cors = require("cors");
    const db = mongoose();
    const app = express();
    
    app.use('*', cors());
    
    const userSchema = require('./graphql/index').userSchema;
    app.use('/graphql', cors(), graphqlHTTP({
      schema: userSchema,
      rootValue: global,
      graphiql: true
    }));
    
    // Up and Running at Port 4000
    app.listen(process.env.PORT || 4000, () => {
      console.log('A GraphQL API running at port 4000');
    });
    
    
    Run
    ====
    
    1. node server.ls
    2. Run GraphQL queries
    
    
    GraphQL Queries
    ===============
    
    {
        users {
            id
            name
        }
    }
    
    mutation {
        addUser(name : "LSPE-User2") {
            id
        }
    }
    
    mutation {
        updateUser(id : "X") {
            id
            name
        }
    }
    
    mutation {
        removeUser(id : "X") {
            id
            name
        }
    }
    
    
    Mongo Commands
    ==============
    
    mongo
    use graphql
    show collections
    db.users.find()
