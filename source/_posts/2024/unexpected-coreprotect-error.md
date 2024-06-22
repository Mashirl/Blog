---
title: 解决自构建 CoreProtect 无法启动的问题
updated: 2024-06-23 00:00:00
date: 2024-06-23 00:00:00
tags: 
- [Minecraft]
- [Minecraft服务器]
---

我最近在尝试Minecraft 1.21服务端的兼容性，[CoreProtect](https://coreprotect.net/download[CoreProtect | Patreon](https://coreprotect.net)一直是一个很重要的插件，可以帮助玩家和管理员查询方块的更改记录，以便监管和阻止恶意破坏行为。

CoreProtect的最新构建是需要付费的，但它的源代码在[它的 GitHub 仓库](https://github.com/PlayPro/CoreProtect/)上是开放的，所以我们理所应当地想到了手动构建。然而，在手动构建之后，却出现了意料之外的报错。

<!--more-->

```log latest.log
[00:08:20] [Server thread/INFO]: [CoreProtect] Enabling CoreProtect v22.4
[00:08:21] [Server thread/INFO]: [CoreProtect] Invalid plugin version (branch has not been set).
[00:08:21] [Server thread/INFO]: [CoreProtect] To continue, set project branch to "development".
[00:08:21] [Server thread/INFO]: [CoreProtect] Running development code may result in data corruption.
[00:08:21] [Server thread/INFO]: [CoreProtect] CoreProtect was unable to start.
[00:08:21] [Server thread/INFO]: [CoreProtect] Disabling CoreProtect v22.4
[00:08:21] [Server thread/INFO]: [CoreProtect] Finishing up data logging. Please wait...
[00:08:21] [Server thread/INFO]: [CoreProtect] Success! Disabled CoreProtect v22.4
```

日志显示：`To continue, set project branch to "development"`。由此得知，需要把branch设置为`development`才可以正常运行。

那么，如何设置branch呢？

我们回到CoreProtect的源码中继续寻找，找到了下面这部分代码

```java net.coreprotect.config.ConfigHandler.
package net.coreprotect.config;

import java.io.File;
import java.io.RandomAccessFile;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Locale;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

import org.bukkit.Bukkit;
import org.bukkit.World;
import org.bukkit.entity.Player;
import org.bukkit.inventory.ItemStack;
import org.bukkit.plugin.Plugin;
import org.bukkit.plugin.PluginManager;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

import net.coreprotect.bukkit.BukkitAdapter;
import net.coreprotect.consumer.Queue;
import net.coreprotect.database.Database;
import net.coreprotect.database.statement.UserStatement;
import net.coreprotect.language.Phrase;
import net.coreprotect.listener.ListenerHandler;
import net.coreprotect.model.BlockGroup;
import net.coreprotect.paper.PaperAdapter;
import net.coreprotect.patch.Patch;
import net.coreprotect.spigot.SpigotAdapter;
import net.coreprotect.utility.Chat;
import net.coreprotect.utility.Color;
import net.coreprotect.utility.Util;

public class ConfigHandler extends Queue {
    public static int SERVER_VERSION = 0;
    public static final int EDITION_VERSION = 2;
    public static final String EDITION_BRANCH = Util.getBranch();
    public static final String EDITION_NAME = Util.getPluginName();
    public static final String JAVA_VERSION = "11.0";
    public static final String SPIGOT_VERSION = "1.15";
    public static String path = "plugins/CoreProtect/";
    public static String sqlite = "database.db";
    public static String host = "127.0.0.1";
    public static int port = 3306;
    public static String database = "database";
    public static String username = "root";
    public static String password = "";
    public static String prefix = "co_";
    public static int maximumPoolSize = 10;
```

这部分代码中的这一部分

```java
public class ConfigHandler extends Queue {
    public static int SERVER_VERSION = 0;
    public static final int EDITION_VERSION = 2;
    public static final String EDITION_BRANCH = Util.getBranch();
    public static final String EDITION_NAME = Util.getPluginName();
```

我们可以看到，这里有一个常量叫做`EDITION_BRANCH`，是通过`Util.getBranch`方法获取的。所以我们可以把这个常量直接设置为`development`，以解决这个问题。

```java
    public static final String EDITION_BRANCH = "development";
```

设置完毕之后，插件便可以正常运行了。
