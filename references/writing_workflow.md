# Writing Workflow Reference

Use this reference when the user wants the skill to write a real WeChat Official Account article, not merely publish an existing Markdown file.

## Table of Contents

- Account voice
- Article structures
- Opening hooks
- Markdown skeleton
- Humanization checklist
- Image planning
- Final gate

## Account Voice

Read the target account from `wechat-publisher.yaml`.

- Use `default` when the user does not specify an account.
- Use the account's `voice` field as the writing baseline.
- Keep each account's tone distinct. Do not make every account sound like the same generic AI assistant.
- If the user gives real details, preserve them: people, time, place, price, product version, error message, command output, and first-hand experience.

## Article Structures

Pick one structure before drafting. Avoid using the same structure for every article.

| Structure | Best For | Flow |
|---|---|---|
| `data-first` | Rankings, benchmarks, funding, model comparisons | number -> context -> why it matters -> implications -> judgment |
| `question-led` | Unclear news, controversy, user confusion | question -> origin -> evidence A/B -> confirmed vs uncertain -> conclusion |
| `contrarian` | One-sided public opinion | common belief -> why incomplete -> evidence -> reframing -> impact |
| `teardown` | Frameworks, protocols, agent architecture, SDKs | overview -> layers -> tradeoffs -> comparison -> boundaries |
| `field-notes` | Tool tests, tutorials, local experiments | what I tried -> how it ran -> what worked -> what failed -> advice |
| `timeline-news` | News, company moves, policy, incidents | what happened -> timeline -> key players -> scope -> what to watch |
| `playbook` | Guides, methods, productivity | who it is for -> shortest path -> key actions -> traps -> when not to use |
| `case-file` | Failure reviews, penalties, security incidents | facts -> evidence -> where it went wrong -> alternative explanation -> lesson |

## Opening Hooks

Use a concrete first screen. Avoid generic openings like "with the rapid development of...".

| Hook | Use It Like This | Avoid |
|---|---|---|
| `hard-number` | Start with a number, price, percentage, version, or ranking | empty emotion after the number |
| `sharp-question` | Ask a question the reader would really ask | several rhetorical questions in a row |
| `contradiction` | Put two conflicting facts side by side | fake reversals |
| `quote-first` | Start with a sourced quote, issue, or commit message | unsourced "someone said" |
| `artifact-first` | Start from logs, command output, screenshot details, or benchmark rows | unexplained technical dump |
| `timeline-first` | Start with a specific date or event chain | broad historical background |
| `scene-first` | Start from a short real scene | long fictional setup |

## Markdown Skeleton

The first `# Title` becomes the WeChat title and is removed from body HTML by `publish.py`.

```markdown
# 标题

> 摘要,1 到 2 句话。

## 开篇

用具体数字、问题、矛盾、引用、命令输出、时间线或真实场景切入。

## 小节一

## 小节二

## 小节三

## 写在最后
```

Use 3 to 6 sections for most articles. Let section lengths vary naturally.

## WeChat Styling Markers

Use these markers sparingly to add visual rhythm:

| Marker | Meaning |
|---|---|
| `**text**` | primary emphasis |
| `==text==` | yellow highlight for key facts |
| `++text++` | blue highlight for tools/concepts |
| `%%text%%` | warning or trap |
| `&&text&&` | positive result or recommendation |
| `!!text!!` | strong red emphasis |
| `@@text@@` | blue emphasis |
| `^^text^^` | orange emphasis |

## Humanization Checklist

Before publishing, revise the draft:

- Replace generic claims with concrete facts.
- Add at least one real detail when available: time, command, price, error, version, screenshot observation, or exact workflow.
- Vary paragraph lengths.
- Remove formulaic transitions such as "值得注意的是" when they do not add information.
- Avoid empty summaries like "总的来说" unless the paragraph actually concludes something.
- Keep uncertainty explicit. If a fact is not verified, say so.
- Match the account `voice`; do not use one universal tone.

## Image Planning

For real articles, plan images before publishing:

- Use a cover image.
- Use body images only when they explain or add value.
- Keep a consistent image style from `assets/image-styles`.
- Local image paths are resolved relative to the Markdown file.
- Cover upload consumes WeChat permanent material quota; body images use `uploadimg` and do not consume permanent material quota.

## Final Gate

Run the AI-score gate for real posts:

```bash
python3 scripts/ai_score.py article.md --threshold 45
```

For connectivity tests only, `--skip-ai-score` is acceptable.

Then publish to draft:

```bash
python3 scripts/publish.py --account main --input article.md --cover cover.png
```
