# Troubleshooting networking issues

The Azure client libraries for Java offer a consistent and robust [logging story](https://github.com/Azure/azure-sdk-for-java/wiki/Logging-with-Azure-SDK) to enable client side troubleshooting. But, the client libraries make network calls, over various protocols, which may lead to troubleshooting scenarios that extend outside of the troubleshooting scope provided. When this happens external tooling to diagnose networking issues is the solution. We'll discuss a few applications that are able to diagnose networking issues of various complexity. The scenarios will range from troubleshooting an unexpected response value from a service to root causing a connection closed exception.

## Fiddler

[Fiddler](https://docs.telerik.com/fiddler-everywhere/introduction) is an HTTP debugging proxy that allows for requests and responses passed through it to be logged as-is. Capturing the raw requests and responses helps aid in troubleshooting scenarios where the service gets an unexpected request or the client receives an unexpected response. To use Fiddler the client library will need to be configured with an HTTP proxy. If HTTPS is being used additional configuration will be needed if the decrypted request and response bodies need to be inspected.

### Adding an HTTP proxy

The Azure HTTP library implementations, `azure-core-http-netty` and `azure-core-http-okhttp`, offer multiple ways to configure a proxy.

#### Environment configuration

By default, when the HTTP client builder isn't passed `Configuration.NONE`, and no proxy has been configured it will inspect the environment for the following
(in order):

- `HTTPS_PROXY`
- `HTTP_PROXY`
- `https.proxy*`
- `http.proxy*`

If the environment is configured with one of the configurations above it will be loaded and used as the proxy for the `HttpClient`.

**_Example_**

This example sets the default Fiddler proxy with `HTTP_PROXY`.

```bash
java -DHTTP_PROXY="http://localhost:8888" -jar example.jar
```

When the default HTTP client builder is used it will scan the environment for proxy configurations and find `HTTP_PROXY=http://localhost:8888` and use it as
the proxy for the constructed `HttpClient`.

#### Using a Configuration object

For finer grain reusable configuration a `Configuration` object with one of the above environment configurations can be passed into the HTTP client builder to
implicitly configure the proxy for the `HttpClient`.

**_Example_**

This example creates a new `Configuration` that sets the default Fiddler proxy with `java.net.httpProxy` configurations.

```java
Configuration configuration = new Configuration()
    .put("java.net.useSystemProxies", "true")
    .put("http.proxyHost", "localhost")
    .put("http.proxyPort", "8888");

HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();

HttpClient okHttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .configuration(configuration)
    .build();
```

`java.net.http*` is split across multiple environment configurations and has a prerequisite configuration to signal that they are allowed to be used. The
HTTP client builder needs to have the constructed `Configuration` passed, otherwise it will use the global environment configuration scope. This scoping allows
for configurations that can be shared but don't affect the entire application.

#### HTTP client builder

Configuring the HTTP client builder with `ProxyOptions` is the most granular way to configure a proxy. This method isn't shareable across multiple locations
but offers the best scoping possible.

**_Example_**

This example sets the proxy using `ProxyOptions`.

```java
HttpClient nettyHttpClient = new NettyAsyncHttpClientBuilder()
    .proxy(new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888)))
    .build();

HttpClient okHttpHttpClient = new OkHttpAsyncHttpClientBuilder()
    .proxy(new ProxyOptions(ProxyOptions.HTTP, new InetSocketAddress("localhost", 8888)))
    .build();
```

### Enable HTTPS decryption

By default Fiddler is only able to capture HTTP traffic. If your application is using HTTPS additional steps must be taken to trust Fiddler's certificate to allow it
to capture HTTPS traffic.

This is a [high-level guide](https://docs.telerik.com/fiddler-everywhere/user-guide/settings/https) on trusting Fiddler's certificate. Below will discuss having your
JRE trust the certificate. Without trusting the certificate HTTPS request through Fiddler may fail with security warnings.

#### Linux/macOS

1. Export Fiddler's certificate
2. Find the JRE's keytool (usually `jre/bin`)
3. Find the JRE's cacert (usually `jre/lib/security`)
4. Run keytool to import the certificate: `sudo keytool -import -file <location of Fiddler certificate> -keystore <location of cacert> -alias Fiddler`
5. Enter a password
6. Trust the certificate

#### Windows

1. Export Fiddler's certificate
2. Find the JRE's keytool (usually `jre/bin`)
3. Find the JRE's cacert (usually `jre/lib/security`)
4. Run keytool to import the certificate: `keytool.exe -import -file <location of Fiddler certificate> -keystore <location of cacert> -alias Fiddler`
5. Enter a password
6. Trust the certificate

## Wireshark

[Wireshark](https://www.wireshark.org/) is a network protocol analyzer that is able to capture traffic going through a network interface without requiring
changes to application code. Wireshark is highly configurable and is able to capture very broad to very specific low level network traffic which allows it to aid
in troubleshooting scenarios such as a remote host closing a connection or having connections closed during operation. The Wireshark GUI differentiates captures
using a color scheme to easily identify unique capture cases such as a TCP retransmission, rst, etc. Captures can also be filtered either at capture time or during
analysis.

### Configuring a capture filter

Capture filters reduce the amount of network calls that are captured for analysis. Without capture filters Wireshark will capture all traffic it is able that goes
through a network interface. This can produce massive amounts of data where most of it may be noise to the investigation. So, using a capture filter helps preemptively
scope the network traffic being captured to help target an investigation.

Wireshark provides an in-depth [guide](https://www.wireshark.org/docs/wsug_html_chunked/ChapterCapture.html) on configuring traffic capture filters.

**_Example_**

This example adds a capture filter to capture network sent to or received from a specific host.

In Wireshark navigate to `Capture > Capture Filters...` and add a new filter with the value `host <host IP or hostname>`. This will add a filter to only capture traffic
to and from that host. If the application communicates to multiple hosts multiple catpure filters can be added or the host IP/hostname can be `or`'d to provide looser
capture filtering.

### Capturing to disk

Reproducing unexpected networking exceptions may requiring running an application for a long time to get the issue to reproduce and see the traffic leading up to it.
It may not be possible to maintain all captures in memory, because of this Wireshark provides the ability to log the captures to disk. Persisting to disk ensures that
the captures are available for post-processing and prevents the risk of running out of memory while reproducing the issue.

Wireshark provides an in-depth [guide](https://www.wireshark.org/docs/wsug_html_chunked/ChapterIO.html) on configuring persisting captured traffic to disk.

**_Example_**

This example sets up Wireshark to persist captures to disk with multiple file where the files split on either 100k capture or 50MB in size.

In Wireshark navigate to `Capture > Options` and navigate to the `Output` tab. Enter a file name to use, this will have Wireshark persist captures to a single file.
Enable multiple files by checking `Create a new file automatically` and then check `after 100000 packets` and `after 50 megabytes`, this will have Wireshark create
a new file after one of the predicates are matched. Each new file will use the same base name as the file name entered and will append a unique identifier. If you
want to limit the number of files that Wireshark is able to create check the `Use a ring buffer with X files`, this will limit Wireshark to logging with only X files
where upon needing a new file after reaching X the oldest is overwritten.

### Filtering captures

Some times it isn't possible to tightly scope the traffic capture by Wireshark, for example your application comminucates with multiple hosts using various protocols.
In this scenario, generally with using persistent capture outlined above, it is easier to run analysis post network capturing. Wireshark provides the ability to use
capture filter-like syntax to be able to analyze captures.

Wireshark provides an in-depth [guide](https://www.wireshark.org/docs/wsug_html_chunked/ChapterWork.html) on filtering captures.

**_Example_**

This example loads a persisted capture file and filters on `ip.src_host==<IP>`.

In Wireshark navigate to `File > Open` and load a persisted capture from the file location used above. Once the file has loaded below the menu bar there is a filter
input. In the filter input enter `ip.src_host==<IP>`, this will limit the capture view to only show captures where the source was from the host with the IP `<IP>`.
