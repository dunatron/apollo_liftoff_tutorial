# Tutorial notes

Apollo studio: https://studio.apollographql.com/

# Tutorial references

- https://www.apollographql.com/tutorials/voyage-part1/04-building-out-the-subgraphs

# Making a subgraph

- install apollo subgraph  
  `npm install @apollo/subgraph`
- publishing a subgraph

```sh
APOLLO_KEY=service:flyby-83y3h:xxxxxxxxx-XX \
  rover subgraph publish flyby-83y3h@current \
  --schema ./products-schema.graphql \
  --name your-subgraph-name \
  --routing-url http://products.prod.svc.cluster.local:4001/graphql
```

# Rover

Rover is Apollo's command-line interface for managing and maintaining graphs with GraphOS:

installation can be found here:

https://www.apollographql.com/docs/rover/getting-started

`npm install --save-dev @apollo/rover`

however it is probably best to install into each of the subgraphs dev dependencies so it can be called with npx

`npx rover`

# Authorization with the CLI

`npx rover config auth`

You will be promted to add your `APOLLO_KEY` to auth. You should get a 'Successfully saved API key.' when authorization goes correct

# rover subgraph publish

```sh
rover subgraph publish <APOLLO_GRAPH_REF> \
  --name <SUBGRAPH NAME> \
  --schema <SCHEMA FILE PATH> \
  --routing-url <ROUTING URL>
```

e.g

```
npx rover subgraph publish flyby-83y3h@current \
  --name locations \
  --schema ./locations.graphql \
  --routing-url http://localhost:4001
```

`--name` What we want to call our subgraph in Apollo Studio  
`--schema` The relative path to our subgraph's schema file  
`--routing-url` The URL where our subgraph runs (locally, for now)

# Apollo router install

- https://www.apollographql.com/docs/router/quickstart/

### Step 1

cd into the router directory and run `curl -sSL https://router.apollo.dev/download/nix/latest | sh`

You will see a new binary file called router in the ./router directory

### Step 2

navigate to the router directory and use the following command to launch your router

```sh
APOLLO_KEY=service:flyby-83y3h:xxxxxxxxxx-XX APOLLO_GRAPH_REF=flyby-83y3h@current ./router
```

### step 3

You should get a message that the server is running on `http://127.0.0.1:4000/` you then need to go to `https://studio.apollographql.com/` and add it to your connection settings
