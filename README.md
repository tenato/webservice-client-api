# WebService Client API

An API to easily built webservice client, based on CXF.

## Quick start Guide

The easiest way is to clone the existing example project. Although you can start from scratch by creating a maven simple jar project.

+ Clone the example project :

`git clone https://github.com/cedricbou/webservice-client-example.git <<your project name here>>`

+ (Optional) Remove the .git folder to "ungit" the project :

`cd <<your project name here>>`
`rm -rf .git`

+ Open the pom.xml file :
    1. Modify the groupId, artifactId, version with yours.
    2. Change description and name to something describing the client you are creating.
    3. Update the `<wsdl>` tag in the cxf plugin with the URL to your WSDL.

+ Clean and generate source

`mvn clean generate-sources`

+ The project will contain compilation error, we need to adapt the Client class :
    1. Open the CRMClient class, rename it to whatever you need, and change the package as well to something you like.
    2. Adapt the CxfClient extended class with generic types corresponding to the Port generated by CXF, and the service manager class generated by CXF :
    `public class CRMClient extends CxfClient<_WebServicePort_, _WebServiceManager_>`
    3. In the `newPort` method, calls the manager method to instanciate your port interface, the manager is passed in parameter.
    4. In the `checkIfPortUp` method, implements a health check. If an exception is thrown the service is considered unhealthy.
    
## Configuration DSL 

### Basic features

```java
builder()
	.withWsseCredentials(username, password) // Configure username and password using the WSSE standard, as PasswordText.
	.withConnectionTimeout(1000) // Set the timeout in millis on socket connection
	.withReceiveTimeout(3000) // Set the timeout in millis to wait for data.
	.withEndpoint("http://....") // Define the endpoint to use.
	.withInLogger(logger) // The SLF4J logger to use for payload sent to the server, the level is INFO.
	.withOutLogger(logger) // The SLF4J logger to use for payload received from the server, the level is INFO.
	.withLogger(logger) // Shortcut for withInLogger(logger).withOutLogger(logger)
```

### Endpoint and servers

During service instanciation, the API endpoint failover. For an endpoint,
you can defined several target servers address.

When instanciating the service, for each server, it will attempt to configure a client. It uses the `checkIfPortUp` method to check if the server is available. If everything is fine, it will use this port, otherwise, it would try the next server. If nothing works, it returns `null`.

For this to work, you need to write your endpoint with the special `{{server}}` tag :

```java
builder()
    .withEndpoint("http://{{server}}/mywebservice")
```

Then you'll configure the server list to use :

```java
builder()
    .withEndpoint("http://{{server}}/mywebservice")
    .withServers("localhost:8080", "10.10.10.10:6541");
```

### Loading from properties

```java
Properties props = new Properties();
props.load(...);

builder()
    .withProperties("com.myprops", props)
    .withServers("localhost:8080", "10.10.10.10:6541");
```

## The Client class

To build the client, use the `build` method : 

```java
builder()
	.withReceiveTimeout(2000)
	.withLogger("com.myservice.Trames")
	.build(CRMClient.class);
```

You need to pass the Client class. This is used to automatically cast to the right type in the return.



    



