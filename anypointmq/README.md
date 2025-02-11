# Anypoint MQ Bindings

This document defines how to describe Anypoint MQ-specific information in AsyncAPI documents. 

[Anypoint MQ](https://docs.mulesoft.com/mq/) is MuleSoft's multi-tenant, cloud messaging service and is fully integrated with [Anypoint Platform](https://www.mulesoft.com/platform/enterprise-integration).

<a name="version"></a>
## Versions

The version of this bindings specification is `0.0.1`.
This is also the `bindingVersion` for all binding objects defined by this specification.
In any given binding object, `latest` MAY alternatively be used to refer to the currently latest published version of this bindings specification.

## Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this bindings specification are to be interpreted as described in IETF [RFC2119](https://www.ietf.org/rfc/rfc2119.txt).

## Protocol

These bindings use the `anypointmq` [protocol](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#definitionsProtocol) in AsyncAPI documents to denote connections to and interactions with Anypoint MQ message brokers.

The Anypoint MQ protocol is based on invocations of the [Anypoint MQ Broker REST API](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-23abe036327a.anypoint-platform/anypoint-mq-broker/).

## Server Object

The fields of the standard [Server Object](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#serverObject) are constrained and interpreted as follows:

Server Object Field Name | Values for Anypoint MQ Protocol | Description
---|:---|:---
<a name="serverObjectProtocolFieldValueAnypointMQ"></a>`protocol`               | `anypointmq`                                                 | **Required**. MUST be `anypointmq` for the scope of this specification.
<a name="serverObjectUrlFieldValueAnypointMQ"></a>`url`                         | e.g., `https://mq-us-east-1.anypoint.mulesoft.com/api`       | **Required**. MUST be the endpoint URL of the Anypoint MQ Broker REST API _excluding_ the final major version indicator (e.g., `v1`). Valid examples are `https://mq-us-east-1.anypoint.mulesoft.com/api` and `https://mq-eu-central-1.eu1.anypoint.mulesoft.com/api` (and _not_ `https://.../api/v1`).
<a name="serverObjectProtocolVersionFieldValueAnypointMQ"></a>`protocolVersion` | e.g., `v1`                                                   | **Optional**, defaults to `v1`. If present MUST be the major version indicator of the Anypoint MQ Broker REST API omitted from the `url`, e.g. `v1`.
<a name="serverObjectSecurityFieldValueAnypointMQ"></a>`security`               | suitably configured OAuth 2.0 client credentials grant type  | **Required**. Authentication against the MuleSoft-hosted Anypoint MQ message brokers uses the OAuth 2.0 client credentials grant type. At runtime, the client ID and client secret values of an Anypoint MQ client app must be supplied. Also, the OAuth 2.0 scopes are currently not client-configurable. The `security` field of the server object MUST correctly match these constraints.

Note that the choice of a particular Anypoint MQ client app (via its client ID and secret) decides the Anypoint Platform organization ID and environment (ID) to be those in which this client app is defined. See the [Anypoint MQ documentation](https://docs.mulesoft.com/mq/mq-client-apps) for details on how to configure Anypoint MQ client apps.

<a name="server"></a>
## Server Binding Object

This object MUST NOT contain any properties. Its name is reserved for future use.

<a name="channel"></a>
## Channel Binding Object

The Anypoint MQ [Channel Binding Object](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#channel-bindings-object) is defined by a [JSON Schema](json_schemas/channel.json), which defines these fields:

Field Name | Type | Description
---|:---:|---
<a name="channelBindingObjectDestination"></a>`destination`       | string | **Optional**, defaults to the channel name. The destination (queue or exchange) name for this channel. SHOULD only be specified if the channel name differs from the actual destination name, such as when the channel name is not a valid destination name in Anypoint MQ.
<a name="channelBindingObjectType"></a>`destinationType`          | string | **Optional**, defaults to `queue`. The type of destination, which MUST be either `exchange` or `queue` or `fifo-queue`. SHOULD be specified to document the messaging model (publish/subscribe, point-to-point, strict message ordering) supported by this channel.
<a name="channelBindingObjectBindingVersion"></a>`bindingVersion` | string | **Optional**, defaults to `latest`. The version of this binding.

Note that an Anypoint MQ exchange can only be sent to, not received from. To receive messages sent to an exchange, [an intermediary queue must be defined and bound to the exchange](https://docs.mulesoft.com/mq/mq-understanding#message-exchanges). In this bindings specification, these intermediary queues are not exposed in the AsyncAPI document. Instead, it is simply assumed that whenever messages must be received from an exchange, such an intermediary queue is involved yet invisible in the AsyncAPI document.

### Examples

The following example shows a `channels` object with two channels, the second having a channel binding object for `anypointmq`:

```yaml
channels:
  user/signup:
    description: |
      This application receives command messages from this channel about users to sign up.
      Minimal configuration, omitting a channel binding object.
    publish:
      #...
  user/signedup:
    description: |
      This application sends events to this channel about users that have signed up.
      Explicitly provides a channel binding object.
    bindings:
      anypointmq:
        destination:     user-signup-exchg
        destinationType: exchange
        bindingVersion:  '0.0.1'
    subscribe:
      #...
```

<a name="operation"></a>
## Operation Binding Object

This object MUST NOT contain any properties. Its name is reserved for future use.

<a name="message"></a>
## Message Binding Object

The Anypoint MQ [Message Binding Object](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#message-bindings-object) is defined by a [JSON Schema](json_schemas/message.json), which defines these fields:

Field Name | Type | Description
---|:---:|---
<a name="messageBindingObjectHeaders"></a>`headers`               | [Schema Object](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#schemaObject) | **Optional**. A Schema object containing the definitions for Anypoint MQ-specific headers (so-called protocol headers). This schema MUST be of type `object` and have a `properties` key. Examples of Anypoint MQ protocol headers are `messageId` and `messageGroupId`.
<a name="messageBindingObjectBindingVersion"></a>`bindingVersion` | string | **Optional**, defaults to `latest`. The version of this binding.

Note that application headers must be specified in the [`headers` field of the standard Message Object](https://github.com/asyncapi/spec/blob/master/spec/asyncapi.md#messageObjectHeaders) and are transmitted in the [`properties` section of the Anypoint MQ message](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-23abe036327a.anypoint-platform/anypoint-mq-broker/).
In contrast, protocol headers such as `messageId` must be specified in the [`headers` field of the message binding object](#messageBindingObjectHeaders) and are transmitted in the [`headers` section of the Anypoint MQ message](https://anypoint.mulesoft.com/exchange/portals/anypoint-platform/f1e97bc6-315a-4490-82a7-23abe036327a.anypoint-platform/anypoint-mq-broker/).

### Examples

The following example shows a `channels` object with two channels, each having one operation (`subscribe` or `publish`) with one message. Only the message of the `subscribe` operation has a message binding object for `anypointmq`:

```yaml
channels:
  user/signup:
    publish:
      message:
        #...
  user/signedup:
    subscribe:
      message:
        headers:
          type: object
          properties:
            correlationId:
              description: Correlation ID set by application
              type: string
        payload:
          type: object
          properties:
            #...
        correlationId:
          description: Correlation ID is specified as a header and transmitted in the Anypoint MQ message properties section
          location:    $message.header#/correlationId
        bindings:
          anypointmq:
            headers:
              type: object
              properties:
                messageId:
                  type: string
            bindingVersion: '0.0.1'
```

## Complete Example

The following is a complete, simple AsyncAPI document illustrating the usage of all binding objects defined in this bindings specification, with all their fields.

```yaml
asyncapi: '2.0.0'
info:
  title: Example with Anypoint MQ
  version: '0.0.1'

servers:
  development:
    protocol: anypointmq
    protocolVersion: v1
    url: https://mq-us-east-1.anypoint.mulesoft.com/api
    description: |
      Anypoint MQ broker for development, in the US East (N. Virginia) runtime plane 
      under management of the US control plane.
    security:
      - oauthDev: []
  production:
    protocol: anypointmq
    protocolVersion: v1
    url: https://mq-eu-central-1.eu1.anypoint.mulesoft.com/api
    description: |
      Anypoint MQ broker for production, in the EU Central (Frankfurt) runtime plane 
      under management of the EU control plane.
    security:
      - oauthProd: []
    bindings:
      anypointmq:
        bindingVersion: '0.0.1'
  
channels:
  user/signup:
    description: |
      This application receives command messages from this channel about users to sign up.
    bindings:
      anypointmq:
        destination:     user-signup-queue
        destinationType: fifo-queue
        bindingVersion:  '0.0.1'
    publish:
      operationId: signUpUser
      description: |
        This application receives command messages via this operation about users to sign up.
      message:
        contentType: application/json
        headers:
          type: object
          properties:
            correlationId:
              description: Correlation ID set by application
              type: string
        payload:
          type: object
          properties:
            username:
              type: string
              minLength: 3
        correlationId:
          description: Correlation ID is specified as a header and transmitted in the Anypoint MQ message properties section
          location: $message.header#/correlationId
        bindings:
          anypointmq:
            headers:
              type: object
              properties:
                messageId:
                  type: string
            bindingVersion: '0.0.1'

components:
  securitySchemes:
    oauthDev:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://mq-us-east-1.anypoint.mulesoft.com/api/v1/authorize
          scopes: {}
    oauthProd:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://mq-eu-central-1.eu1.anypoint.mulesoft.com/api/v1/authorize
          scopes: {}
```
