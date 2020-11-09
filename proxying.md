# Proxying

## Introduction

The Azure client libraries for Java offer multiple ways to configure a proxy for an `HttpClient`. Each method of supply a proxy has their own pros and cons and
provide unique levels of encapsulation. Once a proxy has been configured for an `HttpClient` it will use the proxy for the remainder of its lifetime. Having the
proxy tied to an individual `HttpClient` allows for multiple `HttpClient` to be used in an application where each can use a different proxy to accomplish an
application's proxying requirements.

### ProxyOptions

Azure Core provides a `ProxyOptions` class that acts as the Azure client libraries consistent story for configuring a proxy. `ProxyOptions` is able to be configured
with the network protocol used to send proxy requests, the proxy address, proxy authentication credentials, and non-proxying hosts. Only the proxy network protocol
and proxy address are required. When using authentication credentials both the username and password must be set.

**_Examples_**

This example creates a simple `ProxyOptions` that proxies requests to the default Fiddler address (`localhost:8888`).

```java
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888));
```

This example creates an authenticated `ProxyOptions` that proxies requests to a Fiddler instance requiring proxy authentication.

```java
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddess("localhost", 8888))
    .setCredentials("1", "1"); // Fiddler uses username "1" and password "1" with basic authentication as its proxy authentication requirement.
```

### Using an environment proxy

HTTP client builders, by default, will inspect the environment for proxy configurations. This is done by using `Configuration.getGlobalConfiguration()` as the
default `Configuration` used by the builder. Each HTTP client builder has a `configuration(Configuration)` API that can be used to override this behaviour and
using `Configuration.NONE` explicitly prevents the builder from inspecting the environment for configuration.

When the environment is inspected it will search for the following environment configurations in the order specified:

1. `HTTPS_PROXY`
2. `HTTP_PROXY`
3. `https.proxy*`
4. `http.proxy*`

Where `*` is the [well-known Java](https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html) proxy properties.

If any of the environment configurations are found a `ProxyOptions` will be created from their configuration, if properly formatted, effectively using
`ProxyOptions.fromConfiguration(Configuration.getGlobalConfiguration())`. The well-known Java proxy proerties have an additional prerequisite where the property
`java.net.useSystemProxies` must be `true`.

**_Example_**

This example uses the `HTTP_PROXY` environment variable with value `localhost:8888` to use Fiddler as the proxy.

```bash
export HTTP_PROXY=localhost:8888
```

```java
HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder().build();
HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder().builder();
```

To prevent the environment proxy from being used configure the HTTP client builder with `Configuration.NONE`.

```java
HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .configuration(Configuration.NONE)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .configuration(Configuration.NONE)
    .build();
```

### Using a Configuration proxy

HTTP client builders may be configured to use a custom `Configuration` that will configuration set in it instead of inspecting the global environment. This offers
the ability to have reusable configurations that are scoped to a limited use case. When the HTTP client builder is building the `HttpClient` it will use the
`ProxyOptions` returned from `ProxyOptions.fromConfiguration(<Configuration passed into the builder>)`.

**_Example_**

This example uses the `http.proxy*` configurations set in a `Configuration` object to use a proxy authenticating Fiddler as the proxy.

```java
Configuration configuration = new Configuration()
    .put("java.net.useSystemProxies", "true")
    .put("http.proxyHost", "localhost")
    .put("http.proxyPort", "8888")
    .put("http.proxyUser", "1")
    .put("http.proxyPassword", "1");

HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();
```

### Using an explicit proxy

HTTP client builders may be configured with `ProxyOptions` directly to indicate an explicit proxy to use. This is the most granular way to provide a proxy and
generally isn't as flexible as passing a `Configuration` that can be mutated to update proxying requirements.

**_Example_**

This example uses `ProxyOptions` to use Fiddler as the proxy.

```java
ProxyOptions proxyOptions = new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888));

HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .proxy(proxyOptions)
    .build();

HttpClient okhttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .proxy(proxyOptions)
    .build();
```
