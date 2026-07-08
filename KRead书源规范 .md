# KRead JSON 书源规范 v1（完整文档）

> 版本：v1  
> 适用范围：KRead 当前规则型 JSON 书源  
> 文档目标：让用户只写一个 JSON 文件，就能把普通网站或 JSON 接口站点适配到 KRead，而不需要本地代理、远程中转服务或额外 API。

---

# 目录

- [1. 这份规范解决什么问题](#1-这份规范解决什么问题)
- [2. KRead 书源的工作方式](#2-kread-书源的工作方式)
- [3. 最小可用书源 JSON](#3-最小可用书源-json)
- [4. 顶层字段完整说明](#4-顶层字段完整说明)
- [5. searchUrl 写法详解](#5-searchurl-写法详解)
- [6. header 写法详解](#6-header-写法详解)
- [7. 支持的模板变量](#7-支持的模板变量)
- [8. 规则系统总览](#8-规则系统总览)
- [9. HTML / CSS 规则语法](#9-html--css-规则语法)
- [10. JSON / JSONPath 规则语法](#10-json--jsonpath-规则语法)
- [11. 模板语法 `{{...}}`](#11-模板语法-)
- [12. 正则替换语法 `##`](#12-正则替换语法-)
- [13. JS 规则 `@js:`](#13-js-规则-js)
- [14. ruleSearch 规范](#14-rulesearch-规范)
- [15. ruleBookInfo 规范](#15-rulebookinfo-规范)
- [16. ruleToc 规范](#16-ruletoc-规范)
- [17. ruleContent 规范](#17-rulecontent-规范)
- [18. 目录分页专题](#18-目录分页专题)
- [19. 正文分页专题](#19-正文分页专题)
- [20. 站点正文清洗建议](#20-站点正文清洗建议)
- [21. 导入校验规则](#21-导入校验规则)
- [22. 调试与排错流程](#22-调试与排错流程)
- [23. 常见问题 FAQ](#23-常见问题-faq)
- [24. 最佳实践](#24-最佳实践)
- [25. 案例一：00小说网](#25-案例一00小说网)
- [26. 案例二：大唐小说网](#26-案例二大唐小说网)
- [27. 模板文件说明](#27-模板文件说明)
- [28. 当前边界与限制](#28-当前边界与限制)

---

# 1. 这份规范解决什么问题

KRead 的目标不是把每个网站硬编码进 App，而是提供一套**规则型 JSON 书源协议**：

- 用户自己写 JSON
- KRead 直接请求目标网站
- KRead 按 JSON 里的规则解析：
  - 搜索
  - 在线详情
  - 章节目录
  - 正文内容
- 不依赖本地代理
- 不依赖远程中转 API
- 不要求单独部署服务端

你可以把它理解为：

> **KRead 书源 = 请求定义 + 解析规则 + 分页规则 + 内容清洗规则**

---

# 2. KRead 书源的工作方式

一个 KRead 书源，通常会经过下面这条链路：

```text
用户输入关键词
  ↓
KRead 根据 searchUrl 发起请求
  ↓
用 ruleSearch 解析搜索结果
  ↓
点击某一本书
  ↓
KRead 请求详情页
  ↓
用 ruleBookInfo 解析书名/作者/简介/封面等
  ↓
请求目录页
  ↓
用 ruleToc 解析章节目录
  ↓
点击章节
  ↓
请求正文页
  ↓
用 ruleContent 提取正文
  ↓
如有正文分页，则继续翻页并拼接
```

---

# 3. 最小可用书源 JSON

下面是一个最小能跑通的 HTML 站点模板：

```json
{
  "mode": "kread",
  "bookSourceName": "示例书源",
  "bookSourceUrl": "https://example.com",
  "bookSourceGroup": "小说",
  "enabled": true,
  "header": {
    "User-Agent": "Mozilla/5.0"
  },
  "searchUrl": "{\"url\":\"/search?q={{key}}&page={{page}}\",\"method\":\"GET\",\"charset\":\"utf-8\",\"connectTimeout\":30000}",
  "ruleSearch": {
    "bookList": "@css:.book-item",
    "name": "@css:h3 a@text",
    "author": "@css:.author@text",
    "bookUrl": "@css:h3 a@href",
    "coverUrl": "@css:img@src",
    "kind": "@css:.meta@text",
    "lastChapter": "@css:.last@text",
    "intro": "@css:.intro@text"
  },
  "ruleBookInfo": {
    "name": "@css:meta[property$='book_name']@content",
    "author": "@css:meta[property$='author']@content",
    "coverUrl": "@css:meta[property='og:image']@content",
    "intro": "@css:meta[property$='description']@content",
    "kind": "@css:meta[property$='category']@content",
    "lastChapter": "@css:meta[property$='latest_chapter_name']@content",
    "tocUrl": ""
  },
  "ruleToc": {
    "chapterList": "@css:.chapter-list a",
    "chapterName": "text",
    "chapterUrl": "href",
    "isVolume": "false",
    "nextTocUrl": ""
  },
  "ruleContent": {
    "content": "@css:.content@text",
    "nextContentUrl": "",
    "replaceRegex": ""
  }
}
```

如果一个站点是 JSON 接口站点，也可以写成规则型 JSON，只不过 `ruleSearch` / `ruleBookInfo` / `ruleToc` / `ruleContent` 里的规则改成 JSONPath 即可。

---

# 4. 顶层字段完整说明

## 4.1 `mode`

类型：`String`  
推荐值：

- `kread`
- `rule`

当前两者等价，都会按规则型书源处理。

推荐统一写：

```json
"mode": "kread"
```

---

## 4.2 `bookSourceName`

类型：`String`  
是否必填：**必填**

作用：书源显示名称。

示例：

```json
"bookSourceName": "00小说网"
```

---

## 4.3 `bookSourceUrl`

类型：`String`  
是否必填：**必填**

作用：书源根地址，用来：

- 作为相对路径补全基准
- 作为站点标识

示例：

```json
"bookSourceUrl": "https://m.00shu.la/"
```

建议：

- 写完整协议头 `https://`
- 建议以 `/` 结尾，但不是强制

---

## 4.4 `bookSourceGroup`

类型：`String`  
是否必填：否

作用：用于书源分组显示。

示例：

```json
"bookSourceGroup": "网页书源"
```

---

## 4.5 `enabled`

类型：`Bool`  
是否必填：否  
默认值：`true`

作用：是否启用书源。

---

## 4.6 `header`

类型：

- `Object`
- 或 JSON 字符串

作用：给请求附加请求头。

示例：

```json
"header": {
  "User-Agent": "Mozilla/5.0",
  "Referer": "https://example.com/"
}
```

或者：

```json
"header": "{\"User-Agent\":\"Mozilla/5.0\"}"
```

---

## 4.7 `searchUrl`

类型：`String`  
是否必填：对“可搜索书源”来说，**强烈建议必填**

作用：定义搜索请求。

支持两种形式：

- 纯 URL
- 带 method/body/timeout 的 JSON 字符串

详见后文《searchUrl 写法详解》。 

---

## 4.8 `exploreUrl`

类型：`String`  
是否必填：否

作用：发现页 / 分类入口。

常见格式：

```json
"exploreUrl": "玄幻::/list1/{{page}}.html\n武侠::/list2/{{page}}.html"
```

如果你的 App 当前没有完整用到发现页，可以先不写。

---

## 4.9 `ruleSearch`

类型：`Object`  
作用：搜索结果解析规则。

---

## 4.10 `ruleBookInfo`

类型：`Object`  
作用：书籍详情解析规则。

---

## 4.11 `ruleToc`

类型：`Object`  
作用：章节目录解析规则。

---

## 4.12 `ruleContent`

类型：`Object`  
作用：正文解析规则。

---

# 5. searchUrl 写法详解

KRead 支持两种写法。

## 5.1 直接写 URL

适合最简单的 GET 请求：

```json
"searchUrl": "/search?q={{key}}&page={{page}}"
```

KRead 会：

- 自动用 `bookSourceUrl` 补全相对地址
- 自动发起 GET 请求

---

## 5.2 写成请求对象

适合：

- POST 搜索
- 要带 body
- 要指定超时
- 要单独设 headers

示例：

```json
"searchUrl": "{\"url\":\"/s.php\",\"method\":\"POST\",\"body\":\"searchkey={{key}}&type=articlename\",\"charset\":\"utf-8\",\"connectTimeout\":60000}"
```

### 支持字段

#### `url`
请求地址，可相对可绝对。

#### `method`
支持：

- `GET`
- `POST`

#### `body`
POST body 文本。

#### `charset`
常见：

- `utf-8`
- `utf-16`
- 站点特殊编码时可配，但能否完全生效取决于站点返回

#### `connectTimeout`
单位：毫秒。

例如：

```json
"connectTimeout": 60000
```

表示 60 秒。

#### `headers`
可选的请求头对象。

---

# 6. header 写法详解

## 推荐写法

```json
"header": {
  "User-Agent": "Mozilla/5.0",
  "Referer": "https://example.com/"
}
```

## 字符串写法

```json
"header": "{\"User-Agent\":\"Mozilla/5.0\"}"
```

## 建议

优先写对象格式，可读性更高。

---

# 7. 支持的模板变量

## 7.1 搜索变量

- `{{key}}`
- `{{keyword}}`
- `{{searchKey}}`
- `{{page}}`

通常搜索时最常用的是：

```json
/search?q={{key}}&page={{page}}
```

---

## 7.2 分页变量

- `{{page+1}}`
- `{{page-1}}`

示例：

```json
/list/{{page+1}}.html
```

---

## 7.3 模板变量用法

你可以在规则里嵌入：

```json
"coverUrl": "{{@css:img@src}}"
```

也可以写简单 JS：

```json
"coverUrl": "{{ var href = java.getString(result, 'a@href'); return href + '.jpg'; }}"
```

---

# 8. 规则系统总览

KRead 规则本质上分 4 类：

1. HTML / CSS 规则
2. JSON / JSONPath 规则
3. 模板规则 `{{...}}`
4. JS 规则 `@js:`

再配合：

- `## 正则替换 ##`
- `nextTocUrl`
- `nextContentUrl`

来解决目录/正文分页和内容清洗问题。

---

# 9. HTML / CSS 规则语法

## 9.1 基础 `@css:`

最常用写法：

```json
"name": "@css:h3 a@text"
```

含义：

- 先选中 `h3 a`
- 再取文本

---

## 9.2 常用取值后缀

### `@text`
取文本

```json
"author": "@css:.author@text"
```

### `@href`
取链接

```json
"bookUrl": "@css:h3 a@href"
```

### `@src`
取图片地址

```json
"coverUrl": "@css:img@src"
```

### `@content`
取 meta content

```json
"intro": "@css:meta[property$='description']@content"
```

### `text`
当规则对象已经是元素本身时，可以直接取文字：

```json
"chapterName": "text"
```

### `href`
当规则对象已经是链接本身时：

```json
"chapterUrl": "href"
```

---

## 9.3 位置取值

例如：

```json
"name": "a[0]@text"
"author": "a[1]@text"
```

表示：

- 第一个 a
- 第二个 a

这在搜索结果里很常见。

---

## 9.4 常用选择器写法

支持较稳定的例子：

- `@css:.class-name`
- `@css:div > a`
- `@css:h3 a`
- `@css:meta[property='og:image']`
- `@css:meta[property$='book_name']`
- `@css:.box.hot .row > .col-12.col-md-6`

---

# 10. JSON / JSONPath 规则语法

如果站点返回的是 JSON，可以直接写 JSONPath 风格规则。

## 10.1 搜索列表示例

```json
"bookList": "$.data.list[*]"
```

## 10.2 单字段示例

```json
"name": "$.name"
"author": "$.author"
"bookUrl": "$.bookUrl"
```

## 10.3 目录示例

```json
"chapterList": "$.data.list[*]"
"chapterName": "$.title"
"chapterUrl": "$.url"
```

## 10.4 正文示例

```json
"content": "$.data.content"
```

---

# 11. 模板语法 `{{...}}`

模板语法常用于拼字段、拼地址、做轻量加工。

## 11.1 引用子规则

```json
"kind": "{{@css:.tag@text}} {{@css:.status@text}}"
```

## 11.2 轻量 JS

```json
"coverUrl": "{{ var href = java.getString(result, 'a@href') || ''; return href + '.jpg'; }}"
```

## 11.3 多字段拼接

```json
"kind": "{{@css:meta[property$='category']@content}} {{@css:meta[property$='status']@content}}"
```

---

# 12. 正则替换语法 `##`

格式：

```text
主规则##正则##替换文本
```

## 12.1 例子：去标签前缀

```json
"name": "@css:h3 a@text##^\\[.*?\\]##"
```

作用：

- 原文本：`[网游]斗罗大陆`
- 替换后：`斗罗大陆`

---

## 12.2 例子：正文清洗

```json
"replaceRegex": "第\\(\\d+\\/\\d+\\)页##\nhttps?:\\/\\/[^\\s<>\"]+##"
```

这里是多行规则，每一行一个替换。

---

# 13. JS 规则 `@js:`

适合在这些情况使用：

- 目录分页不规则
- 正文下一页要判断
- 封面地址需要计算
- 某个字段要动态拼装

示例：

```json
"nextContentUrl": "@js: var links = JSON.parse(java.getElements(result, 'a') || '[]'); for (var i = 0; i < links.length; i++) { var text = java.getString(links[i], 'text') || ''; if (text.indexOf('下一页') >= 0) return java.getString(links[i], 'href') || ''; } return '';"
```

### 注意

JS 能力有，但不是完整浏览器环境。  
复杂站点不要一开始就上大量 JS，优先把规则写清楚。

---

# 14. ruleSearch 规范

## 必填字段

- `bookList`
- `name`
- `bookUrl`

## 推荐字段

- `author`
- `coverUrl`
- `kind`
- `lastChapter`
- `intro`

## 典型 HTML 站点例子

```json
"ruleSearch": {
  "bookList": "@css:.book-item",
  "name": "@css:h3 a@text",
  "author": "@css:.author@text",
  "bookUrl": "@css:h3 a@href",
  "coverUrl": "@css:img@src"
}
```

## 典型 JSON 站点例子

```json
"ruleSearch": {
  "bookList": "$.data.list[*]",
  "name": "$.name",
  "author": "$.author",
  "bookUrl": "$.bookUrl",
  "coverUrl": "$.coverUrl"
}
```

---

# 15. ruleBookInfo 规范

## 推荐字段

- `name`
- `author`
- `coverUrl`
- `intro`
- `kind`
- `lastChapter`
- `tocUrl`
- `wordCount`

## 注意

如果详情页目录地址跟详情页地址相同，`tocUrl` 可以留空字符串：

```json
"tocUrl": ""
```

KRead 会回退到当前书链接。

---

# 16. ruleToc 规范

## 必填字段

- `chapterList`
- `chapterName`
- `chapterUrl`

## 可选字段

- `isVolume`
- `nextTocUrl`

## 最简单目录例子

```json
"ruleToc": {
  "chapterList": "@css:.chapter-list a",
  "chapterName": "text",
  "chapterUrl": "href",
  "isVolume": "false"
}
```

---

# 17. ruleContent 规范

## 必填字段

- `content`

## 推荐字段

- `nextContentUrl`
- `replaceRegex`
- `subContent`
- `js`

## 示例

```json
"ruleContent": {
  "content": "@css:.font_max@text",
  "nextContentUrl": "@js: ...",
  "replaceRegex": "第\\(\\d+\\/\\d+\\)页##"
}
```

---

# 18. 目录分页专题

如果一个站点目录不止一页，必须解决“下一页目录”的问题。

## 18.1 最简单：直接有下一页链接

```json
"nextTocUrl": "@css:.next@href"
```

## 18.2 下拉页码型

例如 00小说网：

- 页码藏在 `<option value="...">`

这时可用 `@js:` 扫描 option 并返回下一页地址。

## 18.3 页码数字型

例如大唐小说网：

- 会出现 `1/7`
- 会有 `index_2.html`
- `index_3.html`

这类站点通常需要 JS 或专门规则识别最大页码。

## 18.4 注意事项

如果你没有写 `nextTocUrl`：

- KRead 通常只会抓到第一页目录
- 这也是“明明有 600 章但只显示 100 章”的常见原因

---

# 19. 正文分页专题

很多站点一章正文会拆成：

- `chapter.html`
- `chapter_2.html`
- `chapter_3.html`

这时必须写：

- `ruleContent.nextContentUrl`

## 19.1 最简单：直接有下一页按钮

```json
"nextContentUrl": "@css:.next@href"
```

## 19.2 需要判断是不是“下一页”而不是“下一章”

有些站按钮文案都叫“下一章”或“下一”，这时最好用 JS 判断：

- 当前 URL 是否还是同一章分页
- 如果跳到下一章，就不要继续拼接

---

# 20. 站点正文清洗建议

正文里常见脏内容：

- `最新网址：xxx`
- `第(1/3)页`
- 广告域名
- `本章未完，请点击继续阅读`
- 脚本残留字符串

## 20.1 典型 replaceRegex

```json
"replaceRegex": "第\\(\\d+\\/\\d+\\)页##\nhttps?:\\/\\/[^\\s<>\"]+##\n本章未完.*##"
```

## 20.2 建议

先抓原始正文，再逐步清洗。  
不要一开始把替换规则写太重，否则容易把正文也误删掉。

---

# 21. 导入校验规则

KRead 当前导入后会做基础检查。

## 21.1 常见错误

- 缺少 `ruleSearch.bookList`
- 缺少 `ruleSearch.name`
- 缺少 `ruleSearch.bookUrl`
- 缺少 `ruleToc.chapterList`
- 缺少 `ruleToc.chapterName`
- 缺少 `ruleToc.chapterUrl`
- 缺少 `ruleContent.content`

## 21.2 常见警告

- `searchUrl` 没有 `{{page}}`
- 没写 `ruleToc.nextTocUrl`
- 没写 `ruleContent.nextContentUrl`

---

# 22. 调试与排错流程

推荐按下面顺序调：

## 第一步：先跑搜索
检查：

- 请求地址对不对
- 返回有内容吗
- `bookList` 能匹配到几项
- `name` / `bookUrl` 是否正确

## 第二步：再跑详情
检查：

- 书名
- 作者
- 封面
- 简介
- `tocUrl`

## 第三步：再跑目录
检查：

- 第一页目录是否正常
- 多页目录是否完整
- 是否正序 / 倒序

## 第四步：最后跑正文
检查：

- 正文是否抓到
- 有没有脏广告
- 有没有分页正文没拼全

---

# 23. 常见问题 FAQ

## Q1：搜索有请求，但结果是 0

常见原因：

- `bookList` 规则不对
- `name` 规则不对
- `bookUrl` 规则为空
- 实际返回不是你以为的 HTML/JSON

---

## Q2：详情有书名，但目录为空

常见原因：

- `tocUrl` 错了
- `ruleToc.chapterList` 规则不对
- 目录是分页的，但没写 `nextTocUrl`

---

## Q3：只有 100 章，不是完整目录

常见原因：

- 目录分页没有写好
- `nextTocUrl` 只能翻到第一页
- 站点页码结构复杂，需要 JS 补逻辑

---

## Q4：正文为空

常见原因：

- `ruleContent.content` 指向错了
- 实际节点不是 `div`，而是 `article` / `section`
- 正文被分页拆开，但没写 `nextContentUrl`

---

## Q5：正文里有广告和脏提示

解决方式：

- 补 `replaceRegex`
- 必要时用 JS 做更细清洗

---

## Q6：封面不显示

常见原因：

- `coverUrl` 是相对地址
- 规则取错了属性
- 站点实际上封面在 meta 标签里

---

## Q7：乱码

常见原因：

- 返回内容不是 utf-8
- 站点历史编码是 gbk / gb18030

KRead 会尽量尝试常见编码，但极端站点仍可能要单独处理。

---

# 24. 最佳实践

1. **先写最简单能跑的版本**
2. **先打通搜索，再打通详情，再做目录，最后处理正文**
3. **正文清洗先轻后重**
4. **分页目录 / 分页正文优先单独验证**
5. **复杂逻辑再上 JS，不要一开始全靠 JS**
6. **一个字段只做一件事**，不要在一个规则里塞太多副作用

---

# 25. 案例一：00小说网

文件参考：

- `00小说网_KRead格式.json`

这个站点的特点：

- 搜索是 POST
- 搜索结果封面要根据 `bookUrl` 里的数字计算
- 目录分页藏在 `<option value="...">`
- 正文分页要识别“下一页”
- 正文里有：
  - `最新网址`
  - `第x/x页`
  - `本章未完，请点击继续阅读`
  - 脚本残留

它非常适合作为：

- **分页目录案例**
- **分页正文案例**
- **模板 + JS 混合案例**

---

# 26. 案例二：大唐小说网

文件参考：

- `大唐小说网_KRead格式.json`

这个站点的特点：

- 搜索是 GET
- 搜索结果结构规整
- 详情 meta 信息完整
- 目录分页是 `1/7 + index_x.html` 类型
- 正文节点有些页是 `.font_max`
- 有些页正文在 `article.font_max`

它非常适合作为：

- **HTML 搜索案例**
- **多页目录案例**
- **正文节点变体案例**

---

# 27. 模板文件说明

你当前可以给用户发的模板文件是：

- `KRead标准格式模板.json`

以及可以给用户参考的真实案例：

- `00小说网_KRead格式.json`
- `大唐小说网_KRead格式.json`

建议用户学习顺序：

1. 先看模板
2. 再看 00小说网
3. 再看 大唐小说网

---

# 28. 当前边界与限制

这套规范适合：

- 普通 HTML 小说站
- 结构稳定的 JSON 接口站
- 轻度分页目录 / 正文站

这套规范目前**不保证完整支持**：

- 全量 XPath
- 全量 CSS 语法
- 强依赖浏览器环境的复杂 JS
- 动态签名
- 复杂风控
- 必须执行完整前端逻辑才能拿到正文的站点

遇到这种站点时，可以：

1. 先尝试用规则 + 少量 JS 解决
2. 如果还是不行，再考虑增强引擎能力

---

# 结语

KRead JSON 书源规范 v1 的核心目标只有一句话：

> **让用户自己写 JSON，就能把大多数普通网站接进 KRead。**

如果后续你要继续扩展规范，建议优先补：

- 更完整的字段字典
- 更多站点案例
- 用户调试向导
- 导入后自动检测报告
