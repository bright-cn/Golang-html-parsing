# 如何使用 Golang 解析 HTML？

[![推广](https://github.com/bright-cn/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://www.bright.cn/) 

学习如何在 Go 中使用[节点解析器](https://www.npmjs.com/package/node-html-parser)、[Tokenizer](https://github.com/greim/html-tokenizer) 以及 Goquery、Colly 和 [Bright Data 网页抓取工具](https://www.bright.cn/products/web-scraper)等第三方工具解析 HTML，实现高效的[网页爬虫](https://github.com/bright-cn/Awesome-Web-Scraping)。

## 前提条件

开始之前，建议你先了解 Go（Golang）和网页爬虫的基本概念。请确保你的计算机上已安装 Go。然后创建一个新的项目文件夹，并使用以下命令进行初始化：

```bash
mkdir goparser
cd goparser
go mod init goparser
```

使用以下命令测试你的环境：

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

运行文件：

```bash
go run main.go
```

安装依赖项：

```bash
go get golang.org/x/net/html
```

## 使用节点解析器提取数据

节点解析器可用于遍历 HTML 页面的 DOM（文档对象模型），适合提取引用和作者等特定元素。以下示例展示了如何使用节点解析器实现这一目的：

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
                    fmt.Println("Quote:", n.FirstChild.Data)
                }
            }
        }
        if n.Type == html.ElementNode && n.Data == "small" {
            for _, a := range n.Attr {
                if a.Key == "class" && a.Val == "author" {
                    fmt.Println("Author:", n.FirstChild.Data)
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

Tokenizer 会将 HTML 页面作为 token 流进行处理，每个 token 代表 HTML 的一个独立组成部分（例如标签、属性和文本）。这种方法在处理大型页面时更加节省内存，但需要进行更多手动处理。以下示例展示了如何使用 Tokenizer 提取引用和作者：

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
                fmt.Println("Quote:", strings.TrimSpace(tokenizer.Token().Data))
                inQuote = false
            }
            if inAuthor {
                fmt.Println("Author:", strings.TrimSpace(tokenizer.Token().Data))
                inAuthor = false
            }
        }
    }
}
```

## 第三方替代方案

- **Goquery**：jQuery 的 Go 替代方案，支持 DOM 遍历和 CSS 选择器。
- **htmlquery**：与 Goquery 类似，但使用 XPath 选择器。
- **Colly**：功能完整的 Go 网页爬虫框架。
- **Bright Data 网页抓取工具**：用于抓取页面并以 JSON 格式返回数据的 API 服务。

## 总结

现在，你已经掌握了如何使用 Go 解析 HTML。你可以使用节点解析器遍历整个页面，也可以使用 Tokenizer 解析所需数据。如需更多功能，不妨探索第三方工具。你还可以查看我们的其他抓取指南：

- [**Amazon**](https://github.com/bright-cn/Amazon-Scraper)
- [**LinkedIn**](https://github.com/bright-cn/LinkedIn-Scraper)
- [**Google 地图**](https://github.com/bright-cn/Google-Maps-Scraper)
- [**Google 新闻**](https://github.com/bright-cn/Google-News-Scraper)
