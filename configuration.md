# Configuration

Azure Core offers the `Configuration` class as a standard way to load, store, and share environment and runtime application configuration. 

## Using Configuration

`Configuration` has APIs to get, check existence, put, and remove key-value properties. 

### Get

`Configuration` offers multiple get APIs which offer different levels of convenience but all use the same root logic. Initially the configuration which check to see if it already contains the value in its in-memory cache, if it doesn't it will then attempt to load it from the environment checking Java properties (`System.getProperty`) and the environment (`System.getenv`), in that order, accepting the first non-null value loaded. The default get will return the value as a String with get or default and get and transform convenience APIs also being available.

**_Examples_**

__Load HTTP proxy__

```java
String proxy = configuration.load("HTTP_PROXY");
```

__Load HTTP proxy with default value__

```java
String proxy = configuration.load("HTTP_PROXY", "localhost:8888");
```

__Load HTTP proxy and transform to URL__

```java
URL proxy = configuration.load("HTTP_PROXY", endpoint -> {
    try {
        return new URL(endpoint);
    } catch (MalformedURLException ex) {
        return null;
    }
});
```

### Check existence

`Configuration` offers a single existence check API to determine if it contains a value for a given key. This provides the ability to check and put a key instead of it loading from the environment when get is used and the configuration doesn't contain the key.

**_Example_**

__Check for HTTP proxy then get or put__

```java
String proxy;
if (configuration.contains("HTTP_PROXY")) {
    proxy = configuration.get("HTTP_PROXY");
} else {
    configuration.put("HTTP_PROXY", "<default proxy>");
    proxy = "<default proxy>";
}
```

### Put

`Configuration` offers a single put API to directly set a key-value property without doing an environment look-up. This provides the ability to insert runtime properties into the configuration without having to update the environment.

**_Example_**

__Put HTTP proxy__

```java
configuration.put("HTTP_PROXY", "localhost:8888");
```

### Remove

`Configuration` offers a single remove API to purge a key from its local cache. This provides the ability to have configuration load the key-value from the environment again if it is updated during runtime of the application. Additionally, it also provides the ability to remove specific configurations from a cloned configuration to modify the behavior of code that accepts configuration as a parameter.

**_Example_**

__Remove SERVER_ENDPOINT__

```java
String purgedProxy = configuration.remove("HTTP_PROXY");
```

## Configuration scoping

`Configuration` has the ability to be scope preventing application properties from leaking into other areas of an application.

### Global configuration

`Configuration` as a singleton global configuration accessible using `Configuration.getGlobalConfiguration()` that will be used as a default in most locations when a `Configuration` isn't supplied. Updating this will allow for the changes to be shared in all spots where the global configuration is used.

### Scoped configuration

Constructing a `Configuration` instance will create a scoped configuration that is used only in location where it is passed into APIs using configuration. This will allow for application configuration to be scoped while retaining its ability to be shared across multiple locations.

## No-op/Empty configuration

In some situation you may not want `Configuration` to attempt to load from the environment or affect execution of an application, for this reason a special `Configuration.NONE` is available. This special configuration no-ops all operations and will always return null when get is used. This will prevent APIs that accept configuration from using it to modify their behavior/execution.

**_Example_**

__Use the no-op/empty configuration when building an HttpClient__

```java
HttpClient httpClient = new NettyAsyncHttpClientBuilder()
    .configuration(Configuration.NONE)
    .build();
```
