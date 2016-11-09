### caching

Options: **in memory, redis**

Method: a **try-get-set** pattern will take in a key and a delegate that will be used to set the key if it doesn't exist. This will wrap up a lot of excess logic for simple cache values.

```csharp
async Task<T> TryGet<T>(string key, Func<Task<T>> ifCacheMissAction)
```

Here's a simple example using redis:

```csharp

public async Task<T> TryGet<T>(string key, Func<Task<T>> ifCacheMissAction)
{
    if (ifCacheMissAction == null) throw new ArgumentNullException("ifCacheMissAction");
    if (string.IsNullOrEmpty(key)) throw new ArgumentNullException("key");

    RedisValue redisValue = Cache.StringGet(key);
    // redis returns {} when there's no hit in the cache
    if (redisValue.HasValue && redisValue != "{}")
    {
        return JsonConvert.DeserializeObject<T>(redisValue);
    }

    T item = await ifCacheMissAction();
    SetValue(key, item);
    return item;
}

```

Then the usage becomes something like this:

```csharp

IEnumerable<AgendaItem> agendaItems = await _redisCache.TryGetList<IEnumerable<AgendaItemDto>>(cacheKey, async () =>
{
    RestRequest restRequest = new RestRequest(resource, Method.GET);
    IEnumerable<AgendaItemDto> agendaItemDtos =
        await
            _integrationClient.ExecuteGetRequest<IEnumerable<AgendaItemDto>>(restRequest,
                "There was an issue hitting the endpoint: {0}".FormatWith(resource));
    ...
});

```

Here's an example javascript implementation (typescript!)

```javascript

public tryGetSet<T>(key: string, apiCall: Function): ng.IPromise<T> {
    var deferred = this.$q.defer();
    var value = this.cache.get(key);

    if(value !== undefined && value !== null) {
        deferred.resolve(value);
        return deferred.promise;
    }
    return apiCall().then((response: T) => {
        this.set(key, response);
        return response;
    }, (e) => {
        console.error(e);
    });
}
```

Here's the raw javascript implementation

```javascript

Greeter.prototype.tryGetSet = function (key, apiCall) {
    var deferred = this.$q.defer();
    var value = this.cache.get(key);
    if (value !== undefined && value !== null) {
        deferred.resolve(value);
        return deferred.promise;
    }
    return apiCall().then(function (response) {
        _this.set(key, response);
        return response;
    }, function (e) {
        console.error(e);
    });
};

```

Why? Code reuse and an easy to use cache implementation.