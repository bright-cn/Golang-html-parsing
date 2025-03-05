# 如何使用 Golang 解析 HTML？

[![宣传图](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/)

在 Go 中掌握 HTML 解析技巧，可使用 [Node Parser](https://www.npmjs.com/package/node-html-parser)、[Tokenizer](https://github.com/greim/html-tokenizer) 以及第三方工具，例如 Goquery、Colly 和 [Bright Data 的 Web Scraper](https://www.bright.cn/products/web-scraper)，以高效完成 [网页抓取](https://github.com/bright-cn/Awesome-Web-Scraping)。

## 前提条件

在开始之前，最好先对 Go（Golang）和网页抓取概念有基本了解。请确保你的计算机上已安装 Go。然后，新建一个项目文件夹，并使用以下命令进行初始化：

```bash
mkdir goparser
cd goparser
go mod init goparser
```

用以下 Go 程序测试你的环境：

```go
package main

import "fmt"

func main() {
    fmt.Println("你好，世界！")
}
```

运行该文件：

```bash
go run main.go
```

安装依赖包：

```bash
go get golang.org/x/net/html
```

## 使用 Node Parser 提取数据

Node Parser 允许你遍历 HTML 页面对应的 DOM（文档对象模型）。这对提取特定元素（例如引用内容和作者）非常有用。下面是一个使用 Node Parser 完成此任务的示例：

```go
package main

import (
    "fmt"
    "net/http"
    "golang.org/x/net/html"
)

func main() {
    resp, _ := http.Get("http://quotes.toscrape.com")
    defer resp.Body.Close()
    doc, _ := html.Parse(resp.Body)

    var processNode func(*html.Node)
    processNode = func(n *html.Node) {
        if n.Type == html.ElementNode && n.Data == "span" {
            for _, a := range n.Attr {
                if a.Key == "class" && a.Val == "text" {
                    fmt.Println("引用:", n.FirstChild.Data)
                }
            }
        }
        if n.Type == html.ElementNode && n.Data == "small" {
            for _, a := range n.Attr {
                if a.Key == "class" && a.Val == "author" {
                    fmt.Println("作者:", n.FirstChild.Data)
                }
            }
        }
        for c := n.FirstChild; c != nil; c = c.NextSibling {
            processNode(c)
        }
    }
    processNode(doc)
}
```

## 使用 Tokenizer 提取数据

Tokenizer 将 HTML 页面作为一个令牌流进行处理，这些令牌代表 HTML 的各个组成部分（如标签、属性和文本）。对于大型页面，这种方式更加节省内存，但需要更多的手动处理。以下示例演示了如何使用 Tokenizer 提取引用和作者信息：

```go
package main

import (
    "fmt"
    "net/http"
    "strings"
    "golang.org/x/net/html"
)

func main() {
    resp, _ := http.Get("http://quotes.toscrape.com")
    defer resp.Body.Close()
    tokenizer := html.NewTokenizer(resp.Body)

    inQuote := false
    inAuthor := false

    for {
        tt := tokenizer.Next()
        switch tt {
        case html.ErrorToken:
            return
        case html.StartTagToken:
            t := tokenizer.Token()
            if t.Data == "span" {
                for _, a := range t.Attr {
                    if a.Key == "class" && a.Val == "text" {
                        inQuote = true
                    }
                }
            }
            if t.Data == "small" {
                for _, a := range t.Attr {
                    if a.Key == "class" && a.Val == "author" {
                        inAuthor = true
                    }
                }
            }
        case html.TextToken:
            if inQuote {
                fmt.Println("引用:", strings.TrimSpace(tokenizer.Token().Data))
                inQuote = false
            }
            if inAuthor {
                fmt.Println("作者:", strings.TrimSpace(tokenizer.Token().Data))
                inAuthor = false
            }
        }
    }
}
```

## 第三方替代方案

- **Goquery**：Go 版 jQuery，可用于 DOM 遍历及使用 CSS 选择器。
- **htmlquery**：与 Goquery 类似，但使用 XPath 选择器。
- **Colly**：Go 的全功能网页抓取框架。
- **Bright Data Web Scraper**：一种 API 服务，可抓取网页并以 JSON 格式返回数据。

## 总结

现在你已经了解了如何使用 Go 解析 HTML。Node Parser 适用于完整页面遍历，Tokenizer 更便于处理大型页面中特定数据提取。想要更多功能，可以尝试第三方工具。也欢迎阅读我们其他的抓取教程：

- [**Amazon**](https://github.com/bright-cn/LinkedIn-Scraper)
- [**LinkedIn**](https://github.com/bright-cn/LinkedIn-Scraper)
- [**Google Maps**](https://github.com/bright-cn/Google-Maps-Scraper)
- [**Google News**](https://github.com/bright-cn/Google-News-Scraper)
