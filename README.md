# GraphQL Federation with Ballerina and WunderGraph
This repository includes details on implementing GraphQL federation with the Ballerina GraphQL package and the WunderGraph router. In this example, a product review system will be built using Ballerina and WunderGraph Federation.

This service provides the functionality to retrieve product information, user information, and review information. Additionally, it provides the functionality to add new reviews to the products.

## GraphQL schema
```graphql
type Mutation {
  addReview(input: ReviewInput!): Review!
}

type Product {
  id: ID!
  name: String!
  description: String!
  price: Float!
}

type Query {
  products: [Product!]!
  product(id: ID!): Product
  reviews: [Review!]!
  users: [User!]!
}

type Review {
  id: ID!
  title: String!
  comment: String!
  rating: Int!
}

input ReviewInput {
  title: String!
  comment: String!
  rating: Int!
  authorId: String!
  productId: String!
}

type User {
  id: ID!
  name: String!
  email: String!
}
```

## Subgraphs
This example includes three subgraphs named [**Users**](https://github.com/wunder-bal/ballerina-graphql-federation-users-subgraph), [**Products**](https://github.com/wunder-bal/ballerina-graphql-federation-products-subgraph), and [**Rewiews**](https://github.com/wunder-bal/ballerina-graphql-federation-reviews-subgraph) implemented using Ballerina GraphQL.

## Router
The WunderGraph [router](https://cosmo-docs.wundergraph.com/router/intro) is used as the router of federated GraphQL service. The WunderGraph is an Open-Source GraphQL solution to manage Federated Graphs at scale. In this examle, cosmo router docker image is used.

## Steps to Build Federation Sample
1. Deploy the subgraphs in Choreo as public GraphQL services. Subgraphs should be deployed as no-auth services.
2. Create the following `compose.yaml` file with subgraph details.
   ```yaml
     version: 1
     
     subgraphs:
       - name: <subgraph_name>
         routing_url: <subgraph_url>
         introspection:
           url: <intorpection_endpoint_of_subgraph>
   ```
   ### example
   ```yaml
   version: 1
   
   subgraphs:
    - name: products
      routing_url: http://localhost:9091
      introspection:
        url: http://localhost:9091
    - name: users
      routing_url: http://localhost:9092
      introspection:
        url: http://localhost:9092
    - name: reviews
      routing_url: http://localhost:9093
      introspection:
        url: http://localhost:9093
   ```
4. Install the WunderGraph command line tool and generate the `config.json` file using the following command. To generate the config.json, provide the above `compose.yaml` as input.
   ```shell
   npm install -g wgc@latest
   
   wgc router compose -i compose.yaml -o config.json
   ```
5. Then create the `config.yaml` file, which includes the path of the previously created `config.json` file.
   ```yaml
   dev_mode: true
   # Path to the previous generated file
   router_config_path: config.json
   graph:
     # Can be omitted for local testing.
     token: ""
   ```
6. Create the `docker-compose.yaml` file as follows.
   ```yaml
   version: '2.2'

   name: ballerina-wundergraph-federation
  
   services:
     router:
       image: ghcr.io/wundergraph/cosmo/router:latest
       ports:
         - 3002:3002
       environment:
         - pull=always
         - LISTEN_ADDR=0.0.0.0:3002
       volumes:
         - ./config.yaml:/config.yaml
         - ./config.json:/config.json
   ```
7. Deploy the Docker container in Choreo. The following query can be used to test the federated service.
   ```graphql
   query ExampleQuery {
    products {
      id
      name
    }
    reviews {
      id
      rating
    }
    users {
      id
      name
    }
   }
   ```
