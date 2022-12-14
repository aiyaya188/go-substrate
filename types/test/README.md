# Types Test

The purpose of this test is to ensure that any types can be decoded successfully by using live blockchain
data (i.e. metadata and storage). In an effort to remove dependencies on external service providers such as `subscan` for this test,
the test data can be generated on demand by using the [test-gen](test-gen/main.go) tool.

The generation of test data and test execution can be triggered via the
`generate-test-data` and `test-types-decode` targets, see [Makefile](../../Makefile). 

## Test data generation

The test data is generated by invoking the [test-gen](test-gen/main.go) tool via `go:generate` in [types_test](types_test.go), 
and it's done based on the information provided for each field of the `types.EventRecords` struct found [here](../event_record.go).

### Parsing

#### Field name

The field name, e.g. `Auctions_AuctionStarted`, is used for generating the request data for querying the `subscan` API 
for that particular event, this will result in a request that has the parameters `module=Auctions` 
and `call=AuctionStarted`.

#### Field tags

Field tags are used for providing information related to the endpoints used for retrieving the events defined by the above field names.

The following tags are available:
- `test-gen-skip` - if set to `true`, the field will be skipped, it is used when there are no events available for that 
particular pallet in any of the blockchains that are relevant when testing.


- `test-gen-blockchain` - the name of the blockchain that will be used for querying the events. This blockchain name
is used to generate the API / WS URLs based on the `subscan` / `onfinality` formats defined in the [parser](test-gen/parser.go):
  - `WsURLFormat  = "wss://%s.api.onfinality.io/public-ws"`
  - `APIURLFormat = "https://%s.api.subscan.io/api/scan/events"`


- `test-gen-api` and `test-gen-ws` - the specific API / WS URLs to be used for generating the test data.

### Retrieval

Once the fields are parsed accordingly, the actual test data is retrieved by:
1. Querying the API for each event and the last block number where that event was encountered.
2. Querying the WS for retrieving the latest metadata and the storage data under the `System_Events` storage key, 
for the block number retrieved in the previous step.
3. Saving the raw bytes of the metadata and storage data.