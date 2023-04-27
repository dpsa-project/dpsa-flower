# dpsa-flower
Server and client to use the [flower framework](https://flower.dev/) for differentially private federated learning with secure aggregation.

Made to be used with the [dpsa infrastructure](https://github.com/dpsa-project/overview), head there for an explanation of the system's participants and properties. Setup of additional aggregation servers is required, head [here](https://github.com/dpsa-project/dpsa4fl-testing-infrastructure) for instructions.

## Example code
There is a [repo](https://github.com/dpsa-project/dpsa4fl-example-project) containing an example implementation learning the CIFAR task using a torch model, where learning is federated using flower with differential privacy and secure aggregation.

## Classes
This package exposes two classes, one for the server and one for the client.
### [`DPSAServer`](https://github.com/dpsa-project/dpsa-flower/blob/3f1becb09bb79dfe26f9ee959114cf6c36a31dbb/dpsa_flower/dpsa_server.py#L40)
The server class extends the flower [server class](https://flower.dev/docs/apiref-flwr.html#module-flwr.server) with the necessities for using DPSA for aggregation. Construction requires the following parameters:

- `model_size: int` The number of parameters of the model to be trained.
- `privacy_parameter: float` The desired privacy per learning step. One aggregation step will
    be `1/2*privacy_parameter^2` zero-concentrated differentially private
    for each client.
- `granularity: int` The resolution of the fixed-point encoding used for secure aggregation.
    A larger value will result in a less lossy representation and more
    communication and computation overhead. Currently, 16, 32 and 64 bit are
    supported.
- `aggregator1_location: str` Location of the first aggregator server in URL format including the port.
    For example, for a server running locally: "http://127.0.0.1:9991"
- `aggregator2_location: str` Location of the second aggregator server in URL format including the port.
    For example, for a server running locally: "http://127.0.0.1:9992"


### [`DPSANumPyClient`](https://github.com/dpsa-project/dpsa-flower/blob/3f1becb09bb79dfe26f9ee959114cf6c36a31dbb/dpsa_flower/dpsa_numpy_client.py#L19)
The client class implements the [`NumPyClient`](https://flower.dev/docs/apiref-flwr.html#numpyclient) interface provided by flower. It's a wrapper for existing `NumPyClient`s adding secure aggregation and differential privacy. The constructor requires the following parameters:
 
- `max_privacy_per_round: float` The maximal zero-contentrated differential privacy budget allowed to be spent on a single round of training. If the selected server offers a weaker guarantee, no data will be submitted and an exception will be raised.
- `aggregator1_location: str` Location of the first aggregator server in URL format including the port. For example, for a server running locally: "http://127.0.0.1:9991"
- `aggregator2_location: str` Location of the second aggregator server in URL format including the port. For example, for a server running locally: "http://127.0.0.1:9992"
- `client: NumPyClient` The NumPyClient used for executing the local learning tasks.
- `allow_evaluate: bool` Evaluation is a privacy-relevant operation on the client dataset. If this flag is set to `False`, evaluation always reports infinite loss and zero accuracy to the server. Otherwise, the evaluation function of the wrapped client will be used and the results will be released to the server, potentially compromising privacy. Defaults to `False`.
