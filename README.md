# Spring.Cache.Extensions - 高性能滑动过期单点更新缓存拓展类
Spring.Cache.Extensions 是一个为解决高并发场景缓存过期可能造成雪崩而产出一个类库，主要解决缓存（滑动过期）单点更新，适用于生成缓存逻辑耗时长的对而对查询速度有要求的高并发场景。

# 使用方法

## 注册
```cs

public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        // This lambda determines whether user consent for non-essential cookies is needed for a given request.
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
    });

    services.AddSingleton(typeof(InterfaceA), typeof(ClassA));

    Cluster.UseClusterLock((config, ioc) =>
    {
        config.ConnectionString = "192.168.210.141:7002,192.168.210.141:7001,keepAlive=180,connectTimeout=200000,syncTimeout=200000,abortConnect=false";
        ioc.SetAddSingleton((a) => services.AddSingleton(a));
        ioc.SetAddSingleton((a, b) => services.AddSingleton(a, b));
        ioc.SetGetService((a) => services.BuildServiceProvider().GetService(a));
    });

    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}

```

## 使用

```cs

public class IndexModel : PageModel
{
    public static IMemoryCache cache = new MemoryCache(Options.Create(new MemoryCacheOptions()));
    private AppClusterLocker _locker;
    public IndexModel(AppClusterLocker locker)
    {
        _locker = locker;
    }
    public void OnGet()
    {
        //获取缓存时间
        var value = _locker.Get<string>("Cache_Key_XXX",
        (k) => cache.Get<CacheValueFormat<string>>(k),
        () => DateTime.Now.ToString(),
        (k, a, b) => cache.Set<CacheValueFormat<string>>(k, a, b),
        TimeSpan.FromSeconds(23),
        TimeSpan.FromSeconds(600));

        ViewData.Add("test", value);
    }
}

```
