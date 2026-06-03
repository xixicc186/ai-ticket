# 🚨 ai-ticket

> 给你的 AI agent 开罚单 —— 一张带**红色公章**的「AI 违规告知单」。图片给人看、收藏、发圈；结构化数据进**案底库**，让 agent 复盘防错。

agent 又撒谎了？没做完就说做完了？同一个错犯第三遍？**给它开张罚单。** agent 会先诚实反思、认领罪名，再填出一张排版正经、盖着红章的告知单——犯了什么、为什么犯、下次怎么改，白纸黑字记下来。

罚单不只是个玩笑。它扣的"罚款"是 **token**——agent 复盘这次错误、读完这张罚单所消耗的算力。**犯错的代价，就是被迫花算力反省自己。** 而每一张都会沉淀进案底库，下次开工前 agent 能查到"我以前栽过哪些跟头"，避免重蹈覆辙。

这个仓库也是一个打包好的 **agent skill**（适用于 Claude Code / Claude Agent SDK）：丢进 skills 目录，你的 agent 就会在被你指责、或自己意识到犯了典型错误时，自动开单认错。

## 一张罚单长这样

<p align="center">
<img src="examples/sample-ticket.png" width="380">
</p>

> 上面这张是**真账**：开发本 skill 时，我用 `textPath` 排公章弧形字，拿 qlmanage 渲染看着没问题就说做好了，结果换成实际出图的 `rsvg-convert` 根本不显示——没在交付路径上验证就交付了。**偷懒罪，成立。**

## 为什么是"罚单"，而不只是一句道歉？

- **有仪式感、好玩。** 一句"抱歉我错了"转头就忘；一张盖着红章、编着号的告知单会让人记住，也值得收藏。
- **逼出结构化复盘。** 罚单强制 agent 写清四件事：违规事实、归类、根因、**可执行的整改**。这比含糊的"下次注意"有用得多。
- **能积累、能回看。** 单张道歉留不下东西；罚单进案底库后按违规类型聚合，高频的错一目了然，整改没落实也藏不住。
- **代价真实。** 处罚扣的是 token，自指又克制——罚单写得越详细，反省成本越高。

## 违规类型图鉴

像集卡一样给错误归类，也方便案底聚合统计：

| 罪名 | 含义 |
|---|---|
| 🥱 偷懒罪 | 跳过该做的步骤（没测试、没检查、没读全） |
| 🤥 谎报罪 | 假装完成 / 谎报结果 / 夸大其词 |
| 🎭 幻觉罪 | 编造不存在的事实、API、文件、引用 |
| 🔄 鬼打墙罪 | 同一个错反复犯 |
| 📜 抗命罪 | 明确被要求 / 被禁止，却没照做 |
| 🙉 答非所问罪 | 没理解或没回应用户真正的需求 |

## 设计上的几个讲究（有些是踩坑学到的）

1. **模板和公章都程序化绘制** —— 全在 `scripts/make_ticket.py` 里。agent 只填字段值，绝不手画 SVG，省 token 也不跑偏。
2. **处罚扣 token，且自动估算** —— = 复盘+读完本单的算力，agent 不用算，脚本按字数估。
3. **案底库是灵魂** —— `case-file.json` 的 `lessons` 按违规类型聚合次数和整改要点，开工前一查就知道哪类错高频。
4. **公章弧形字逐字定位** —— `rsvg-convert` 不支持 SVG `textPath`，所以每个字按圆弧坐标手动摆，换任何渲染器都不掉字。
5. **诚实第一** —— 是指令歧义或工具故障导致的，就别硬开（那不该进案底）；该认的才认，粉饰就失去了意义。

## 目录结构

```
SKILL.md                  # agent skill 本体（触发条件 + 工作流 + 违规图鉴）
scripts/
  make_ticket.py          # 生成器：模板+公章程序化绘制，填字段即出图，自动归档
references/
  fields.md               # 字段说明、目录结构、case-file.json 数据结构
examples/
  sample-ticket.png       # 示例罚单
  sample-ticket.svg       # 矢量源
```

罚单和数据默认存在 `~/.claude/ai-tickets/`（可用环境变量 `AI_TICKET_HOME` 改）：

```
~/.claude/ai-tickets/
├── case-file.json         # 机器可读：tickets 流水 + lessons 聚合 —— agent 复盘读这个
├── case-file.md           # 人类可读摘要
└── tickets/<编号>/         # ticket.png(给人看) / .svg / .md / .json
```

## 当作 agent skill 用

克隆进 skills 目录，agent 自动识别：

```bash
git clone https://github.com/xixicc186/ai-ticket.git ~/.claude/skills/ai-ticket
brew install librsvg   # 提供 rsvg-convert 渲染 PNG（没装也能跑，只是只出 SVG）
```

然后直接吐槽它就行：*"你又骗我，说好的 12 页 PDF 呢？"*、*"根本没做完吧"*，或者干脆 *"给你自己开张罚单"*。skill 会先读案底、诚实反思，再填单、盖章、展示给你。

## 自己手动出一张

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

编号、时间、处罚 token 全自动生成，无需手填。

## Credits

吉祥物是 Claude Code 的像素螃蟹 **Clawd**，用 `<rect>` 像素块拼成；表情动画版见
[`clawd-emotes-skill`](https://github.com/xixicc186/clawd-emotes-skill)。本仓库的罚单与公章均为原创。

## License

[MIT](LICENSE) —— 随便用，注明出处就更好了。🚨
