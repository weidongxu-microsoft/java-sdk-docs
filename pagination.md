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
   System.out.println("Secret is: " + it.next());
}
```

### Iterating over Pages

When working with individual pages is required, for example for when HTTP response information is required, or when continuation tokens are important to retain iteration history, it is possible to iterate per page.

#### Using a _for-each_ loop

```java
Iterable<PagedResponse<Secret>> secretPages = client.listSecrets().iterableByPage();
for (PagedResponse<Secret> page : secretPages) {
   System.out.println("Response code: " + page.getStatusCode());
   System.out.println("Continuation Token: " + page.getContinuationToken());
   page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
}
```

#### Using Stream

```java
client.listSecrets()
      .streamByPage()
      .forEach(page -> {
          System.out.println("Response code: " + page.getStatusCode());
          System.out.println("Continuation Token: " + page.getContinuationToken());
          page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
      });
```

## Asynchronously observing pages and individual elements

### Observing individual elements

The code sample below shows how the `PagedFlux` API allows users to observe individual elements asynchronously.
 
 ```java
asyncClient.listSecrets()
    .subscribe(secret -> System.out.println("Secret value: " + secret),
        ex -> System.out.println("Error listing secrets: " + ex.getMessage()),
        () -> System.out.println("Successfully listed all secrets"));

TimeUnit.SECONDS.sleep(5);
 ```
 
>**Note:** Since the network calls happens in a different thread than the main-thread that calls `subscribe()`, the main-thread may terminate before the result is available; to avoid this, the main-thread sleeps few seconds.
 
 ### Observing pages
 
 The code sample below shows how the `PagedFlux` API allows users to observe each page asynchronously.
 
  ```java
asyncClient.listSecrets().byPage()
    .subscribe(page -> {
            System.out.println("Response code: " + page.getStatusCode());
            System.out.println("Continuation Token: " + page.getContinuationToken());
            page.getElements().forEach(secret -> System.out.println("Secret value: " + secret))
        },
        ex -> System.out.println("Error listing pages with secret: " + ex.getMessage()),
        () -> System.out.println("Successfully listed all pages with secret"));
        
TimeUnit.SECONDS.sleep(5);
 ```
 
>**Note:** Since the network calls happens in a different thread than the main-thread that calls `subscribe()`, the main-thread may terminate before the result is available; to avoid this, the main-thread sleeps few seconds.
