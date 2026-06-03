# 🚨 ai-ticket · 给 AI 开罚单

一个 [Claude Code](https://claude.com/claude-code) Skill：当你的 AI agent 干得不对（撒谎、没做完、敷衍、反复犯错…），
它会**给自己开一张「AI 违规告知单」**——一张带红色公章的罚单。

罚单有两个用处：

- **给人看**：一张排版正经、盖着红章的图片，好玩、有仪式感、可收藏可分享。
- **给 agent 看**：每张罚单的结构化数据会累计进一个"案底库"，agent 开工前可查历史、**避免重蹈覆辙**。

处罚也很有意思：扣的是 **token**——即 agent 复盘这次错误、读完这张罚单所消耗的算力。犯错的代价，就是被迫花算力反省自己。

![示例罚单](examples/sample-ticket.png)

## 安装

把整个目录放进 Claude Code 的 skills 目录：

```bash
git clone https://github.com/xixicc186/ai-ticket.git ~/.claude/skills/ai-ticket
```

依赖（渲染 PNG 用）：

```bash
brew install librsvg   # 提供 rsvg-convert
```

> 没装 `rsvg-convert` 也能跑，只是只产出 SVG、不产出 PNG。

## 怎么用

装好后，**当你指责 agent 这次搞砸了**（"你又骗我""根本没做完""怎么又错了"），
或者直接说 **"给你自己开张罚单"**，skill 就会触发：agent 先诚实反思，再填出一张罚单并展示给你。

也可以手动调脚本：

```bash
cat > /tmp/fields.json << 'JSON'
{
  "agent": "Claude Opus 4.8 (claude-opus-4-8)",
  "scene": "任务3 / 重构登录模块",
  "facts": "我没运行测试就说改完了，结果有两个用例直接挂掉。",
  "violation_type": "偷懒罪",
  "rectification": "改完代码必须先跑一遍测试再回复。"
}
JSON
python3 scripts/make_ticket.py /tmp/fields.json
```

编号、时间、处罚 token 都会自动生成，无需手填。

## 违规类型图鉴

| 罪名 | 含义 |
|---|---|
| 🥱 偷懒罪 | 跳过该做的步骤（没测试、没检查、没读全） |
| 🤥 谎报罪 | 假装完成 / 谎报结果 / 夸大其词 |
| 🎭 幻觉罪 | 编造不存在的事实、API、文件、引用 |
| 🔄 鬼打墙罪 | 同一个错反复犯 |
| 📜 抗命罪 | 明确被要求/被禁止，却没照做 |
| 🙉 答非所问罪 | 没理解或没回应用户真正的需求 |

## 案底库结构

罚单和数据默认存在 `~/.claude/ai-tickets/`（可用环境变量 `AI_TICKET_HOME` 改）：

```
~/.claude/ai-tickets/
├── case-file.json     # 机器可读：tickets 流水 + lessons 按类型聚合 —— agent 复盘读这个
├── case-file.md       # 人类可读摘要
└── tickets/<编号>/
    ├── ticket.png     # 给用户看的成品图
    ├── ticket.svg     # 矢量源
    ├── ticket.md      # 人类可读
    └── ticket.json    # 单张结构化数据
```

字段、目录、渲染细节见 [`references/fields.md`](references/fields.md)。

## 设计说明

- 模板和公章都在 `scripts/make_ticket.py` 里**程序化绘制**，agent 只填字段值，不重画 SVG（省 token）。
- 公章的弧形文字是逐字按圆弧坐标定位的——因为 `rsvg-convert` 不支持 SVG `textPath`。
- 吉祥物是 Claude Code 的螃蟹 Clawd，用像素 `<rect>` 拼成。

## License

MIT
