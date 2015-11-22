---
layout:     post
title:      "可视化管理各种配置"
subtitle:   "解决.NET项目配置文件种配置多、乱、杂的问题."
date:       2015-10-21 16:00:00
author:     "极品拖拉机"
header-img: "img/post-bg-01.jpg"
tags:
    - .net
---

在开发.NET项目时，特别是web项目，经常通过配置文件设置各种变量，数据库连接字符串等，常用的做法是放到web.config中。但项目大了，配置越来越多，发现web.config配置越来越大，并且有可能几个项目，都有同样的配置，但只能在各自的web.config配置，这样一个容易配错，而且要做重复工作，也有可能这个项目配置了，但另外一个项目漏掉了，发现上线后出错了。

为了偷懒，我采用了一个简单的思路，全局统一管理配置，把配置存储到数据库，然后项目在启动的时候，先从数据库读取到本地内存中，然后每次读取配置，都直接从内存中读取。

## 基础表结构

### 环境表

```csharp

/// <summary>
/// 这个表的意义就类似于sandbox的概念。
/// 每个环境都可以有自己的配置值，如果当前环境没有找到对应的配置，则找到其父节环境的配置。
/// 子环境的配置覆盖父级配置。
/// </summary>
public class Env
{
    public int Id {get; set;}
    public string Name {get; set;}
    public int? ParentId {get; set;}
}
```

### 服务器表

```csharp

/// <summary>
/// 此表记录所有的服务器。
/// </summary>
public class Server
{
    public int Id {get; set;}
    //电脑的名称
    public string ComputerName {get; set;}
    //所属环境
    public int EnvId {get; set;}
    //内网IP
    public string IntranetIP {get; set;}
    //外网IP
    public string IP {get; set;}
    public string Remark {get; set;}
}
```

### 键值配置表

```csharp

  /// <summary>
  /// 对应web.config的AppSetting, 健值配置
  /// </summary>
  public class ConfigValue
  {
      public int Id {get; set;}
      //健名称，一般是namespace.key
      public string Key {get; set;}
      //对应的值
      public string Value {get; set;}
      ///有时候一台服务器，存在好几个项目.
      ///但同一个配置，单独的项目也不一样，就根据web.config的appcode来读取
      public int? AppCode {get; set;}
      //是否启用
      public bool IsActived {get; set;}
      public string Remark {get; set;}
  }
```

### 数据库连接表

```csharp

/// <summary>
/// 对应web.config的ConnectionString, 记录各个数据库的连接字符串
/// </summary>
public class ConnectionString
{
    public int Id {get; set;}
    public string Name {get; set;}
    public string ConnectionValue {get; set;}
    /// <summary>
    /// 有时候一台服务器，存在好几个项目.
    /// 但同一个配置，单独的项目也不一样，就根据web.config的appcode来读取
    /// </summary>
    public int? AppCode {get; set;}
    //是否启用
    public bool IsActived {get; set;}
    public string Remark {get; set;}
}

```

## 程序设计

1. 上面几个表存储在mongodb中(可存储到任意数据库)。
2. 修改每台服务器的 `C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config` 的connectionstrings节点， 增加一个connectionString, name = "config", connectionString为连接到mongodb的字符串。

3.  获取当前服务器所在的环境

```csharp

public class ServerHelper
{
    static ServerHelper()
    {
        ServerName = Environment.MachineName.ToLower();
        SetEnvId();
    }

    private static int? _envId;
    public static int EnvId
    {
        get
        {
            if (!_envId.HasValue)
            {
                SetEnvId();

            }
            return _envId.Value;
        }
    }

    public static int? ParentEnvId
    {
        get; set;
    }

    public static string ServerName
    {
        get; private set;
    }

    public static string ConnectionString
    {
        get; set;
    }

    private static void SetEnvId()
    {
        ConnectionString = ConfigurationManager.ConnectionStrings["config"].ConnectionString;
        if (String.IsNullOrEmpty(ConnectionString))
        {
            throw new Exception("缺少Config基础库的连接字符串");
        }
        var client = new MongoDbBase<Server>(ConnectionString, "config");
        var server = client.GetAll().Single(t => t.ServierName.ToLower() == ServerName);
        _envId = server.EnvId;

        var client2 = new MongoDbBase<Env>(ConnectionString, "config");
        var env = client2.GetAll().Single(t => t.Id == EnvId);
        ParentEnvId = env.ParentId;
    }
}
```

4.  获取ConfigValue的值

```csharp

/// <summary>
/// 获取appsetting配置的值
/// </summary>
public static class ConfigValueHelper
{
    static ConfigValueHelper()
    {
        Init();
    }

    private static Dictionary<string, string> ConfigValues
    {
        set;
        get;
    }

    public static string GetString(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            return String.Empty;
        }

        key = key.ToLower();
        if (!ConfigValues.ContainsKey(key))
        {
            return String.Empty;
        }

        return ConfigValues[key];
    }

    /// <summary>
    /// 获取bool类型的配置值，如果不存在，则返回false
    /// </summary>
    /// <param name="key"></param>
    /// <returns></returns>
    public static bool GetBool(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            return false;
        }

        key = key.ToLower();
        if (!ConfigValues.ContainsKey(key))
        {
            return false;
        }

        bool retValue;
        if (Boolean.TryParse(ConfigValues[key], out retValue))
        {
            return retValue;
        }

        return false;
    }

    /// <summary>
    /// 返回int类型的配置值，如果不存在，则返回0
    /// </summary>
    /// <param name="key"></param>
    /// <returns></returns>
    public static int GetInt(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            return 0;
        }
        key = key.ToLower();
        if (!ConfigValues.ContainsKey(key))
        {
            return 0;
        }

        int retValue;
        if (int.TryParse(ConfigValues[key], out retValue))
        {
            return retValue;
        }

        return 0;
    }

    public static DateTime? GetDateTime(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            return null;
        }
        key = key.ToLower();
        if (!ConfigValues.ContainsKey(key))
        {
            return null;
        }

        DateTime retValue;
        if (DateTime.TryParse(ConfigValues[key], out retValue))
        {
            return retValue;
        }

        return null;
    }

    /// <summary>
    /// 返回double类型的配置值，如果不存在，则返回0.0
    /// </summary>
    /// <param name="key"></param>
    /// <returns></returns>
    public static double GetDouble(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            return 0.0;
        }
        key = key.ToLower();
        if (!ConfigValues.ContainsKey(key))
        {
            return 0.0;
        }

        double retValue;
        if (double.TryParse(ConfigValues[key], out retValue))
        {
            return retValue;
        }

        return 0.0;
    }

    #region Private methods

    private static void Init()
    {
        ConfigValues = new Dictionary<string, string>();
        var client = new MongoDbBase<AppSetting>(ServerHelper.ConnectionString, "config");
        var query = client.GetAll();
        if (ServerHelper.ParentEnvId.HasValue)
        {
            query = query.Where(t => t.EnvId.In(new[] { ServerHelper.EnvId, ServerHelper.ParentEnvId.Value }) && t.IsActive);
        }
        else
        {
            query = query.Where(t => t.EnvId == ServerHelper.EnvId && t.IsActive);
        }

        var list = query.ToList();

        if (!list.Any())
        {
            return;
        }

        var dict = new Dictionary<string, int>();

        var appCode = ConfigHelper.GetAppSettingIntValue("appcode");

        foreach (var item in list)
        {
            if (item.AppCode.HasValue && item.AppCode.Value != appCode)
            {
                continue;
            }

            int level = 1;

            if (item.EnvId == ServerHelper.EnvId
                && (item.AppCode.HasValue && item.AppCode == appCode))
            {
                level = 3;
            }
            else if (item.EnvId == ServerHelper.EnvId)
            {
                level = 2;
            }
            string key = item.Key.ToLower();
            if (dict.ContainsKey(key))
            {
                if (dict[key] > level)
                {
                    continue;
                }
            }

            ConfigValues[key] = item.Value;
            dict[key] = level;
        }
    }

    #endregion

}
```

5. 获取ConnectionString的值

```csharp

public static class ConnectionStringHelper
{
    static ConnectionStringHelper()
    {
        Init();
    }

    /// <summary>
    /// 根据key值获取连接字符串，如果不存在，则throw exception
    /// </summary>
    /// <param name="key"></param>
    /// <returns></returns>
    public static string GetConnectionString(string key)
    {
        if (String.IsNullOrEmpty(key))
        {
            throw new ArgumentNullException("获取连接字符串的key为空");
        }

        key = key.ToLower();

        if (!ConnectionStrings.ContainsKey(key.ToLower()))
        {
            throw new Exception(String.Format("不存在[{0}]的连接字符串", key));
        }

        return ConnectionStrings[key];
    }

    private static Dictionary<string, string> ConnectionStrings
    {
        get;
        set;
    }

    private static void Init()
    {
        ConnectionStrings = new Dictionary<string, string>();
        var client = new MongoDbBase<ConnectionString>(ServerHelper.ConnectionString, "config");
        var query = client.GetAll();
        if (ServerHelper.ParentEnvId.HasValue)
        {
            query = query.Where(t => t.EnvId.In(new[] { ServerHelper.EnvId, ServerHelper.ParentEnvId.Value }) && t.IsActive);
        }
        else
        {
            query = query.Where(t => t.EnvId == ServerHelper.EnvId && t.IsActive);
        }

        var list = query.ToList();

        if (!list.Any())
        {
            return;
        }

        var dict = new Dictionary<string, int>();

        var appCode = ConfigHelper.GetAppSettingIntValue("appcode");

        foreach (var item in list)
        {
            if (item.AppCode.HasValue && item.AppCode.Value != appCode)
            {
                continue;
            }

            int level = 1;

            if (item.EnvId == ServerHelper.EnvId
                && (item.AppCode.HasValue && item.AppCode == appCode))
            {
                level = 3;
            }
            else if (item.EnvId == ServerHelper.EnvId)
            {
                level = 2;
            }

            string key = item.Name.ToLower();

            if (dict.ContainsKey(key))
            {
                if (dict[key] > level)
                {
                    continue;
                }
            }

            ConnectionStrings[key] = item.ConnectionValue;
            dict[key] = level;
        }
    }

}
```

6. 再做一个可视化后台界面，对这几个表进行增、删、改、查。

## 使用
1. 在后台界面的环境页面中，录入你想区分的环境。 一般是录一个最基础的base环境，然后根据需要，再添加其他环境继承这个基础的环境。
2. 在服务器页面中，录入当前所有的服务器，电脑名称必须是各个电脑系统里面对应的名称。 然后各个服务器选择对应的环境，如果没有特殊情况，可以选择基础环境。
3. 在配置页面和数据库连接页面录入对应的配置，并选择相应的环境。
4. 因为数据是在项目启动后放入内存中的，所以中间如果有修改，需要回收IIS POOL或者重启IIS。 可以在之前的后台管理系统中，获取所有服务器的应用程序池，然后通过写代码去进行回收某个IIS POOL。这个可以查询围绕相关资料。
