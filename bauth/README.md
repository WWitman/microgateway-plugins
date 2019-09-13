# bauth plugin

## Summary
The bauth plugin enables basic authentication - `Authorization: Basic` - in the Microgateway.  However, the Microgateway does not validate the username and password that is provided in the request.  It extracts the username and password from the Authorization header and creates two new request flow variables - `request.username` and `request.password`.  Create a custom plugin that sends a request to an authorization server to validate the username and password.  

## When do I use this plugin?
Use this plugin when you want to enable basic authentication, but you are required to validate the username and password.  

## Prerequisites
Please complete the following tasks before you use this plugin.  

1. [Install MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Prerequisite)   

2. [Configure MG](https://docs.apigee.com/api-platform/microgateway/3.0.x/setting-and-configuring-edge-microgateway#Part1)


## Plugin configuration properties
The following properties can be set in the `bauth` stanza in the Microgateway configuration file.

```yaml
bauth:
  # set to true if you want to send the Authorization header to the target server; set to false when you want this plugin to remove the header after it is validated.
  keep-authorization-header: true # defaults to false
```

## Enable the plugin
In the Edge Microgateway configuration file (`org-env-config.yaml`) make sure that your plugin sequence is as shown below.

```yaml
plugins:
    sequence:
      - bauth
      # other plugins can be listed here
```

## Configure the plugin
In the same configuration file you also need to configure the `bauth` plugin if you want to change the default behavior.  This example keeps the authorization header to ensure the the Basic Authorization header is included on the request to the target server.    

```yaml
bauth:
  keep-authorization-header: true
```

## Errors
The plugin returns the following error messages.

### 401 Missing Authorization
This error is returned when the Authorization header is not included on the request.  
```
401 Unauthorized

{
  "error": "missing_authorization",
  "error_description": "Missing Authorization header"
}
```

### 400 Bad Request
This error is returned when the Authorization header is invalid, such as when the header is not `Authorization: Basic base64encoded(username:password)` or if the Authorization header does not include the Base64 encoded username and password.  
```
400 Bad Request

{
  "error": "invalid_request",
  "error_description": "Invalid Authorization header"
}
```
