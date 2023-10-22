# AxelART-Subgraph
Delve into AxelART's decentralized art platform with our tailored custom subgraph, enabling swift and precise data queries for an enhanced user experience in the realm of interchain AI art NFTs. Custom AxelART Subgraph + Safe + Axelar.

Build completed: QmefAwLQKZRu4oXJyVnHkLG1cJXyRcMB3AW8h1pyZa1Zmu

Deployed to https://thegraph.com/hosted-service/subgraph/amaansayyad/axelart

Subgraph endpoints:
Queries (HTTP):     https://api.thegraph.com/subgraphs/name/amaansayyad/axelart

```graphql
{
  initializeds(first: 5) {
    id
    version
    blockNumber
    blockTimestamp
  }
  roleAdminChangeds(first: 5) {
    id
    role
    previousAdminRole
    newAdminRole
  }
}
```
