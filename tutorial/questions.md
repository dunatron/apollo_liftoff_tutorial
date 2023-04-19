## Agreeing on a schema

![My Image](assets/pq-1.png)

### The homepage

![My Image](assets/fly-by-home-page-mock.png)

```gql
query GetHomePageLocationsAndReviews {
  latestReviews {
    id
    comment
    rating
    location {
      name
    }
  }
  locations {
    id
    name
    overallRating
    photo
    reviewsForLocation {
      id
      comment
      rating
    }
  }
}
```

### The location details page

The next mock-up is for the location details page, which shows information about the destination, as well as all of its reviews.

![My Image](assets/locations-page-mock.jpg)

The location details page needs data for one particular location, as well as other details and reviews for the location.

```gql
query GetLocationDetails($locationId: ID!) {
  location(id: $locationId) {
    id
    name
    description
    photo
    overallRating
    reviewsForLocation {
      id
      comment
      rating
    }
  }
}
```

This page also needs a way for users to submit new reviews. We'll write a mutation that allows us to do just that.

```gql
mutation SubmitReview($locationReview: LocationReviewInput) {
  submitReview(locationReview: $locationReview) {
    code
    success
    message
    locationReview {
      id
      comment
      rating
    }
  }
}
```

### Key takeaways

- Annotating UI mock-ups is one way to collaborate with the frontend team and implement a schema-first design process.
- Agreeing on your schema structure at the start of a new project means your frontend and backend teams can work in parallel.
- To decide how to split your schema into multiple subgraphs, you can group types and fields related to similar concerns.
- Each subgraph schema contains only the types and fields that particular subgraph is responsible for resolving.

![My Image](assets/pq-12.png)

## Building out the subgraphs

We'll start with the locations subgraph. Here's an architecture diagram showing the files in the subgraph-locations directory and how they're connected:

![My Image](assets/subgraph-locations-architecture.png)

- `index.js`: Creates an ApolloServer instance that runs on port 4001. So far, this is just a normal GraphQL server, not a subgraph.
- `locations.graphql`: A schema that defines the types and fields owned by the locations subgraph.
- `resolvers.js`: Defines resolver functions for the fields in the locations schema.
- `datasources/LocationsApi.js`: Retrieves location data from the locations_data.json file.
  Note: In a real-world application, this data source would talk to a REST API or a database, but we're in tutorial-land so we'll stick with the hard-coded fake data.
- `datasources/locations_data.json`: A JSON object with hard-coded location data.

![My Image](assets/schema-example-fly-by.png)

![My Image](assets/dnd-1.png)

### key takeaways

- Adding a Federation 2 definition to the top of our schema file lets us opt in to the latest features available in Apollo Federation 2.
- To make an ApolloServer instance a federation-ready subgraph, use the buildSubgraphSchema function from the @apollo/subgraph package.

## Managed federation & the supergraph

`Managed federation` is an approach to maintaining a supergraph. With managed federation, updates to your supergraph schema are handled by Apollo Studio and the schema registry, all with zero downtime for your router - which we'll get to in a bit.

At a high-level, the managed federation workflow looks like this:

1. Backend developers design and build their subgraphs.
2. Someone creates a new supergraph in Apollo Studio.
3. Backend developers publish their subgraph schemas to the Apollo schema registry.
4. The schema registry automatically composes the subgraph schemas together into a supergraph schema. and makes it available via Apollo Uplink.
5. The router automatically polls Uplink for any new versions of the supergraph schema.

![My Image](assets/federation-journey-1.png)

![My Image](assets/dnd-2.png)

## Publishing the subgraphs with rover

To publish our subgraphs using Rover, we'll need to save two environment variables from Apollo Studio:

- `APOLLO_KEY`: An API key for authenticating Rover. It starts with something like `service:your-graph-name`.
- `APOLLO_GRAPH_REF`: The graph reference (or graph ref) for our supergraph, which we'll use to tell Rover where to publish our subgraphs.
  - A graph ref starts with the graph's ID, followed by an @ symbol, followed by the graph variant.

![My Image](assets/pq-2.png)

- [Tutorial Link](https://www.apollographql.com/tutorials/voyage-part1/06-publishing-the-subgraphs-with-rover)

# 7. How the router resolves data

We know that the router uses the supergraph schema to resolve incoming GraphQL operations from the client. But how exactly does that process work?

- Trace the journey of a client request through the supergraph
- Describe how the router creates query plans to resolve GraphQL operations across multiple subgraphs

## Step 1: The client request

First, the client sends a GraphQL operation to the router. The client has no clue which fields belong to which subgraphs—or even that there are subgraphs at all!

## Step 2: Building a query plan

The router looks at the fields in the operation and uses the supergraph schema to figure out which subgraphs are responsible for resolving each field.

It uses this information to build a query plan, a list of smaller GraphQL operations to execute on the subgraphs. The query plan also specifies the order in which the subgraph operations need to run.

## Step 3: Executing the query plan

Next, the router carries out the query plan by sending the smaller GraphQL operations to each of the subgraphs it needs data from.

The subgraphs resolve the operations the same way as any other GraphQL server: they use their resolvers and data sources to retrieve and populate the requested data.

## Step 4: The subgraph responses

The subgraphs send back the requested data to the router, and then the router combines all those responses into a single JSON object.

## Step 5: Sending data back to the client

Finally, the router sends the final JSON object back to the client. And that's the end of our operation's journey!

Here's the entire journey of a GraphQL operation through the supergraph, summarized in a single diagram:
![federated graph query animations](assets/step-7-recap.png)

![Step seven practice question one](assets/step-7-pq-1.png)

## Key takeaways

- The router uses the supergraph schema to create a query plan for the incoming GraphQL operation. The query plan is a list of smaller operations the router can execute on different subgraphs to fully resolve the incoming operation.
- The router carries out the query plan by executing the list of operations on the appropriate subgraphs.
- The router combines all the responses from the subgraphs into a single JSON object, which it sends back to the client.

# 8. How the router resolves data

So far, FlyBy's subgraphs are running and their schemas have been published, but we still need one piece to tie everything together: the router.

- Set up the Apollo Router locally
- Connect our router to Apollo Studio
- Send our first query to our supergraph

## Downloading the router

The Apollo Router is a high-performance graph router built in Rust. It's available as an executable binary that you can add to your project in a few steps:

1. Open a terminal window and navigate to the `router` directory in the FlyBy project.

2. We'll download the Router by running the install command in the terminal.
   - Note: Visit the [official documentation](https://www.apollographql.com/docs/router/quickstart/#manual-download) to explore alternate methods of downloading the Router.

## Running the router

### 1

Back in the same terminal window, run the command below. You'll need to replace the `<APOLLO_KEY>` and `<APOLLO_GRAPH_REF>` placeholder values with your supergraph's corresponding values from the router/.env file. This command starts up the router locally and tells the router which supergraph to connect to.

```
APOLLO_KEY=<APOLLO_KEY> APOLLO_GRAPH_REF=<APOLLO_GRAPH_REF> ./router
```

### 2.

We'll see a few lines of router output, and finally a message that our router is running on port 4000, ready to receive queries!

Let's copy this address, we'll need it to set our connection settings in Studio. This tells outside consumers of our API what endpoint they can use to query our schema.

[Tutorial link](https://www.apollographql.com/tutorials/voyage-part1/08-router-configuration-and-uplink)

![Step 8 practice question 8](assets/step-8-pq-1.png)

## Key takeaways

- The Apollo Router is an executable binary file that can be downloaded and run locally.
- The Query Plan Preview inspects the GraphQL operation in the Explorer and outputs the query plan the router will execute to resolve the operation.
