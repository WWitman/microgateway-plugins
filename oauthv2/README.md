# oauthv2 plugin

## Summary
This plugin allows customers to secure API requests to the Apigee Edge Microgateway (EM) with OAuth 2.0 (JWT validation).  Therefore, it was created to separate API key validation from OAuth 2.0 token validation and simply its configuration.

## OAuth plugin links
Please review the following OAuth plugin documentation.  
* [Microgateway existing plugins](https://docs.apigee.com/api-platform/microgateway/2.5.x/use-plugins#existingpluginsbundledwithedgemicrogateway)
* [Creating entities on Apigee Edge plugin](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part2createentitiesonapigeeedge)
* [Secure the Microgateway with OAuth2.0](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway)
* [Secure the Microgateway with API Key validation](https://docs.apigee.com/api-platform/microgateway/2.5.x/setting-and-configuring-edge-microgateway.html#part4secureedgemicrogateway-securingtheapiwithanapikey)


## When do I use this plugin?
Use this plugin when you want to enable OAuth 2.0 JWT validation.

## Process Summary

1. The client will obtain an client ID and secret.
2. The client will include either the API Key on the request or exchange the client ID and secret for a JWT and include it the request to the Microgateway.
3. The `oauthv2` plugin will validate either the JWT.
4. If the JWT is valid, then the request will continue to the next plugin, otherwise an error message will be returned to the client application.

## Prerequisites
Please complete the following tasks before you use this plugin.  

1. [Install MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)

3. [Create entities on Apigee Edge](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part2)


## Plugin configuration properties
The following properties can be set in the `oauthv2` stanza in the Microgateway configuration file.

```yaml
oauth:
  # header name used to send the JWT to the Microgateway
  authorization-header: "x-custom-auth-header" # defaults to Authorization
  # set to true if you want to send the Authorization header to the target server; set to false when you want this plugin to remove the header after it is validated.
  keep-authorization-header: true # defaults to false
  # gracePeriod is the number of seconds before the token is removed from the cache.
  # If you set this to 5 (seconds) then MG will check if the difference between the expiry time and the current time [abs(expiry time - current time)] is less than or equal (<=) to the grace period. If true, then MG will remove the token from the cache.  
  gracePeriod: 5 # defaults to 0 seconds
  # set to true to enable the Microgateway to check against the resource paths only.  In this case it ignores the proxy name check.  
  productOnly: true # defaults to false, which enables the Microgateway to check if the proxy name is included in the product.

  ## Note that if you set the tokenCacheSize, then you should also enable it (tokenCache: true)
  # set tokenCache to true if you want to cache the access token (JWT) locally
  tokenCache: true # defaults to false, which does not cache the access token.
  # set the number of tokens allowed to be cached locally
  tokenCacheSize: 150 # defaults to 100
```

## Enable the plugin
In the EM configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - oauthv2
      # other plugins can be listed here
```

## Configure the plugin
In the same configuration file you also need to configure the `oauthv2` plugin if you want to change the default behavior.  The example below changes the `oauthv2` to enable access token caching and changes the `tokenCacheSize` to 150 tokens.    

```yaml
auth:
  tokenCache: true
  tokenCacheSize: 150
```

## Caching
JWT token caching is **not** enabled by default.  

### Cache Headers
The oauthv2 plugin **does not** observe the [`cache-control`](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching) header.
The only way to cache the JWT is to set the `tokenCache` property to `true`.

### Access Tokens
Access tokens (JWTs) can also be cached in the Microgateway to avoid validating the JWT on every request.  Access tokens are only cached if you set `tokenCache` to `true` and the JWT is valid.  

## Best Practices for configuring this plugin
* The oauthv2 plugin is typically listed first in the plugin sequence.  
* If you need one set of APIs to be validated by JWTs and another set to be validated by API Keys (lower security), then consider the following:
  * create two products, one for API Key validation and one set for access token (JWT) validation, then configure two sets for Microgateway instances - one set for API key validation and one set for access token validation.
  * routing between these Microgateway instances should be handled with a HTTPS load balancer.  
