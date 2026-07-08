# KRead / 读一刀

一个面向 **iOS 原生阅读体验** 的小说阅读器项目，支持：

- **本地阅读**：TXT / EPUB 导入、解析、阅读
- **在线阅读**：搜索、详情、目录、正文
- **KRead JSON 书源规范**：用户可通过 JSON 自助适配站点
- **阅读器增强**：滚动 / 分页 / 仿真翻页、标注、书签、历史、统计、签到等

---

## 项目定位

KRead 不是一个只做 UI 的壳项目，也不是只做本地文件阅读的简化 Demo。

它的目标是做一个 **“本地阅读 + 在线书源 + 原生阅读体验”** 的 iOS 阅读器，核心方向包括：

1. **本地 TXT / EPUB 阅读体验**
2. **在线书籍搜索 → 详情 → 目录 → 正文 主链路**
3. **KRead JSON 书源规范**
4. **原生阅读器交互与数据能力**

简单说：

> 这是一个以 iOS 原生体验为核心、同时兼顾本地阅读和在线书源扩展能力的阅读器项目。

---

## 当前能力总览

### 一、本地阅读

- 支持导入 `.txt`
- 支持导入 `.epub`
- 支持 TXT 自动识别章节
- TXT 无章节时整本作为一章
- 支持 UTF-8 / UTF-16 / GB18030 等常见编码尝试
- EPUB 基础解析：`container.xml / OPF / manifest / spine`
- EPUB 封面提取
- EPUB 懒加载正文

### 二、在线阅读

- 搜索页
- 在线详情页
- 在线目录页
- 在线正文阅读
- 目录分页处理
- 正文分页拼接
- 规则型 JSON 书源导入
- 在线书源调试页
- 批量检测书源
- 删除无效书源
- 支持在线书源 URL 导入

### 三、阅读器

- 滚动阅读
- 分页阅读
- 仿真翻页
- 字体大小调整
- 行距 / 段距 / 页边距等配置
- 自定义字体导入
- 背景图
- 音量键翻页
- TTS 朗读
- 长按选中文本
- 高亮 / 笔记 / 批注
- 书签

### 四、书架与数据

- 书架管理
- 搜索 / 筛选 / 排序
- 置顶
- 批量选择 / 批量删除
- 阅读进度保存
- 阅读历史
- 阅读统计
- 每日签到
- 备份 / 恢复

---

## 技术栈

### 客户端

- **Swift**
- **SwiftUI**
- **Combine**
- **UIKit Bridge**（如文件选择、部分系统能力封装）

### 规则与解析

- **QuickJS**
- 自研 **Legado / KRead 规则解析能力**
- 自研 HTML / JSON / 部分 JS 处理链路

### 数据存储

- 本地 JSON 持久化
- App 沙盒文件存储
- 章节缓存

---

## KRead JSON 书源规范

项目已经支持 **KRead 规则型 JSON 书源**。

这意味着：

- 用户可以通过一个 JSON 文件自定义书源
- KRead 直接请求目标站点
- KRead 按规则解析搜索 / 详情 / 目录 / 正文
- **不依赖本地代理**
- **不依赖远程中转 API**

### 规范文档

请查看：

- `KRead/KRead书源规范.md`
- `KRead/KRead（对外发布版).md`

### 对外发布版文档

如果你是准备发给外部用户或书源编写者，推荐使用：

- `KRead/KRead（对外发布版).md`

### 模板与示例

项目当前可参考：

- `KRead标准格式模板.json`
- `00小说网.json`
- `大唐小说网.json`

---

## 目录结构

```text
KRead/
├── App/                     App 根视图、Tab 框架
├── Models/                  数据模型
├── Stores/                  本地状态与持久化存储
├── Services/                业务服务层
├── Parsers/                 TXT / EPUB 解析器
├── Views/                   各页面与阅读器视图
├── QuickJS/                 QuickJS 与底层桥接能力
├── Utils/                   工具类
├── README_IMPLEMENTATION.md 当前实现说明
├── KREAD_SOURCE_JSON_V1.md  KRead JSON 书源规范
└── KREAD_SOURCE_JSON_V1_PUBLIC.md 对外发布版书源规范
```

---

## 主要模块说明

### 1. 本地导入与解析

相关文件：

- `KRead/Services/ImportService.swift`
- `KRead/Services/LocalBookService.swift`
- `KRead/Parsers/TxtParser.swift`
- `KRead/Parsers/EpubParser.swift`
- `KRead/Parsers/SimpleZipReader.swift`

负责：

- 文件导入
- 复制到沙盒
- 章节解析
- 本地书籍数据组织

---

### 2. 在线搜索与详情

相关文件：

- `KRead/Services/OnlineSearchService.swift`
- `KRead/Services/OnlineBookService.swift`
- `KRead/Views/Search/OnlineSearchView.swift`
- `KRead/Views/Search/OnlineBookDetailView.swift`

负责：

- 搜索请求
- 搜索结果解析
- 详情页请求
- 目录页请求
- 正文请求

---

### 3. 规则解析能力

相关文件：

- `KRead/Services/LegadoAnalyzeRule.swift`
- `KRead/Services/LegadoAnalyzeUrl.swift`
- `KRead/Services/LegadoQuickJS.swift`

负责：

- URL 模板处理
- HTML / JSON 规则解析
- 模板变量替换
- 基础 JS 执行

---

### 4. 阅读器

相关文件：

- `KRead/Views/Reader/ReaderView.swift`
- `KRead/Views/Reader/ScrollReadingView.swift`
- `KRead/Views/Reader/PlainPagedReaderView.swift`
- `KRead/Views/Reader/PageCurlReaderView.swift`

负责：

- 不同阅读模式切换
- 章节内容加载
- 分页逻辑
- 阅读交互

---

### 5. 数据与状态存储

相关文件：

- `KRead/Stores/BookshelfStore.swift`
- `KRead/Stores/BookSourceStore.swift`
- `KRead/Stores/ChapterCacheStore.swift`
- `KRead/Stores/ReaderBookmarkStore.swift`
- `KRead/Stores/ReaderAnnotationStore.swift`
- `KRead/Stores/ReadingHistoryStore.swift`

负责：

- 书架数据
- 书源数据
- 章节缓存
- 书签 / 标注 / 历史 / 统计

---

## 已重点适配 / 验证过的规则型书源示例

当前项目中，已经围绕一批真实书源做过适配与验证，包括但不限于：

- 00小说网
- 大唐小说网
- 神魔小说
- 唐诗宋词
- 大灰狼融合源

这些案例对外最有价值的不是“固定支持这些站”，而是：

> 它们说明 KRead 的规则引擎已经具备适配常见网页书源的能力。

---

## 构建与运行

### 要求

- macOS
- Xcode
- iOS Simulator 或真机

### 构建命令

```bash
xcodebuild -project KRead.xcodeproj \
  -scheme '读一刀' \
  -configuration Debug \
  -destination 'generic/platform=iOS Simulator' \
  build CODE_SIGNING_ALLOWED=NO
```

### 本地运行

直接用 Xcode 打开：

```text
KRead.xcodeproj
```

选择 scheme：

```text
读一刀
```

运行即可。

---

## 书源导入方式

当前支持两种导入方式：

### 1. 本地 JSON 文件导入

在 App 内：

- 设置
- 书源管理
- 导入书源 JSON

### 2. 在线书源 URL 导入

支持：

- 普通 JSON 直链
- GitHub `blob` 页面链接（会自动转换 raw 地址）

例如：

```text
https://github.com/Mac-XK/KRead/blob/main/00小说网.json
```

---

## 当前已知限制

### EPUB

当前 EPUB 解析仍然是基础版，主要面向普通 EPUB：

- 暂不完整还原复杂 CSS
- 暂不完整支持图片 / 字体 / 脚注等高级内容
- 复杂 ZIP 情况后续仍建议接入 `ZIPFoundation`

### 规则引擎

当前规则引擎并不是“完整 Legado 100% 兼容版”，仍有边界：

- 复杂 XPath 不保证完整
- 全量 CSS 语法不保证完整
- 重度动态签名站点不一定能直接适配
- 强浏览器环境依赖站点不一定能直接适配

### 站点稳定性

在线书源最终仍受目标站点影响：

- 站点改版
- 站点超时
- 站点分页变化
- 站点正文节点变动

这些都会影响规则型书源的可用性。

---

## 项目当前状态

当前项目已经不是概念阶段，而是具备以下特征：

- 本地阅读主链路可用
- 在线阅读主链路可用
- KRead JSON 书源规范已落地
- 书源导入 / 调试 / 批量检测能力已具备
- 阅读器体验已具备较完整雏形

如果一句话概括：

> KRead 当前已经是一个可运行、可扩展、可继续打磨的 iOS 原生阅读器项目。

---

## 后续方向

建议优先级：

### 1. 继续增强 KRead 书源规范

- 更多案例
- 更强校验
- 更好的调试反馈
- 新手友好的快速上手文档

### 2. 增强阅读器体验

- 更强的标注管理
- 更完整的分页体验
- 更细粒度阅读设置

### 3. 增强在线书源适配能力

- 更完整的规则语法支持
- 更稳定的目录 / 正文分页处理
- 更复杂站点的兼容能力

---

## 适合谁使用 / 参与

这个项目适合：

- 想做 iOS 阅读器的开发者
- 想做规则型书源生态的人
- 想研究 SwiftUI + 阅读器架构的人
- 想做本地阅读 / 在线阅读融合方案的人

---

## 说明

本项目当前文档、书源规范、示例文件都在持续完善中。  
如果你准备把 KRead 作为长期项目推进，建议优先维护好这三部分：

1. **项目 README**
2. **KRead JSON 书源规范**
3. **可运行的书源示例**

这样对后续维护、协作和对外发布帮助最大。
