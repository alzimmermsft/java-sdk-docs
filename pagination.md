# Pagination & iteration

Many operations provided by the client libraries within the Azure Java SDK return more than one result. The Azure Java SDK uses standard return types in these cases to ensure developer experience is maximized. The return types used are `PagedIterable` for sync APIs and `PagedFlux` for async APIs. The APIs differ slightly on account of their different use cases, but conceptually they offer the same functionality:

1. By default, both `PagedIterable` and `PagedFlux` enable the common case to be quickly and easily achieved: iterating over a paginated response deserialized into a given type `T`. In the case of `PagedIterable`, it implements the `Iterable` interface, and offers API to receive a `Stream`. In the case of `PagedFlux`, it is a `Flux`.

2. Both `PagedIterable` and `PagedFlux` have methods that will return appropriate types to iterate by page, rather than by individual element. This allows for easy retrieval of response information.

## Synchronous Pagination and Iteration

### Iterating over Individual Elements

The most common use case is to iterate over each element individually, rather than per page. The code samples below show how the `PagedIterable` API allows for users to use the iteration style they prefer to implement this functionality.

#### Using a _for-each_ loop

Because `PagedIterable` implements `Iterable`, it is possible to iterate through the elements using code such as that shown below:

```java
PagedIterable<Secret> secrets = client.listSecrets();
for (Secret secret : secrets) {
   System.out.println("Secret is: " + secret);
}
```

#### Using Stream

```java
client.listSecrets()
      .stream()
      .forEach(secret -> System.out.println("Secret is: " + secret));
```

#### Using Iterator

```java
Iterator<Secret> secrets = client.listSecrets().iterator();
while (it.hasNext()) {
   System.out.println("Secret is: " + it.next);
}
```

### Iterating over Pages

When working with individual pages is required, for example for when HTTP response information is required, or when continuation tokens are important to retain iteration history, it is possible to iterate per page.

#### Using a _for-each_ loop

```java
Iterable<PagedResponse<Secret>> secretPages = client.listSecrets().iterableByPage();
for (PagedResponse<Secret> page : secretPages) {
   System.out.println("Secret page is: " + page);
}
```

#### Using Stream

```java
client.listSecrets()
      .streamByPage()
      .forEach(page -> System.out.println("Secret page is: " + page));
```

When using PagedIterable, iterating by page will result in fetching the first page of results and in addition to the first page, an additional page is eagerly retrieved as the underlying publishers for the first page and the next page are concatenated. This is because the iterable is created by stringing together a series of `Mono` publishers and when the first page is retrieved, the underlying flux needs to know whether to complete the flux or to wait for more data from the concatenated `Mono`. `PagedFlux` doesn't require this eager retrieval as the flux is directly handed to the user and nothing happens until user subscribes and depending on the subscription type, the flux will fetch only the necessary pages lazily.

## Lazy generation of continuation token
Most Azure services that support pagination will provide a continuation token in the response and clients can directly use this continuation token to request for the next page. However, some services, like Cosmos, generate continuation token on the client-side using complex operations which can be quite expensive. To support such scenarios, Java has a `Page` interface that can be implemented by custom paged responses to lazily generate the continuation token when requested. See below example:

```java
public class LazyPagedResponse implements PagedResponse<String> {

    private final List<String> items;
    private final Supplier<String> lazyContinuationTokenSupplier;
    private final int statusCode;
    private final HttpHeaders headers;
    private final HttpRequest httpRequest;

    public LazyPagedResponse(List<String> items, Supplier<String> lazyContinuationTokenSupplier, int statusCode,
        HttpHeaders headers, HttpRequest httpRequest) {
        this.items = items;
        this.lazyContinuationTokenSupplier = lazyContinuationTokenSupplier;
        this.statusCode = statusCode;
        this.headers = headers;
        this.httpRequest = httpRequest;
    }
    @Override
    public List<String> getItems() {
        return this.items;
    }

    @Override
    public String getContinuationToken() {
        return this.lazyContinuationTokenSupplier.get();
    }

    @Override
    public int getStatusCode() {
        return this.statusCode;
    }

    @Override
    public HttpHeaders getHeaders() {
        return this.headers;
    }

    @Override
    public HttpRequest getRequest() {
        return this.httpRequest;
    }

    @Override
    public void close() throws IOException {

    }
}
```
To test:

```java
    @Test
    public void testLazyPagedResponse() {
        Supplier<String> expensiveContinuationTokenGenerator = () -> {
            try {
                // expensive operation
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
            }
            System.out.println("Generating new continuation token");
            return UUID.randomUUID().toString();
        };

        LazyPagedResponse response = new LazyPagedResponse(Arrays.asList("foo", "bar"),
            expensiveContinuationTokenGenerator, 200, null, null);
        System.out.println(response.getItems());
        System.out.println(response.getStatusCode());
        // Continuation token is not generated until it's required
        System.out.println(response.getContinuationToken());
   }
```

## Asynchronous Pagination and Iteration

> TODO