There's two things that combined have the potential to increase the speeds of the queries exponentially:
##### JSON-RPC batch request
Consist in grouping a bunch of transactions in the same batch before sending to the provider. This is a native method from JSON-RPC that reduces the numbers of travels to the back-end.
The  downside is the we'd need to implement the type check of the response manually since alloy.rs doesn't provide batch requests

##### Multi-thread requests
Split the batch requests into different threads, so we can


#### Challenges
This is probably going to result into hitting the rate-limits way to many times. Need to make sure to counter-balance with some waiting periods between the requests