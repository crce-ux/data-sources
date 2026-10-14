# scripts/

**这里是运行版。正本在 [`research-workflow`](https://github.com/crce-ux/research-workflow) 的 `scripts/`**——开发与测试都在那边。

改脚本请改总仓，再同步过来；**不要只改这里**。

| 脚本 | 干什么 |
|---|---|
| `check_skill_structure.py` | 结构校验（官方 Agent Skills 规范）：name / description / 正文长度 / 深链 / **声明落地** |
| `selfcheck.py` | **一条命令自检**：结构 + 语法 + 依赖 |

> ⚠️ **本 skill 不分发取数脚本。**
> NMPA / CDE / 巨潮 / 港交所的取数代码与本机浏览器路径、profile、登录态绑定，
> 换机即坏——**发一份坏的比不发更糟**。方法见 SKILL.md 与 `references/regulatory-databases.md`。

## 自检

```bash
python scripts/selfcheck.py --strict
```
