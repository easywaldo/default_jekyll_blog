---
layout: post
title: Elasticsearch.Net Connection Pooling
description: 
date:   2019-08-08 01:10:00 +0530
categories: code
tags: [ software development ]
author: easywaldo
comments: true
---

### Elasctic.Net Connection Pool 성능테스트

[Connection-pooling] 에 대한 문서는 링크 참조
아래의 3가지 방식에 대해서 각각 500개의 질의를 할 때 모두 완료가 된 시간을 기준으로 측정하였다.

- Static
- SingleNode
- StickySniffing

### 테스트를 시작하기에 앞서

어떤것이 가장 빠른 방식인지는 ES 구성과 서버의 상태와 상황에 따라 다를 수 있기에
예측하기가 어려웠다.

그래서 처음에는 `테스트 코드의 이름도 아래 처럼 가장 빠른 것이 무슨 방식이다.` 라고 명명하지는 않았다.
테스트 코드를 구현하고 여러번 수행을 하면서 StickySniffing 방식이 작성된 테스트 코드 기준으로는 가장
빠른 방식임을 알 수 있었고 결국 테스트 코드의 이름을 아래 처럼 변경할 수 있었다.

```csharp
[TestMethod]
public void sticky_sniffing_connection_pool_type_should_be_faster_than_others()
{
    // Arrange
    var uris = Enumerable.Range(9200, 1)
        .Select(port => new Uri(base._setting.EndPoint));
    var stickySniffingPool = new StickySniffingConnectionPool(uris, node =>
        {
            var weight = 0f;
            if (node.ClientNode)
            {
                weight += 10;
            }

            if (node.Settings.TryGetValue("node.attr.rack_id", out var rackId) && 
                rackId.ToString() == "rack_one")
            {
                weight += 10;
            }
            return weight;
        });

    var singleNodePool = new SingleNodeConnectionPool(new Uri(base._setting.EndPoint));
    var staticPool = new StaticConnectionPool(uris);

    var stickySniffingClient = new ElasticClient(
        new ConnectionSettings(stickySniffingPool)
            .ConnectionLimit(300)
            .MaximumRetries(3)
            .RequestTimeout(new TimeSpan(0, 0, 0, 0, 300)));
    var singleNodeClient = new ElasticClient(
        new ConnectionSettings(singleNodePool)
            .ConnectionLimit(300)
            .MaximumRetries(3)
            .RequestTimeout(new TimeSpan(0, 0, 0, 0, 300)));
    var staticClient = new ElasticClient(
        new ConnectionSettings(staticPool)
            .ConnectionLimit(300)
            .MaximumRetries(3)
            .RequestTimeout(new TimeSpan(0, 0, 0, 0, 300)));


    var sut = new ProductService(_coreService, stickySniffingClient);
    var testRequest = new SearchRequest
    {
        From = 0,
        Size = 1000,
        Query = new BoolQuery
        {
            Must = new QueryContainer[]
            {
                new LongRangeQuery
                {
                    GreaterThan = 0,
                    Field = "eventsaleprice",
                },

                new DateRangeQuery
                {
                    Field = "eventstartdate",
                    GreaterThanOrEqualTo = "now",
                    LessThanOrEqualTo = "now+30d",
                    TimeZone = "+09:00",
                }
            }
        }
    };

    var elaspedList = new List<(string poolType, double time)> { };

    // Act
    var stopWatch = new Stopwatch();
    stopWatch.Start();
    var task = Task.Run(async delegate
    {
        var commands = Enumerable.Range(1, 500);
        foreach (var c in commands)
        {
            await stickySniffingClient.SearchAsync<ProductSearch>(testRequest);
        }
    });
    var completed = Task.WhenAll(task);
    try
    {
        completed.Wait();
    }
    catch { }
    stopWatch.Stop();
    elaspedList.Add(("stickySniffingPool", stopWatch.Elapsed.TotalMilliseconds));
    stopWatch.Start();
    task = Task.Run(async delegate
    {
        var commands = Enumerable.Range(1, 500);
        foreach (var c in commands)
        {
            await singleNodeClient.SearchAsync<ProductSearch>(testRequest);
        }
    });
    completed = Task.WhenAll(task);
    try
    {
        completed.Wait();
    }
    catch { }
    stopWatch.Stop();
    elaspedList.Add(("singleNodePool", stopWatch.Elapsed.TotalMilliseconds));
    stopWatch.Start();
    task = Task.Run(async delegate
    {
        var commands = Enumerable.Range(1, 500);
        foreach (var c in commands)
        {
            await staticClient.SearchAsync<ProductSearch>(testRequest);
        }
    });
    completed = Task.WhenAll(task);
    try
    {
        completed.Wait();
    }
    catch { }
    stopWatch.Stop();
    elaspedList.Add(("staticPool", stopWatch.Elapsed.TotalMilliseconds));

    // Assert
    elaspedList.OrderBy(x => x.time).First().poolType.Should().Be("stickySniffingPool");
}
```

### 테스트를 하며 느낀 점

사실 이렇게 성능 비교같은 테스트는 아래 처럼 단위테스트 코드 내지 자동화된 테스트 툴이 없다면 비교 검증하기가
수월하지는 않을 것이다. 더욱이 프로덕션 코드를 수정하면서 테스트 하기에도 시간과 공이 더 많이 소비가 될 것은
쉽게 짐작 가능한 일이다.




[Connection-pooling]: https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/connection-pooling.html



