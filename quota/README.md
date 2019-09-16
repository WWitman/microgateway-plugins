# quota plugin

## Summary
The quota plugin allows you to restrict the number of requests that the Microgateway processes during a specific interval for a specific client.  It is similar to, but not exactly the same as the [Quota policy](https://docs.apigee.com/api-platform/reference/policies/quota-policy) in Apigee Edge SaaS.  In fact, the quota plugin relies on a proxy deployed in Apigee Edge with a Quota policy attached to the flow.  It also requires that you create a product and include the quota interval and quota limit.  As a best practice you should include the oauth plugin before this plugin so that the developer application and product is used as the identifier. Please read the detailed [Apigee community article](https://community.apigee.com/articles/39793/edge-microgateway-quota-plugin.html) that describes this plugin.

Also note that the quota counter is **not** an exact counter due to the fact that Node.js utilizes an event loop with callbacks and the Microgateway starts with multiple child processes and each one gets a quota counter.  Therefore, if the Microgateway starts with 8 child processes, then it will have 8 quota counters.  Additionally, if you run N Microgateway instances across multiple VMs, each one with 8 child processes, then you will have N * 8 quota counters.  This is the reason why the quota plugin stores the quota in Apigee Edge SaaS; the quota counter stored in the cloud helps synchronize each local quota counter across multiple instances of the Microgateway across requests.  

## Process Summary
1. The client will obtain a client ID and secret.
2. The client will include either the API Key on the request or exchange the client ID and secret for a JWT and include it the request to the Microgateway.
3. The `oauth` plugin will validate either the API Key or the JWT.
4. If the API Key or JWT is valid, then the request will continue to the next plugin, otherwise an error message will be returned to the client application.
5. The `quota` plugin will store the quota counter in Apigee Edge SaaS and fetch the latest update.  
6. The `quota` plugin will determine if the request exceeds the counter for the configured interval.  

## Prerequisites
Please complete the following tasks before you use this plugin.  

1. [Install MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)

3. [Create entities on Apigee Edge](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part2)

## When do I use this plugin?
Use this plugin when you want to restrict the number of request to the Microgateway for a particular client and time interval.  This is not an exact quota counter.  

## Plugin configuration properties
You cannot set any properties for the quota plugin.

## Enable the plugin
In the Edge Microgateway configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauth
      - quota
      # other plugins can be listed here
```

## Configure the plugin
This plugin cannot be configured.  

## Best Practices for configuring this plugin
* The quota plugin requires a consistent connection to Apigee Edge SaaS in order to store updates to the quota counter and fetch the most current value.  If this connection goes down, then the quota counter will not be applied.  
* Always include the `oauth` plugin before the `quota` plugin so that a quota identifier is included in the quota counter.  
* Configure the quota limit in Apigee Edge SaaS when you create the product.  

## Errors
The plugin returns the following error messages.

### 403 Forbidden
This error is returned when the quota counter is exceed on the in the first child process .  
```
403 Forbidden

{
  "message":"exceeded quota",
  "status":403
}
```
