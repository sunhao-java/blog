title: IDE安装wakatime插件及介绍
tags: [wakatime]
date: 2025-09-03 09:27
categories: 中间件
---

> 1. Wakapi私服地址：https://xxx.your-domain.com
> 2. Wakapi私服api地址：https://xxx.your-domain.com/api
> 3. 注册好后，点击右上角头像->Show API Key获得API Key值

# 获得插件

1. 访问：https://wakatime.com/plugins
2. 输入您使用的IDE名称，回车，点击图标进入详情页面

## 【示例】IntelliJ IDEA

1. 在 IntelliJ IDEA 中，打开插件设置菜单。
   1. Mac：Preferences → Plugins
   2. Win：File → Settings… → Plugins
2. 搜索 wakatime。
3. 点击绿色的 Install 按钮。
4. 重新启动 IntelliJ IDEA。
5. 输入您的 API Key，然后点击 Save。如果未提示输入 API Key，请进入 IntelliJ IDEA 内的 Tools → WakaTime Settings 菜单进行设置。
6. 像平常一样使用 IntelliJ IDEA，您的编码活动将会显示在 WakaTime 仪表板上。

## 【示例】VS Code/Trae

1. 按 F1 或 CMD + Shift + P，然后输入 install。选择扩展：安装扩展。

   ![](https://imgs.lodsve.com:9000/images/2025/09/03/92d8cc194c9a.png)

2. 输入 wakatime 然后按回车。

![](https://imgs.lodsve.com:9000/images/2025/09/03/4b881572c35f.png)

3. 输入你的API密钥，然后按回车。（如果没有提示，请按F1或CMD + Shift + P，然后输入 WakaTime API Key。）

![](https://imgs.lodsve.com:9000/images/2025/09/03/66f656c288af.png)

4. 像平常一样使用 VS Code/Trae，你的编码活动将会显示在你的 WakaTime 仪表板上。

# 配置插件

正常来讲，第一次配置时，各个IDE都没有提示配置`API_URL`，可在配置`API_KEY`之后，在用户目录下找到`.wakatime.cfg`文件进行手动编辑

1. 找到`.wakatime.cfg`文件
   1. Windows：`C:\Users\{Your UserName}\.wakatime.cfg`
   2. Mac: `~/.wakatime.cfg`
2. 在`settings`块下，加入`api_url`，如下：

```YAML
[settings]
api_url = https://wakapi.url.mo.cn/api
api_key = Your API Key In Wakaapi
debug = false
status_bar_enabled = true
proxy = 
```

3. 重启各个IDE

# Wakapi私服介绍

## Dashboard（仪表盘）

![](https://imgs.lodsve.com:9000/images/2025/09/03/ed273189cda0.png)

## Leaderboard（排行榜）

1. Wakapi 的排行榜显示了此服务器上最活跃的用户排名，前提是这些用户**选择加入**公开排行榜。统计数据至少每 12 小时更新一次，基于用户在预设时间段内的总编码时间。要参与排行榜，请登录后前往**`Settings 🠒 Permissions`**，并**启用**排行榜功能。

   ![](https://imgs.lodsve.com:9000/images/2025/09/03/f4d78aed4686.png)

2. 生成数据时间：**每天下午6点半**，统计全量数据

3. 总时长排序

   ![](https://imgs.lodsve.com:9000/images/2025/09/03/a6510c27bb2f.png)

4. 按语言排序

   ![](https://imgs.lodsve.com:9000/images/2025/09/03/1d0c6add0079.png)