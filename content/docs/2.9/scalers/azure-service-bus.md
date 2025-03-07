+++
title = "Azure Service Bus"
maintainer = "Microsoft"
description = "Scale applications based on Azure Service Bus Queues or Topics."
availability = "v1.0+"
go_file = "azure_servicebus_scaler"
+++

### Trigger Specification

This specification describes the `azure-servicebus` trigger for Azure Service Bus Queue or Topic.

> ⚠️ **WARNING:** KEDA is not in charge of managing entities. If the queue, topic or subscription does not exist, it will not create them automatically.

```yaml
triggers:
- type: azure-servicebus
  metadata:
    # Required: queueName OR topicName and subscriptionName
    queueName: functions-sbqueue
    # or
    topicName: functions-sbtopic
    subscriptionName: sbtopic-sub1
    # Optional, required when pod identity is used
    namespace: service-bus-namespace
    # Optional, can use TriggerAuthentication as well
    connectionFromEnv: SERVICEBUS_CONNECTIONSTRING_ENV_NAME # This must be a connection string for a queue itself, and not a namespace level (e.g. RootAccessPolicy) connection string [#215](https://github.com/kedacore/keda/issues/215)
    # Optional
    messageCount: "5" # Optional. Count of messages to trigger scaling on. Default: 5 messages
    activationMessageCount: "2"
    cloud: Private # Optional. Default: AzurePublicCloud
    endpointSuffix: servicebus.airgap.example # Required when cloud=Private
```

**Parameter list:**

- `messageCount` - Amount of active messages in your Azure Service Bus queue or topic to scale on.
- `activationMessageCount` - Target value for activating the scaler. Learn more about activation [here](./../concepts/scaling-deployments.md#activating-and-scaling-thresholds).(Default: `0`, Optional)
- `queueName` - Name of the Azure Service Bus queue to scale on. (Optional)
- `topicName` - Name of the Azure Service Bus topic to scale on. (Optional)
- `subscriptionName` - Name of the Azure Service Bus queue to scale on. (Optional*, Required when `topicName` is specified)
- `namespace` - Name of the Azure Service Bus namespace that contains your queue or topic. (Optional*, Required when pod identity is used)
- `connectionFromEnv` - Name of the environment variable your deployment uses to get the connection string of the Azure Service Bus namespace. (Optional)
- `cloud` - Name of the cloud environment that the service bus belongs to. Must be a known Azure cloud environment, or `Private` for Azure Stack Hub or Air Gapped clouds. (valid values: `AzurePublicCloud`, `AzureUSGovernmentCloud`, `AzureChinaCloud`, `AzureGermanCloud`, `Private`; default: `AzurePublicCloud`)

When `cloud` is set to `Private`, the `endpointSuffix` parameter is required. Otherwise, it is automatically generated based on the cloud environment. `endpointSuffix` represents the service bus endpoint suffix of the cloud environment that the service bus belongs to, e.g. `servicebus.usgovcloudapi.net` for `AzureUSGovernmentCloud`.

> 💡 **NOTE:** Service Bus Shared Access Policy needs to be of type `Manage`. Manage access is required for KEDA to be able to get metrics from Service Bus.

### Authentication Parameters

You can authenticate by using pod identity or connection string authentication.

**Connection String Authentication:**

- `connection` - Connection string for Azure Service Bus Namespace.

### Example

Here is an example of how to use managed identity:

```yaml
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: azure-servicebus-auth
spec:
  podIdentity:
    provider: azure
---
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: azure-servicebus-queue-scaledobject
  namespace: default
spec:
  scaleTargetRef:
    name: azure-servicebus-queue-function
  triggers:
  - type: azure-servicebus
    metadata:
      # Required: queueName OR topicName and subscriptionName
      queueName: functions-sbqueue
      # or
      topicName: functions-sbtopic
      subscriptionName: sbtopic-sub1
      # Required: Define what Azure Service Bus to authenticate to with Managed Identity
      namespace: service-bus-namespace
      # Optional
      messageCount: "5" # default 5
      cloud: AzureGermanCloud # Optional. Default: AzurePublicCloud
    authenticationRef:
        name: azure-servicebus-auth # authenticationRef would need either podIdentity or define a connection parameter
```
