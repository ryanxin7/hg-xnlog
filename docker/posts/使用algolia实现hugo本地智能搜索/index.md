# 使用Algolia实现Hugo本地智能搜索









## 1.简介



Algolia是一家提供搜索即服务的技术公司，帮助开发者为他们的应用程序或网站构建高速、精准的搜索功能。



![algoliaPricing](http://cdn1.ryanxin.live/image-20230815102738980.png)



免费的计划每个月可以查询10000次，对于个人站点也是够用了。



## 2. 配置Algolia



Algolia的配置步骤通常包括以下几个主要方面：创建账户、导入数据、设置索引、集成到应用程序中以及调整搜索体验。



### 2.1 创建账号

访问Algolia的官方网站（https://www.algolia.com/），注册一个账户并登录。Google和Github账号可以直接登录。



![algolia-login](http://cdn1.ryanxin.live/image-20230815104008760.png)



### 2.2 创建应用程序

选择免费套餐

![image-20230815105056951](http://cdn1.ryanxin.live/image-20230815105056951.png)



选择位置

![image-20230815105414862](http://cdn1.ryanxin.live/image-20230815105414862.png)





### 2.3 创建索引

在Algolia控制台中，创建一个新的索引。索引是存储数据的容器，用于执行搜索操作。

![image-20230815110845372](http://cdn1.ryanxin.live/image-20230815110845372.png)





### 2.4 导入数据

您可以使用Algolia的API、SDK或工具来导入数据，确保数据字段与搜索需求匹配。

![image-20230815140356504](http://cdn1.ryanxin.live/image-20230815140356504.png)



### 2.5 配置搜索属性

定义索引字段，为每个索引定义需要搜索和显示的字段。也可以设置字段的搜索权重和过滤条件。

![image-20230815104404886](http://cdn1.ryanxin.live/image-20230815104404886.png)



### 2.6 配置排名规则

调整搜索结果的排名规则，以确保最相关的结果显示在顶部

![image-20230815111353856](http://cdn1.ryanxin.live/image-20230815111353856.png)



### 2.6 测试搜索

![image-20230815143410285](http://cdn1.ryanxin.live/image-20230815143410285.png)

通过Algolia UI界面，测试搜索效果。

![image-20230815143126719](http://cdn1.ryanxin.live/image-20230815143126719.png)





## 3.配置hugo主题使用Algolia搜索



### 3.1 获取Algolia Key

点击右上角头像—>setting—> API Keys

![image-20230815141249131](http://cdn1.ryanxin.live/image-20230815141249131.png)



### 3.2 配置主题相关参数



为了生成搜索功能所需要的 `index.json`, 请在你的 [网站配置](https://hugoloveit.com/zh-cn/theme-documentation-basics/#site-configuration) 中添加 `JSON` 输出文件类型到 `outputs` 部分的 `home` 字段中。





在 Hugo 中，可以通过修改网站配置文件（通常是 `config.toml`、`config.yaml` 或 `config.json`）来指定不同部分的输出格式。这样，可以在生成网站页面时，为不同的页面部分选择不同的输出格式，包括 JSON。

1. 打开网站配置文件。
2. 寻找名为 `outputs` 的部分，如果没有则创建它。
3. 在 `outputs` 部分中，可以为不同的页面部分指定输出格式。

```yaml
# Options to make hugo output files
home = ["HTML", "RSS", "JSON", "BaiduUrls"]
page = ["HTML", "MarkDown"]
section = ["HTML", "RSS"]
taxonomy = ["HTML", "RSS"]
taxonomyTerm = ["HTML"]
```



**保存网站配置文件**

在运行 Hugo 构建命令（例如 `hugo` 或 `hugo build`）以生成网站时，Hugo 将生成 `index.json` 文件作为上述部分的输出。





**outputFormats**

在 Algolia 中，`outputFormats` 是用于控制返回搜索结果的格式的设置选项之一。

通过调整 `outputFormats`，可以决定搜索结果以何种格式返回应用程序。

```yaml
[MarkDown]
  mediaType = "text/markdown"
  isPlainText = true
  isHTML = false
# Options to make output baidu_urls.txt file
[BaiduUrls]
  baseName = "baidu_urls"
  mediaType = "text/plain"
  isPlainText = true
  isHTML = false
```





这段代码的作用是为 Hugo 网站生成 Algolia 搜索所需的 JSON 索引文件，以便在搜索时快速检索和展示内容。

```json
{{- if .Site.Params.search -}}
    {{- $index := slice -}}
    {{- $pages := where .Site.RegularPages "Params.password" "eq" nil -}}
    {{- if .Site.Params.page.hiddenFromSearch -}}
        {{- $pages = where $pages "Params.hiddenfromsearch" false -}}
    {{- else -}}
        {{- $pages = where $pages "Params.hiddenfromsearch" "!=" true -}}
    {{- end -}}
    {{- range $pages -}}
        {{- $uri := .RelPermalink -}}
        {{- if $.Site.Params.search.absoluteURL -}}
            {{- $uri = .Permalink -}}
        {{- end -}}
        {{- $meta := dict "uri" $uri "title" .Title "tags" .Params.tags "categories" .Params.categories -}}
        {{- $meta = $.Site.Params.dateFormat | default "2006-01-02" | .PublishDate.Format | dict "date" | merge $meta -}}
        {{- with .Description -}}
            {{- $index = $index | append (dict "content" . "objectID" $uri | merge $meta) -}}
        {{- end -}}
        {{- $params := .Params | merge $.Site.Params.page -}}
        {{/* Extended Markdown syntax */}}
        {{- $content := dict "Content" .Content "Ruby" $params.ruby "Fraction" $params.fraction "Fontawesome" $params.fontawesome | partial "function/content.html" -}}
        {{/* Remove line number for code */}}
        {{- $content = $content | replaceRE `<span class="lnt?"> *\d*\n?</span>` "" -}}
        {{- range $i, $contenti := split $content "<h2 id=" -}}
            {{- if gt $i 0 -}}
                {{- $contenti = printf "<h2 id=%v" $contenti -}}
            {{- end -}}
            {{- range $j, $contentj := split $contenti "<h3 id=" -}}
                {{- if gt $j 0 -}}
                    {{- $contentj = printf "<h3 id=%v" $contentj -}}
                {{- end -}}
                {{/* Plainify, unescape and remove (\n, \t) */}}
                {{- $contentj = $contentj | plainify | htmlUnescape | replaceRE `[\n\t ]+` " " -}}
                {{- if gt $.Site.Params.search.contentLength 0 -}}
                    {{- $contentj = substr $contentj 0 $.Site.Params.search.contentLength -}}
                {{- end -}}
                {{- if $contentj | and (ne $contentj " ") -}}
                    {{- $one := printf "%v:%v:%v" $uri $i $j | dict "content" $contentj "objectID" | merge $meta -}}
                    {{- $index = $index | append $one -}}
                {{- end -}}
            {{- end -}}
        {{- end -}}
    {{- end -}}

    {{- $index | jsonify | safeJS -}}
{{- end -}}

```





### 3.3 配置hugo主题应用Algolia

```yaml
  #  搜索配置
  [params.search]
    enable = true
    # 搜索引擎的类型 ["lunr", "algolia", "fuse"]
    type = "algolia"
    # 文章内容最长索引长度
    contentLength = 4000
    # 搜索框的占位提示语
    placeholder = ""
    #  最大结果数目
    maxResultLength = 10
    #  结果内容片段长度
    snippetLength = 50
    #  搜索结果中高亮部分的 HTML 标签
    highlightTag = "em"
    #  是否在搜索索引中使用基于 baseURL 的绝对路径
    absoluteURL = false
    [params.search.algolia]
      index = "xx-log"
      appID = "SFSFN4DBN1"
      searchKey = "bd48328538sdb2f38b20753c17c60ba92f"
```







## 4.测试效果

![image-20230815143700010](http://cdn1.ryanxin.live/image-20230815143700010.png)

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/%E4%BD%BF%E7%94%A8algolia%E5%AE%9E%E7%8E%B0hugo%E6%9C%AC%E5%9C%B0%E6%99%BA%E8%83%BD%E6%90%9C%E7%B4%A2/  

