#Clients for Riak CS

As Riak CS is API-compliant with Amazon S3, any Amazon s3 client should allow you to communicate with your Riak CS instance. Here are a few we have validated to work with Riak CS for Pivotal Cloud Foundry.

- [aws-cli](#aws-cli) - The AWS Command Line Interface (CLI) is a unified tool to manage your AWS services (Apache 2.0 license)
- [fog](#fog) - this Ruby library is the swiss-army knife for the cloud (MIT license)

##Prerequisites

You have created a service instance, bound it to an application, and have binding credentials from the VCAP\_SERVICES environment variable.

```
"VCAP_SERVICES":
{
  "p-riakcs": [
    {
      "name": "mybucket",
      "label": "p-riakcs",
      "tags": [
        "riak-cs",
        "s3"
      ],
      "plan": "developer",
      "credentials": {
        "uri": "https://my-access-key-id:url-encoded-my-secret-access-key@p-riakcs.mydomain/service-instance-id",
        "access_key_id": "my-access-key-id",
        "secret_access_key": "my-secret-access-key"
      }
    }
  ]
}
```

You can also get access credentials without binding an app using `cf create-service-key` and `cf service-key`.

##<a id='aws-cli'></a>aws-cli

See the [AWS CLI](https://aws.amazon.com/cli/) docs for installation instructions.

First, run `aws configure` to add your riak-cs `access_key_id` and `secret_access_key`. The other options can be left as their defaults. Make sure to 
note your service-instance-id in the credentials, as above.

An example command using the `aws-cli` on a bosh-lite with a cf-riak-cs deployment:

```
$ aws s3 --endpoint-url https://p-riakcs.bosh-lite.com cp my-local-file s3://service-instance-e689c062-dee6-45d7-90fe-39e63256915f
$ aws s3 --endpoint-url https://p-riakcs.bosh-lite.com ls s3://service-instance-e689c062-dee6-45d7-90fe-39e63256915f
2015-09-29 15:57:16   23808604 my-local-file
```
**Note**: If running Cloud Foundry with self-signed SSL certificates, you'll need to provide the `--no-verify-ssl` option to the `aws` command. This will produce some ugly errors, but should function normally.

##<a id='fog'></a>fog

[Fog](http://fog.io) requires Ruby to be installed.

Install the fog gem:

<pre class="terminal">
$ gem install fog
</pre>

Create a ruby client object (requires fog):

```
require 'fog'

basic_client = Fog::Storage.new(
  provider: 'AWS',
  path_style: true,
  host: 'p-riakcs.mydomain',
  port: 80,
  scheme: 'http',
  aws_access_key_id: 'my-access-key-id',
  aws_secret_access_key: 'my-secret-access-key')
```
*Note: try this in irb with `irb -r 'fog'` and then create the client object.*

To list bucket contents at service-instance-location:

`basic_client.get_bucket('service-instance-id')`

To put text to bucket with key `mykey`:

`basic_client.put_object('service-instance-id','mykey','my text here')`

To get file with key `mykey` from bucket:

`basic_client.get_object('service-instance-id', 'mykey')`
