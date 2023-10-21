# 五花八门的 Markdown Admonition







## 语法



`admonition` shortcode 支持 **12** 种 帮助你在页面中插入提示的横幅。

*支持 Markdown 或者 HTML 格式。*

{{< admonition >}}
一个 **注意** 横幅
{{< /admonition >}}

{{< admonition abstract >}}
一个 **摘要** 横幅
{{< /admonition >}}

{{< admonition info >}}
一个 **信息** 横幅
{{< /admonition >}}

{{< admonition tip >}}
一个 **技巧** 横幅
{{< /admonition >}}

{{< admonition success >}}
一个 **成功** 横幅
{{< /admonition >}}

{{< admonition question >}}
一个 **问题** 横幅
{{< /admonition >}}

{{< admonition warning >}}
一个 **警告** 横幅
{{< /admonition >}}

{{< admonition failure >}}
一个 **失败** 横幅
{{< /admonition >}}

{{< admonition danger >}}
一个 **危险** 横幅
{{< /admonition >}}

{{< admonition bug >}}
一个 **Bug** 横幅
{{< /admonition >}}

{{< admonition example >}}
一个 **示例** 横幅
{{< /admonition >}}

{{< admonition quote >}}
一个 **引用** 横幅
{{< /admonition >}}

`admonition` shortcode 有以下命名参数：

* **type** *[必需]*（**第一个**位置参数）

    `admonition` 横幅的类型，默认值是 `note`。

* **title** *[可选]*（**第二个**位置参数）

    `admonition` 横幅的标题，默认值是 **type** 参数的值。（支持 markdown）{{< version 0.2.14 changed >}}

* **open** *[可选]*（**第三个**位置参数） {{< version 0.2.0 changed >}}

    横幅内容是否默认展开，默认值是 `true`。

一个 `admonition` 示例：
```
{{</* admonition type=tip title="This is a tip" open=false */>}}
一个 **技巧** 横幅
{{</* /admonition */>}}
或者
{{</* admonition tip "This is a tip" false */>}}
一个 **技巧** 横幅
{{</* /admonition */>}}
```

呈现的输出效果如下：

{{< admonition tip "This is a tip" false >}}
一个 **技巧** 横幅
{{< /admonition >}}

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/blog/%E4%BA%94%E8%8A%B1%E5%85%AB%E9%97%A8%E7%9A%84-markdown-admonitions/  

