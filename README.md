# mySkills

个人 Claude Code 技能集。每个子目录是一个可独立安装的 skill。

## 安装

把某个 skill 目录整个复制到 Claude Code 的技能目录：

```bash
git clone https://github.com/<你的用户名>/mySkills.git
cp -r mySkills/flow-player ~/.claude/skills/
```

Windows 下即复制到 `C:\Users\<用户>\.claude\skills\`。之后新会话的技能列表会自动出现该 skill。

## 目录

### flow-player — 离散流程可视化（图 + 播放器）

把"任意沿时间或因果方向推进的离散流程"做成可视化讲解，**两级交付**：

1. **静态对照图**（轻量）：左边过程流、右边被逐步改变的状态结构，手工布局、渲染目检迭代；
2. **交互播放器**（完整）：单文件零依赖 HTML，逐跳步进、聚光灯、故障注入、确定性回放。

核心是十条不可协商的产出契约（逐跳三段式「产生 → 移动 → 落效」、状态只在落效后变化、布局零位移、真实载荷照录、断点用 id 定位、任意 seek 可重建……），每条都来自真实使用反馈，与具体领域无关。

**适用**：数据传输 / 请求生命周期 / 管道 / 工作流 / 状态机 / 代理循环——凡有离散单元沿时间推进的流程。
**不适用**：连续模拟量、无时间维的纯静态结构。

详见 [flow-player/SKILL.md](flow-player/SKILL.md) 与 [flow-player/references/engine-skeleton.md](flow-player/references/engine-skeleton.md)（通用时间轴引擎骨架）。
