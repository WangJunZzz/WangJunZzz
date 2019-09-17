---
title: Apollo
date: 2019-09-16 15:37:45
tags:
---

+ 携程Apollo再.Net FrameWork上的使用
    - 新建一个.Net Framework4.7.2的项目
    - 添加Com.Ctrip.Framework.Apollo.ConfigurationManager包引用
    - 再Web.config的appSettings目录下添加
``` csharp
    <add key="Apollo.AppId" value="framework.web" />
    <add key="Apollo.MetaServer" value="http://192.168.19.48:8080/" />
```
    - 获取配置文件
``` csharp
    // ApolloConfigurationManager 不过这个是过时的
    IConfig config = await ApolloConfigurationManager.GetAppConfig();
    // 获取apollo配置好的连接字符串
    return config.GetProperty("ConnectionStrings", string.Empty);
```

+ 携程Apollo再.NetCore2.2上的使用
    - 新建一个.NetCore2.2项目
    - 添加Com.Ctrip.Framework.Apollo.Configuration包引用
    - 配置appsettings.json
``` Json
    {
      "apollo": {
        "AppId": "qa.fanyou.001",
        //"MetaServer": "http://192.168.19.48:8081",
        //"Env": "FAT",
        "Meta": {
          "DEV": "http://192.168.19.48:8080",
          "FAT": "http://192.168.19.48:8081",
          "UAT": "http://192.168.19.48:8082"
        }
      }
    }
```
    - 在Program->Main函数中添加
``` csharp
        public static IWebHostBuilder CreateWebHostBuilder(string[] args)
        {
            return WebHost.CreateDefaultBuilder(args)
                 .ConfigureAppConfiguration(builder => builder
                   .AddApollo(builder.Build().GetSection("apollo"))
                   //.AddNamespace("fanyou") // 命名空间
                   .AddDefault())
                   .UseStartup<Startup>();
        }
```
    - 使用方式一(注入IConfiguration)：
``` csharp
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        private readonly IConfiguration configuration;
        public ValuesController(IConfiguration config)
        {
            configuration = config;
        }
        // GET api/values
        [HttpGet]
        public string Get()
        {
            return configuration["Test"].ToString();
        }
    }
```
    - 使用方式二(POCO):
        - 添加Tuhu.Extensions.Configuration.ValueBinder.Json包引用
``` csharp
    // 在apollo中配置文件是Json格式。项目中建立对于的实体
    services.ConfigureJsonValue<ConfigMsg>(Configuration.GetSection("Test"));
    // 获取配置文件信息
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        private IOptions<ConfigMsg> config;

        public ValuesController(IOptions<ConfigMsg> configMsg)2
        {
            config = configMsg;
        }

        [HttpGet]
        public int Get()
        {
            return config.Value.request_timeout;
        }
    }
```

[Github源码地址](https://github.com/WangJunZzz/Apollo.git)