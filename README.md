# data-sources

[![Skill self-check](https://github.com/crce-ux/data-sources/actions/workflows/structure.yml/badge.svg)](https://github.com/crce-ux/data-sources/actions/workflows/structure.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**那些不能直接 curl 拿到、且已经被试过一遍的源——路怎么走，坑在哪里。**

> 🔴 **本 skill 的一半价值是「已证伪清单」。**
> 下面每一条都是实测复现的，不是推测——**别再花一遍时间。**

```
NMPA 批准文号 · CDE 受理品种 · 医保目录 · 巨潮公告 · 港交所披露
        ↓
❓ curl 412？无头 412？起了浏览器还是空？
        ↓
本 skill：能直连的先直连，不能的用 CDP 接管真实浏览器
```

---

## 一、先试直连，再上浏览器

```
1  curl / requests         —— 主页多失败，但附件、接口常可行
2  官方 API（CT.gov v2 …） —— 直连
3  DrissionPage + 真实浏览器 —— 兜底
```

**浏览器是兜底，不是默认。** 上来就开浏览器 = 慢、脆、多数时候没必要。

---

## 二、共用技术：CDP 接管真实浏览器

政府站有**瑞数**动态令牌。已证伪的路：

```
❌ curl / requests          → 412
❌ Playwright 无头           → 412
❌ Playwright 有头 + stealth → 412
❌ goto → reload 换 Cookie   → 无效
❌ 反混淆 api.js             → 不值得

✅ DrissionPage + 真实浏览器（有头）
```

**为什么行**：经 **CDP 接管真实进程**，不留 Playwright 那套自动化指纹。

---

## 三、逐站

| 站点 | 通路 | 关键点 |
|---|---|---|
| **NMPA** | DrissionPage + **localStorage 注入** | 首页搜索框驱动不了；**结果页可独立访问** |
| **CDE** | UI 驱动，**按回车提交** | 点「查询」按钮不触发；接口带一次性令牌 |
| **国家医保局** | ✅ curl 直下 | 目录附件 PDF |
| **巨潮资讯** | ✅ 接口 | 先查表拿 `orgId` |
| **港交所** | ✅ 接口 | 先 `prefix.do` 拿 `stockId` |
| **卫健委** | ❌ 412 | 用 DrissionPage |

详见 [`references/regulatory-databases.md`](references/regulatory-databases.md)。

---

## 四、两条一般纪律 ⭐

1. **「不报错」≠「检索式生效」** —— 必须配对照组
2. **解析条数必须与站点标称的「命中 N 条」对账** —— 漏解析不报错

## ⚠️ 一条最贵的教训

**法定披露 ≠ 申报状态。**

上市公司公告**滞后**；「某药是否已申报」**以 CDE 受理目录为第一手**。
**交易所只记「公司说了什么」，不记「事情发生了没有」。**

---

## 自检

```bash
python scripts/selfcheck.py --strict
```

---

## 边界

| | 管什么 |
|---|---|
| **本 skill** | **非文献**的监管 / 政务 / 产业数据源 |
| `literature-pipeline` | **文献检索**——英文直连 + 中文库（维普 / 知网） |

> 中文库的坑单在 `literature-pipeline/references/pitfalls.md` §五——
> **本 skill 不维护第二份**。

---

## 许可

MIT。
