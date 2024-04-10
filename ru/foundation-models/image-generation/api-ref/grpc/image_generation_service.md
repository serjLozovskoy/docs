---
editable: false
sourcePath: en/_api-ref-grpc/foundation-models/image-generation/image-generation/api-ref/grpc/image_generation_service.md
---

# Foundation Models Image Generation API, gRPC: ImageGenerationAsyncService

Service for obtaining images from input data.

| Call | Description |
| --- | --- |
| [Generate](#Generate) | RPC method for obtaining image from inpt data. |

## Calls ImageGenerationAsyncService {#calls}

## Generate {#Generate}

RPC method for obtaining image from inpt data.

**rpc Generate ([ImageGenerationRequest](#ImageGenerationRequest)) returns ([operation.Operation](#Operation))**

Response of Operation:<br>
	&nbsp;&nbsp;&nbsp;&nbsp;Operation.response:[ImageGenerationResponse](#ImageGenerationResponse)<br>

### ImageGenerationRequest {#ImageGenerationRequest}

Field | Description
--- | ---
model_uri | **string**<br>The identifier of the model to be used for image generation. 
messages[] | **[Message](#Message)**<br>A list of messages representing the context for the image generation model. 
generation_options | **[ImageGenerationOptions](#ImageGenerationOptions)**<br>Image generation options 


### Message {#Message}

Field | Description
--- | ---
text | **string**<br>text 
weight | **double**<br>Message weight. Negative values for negative messages 


### ImageGenerationOptions {#ImageGenerationOptions}

Field | Description
--- | ---
mime_type | **string**<br>generated image format 
seed | **int64**<br>Seed for image generation 


### Operation {#Operation}

Field | Description
--- | ---
id | **string**<br>ID of the operation. 
description | **string**<br>Description of the operation. 0-256 characters long. 
created_at | **[google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#timestamp)**<br>Creation timestamp. 
created_by | **string**<br>ID of the user or service account who initiated the operation. 
modified_at | **[google.protobuf.Timestamp](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf#timestamp)**<br>The time when the Operation resource was last modified. 
done | **bool**<br>If the value is `false`, it means the operation is still in progress. If `true`, the operation is completed, and either `error` or `response` is available. 
metadata | **[google.protobuf.Any](https://developers.google.com/protocol-buffers/docs/proto3#any)**<br>Service-specific metadata associated with the operation. It typically contains the ID of the target resource that the operation is performed on. Any method that returns a long-running operation should document the metadata type, if any. 
result | **oneof:** `error` or `response`<br>The operation result. If `done == false` and there was no failure detected, neither `error` nor `response` is set. If `done == false` and there was a failure detected, `error` is set. If `done == true`, exactly one of `error` or `response` is set.
&nbsp;&nbsp;error | **[google.rpc.Status](https://cloud.google.com/tasks/docs/reference/rpc/google.rpc#status)**<br>The error result of the operation in case of failure or cancellation. 
&nbsp;&nbsp;response | **[google.protobuf.Any](https://developers.google.com/protocol-buffers/docs/proto3#any)<[ImageGenerationResponse](#ImageGenerationResponse)>**<br>if operation finished successfully. 


### ImageGenerationResponse {#ImageGenerationResponse}

Field | Description
--- | ---
image | **bytes**<br>Image serialized as bytes array 
model_version | **string**<br>Model version (changes with model releases). 


