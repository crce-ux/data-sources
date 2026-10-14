---
name: data-sources
description: 非常规数据源的取数方法、本机接线与**已证伪路径**——监管/政务/产业库：NMPA 药品批准文号、CDE 受理品种、国家医保局目录、巨潮资讯、港交所披露易、卫健委、ClinicalTrials.gov。核心是**别重复踩已经证伪的坑**（curl/无头浏览器一律 412），以及**两条一般纪律**：「不报错」不等于「检索式生效」（必须配对照组）、解析条数必须与库里标称命中数对账。当用户说「NMPA」「药监局」「批准文号」「CDE」「受理号」「医保目录」「巨潮」「上市公司公告」「港交所」「披露易」「临床注册」「瑞数」「412」，或要查药品审批/医保/上市公司披露数据时使用。**文献检索不要用本 skill**——英文库走 literature-pipeline，中文库（维普/知网）也在那里。
compatibility: 需 DrissionPage + 本机真实浏览器（Chrome/Edge，有头）。部分站点需登录态或人工过一次验证。取数脚本与本机路径绑定，不随包分发。
---

# 非常规数据源 · 取数方法与已证伪路径

## 这个 skill 管什么

**那些「不能直接 curl 拿到」的源。**

| 常规（直连，不用本 skill） | 非常规（本 skill） |
|---|---|
| PubMed / Europe PMC / OpenAlex / CrossRef | **NMPA · CDE · 国家医保局** |
| ClinicalTrials.gov **API v2** | **巨潮资讯 · 港交所披露易 · 卫健委** |

> 🔴 **本 skill 的一半价值是「已证伪清单」**——哪些路走不通。
> 下面每一条都是**实测复现**的，不是推测；**别再花一遍时间**。

> **文献检索不是本 skill 的活**：英文库走 [`literature-pipeline`](https://github.com/crce-ux/literature-pipeline)；
> 中文库（维普 / 知网）也在那里（`literature-pipeline/scripts/cnlit_search.py`
> + `literature-pipeline/references/pitfalls.md` §五）。
> 本 skill 只承接**非文献**的监管 / 政务 / 产业数据。

---

## 一、共用技术：CDP 接管真实浏览器

政府站有反自动化（NMPA 用**瑞数动态令牌**）。

```
❌ 已证伪：curl / requests           → 412
❌ 已证伪：Playwright 无头            → 412
❌ 已证伪：Playwright 有头 + stealth  → 412
❌ 已证伪：goto → reload 换 Cookie    → 无效
❌ 已证伪：反混淆 api.js（jsjiami + 自定义 VM）→ 不值得

✅ 唯一可用：DrissionPage + 本机真实浏览器（有头）
```

```python
from DrissionPage import ChromiumPage, ChromiumOptions
co = ChromiumOptions()
co.set_browser_path(r"<本机 Edge / Chrome 路径>")
co.set_argument("--disable-blink-features=AutomationControlled")
page = ChromiumPage(co)
```

**为什么它行**：DrissionPage 经 **CDP 接管真实浏览器进程**，
**不留 Playwright 那套自动化指纹**。

> ⚠️ **必须是有头**。加 `--headless` 一样被识别。

---

## 二、两条一般纪律 ⭐（**不限于本 skill**）

这两条是**通用**的——凡是「脚本代替人操作一个会返回结果的界面」都适用。

### 1. 「不报错」≠「检索式生效」

**必须配对照组**（单概念的命中数）验证检索式真被解析了。

> **实测**（中文库）：裸词静默返回**与检索词完全无关**的通用最新列表，不报错。

### 2. 解析条数必须与库里标称的命中数**对账**

对不上 = 解析漏了，**而且不会报错**。

> **实测**（中文库）：单元格有两种写法，只匹配一种时
> **87 条只解析出 67 条，不报错**。

> ⭐ 写在**文本里**的「命中 N 条」是**唯一可信的分母**——
> 它来自站点自己的计数，是你唯一能拿来做对账的东西。

---

## 三、⚠️ 一条最贵的教训：**法定披露 ≠ 申报状态**

**上市公司公告存在滞后**；「某药是否已申报」**以 CDE 受理目录为第一手**。

> **实例**：某药已在 CDE 受理（受理号有），而公司**当时尚未发公告**
> → 曾因此误判「媒体无据」。

**推论**：查一个「已发生但未公告」的事实，
**去业务主管站的业务目录查，不要去交易所查披露**。
交易所只记「公司说了什么」，不记「事情发生了没有」。

---

## 四、逐站要点

| 站点 | 通路 | 关键点 |
|---|---|---|
| **NMPA** 批准文号 | DrissionPage + **localStorage 注入** | 首页搜索框驱动不了；**结果页可独立访问**，查询条件持久化在 localStorage |
| **CDE** 受理品种 | UI 驱动，**按回车提交** | 点「查询」按钮**不触发**；直接调接口不可靠（URL 带一次性令牌） |
| **国家医保局** | ✅ **curl 直下** | 目录附件 PDF → 解析后 grep 药名 |
| **巨潮资讯** | ✅ 接口 | 股票表取 `orgId`；公告接口 `hisAnnouncement/query` |
| **港交所披露易** | ✅ 接口 | 先 `prefix.do` 取 `stockId` |
| **卫健委** | ❌ 412 | 用 DrissionPage |
| **ClinicalTrials.gov** | ✅ **API v2** | 直连，不用浏览器——**别把它当非常规源** |

> **能直连的就直连**——先试 curl / 官方 API，失败再上浏览器。
> **浏览器是兜底，不是默认。**

逐站代码、参数、返回结构见
[`references/regulatory-databases.md`](references/regulatory-databases.md)。

---

## Anti-Patterns

- **Never** 对政府站用 curl / requests——已证伪，412
- **Never** 用无头浏览器跑这些站——同样 412，**必须是有头**
- **Never** 把「脚本没报错」当成「检索生效了」——**必须配对照组**
- **Never** 不把解析条数与站点标称的命中数对账——**漏解析不报错**
- **Never** 一上来就上浏览器——**先试直连 / 官方 API**，浏览器是兜底
- **Never** 反混淆瑞数令牌——**接管真实浏览器比破解便宜得多**
- **Never** 拿上市公司公告判断「某药是否已申报」——**以 CDE 受理目录为第一手**
- **Never** 在 `--attach` 模式里 `quit()` 一个人工验证过的浏览器——关掉就得重过验证
- **Never** 在文档里硬编码本机浏览器路径却不写环境变量兜底——换机必坏

---

## 脚本

| 脚本 | 干什么 |
|---|---|
| `scripts/check_skill_structure.py` · `scripts/selfcheck.py` | skill 自检（结构 + 语法 + 依赖） |

> ⚠️ **取数脚本不随包分发。** 它们与本机浏览器路径 / profile / 登录态绑定，
> 换机即坏——**发一份坏的比不发更糟**。
> 方法见本文件；**本机接线**（浏览器路径、profile、venv）见
> [`references/regulatory-databases.md`](references/regulatory-databases.md) §本机接线。

```bash
python scripts/selfcheck.py --strict
```

---

## ⚠️ 本机接线（**首次使用前必看**）

| 项 | 处置 |
|---|---|
| **venv** | 重建并装 `DrissionPage` |
| **浏览器路径** | 改脚本，或设环境变量；**不要把路径写死在正文里** |
| **登录态 profile** | 部分站点需登录——profile 路径独立于脚本 |
| **NMPA** | 首次需在真实浏览器里过一次验证；成功后条件存 localStorage |

> 🔴 **第一次跑通后，把成功的那套参数（浏览器路径 / profile / 超时）记进
> `references/regulatory-databases.md` 的本机接线表**——
> 下次换机照它重建，不用重新试错。

---

## 参考文件

| 文件 | 何时读 |
|---|---|
| `references/regulatory-databases.md` | 查药政 / 产业数据时——逐站通路、参数、返回结构、本机接线表 |

---

## 来源

整理自作者 2026-09 的《监管/政务数据库取数》手册，内容为**实测复现**的通路与坑，非推测。
两条一般纪律与「法定披露 ≠ 申报状态」的实例也源于同一批实测。
