---
editable: false
sourcePath: en/_cli-ref/cli-ref/managed-services/managed-mongodb/hosts/update.md
---

# yc managed-mongodb hosts update

Update specified hosts in the cluster.

#### Command Usage

Syntax: 

`yc managed-mongodb hosts update [Flags...] [Global Flags...]`

#### Flags

| Flag | Description |
|----|----|
|`--cluster-id`|<b>`string`</b><br/>ID of the MongoDB cluster.|
|`--cluster-name`|<b>`string`</b><br/>Name of the MongoDB cluster.|
|`--async`|Display information about the operation in progress, without waiting for the operation to complete.|
|`--host`|<b>`PROPERTY=VALUE[,PROPERTY=VALUE...]`</b><br/>Configurations for MongoDB hosts that should be updated.<br/><br/>Possible property names:<br/><ul> <li><code>hostname</code>:     Hostname of the host to update (required)</li> <li><code>assign-public-ip</code>:     Assign a public IP address to the host being updated.</li> <li><code>hidden</code>:     Set hidden parameter to the host being updated</li> <li><code>secondary-delay-secs</code>:     Set secondaryDelaySecs parameter to the host being updated</li> <li><code>priority</code>:     Set priority parameter to the host being updated</li> <li><code>tags</code>:     Set tags parameter to the host being updated</li> </ul>|

#### Global Flags

| Flag | Description |
|----|----|
|`--profile`|<b>`string`</b><br/>Set the custom configuration file.|
|`--debug`|Debug logging.|
|`--debug-grpc`|Debug gRPC logging. Very verbose, used for debugging connection problems.|
|`--no-user-output`|Disable printing user intended output to stderr.|
|`--retry`|<b>`int`</b><br/>Enable gRPC retries. By default, retries are enabled with maximum 5 attempts.<br/>Pass 0 to disable retries. Pass any negative value for infinite retries.<br/>Even infinite retries are capped with 2 minutes timeout.|
|`--cloud-id`|<b>`string`</b><br/>Set the ID of the cloud to use.|
|`--folder-id`|<b>`string`</b><br/>Set the ID of the folder to use.|
|`--folder-name`|<b>`string`</b><br/>Set the name of the folder to use (will be resolved to id).|
|`--endpoint`|<b>`string`</b><br/>Set the Cloud API endpoint (host:port).|
|`--token`|<b>`string`</b><br/>Set the OAuth token to use.|
|`--impersonate-service-account-id`|<b>`string`</b><br/>Set the ID of the service account to impersonate.|
|`--no-browser`|Disable opening browser for authentication.|
|`--format`|<b>`string`</b><br/>Set the output format: text (default), yaml, json, json-rest.|
|`-h`,`--help`|Display help for the command.|
