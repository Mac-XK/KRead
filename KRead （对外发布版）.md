# KRead JSON 书源规范 v1
## 对外发布版

> 文档类型：对外发布手册  
> 版本：v1  
> 适用对象：KRead 书源编写者 / 维护者 / 高级用户  
> 目标：让用户只写一个 JSON 文件，就能把普通网页站点或 JSON 接口站点接入 KRead。

---

# 一、这份文档解决什么问题

KRead 的设计目标不是把每一个网站都硬编码进 App，
而是提供一套**规则型 JSON 书源规范**，让用户自己写规则文件完成接入。

也就是说：

- 用户写一个 JSON 文件
- KRead 直接请求目标站点
- KRead 按规则解析：
  - 搜索结果
  - 在线详情
  - 章节目录
  - 正文内容
- 不依赖本地代理
- 不依赖远程中转 API
- 不要求单独部署服务端

你可以把 KRead 书源理解成下面 4 件事的组合：

1. **怎么请求站点**
2. **怎么从返回结果里取字段**
3. **怎么翻分页目录 / 分页正文**
4. **怎么清理广告和脏内容**

---

# 二、适用范围

这份规范最适合这些场景：

- 普通 HTML 小说站
- 搜索页 / 详情页 / 目录页 / 正文页结构稳定的站点
- 轻量分页目录站点
- 轻量分页正文站点
- 返回 JSON 的简单接口站点

这份规范**不保证完全覆盖**下面这些站点：

- 强依赖复杂前端执行的站点
- 需要动态签名的站点
- 强风控 / 强反爬站点
- 必须执行完整浏览器环境 JS 才能拿到正文的站点
- 依赖完整 XPath / 完整 CSS 特性才能解析的站点

如果你的目标站点属于后者，建议：

- 先尝试规则 + 少量 JS
- 不行再增强引擎能力

---

# 三、KRead 书源的基本工作流

一个 KRead 书源通常按下面链路工作：

```text
输入关键词
  ↓
根据 searchUrl 发起搜索请求
  ↓
用 ruleSearch 解析书籍列表
  ↓
点击某本书
  ↓
请求详情页
  ↓
用 ruleBookInfo 解析书名 / 作者 / 简介 / 封面 / 目录地址
  ↓
请求目录页
  ↓
用 ruleToc 解析章节目录
  ↓
点击某一章
  ↓
请求正文页
  ↓
用 ruleContent 提取正文
  ↓
如有正文分页，则继续翻页并拼接
```

所以一个完整书源，最少要覆盖 4 组规则：

- `ruleSearch`
- `ruleBookInfo`
- `ruleToc`
- `ruleContent`

---

# 四、快速开始

如果你第一次写 KRead 书源，建议按这个顺序：

## 第一步：先从模板开始
直接复制模板文件：

- `KRead标准格式模板.json`

再根据目标站点一点点替换。

## 第二步：先打通搜索
先确认下面 3 个字段能跑通：

- `ruleSearch.bookList`
- `ruleSearch.name`
- `ruleSearch.bookUrl`

## 第三步：再做详情
重点确认：

- `ruleBookInfo.name`
- `ruleBookInfo.author`
- `ruleBookInfo.tocUrl`

## 第四步：再做目录
重点确认：

- `ruleToc.chapterList`
- `ruleToc.chapterName`
- `ruleToc.chapterUrl`

## 第五步：最后做正文
重点确认：

- `ruleContent.content`
- `ruleContent.nextContentUrl`
- `ruleContent.replaceRegex`

---

# 五、最小可用 JSON 示例

下面是一个最小能工作的 KRead 书源结构：

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

---

# 六、顶层字段说明

## 6.1 顶层字段总表

| 字段 | 类型 | 必填 | 说明 |
|---|---|---:|---|
| `mode` | String | 否 | 推荐写 `kread`，当前 `rule` 与 `kread` 等价 |
| `bookSourceName` | String | 是 | 书源名称 |
| `bookSourceUrl` | String | 是 | 书源根地址 / 相对路径补全基准 |
| `bookSourceGroup` | String | 否 | 分组显示名称 |
| `enabled` | Bool | 否 | 是否启用，默认 `true` |
| `header` | Object / String | 否 | 公共请求头 |
| `searchUrl` | String | 推荐 | 搜索请求定义 |
| `exploreUrl` | String | 否 | 发现页入口 |
| `ruleSearch` | Object | 推荐 | 搜索解析规则 |
| `ruleBookInfo` | Object | 推荐 | 详情解析规则 |
| `ruleToc` | Object | 推荐 | 目录解析规则 |
| `ruleContent` | Object | 推荐 | 正文解析规则 |

---

## 6.2 `mode`

推荐写法：

```json
"mode": "kread"
```

兼容值：

- `kread`
- `rule`

当前这两个都按规则型书源处理。

---

## 6.3 `bookSourceName`

书源显示名称，必须可读、可区分。

示例：

```json
"bookSourceName": "00小说网"
```

---

## 6.4 `bookSourceUrl`

书源的根地址，用于：

- 补全相对 URL
- 作为站点标识
- 某些规则的 baseURL

示例：

```json
"bookSourceUrl": "https://m.00shu.la/"
```

建议：

- 带协议头
- 尽量写完整域名
- 建议尾部带 `/`

---

## 6.5 `bookSourceGroup`

可用于 UI 上分类：

```json
"bookSourceGroup": "网页书源"
```

---

## 6.6 `enabled`

是否启用该书源：

```json
"enabled": true
```

---

## 6.7 `header`

给请求统一附加请求头。支持：

- 对象
- JSON 字符串

推荐对象形式：

```json
"header": {
  "User-Agent": "Mozilla/5.0",
  "Referer": "https://example.com/"
}
```

---

## 6.8 `searchUrl`

定义搜索请求。

最简单可以直接写 URL：

```json
"searchUrl": "/search?q={{key}}&page={{page}}"
```

更常见的是请求对象字符串，详见后文。

---

## 6.9 `exploreUrl`

用于发现页 / 分类页入口，例如：

```json
"exploreUrl": "玄幻::/list1/{{page}}.html\n武侠::/list2/{{page}}.html"
```

如果当前不用发现页，可以不写。

---

# 七、`searchUrl` 写法详解

KRead 当前支持两种写法。

## 7.1 纯字符串 URL

适合简单 GET 搜索：

```json
"searchUrl": "/search?q={{key}}&page={{page}}"
```

行为：

- 相对地址会自动补全
- 默认 GET

---

## 7.2 请求对象字符串

适合：

- POST 搜索
- 需要请求 body
- 需要单独 timeout
- 需要单独 headers

示例：

```json
"searchUrl": "{\"url\":\"/s.php\",\"method\":\"POST\",\"body\":\"searchkey={{key}}&type=articlename\",\"charset\":\"utf-8\",\"connectTimeout\":60000}"
```

## 7.3 支持字段表

| 字段 | 类型 | 说明 |
|---|---|---|
| `url` | String | 请求地址 |
| `method` | String | `GET` / `POST` |
| `body` | String | POST body |
| `charset` | String | 编码提示 |
| `connectTimeout` | Number | 超时毫秒数 |
| `headers` | Object | 局部请求头 |

---

# 八、`header` 写法详解

## 8.1 推荐写法

```json
"header": {
  "User-Agent": "Mozilla/5.0",
  "Referer": "https://example.com/"
}
```

## 8.2 兼容写法

```json
"header": "{\"User-Agent\":\"Mozilla/5.0\"}"
```

## 8.3 什么时候要写 header

常见场景：

- 搜索必须带 Referer
- 站点会检查 User-Agent
- 站点要求特殊 Header

---

# 九、支持的模板变量

## 9.1 搜索变量

| 变量 | 说明 |
|---|---|
| `{{key}}` | 搜索关键词 |
| `{{keyword}}` | 搜索关键词别名 |
| `{{searchKey}}` | 搜索关键词别名 |
| `{{page}}` | 页码 |

示例：

```json
/search?q={{key}}&page={{page}}
```

---

## 9.2 页码运算变量

| 变量 | 说明 |
|---|---|
| `{{page+1}}` | 当前页 + 1 |
| `{{page-1}}` | 当前页 - 1 |

示例：

```json
/list/{{page+1}}.html
```

---

## 9.3 模板插值 `{{...}}`

你可以在规则里使用模板：

```json
"kind": "{{@css:.tag@text}} {{@css:.status@text}}"
```

也可以做轻量 JS 计算：

```json
"coverUrl": "{{ var href = java.getString(result, 'a@href') || ''; return href + '.jpg'; }}"
```

---

# 十、规则系统总览

KRead 当前规则大体分为 4 类：

1. **HTML / CSS 规则**
2. **JSON / JSONPath 规则**
3. **模板规则 `{{...}}`**
4. **JS 规则 `@js:`**

再配合：

- `## 正则替换 ##`
- `nextTocUrl`
- `nextContentUrl`

来处理目录分页、正文分页和内容清洗。

---

# 十一、HTML / CSS 规则语法

## 11.1 基础写法 `@css:`

最常用形式：

```json
"name": "@css:h3 a@text"
```

含义：

- 先匹配 `h3 a`
- 再取文本

---

## 11.2 常见后缀

| 后缀 | 作用 |
|---|---|
| `@text` | 取文本 |
| `@href` | 取链接 |
| `@src` | 取图片地址 |
| `@content` | 取 meta content |
| `text` | 取当前元素文本 |
| `href` | 取当前元素 href |

示例：

```json
"bookUrl": "@css:h3 a@href"
"coverUrl": "@css:img@src"
"author": "@css:.author@text"
```

---

## 11.3 位置取值

例如：

```json
"name": "a[0]@text"
"author": "a[1]@text"
```

表示：

- 第 1 个 `a`
- 第 2 个 `a`

适合结构固定的结果块。

---

## 11.4 当前较稳定支持的选择器形式

- `@css:.class-name`
- `@css:div > a`
- `@css:h3 a`
- `@css:meta[property='og:image']`
- `@css:meta[property$='book_name']`
- `@css:.box.hot .row > .col-12.col-md-6`

---

# 十二、JSON / JSONPath 规则语法

如果站点返回的是 JSON，可以直接写 JSONPath 风格规则。

## 12.1 搜索列表示例

```json
"bookList": "$.data.list[*]"
```

## 12.2 单字段示例

```json
"name": "$.name"
"author": "$.author"
"bookUrl": "$.bookUrl"
```

## 12.3 目录示例

```json
"chapterList": "$.data.list[*]"
"chapterName": "$.title"
"chapterUrl": "$.url"
```

## 12.4 正文示例

```json
"content": "$.data.content"
```

---

# 十三、模板语法 `{{...}}`

模板用于：

- 拼接字段
- 调用子规则
- 轻量加工结果

## 13.1 拼接多个字段

```json
"kind": "{{@css:.tag@text}} {{@css:.status@text}}"
```

## 13.2 子规则嵌套

```json
"coverUrl": "{{@css:img@src}}"
```

## 13.3 JS 表达式

```json
"coverUrl": "{{ var href = java.getString(result, 'a@href') || ''; return href + '.jpg'; }}"
```

---

# 十四、正则替换语法 `##`

格式：

```text
主规则##正则##替换文本
```

## 14.1 示例：去书名前缀

```json
"name": "@css:h3 a@text##^\\[.*?\\]##"
```

作用：

- `[网游]斗罗大陆`
- 替换后变成：`斗罗大陆`

---

## 14.2 `replaceRegex` 多行规则

正文清洗常用：

```json
"replaceRegex": "第\\(\\d+\\/\\d+\\)页##\nhttps?:\\/\\/[^\\s<>\"]+##"
```

含义：

- 一行一个替换规则
- 每一行都按 `正则##替换` 处理

---

# 十五、JS 规则 `@js:`

适合这些场景：

- 目录下一页地址需要动态推导
- 正文下一页地址需要判断
- 封面地址需要计算
- 某些字段需要脚本拼接

示例：

```json
"nextContentUrl": "@js: var links = JSON.parse(java.getElements(result, 'a') || '[]'); for (var i = 0; i < links.length; i++) { var text = java.getString(links[i], 'text') || ''; if (text.indexOf('下一页') >= 0) return java.getString(links[i], 'href') || ''; } return '';"
```

## 15.1 建议

- 能不用 JS 就先不用
- 能用 CSS/JSONPath 解决就优先用规则
- JS 只补“规则做不到”的地方

---

# 十六、`ruleSearch` 规范

## 16.1 推荐字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `bookList` | 是 | 搜索结果列表规则 |
| `name` | 是 | 书名 |
| `bookUrl` | 是 | 书详情地址 |
| `author` | 否 | 作者 |
| `coverUrl` | 否 | 封面 |
| `kind` | 否 | 分类 / 状态等 |
| `lastChapter` | 否 | 最新章节 |
| `intro` | 否 | 简介 |

## 16.2 HTML 站点示例

```json
"ruleSearch": {
  "bookList": "@css:.book-item",
  "name": "@css:h3 a@text",
  "author": "@css:.author@text",
  "bookUrl": "@css:h3 a@href",
  "coverUrl": "@css:img@src"
}
```

## 16.3 JSON 站点示例

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

# 十七、`ruleBookInfo` 规范

## 17.1 推荐字段

| 字段 | 说明 |
|---|---|
| `name` | 书名 |
| `author` | 作者 |
| `coverUrl` | 封面 |
| `intro` | 简介 |
| `kind` | 分类 / 状态 / 更新时间 |
| `lastChapter` | 最新章节 |
| `tocUrl` | 目录地址 |
| `wordCount` | 字数 |

## 17.2 `tocUrl` 说明

如果目录页和详情页同地址，可以直接写空字符串：

```json
"tocUrl": ""
```

KRead 会自动回退到当前书地址。

---

# 十八、`ruleToc` 规范

## 18.1 必填字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `chapterList` | 是 | 章节列表规则 |
| `chapterName` | 是 | 章节标题 |
| `chapterUrl` | 是 | 章节地址 |
| `isVolume` | 否 | 是否分卷 |
| `nextTocUrl` | 否 | 目录下一页 |

## 18.2 最简单目录例子

```json
"ruleToc": {
  "chapterList": "@css:.chapter-list a",
  "chapterName": "text",
  "chapterUrl": "href",
  "isVolume": "false"
}
```

---

# 十九、`ruleContent` 规范

## 19.1 推荐字段

| 字段 | 必填 | 说明 |
|---|---:|---|
| `content` | 是 | 正文内容 |
| `nextContentUrl` | 否 | 正文下一页 |
| `replaceRegex` | 否 | 正文清洗规则 |
| `subContent` | 否 | 补充正文 |
| `js` | 否 | 正文后处理 JS |

## 19.2 最简单正文例子

```json
"ruleContent": {
  "content": "@css:.content@text"
}
```

---

# 二十、目录分页专题

如果一个站点目录不止一页，必须解决“下一页目录”规则。

## 20.1 直接下一页链接型

```json
"nextTocUrl": "@css:.next@href"
```

## 20.2 下拉页码型

例如 00小说网：

- 页码藏在 `<option value="...">`
- 需要扫 option 里的值

这时通常用 `@js:` 规则最稳定。

## 20.3 数字页码型

例如大唐小说网：

- 页面显示 `1/7`
- 页码 URL 是 `index_2.html`

这类目录要注意：

- 不能只抓首页
- 必须把总页数解析出来
- 再把每一页目录都抓下来

## 20.4 如果没写好会发生什么

常见表现：

- 明明有 600 章，只显示 100 章
- 详情页目录总是第一页
- 目录直接请求失败

---

# 二十一、正文分页专题

很多站点一章正文会拆成多页：

- `chapter.html`
- `chapter_2.html`
- `chapter_3.html`

## 21.1 最基础规则

```json
"nextContentUrl": "@css:.next@href"
```

## 21.2 复杂场景

有些站点“下一页”和“下一章”很像，甚至都叫“下一章”或“下一”。

这时建议用 JS 判断：

- 下一页是否仍然属于同一章
- 如果已经跳到下一章，就不要继续拼接

## 21.3 如果没写好会发生什么

常见表现：

- 正文只显示第一页
- 正文里出现“本章未完，请点击继续阅读”
- 把下一章内容拼进当前章

---

# 二十二、正文清洗建议

正文常见脏内容包括：

- `最新网址：xxx`
- `第(1/3)页`
- `本章未完，请点击继续阅读`
- 广告域名
- 脚本残留字符串
- 空白行过多

## 22.1 常用 `replaceRegex`

```json
"replaceRegex": "第\\(\\d+\\/\\d+\\)页##\nhttps?:\\/\\/[^\\s<>\"]+##\n本章未完.*##"
```

## 22.2 建议策略

- 先保证正文能取出来
- 再一步步清洗
- 不要一开始写太重的替换
- 否则容易把正文也误删掉

---

# 二十三、导入校验规则

KRead 当前导入后会做基础字段检查。

## 23.1 常见错误（通常会导致无法运行）

- 缺少 `ruleSearch.bookList`
- 缺少 `ruleSearch.name`
- 缺少 `ruleSearch.bookUrl`
- 缺少 `ruleToc.chapterList`
- 缺少 `ruleToc.chapterName`
- 缺少 `ruleToc.chapterUrl`
- 缺少 `ruleContent.content`

## 23.2 常见警告（通常会导致不完整）

- `searchUrl` 没有 `{{page}}`
- 没写 `ruleToc.nextTocUrl`
- 没写 `ruleContent.nextContentUrl`

---

# 二十四、推荐调试流程

推荐按下面顺序调试：

## 第一步：搜索
检查：

- 请求地址是否正确
- 返回内容是否正常
- `bookList` 是否匹配到结果
- `name` / `bookUrl` 是否正确

## 第二步：详情
检查：

- 书名是否对
- 作者是否对
- 封面是否出
- `tocUrl` 是否可用

## 第三步：目录
检查：

- 第一页目录是否对
- 分页目录是否完整
- 顺序是否正序 / 倒序

## 第四步：正文
检查：

- 正文是否能取到
- 是否有分页没拼完
- 是否有广告和脏内容

---

# 二十五、常见问题 FAQ

## Q1：搜索有返回，但结果数是 0

常见原因：

- `bookList` 规则不对
- `name` 规则不对
- `bookUrl` 规则为空
- 实际返回不是你以为的 HTML/JSON

---

## Q2：详情正常，但目录为空

常见原因：

- `tocUrl` 错了
- `ruleToc.chapterList` 规则不对
- 目录分页没写 `nextTocUrl`

---

## Q3：只有 100 章，不是完整目录

常见原因：

- 只抓到了目录第一页
- 分页规则没写对
- 页码结构复杂，需要 JS 补逻辑

---

## Q4：正文为空

常见原因：

- `ruleContent.content` 指向错了
- 实际节点不是 `div`，可能是 `article` 或 `section`
- 正文分页没拼

---

## Q5：正文里有脚本残留或广告

解决：

- 补 `replaceRegex`
- 必要时加 JS 后处理

---

## Q6：封面不显示

常见原因：

- `coverUrl` 是相对地址
- 实际封面在 meta 标签里
- 取错属性

---

## Q7：乱码

常见原因：

- 页面编码不是 UTF-8
- 站点历史编码是 GBK / GB18030

KRead 已尽量尝试常见编码，但极端站点仍可能需要额外处理。

---

# 二十六、最佳实践

1. **先做最小可用版本**  
2. **先打通搜索，再做详情，再做目录，最后做正文**  
3. **先轻量清洗，再做复杂清洗**  
4. **复杂逻辑最后再上 JS**  
5. **分页目录和分页正文优先单独验证**  
6. **一个字段只负责一个目标，不要过度堆逻辑**

---

# 二十七、案例一：00小说网

参考文件：

- `00小说网_KRead格式.json`

## 27.1 站点特点

- 搜索是 POST
- 搜索封面要根据书籍 ID 计算
- 目录分页藏在 `<option value>`
- 正文分页需要识别“下一页”
- 正文有大量脏提示需要清洗

## 27.2 适合作为哪些案例

- POST 搜索案例
- 目录分页案例
- 正文分页案例
- 模板 + JS 混合案例

---

# 二十八、案例二：大唐小说网

参考文件：

- `大唐小说网_KRead格式.json`

## 28.1 站点特点

- 搜索结构规整
- 详情 meta 完整
- 目录分页是 `1/7 + index_x.html`
- 有些正文节点是 `.font_max`
- 有些正文节点是 `article.font_max`

## 28.2 适合作为哪些案例

- HTML 搜索案例
- 多页目录案例
- 正文节点变体案例

---

# 二十九、推荐给用户的配套文件

当前建议一并给用户这两个文件：

## 29.1 规范文档

- `KRead JSON书源规范 v1.md`

## 29.2 模板文件

- `KRead标准格式模板.json`

另外也可以把两个示例发给用户参考：

- `00小说网_KRead格式.json`
- `大唐小说网_KRead格式.json`

---

# 三十、当前边界与后续方向

KRead JSON 书源规范 v1 当前已经适合：

- 普通小说站点接入
- 站点结构规则化适配
- 高级用户自定义维护书源

后续可以继续增强的方向：

1. 更完整的规则语法表
2. 更多真实案例
3. 导入后自动检测报告
4. 书源调试向导
5. 面向普通用户的简明版文档

---

# 结语

这份规范的核心目标只有一句话：

> **让用户自己写 JSON，就能把大多数普通站点接进 KRead。**

如果你后续要继续扩展生态，建议同时准备两份文档：

- **完整规范版**：给维护者 / 高级用户
- **快速上手版**：给普通使用者
