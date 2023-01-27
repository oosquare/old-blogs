---
title: "Python 网络爬虫获取 SYZOJ AC 代码"
subtitle: ""
date: 2022-04-18T15:23:12+08:00
draft: false
author: "ctj12461"
authorLink: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC 4.0 许可证发布"
comment: false
weight: 0

tags:
  - Python 3
  - 网络爬虫
  - aiohttp
  - BeautifulSoup
  - 异步
categories:
  - Tech

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: true
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

写本文之前，我共有两百多道题目在校内 `OJ` 上通过，由于某些原因，想要保存这些代码，于是想到使用 `Python` 实现自动爬取代码。同时考虑到效率问题，决定使用 `aiohttp` 编写一个高性能异步爬虫。

## 实现目标分析
这个爬虫需要能够爬取所有的已通过题目的列表，并继续爬取这些已通过题目的代码，随后保存到文件中。

校内 `OJ` 基于 `SYZOJ` 搭建，该 `OJ` 的项目地址为 <https://github.com/syzoj/syzoj>，故接下来的代码都是基于其实现的。

由于这个 `OJ` 的外网域名的带宽很小，一个网页最坏情况下需要花费 `2～3` 秒的时间，所以必须采用异步实现。这里就使用 `aiohttp` 了。

## 获取 Cookie
由于直接从浏览器里获取的 `Cookie` 无法正常使用，传给服务器无法识别，猜测是编码问题，所以使用在爬虫运行时即时获取 `Cookie` 的办法。

分析 `SYZOJ` 的登陆[页面源码](https://github.com/syzoj/syzoj/blob/master/views/login.ejs)，找到如下代码片段：

```js
function login() {
    password = md5($("#password").val() + "syzoj2_xxx");
    $("#login").addClass("loading");
    $.ajax({
        url: "/api/login",
        type: 'POST',
        data: {
            "username": $("#username").val(),
            "password": password
        },
        async: true,
        success: function(data) {
            error_code = data.error_code;
            switch (error_code) {
                case 1001:
                    show_error("用户不存在");
                    break;
                case 1002:
                    show_error("密码错误");
                    break;
                case 1003:
                    show_error("您尚未设置密码，请通过下方「找回密码」来设置您的密码。");
                    break;
                case 1:
                    success(data.session_id);
                    return;
                default:
                    show_error("未知错误");
                    break;
            }
            $("#login").text("登录");
            $("#login").removeClass("loading");
        },
        error:  function(XMLHttpRequest, textStatus, errorThrown) {
            alert(XMLHttpRequest.responseText);
            show_error("未知错误");
            $("#login").text("登录");
        }
    });
}
```

可以发现，`SYZOJ` 通过 `/api/login` 这个 `Web API` 发送请求获取 `Cookie`，使用 `POST` 方法，数据为 `username` 和 `password`，分别为用户名和密码加上 `syzoj2_xxx` 这个 `salt` 的 `MD5`，所以可以使用以下 `Python` 代码获取 `Cookie`

```python
def get_password_md5(password: str) -> str:
    password += "syzoj2_xxx"
    return hashlib.md5(password.encode("utf-8")).hexdigest()


async def init_cookie(username: str, password: str) -> typing.Dict[str, str]:
    async with aiohttp.ClientSession() as session:
        # host 为 OJ 域名
        async with session.post(
            host + "/api/login",
            data={
                "username": username,
                "password": get_password_md5(password),
            },
        ) as response:
            return response.cookies
```

随后就直接将 `Cookie` 传给 `ClientSession`：

```python
async def main():
    cookie = await init_cookie(username, password)

    async with aiohttp.ClientSession(cookies=cookie) as session:
        if len(cookie) == 0:
            print("Failed to login.")
            return

        print("Succeeded to get cookie.")
        
        # ...
```

## 获取通过题目列表
这里通过抓取题目页面来获取题目列表，如果一道题目已经通过，题目前面就会有一个绿色的勾，指向 `AC` 记录。

{{< image src="problem-page.png" caption="题目列表" >}}

可以发现题目列表被放在整个页面的唯一一个 `<table>` 标签中，每一道题目被放在表格中的每一行 `<tr>` 中，具体结构如下：

```html
<table class="ui very basic center aligned table">
    <thead>
    <!-- ... -->
    </thead>
    <tbody>
        <tr style="height: 44px; ">
            <!-- 如果没有提交记录，则接下来这个 td 为空 -->
            <td>
                <!-- 通过记录 -->
                <a href="/submission/{id}">
                    <!-- AC 状态 -->
                    <span class="status accepted">
                        <i class="checkmark icon"></i>
                    </span>
                </a>
            </td>
            <!-- 题目编号 -->
            <td><b>1</b></td>
            <td class="left aligned">
                <!-- 题目名称和链接 -->
                <a style="vertical-align: middle; " href="/problem/1">A + B Problem</a>
            </td>
            <!-- ... -->
        </tr>
    </tbody>
</table>
```

所以可以先用以下代码获取题目页面 `HTML`：

```python
async def get_problem_page_content(
    session: aiohttp.ClientSession, page_num: int
) -> str:
    async with session.get(
        host + "/problems", params={"page": str(page_num)}
    ) as response:
        return await response.text()
```

然后使用 `BeautifulSoup` 处理 `HTML`：

```python
def check_problem_accepted(tag: bs4.Tag) -> typing.Tuple[str, str] | None:
    if tag.find("span", class_="status accepted") == None:
        return None

    problem_id = tag.b.string.strip()
    # 获得保存文件名
    problem_name = (
        tag.find("a", style="vertical-align: middle; ")
        .contents[0] # 如果某些题目要特殊权限，则会在后面在显示一个 <span>, 不可以直接用 tag.string
        .strip()
        .replace("/", "-") # / 不可以作为文件名的一部分
    )
    submission_url = tag.a["href"].strip()
    return f"{problem_id}-{problem_name}", host + submission_url


def get_accepted_problems(content: str) -> typing.Dict[str, str]:
    soup = bs4.BeautifulSoup(content, "html.parser")
    problem_table = soup.find("tbody")
    accepted_problems_dict = {}
    # 处理每一行，如果是通过题目则加入字典
    for row in problem_table.find_all("tr", recursive=False):
        res = check_problem_accepted(row)

        if res == None:
            continue

        accepted_problems_dict[res[0]] = res[1]

    return accepted_problems_dict
```

异步爬取所有页面的题目：

```python
async def process_problem_page(
    session: aiohttp.ClientSession, page_num: int
) -> typing.Dict[str, str]:
    content = await get_problem_page_content(session, page_num)
    return get_accepted_problems(content)


async def get_problems(
    session: aiohttp.ClientSession, page_nums: typing.Iterable[int]
) -> typing.Dict[str, str]:
    problems_dict = {}
    result = await asyncio.gather(
        *[process_problem_page(session, num) for num in page_nums]
    )

    # 合并每个页面的题目
    for page in result:
        problems_dict.update(page)

    return problems_dict
```

## 获取通过代码
有了提交记录 `URL`，就可以爬取代码了。

但是代码并不是直接放在 `HTML` 里传回来的，而是放在 `Javascript` 中再由浏览器处理得到的，具体可以查看网页源代码，发现以下两行：

```js
const unformattedCode = "\u003Cspan class=\"pl-cp\"\u003E#include\u003C...";
const formattedCode = "\u003Cspan class=\"pl-cp\"\u003E#include\u003C...";
```

因为我的代码已经格式化过了，而且 `OJ` 默认的格式化风格和我不一样，所以我就抓取 `unformattedCode` 里的就可以了：

```python 
async def get_submission_content(session: aiohttp.ClientSession, url: str) -> str:
    async with session.get(url) as response:
        return await response.text()


async def get_code_html(session: aiohttp.ClientSession, url: str) -> str:
    content = await get_submission_content(session, url)
    # 截取 unformattedCode 里的内容
    start = content.find('const unformattedCode = "') + len('const unformattedCode = "')
    end = content.find("\";", start, content.find('const formattedCode = "', start))
    # 把 \u003C 这样的 Unicode 转义字符转换为正常的字符
    return re.sub(
        r"(\\u[0-9a-fA-F])",
        lambda match: match.group(1).encode("utf-8").decode("unicode-escape"),
        content[start:end],
        0,
        re.M,
    )
```

上面的代码判断会返回一个 `HTML` 字符串，含有非常多的 `<span>` 和 `&amp;` 这样的东西，所以接下来的就是正常的 `HTML` 的处理了，把标签去掉，在把一些特殊符号转换回来就可以了：

```python
async def get_code(session: aiohttp.ClientSession, url: str) -> str:
    code = await get_code_html(session, url)
    code = re.sub("</?[^>]+>", "", code, 0, re.M) # 去掉 HTML 标签
    code = re.sub("&lt;", "<", code, 0, re.M)
    code = re.sub("&gt;", ">", code, 0, re.M)
    code = re.sub("&amp;", "&", code, 0, re.M)
    code = re.sub("&quot;", '"', code, 0, re.M)
    code = re.sub("&apos;", "'", code, 0, re.M)
    code = re.sub("&#39;", "'", code, 0, re.M)
    return code


async def scrape_code(session: aiohttp.ClientSession, problem: str, url: str) -> None:
    try:
        code = await get_code(session, url)
        # 写入代码到文件
        with open(problem + ".cpp", "w") as writer:
            writer.write(code)
    except Exception as e:
        print(f"Failed to scrape {problem} from {url}: {e}")
```

## 完善主程序
```python
async def main() -> None:
    with open("passwd.txt") as f:
        username = f.readline().strip()
        password = f.readline().strip()

    cookie = await init_cookie(username, password)

    async with aiohttp.ClientSession(cookies=cookie) as session:
        if len(cookie) == 0:
            print("Failed to login.")
            return

        print("Succeeded to get cookie.")
        # 指定爬取的题目页面编号
        problems = await get_problems(session, range(1, 15))
        print(f"Succeeded to get the problem list. {len(problems)} problems in total.")
        # 需要将这些协程加入 Eventloop 实现异步爬取
        await asyncio.gather(
            *[scrape_code(session, problem, url) for problem, url in problems.items()]
        )

        print("All tasks have finished.")


if __name__ == "__main__":
    asyncio.run(main())
```

完整代码见 [GitHub](https://github.com/ctj12461/syzoj-crawler)。

## 测试
使用同学的帐号进行测试，爬取 `465` 道题目仅需 `27` 秒：

{{< image src="crawler-output.png" caption="爬虫输出结果" >}}

如果使用同步爬虫，则需要 `1` 分 `18` 秒，效率提升还是很明显的。

## 性能提升
如果网络延迟较低，字符串相关的处理可能会成为性能瓶颈，可以考虑使用多线程/多进程进行优化。具体也没有我也尝试过，因为大多数情况下网络延迟还是比较严重的，所以在其他方面做优化不一定有用。大家可以自己实现。