# RSVP Dapp Subgraph
## Creating Subgraph
- The Graph allows devs to create GraphQL APIs to query data from any smart contract.
	- This makes it fast and easy to create dashboards and websites that show live data about how users are interacting with your contract.
- Anyone can deploy their own API, aka subgraph
	- Our subgraph will allow us to connect our frontend website to our contract so we can easily fetch the data we need.
- You can use The Graph's hosted service for free w/ any chain 
	- Full list here: https://thegraph.com/hosted-service/
### CLI Installation
```bash
# NPM
npm install -g @graphprotocol/graph-cli

# Yarn
yarn global add @graphprotocol/graph-cli
```
- Testing to see if it was installed properly
```bash
graph -v
```

### Setup
- Go to https://thegraph.com/en/ and select Products -> Hosted Service
	- Go to Dashboard and click "Add a Subgraph" button
	- Fill out fields, click "Create Subgraph" button
-  Navigate to where you wanna save your subgraph in your terminal and type the command below:
```bash
graph init --product hosted-service <GITHUB_USER>/<SUBGRAPH NAME>
```
- In terminal:
	- Choose protocol
	- Choose network
	- Enter contract address you deployed
	- Choose contract name
### Structure
- Make `.gitignore` file for `node_modules`
- add `@protofire/subgraph-toolkit` as a dependency
### Schema
- An entity (`@entity`) is like an object
	- It's a "thing" that has keys and values, with each key being a schema field that can be queried for its value.
- Each entity also has an ID field for a unique id, and some fields for information about that entity we want to be able to query.
	- Each field has a type, and if it's required it has a `!` (the value can't be null).
	- Ex: the event name, description link, and imageURL fields are not required, which means that they can return a `null` value.
- In the `Event` and `Account` entities for the `rsvps` and `confirmedAttendees` fields we are using a special keyword `@derivedFrom`, also called a reverse lookup, which allows us to reference data from another entity.
	- Ex:
```
rsvps: [RSVP!] @derivedFrom(field: "event")
```
- Basically whenever someone creates a new RSVP, that RSVP instance gets added to this array if the event id from the "event" field in the RSVP entity matches the id for this Event.
	- Ex:
```
RSVP {
	id: 20
	attendee: 300
	event: 5000
}
```
- This RSVP is for an event with the id 5000.
	- The Event with the matching id will have this RSVP added to the rsvps array.
```
Event {
	id: 5000
	rsvps: [ {id: 20, attendee: 300, event: 5000} ]
}
```
- Each new RSVP for this event will also get added here.
	- Using reverse lookups like this, we are easily able to query all of the RSVPs and Confirmations for any Event or Account
## Subgraph Manifest
- In `subgraph.yaml`, above `dataSources` add a `features` section where we can add `ipfsOnEthereumContracts`
```
features:
	- ipfsOnEthereumContracts
```
- Update `Entity` names
- In the `eventHandlers` section, we can tell the subgraph how to connect each of our mappings to different event triggers.
	- Each time an event that is defined here is emitted from our contract, the corresponding mapping function set as the handler will run
- Import `Address, ipfs, json` from `@graphprotocol/graph-ts`
- Should look like this
`import { Address, ipfs, json } from "@graphprotocol/graph-ts"`
### Mappings
- Running `graph codegen` in the terminal generates AssemblyScript types for our entities and events, and import them at the top of our mappings file
	- **do this in subgraph folder**
- If you make any other changes to the schema or subgraph manifest, you can always run `graph codegen` again to generate new types.
### Deploying
```
graph auth --product hosted-service <ACCESS_TOKEN>
```
- This will authenticate you as the subgraph owner so that you're able to push your code changes.
	- **do this in subgraph folder**
- Run this command to deploy your subgraph to The Graph's hosted service:
```
graph deploy --product hosted-service <GITHUB_USER>/<SUBGRAPH NAME>
```
#### Updating your subgraph
- If you make changes to your smart contract and re-deploy to a new address, you can easily upgrade your subgraph to index the new contract.
- You can update the contract address in the manifest (`subgraph.yaml`), and copy + paste the new abi in the `abis` folder.
	- If you made any changes to the schema or emitted events from your contract, make sure to run `graph codegen` to generate new types.
- When you're ready to redeploy, you can run the same deploy command above.
	- You can view the pending version by toggling the version in the upper right corner.
