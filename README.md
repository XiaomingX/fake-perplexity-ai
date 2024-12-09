# Perplexity.ai

这是一个 **API 封装器和账号生成工具**，它可以通过免费使用的 Copilot 账户，提供简单的 API 接口。

---

## 📘 主要功能
该模块使用 [emailnator](https://emailnator.com/) 来生成新账户。每次创建新账户，您将获得 5 个免费的 Copilot 额度。通过这个工具，您可以批量创建 Gmail（实际上是 `@googlemail.com`）账户，从而**获得无限的 Copilot 额度！**

---

## ⚙️ 使用前提

<details>
<summary>点击展开详细依赖项</summary>
<br>

需要安装以下依赖包：

- [requests](https://pypi.org/project/requests/)
- [requests-toolbelt](https://pypi.org/project/requests-toolbelt/)
- [websocket-client](https://pypi.org/project/websocket-client/)
- [aiohttp](https://pypi.org/project/aiohttp/)（如果使用异步版本）

### 安装命令
```sh
pip3 install -r requirements.txt
```

或者使用一条命令安装：
```sh
pip3 install requests && pip3 install websocket-client && pip3 install requests-toolbelt
```

如果使用异步版本，则需要额外安装：
```sh
pip3 install aiohttp
```
</details>

---

## 🚀 使用方法

### 1️⃣ 获取 Cookie
因为 [Perplexity.ai](https://perplexity.ai/) 和 [emailnator](https://emailnator.com/) 受 Cloudflare 保护，所以必须手动获取 Cookies。  
这些 Cookies 是**临时的**，需要定期更新。

#### 如何获取 Cookies
1. 访问 [emailnator](https://emailnator.com/) 网站。
2. 按 `F12` 或 `Ctrl + Shift + I` 打开开发者工具。
3. 转到 “Network” 选项卡，勾选 **Preserve log** 选项。
4. 点击“Go!”按钮，右键点击网络请求列表中的“message-list”，选择 **Copy as cURL (bash)**。
5. 将 cURL 代码粘贴到 [curlconverter](https://curlconverter.com/python/)，生成 Python 中的 Headers 和 Cookies 字典。

6. 同样的方式，访问 [Perplexity.ai](https://perplexity.ai/)，点击“Sign Up”，输入一个随机的电子邮件地址并点击“Continue with Email”，右键“email”请求，选择 **Copy as cURL (bash)**，再将其转换为 Python 代码，提取出 Headers 和 Cookies。

---

### 2️⃣ 代码示例

```python
import perplexity

perplexity_headers = { 
    <您的 Perplexity 头信息>
}

perplexity_cookies = { 
    <您的 Perplexity Cookies>
}

emailnator_headers = { 
    <您的 Emailnator 头信息>
}

emailnator_cookies = { 
    <您的 Emailnator Cookies>
}

# 如果您要使用自己的账户，请手动登录并将 header/cookie 信息复制进来
perplexity_cli = perplexity.Client(perplexity_headers, perplexity_cookies, own=False)

# 生成一个新账户，每次都会有 5 个新的 Copilot 额度
perplexity_cli.create_account(emailnator_headers, emailnator_cookies)

# 发起搜索请求
response = perplexity_cli.search(
    '您想查询的内容', 
    mode='copilot', 
    focus='internet', 
    files=[(open('示例.txt', 'rb').read(), 'txt'), (open('示例.pdf', 'rb').read(), 'pdf')], 
    ai_model='default', 
    solvers={
        'text': lambda query: input(f'{query}: '),
        'checkbox': lambda desc, opts: [int(input('--> '))]
    }
)

print(response)
```

---

## 📘 详细说明

### 1️⃣ Copilot 搜索参数
- **mode**: 可选模式，值为 `['concise', 'copilot']`
- **focus**: 关注领域，值为 `['internet', 'scholar', 'writing', 'youtube', 'reddit']`
- **files**: 文件列表，格式为 [(数据, 文件名)]
- **ai_model**: AI 模型，值为 `['default', 'gpt-4', 'claude-2.1']`
- **solvers**: Copilot 使用的解答器，包含 `text` 和 `checkbox` 两种类型

---

## 🔁 使用“追问”功能

**示例代码：**
```python
response = perplexity_cli.search(
    '您的问题', 
    mode='copilot', 
    focus='internet', 
    solvers={
        'text': lambda query: input(f'{query}: '),
        'checkbox': lambda desc, opts: [int(input('--> '))]
    }
)

# 继续追问
follow_up_response = perplexity_cli.search(
    '继续的追问内容', 
    mode='copilot', 
    focus='internet', 
    follow_up=response, 
    solvers={
        'text': lambda query: input(f'{query}: '),
        'checkbox': lambda desc, opts: [int(input('--> '))]
    }
)

print(follow_up_response)
```

---

## 📘 实验室（Labs）功能

通过实验室功能，可以访问更多模型，比如 `llama`、`mistral` 和 `Liquid LFM 40B`。

**实验室示例代码：**
```python
import perplexity

labs_headers = { 
    <您的实验室 Headers> 
}

labs_cookies = { 
    <您的实验室 Cookies> 
}

labs_cli = perplexity.LabsClient(labs_headers, labs_cookies)

response = labs_cli.ask('你好', model='llama-3.1-sonar-large-128k-online')
print(response)
```

---

## 📘 账户池（Pool）功能

如果您不想每次都手动创建账户，可以使用 Pool 来**自动后台创建账户**。  

### Pool 功能示例
```python
import perplexity

pool = perplexity.Pool(
    perplexity_headers, 
    perplexity_cookies, 
    emailnator_headers, 
    emailnator_cookies, 
    copilots=10, 
    file_uploads=10, 
    threads=1
)

response = pool.search(
    '查询内容', 
    mode='copilot', 
    focus='internet', 
    solvers={
        'text': lambda query: input(f'{query}: '),
        'checkbox': lambda desc, opts: [int(input('--> '))]
    }
)

print(response)
```

---

## 📘 异步版本

如果想提高运行效率，可以使用 **异步版本**。

### 异步示例代码
```python
import perplexity_async
import asyncio

async def test():
    perplexity_cli = await perplexity_async.Client(perplexity_headers, perplexity_cookies, own=False)
    await perplexity_cli.create_account(emailnator_headers, emailnator_cookies)

    response = await perplexity_cli.search(
        '您的查询内容', 
        mode='copilot', 
        focus='internet', 
        solvers={
            'text': lambda query: input(f'{query}: '),
            'checkbox': lambda desc, opts: [int(input('--> '))]
        }
    )
    print(response)

    # 继续追问
    follow_up_response = await perplexity_cli.search(
        '继续的追问内容', 
        mode='copilot', 
        focus='internet', 
        follow_up=response, 
        solvers={
            'text': lambda query: input(f'{query}: '),
            'checkbox': lambda desc, opts: [int(input('--> '))]
        }
    )

    print(follow_up_response)

asyncio.run(test())
```
