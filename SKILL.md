---
name: wechat-publisher
description: |
  微信公众号文章创作、排版与发布到草稿箱的 Skill。适用于把话题、参考文章、Markdown、笔记或文档整理成微信公众号文章,生成或处理配图,转换为微信兼容 HTML,并调用公众号接口创建草稿。用户提到公众号、微信文章、推文、公号、草稿箱、发文、Markdown 转公众号时应使用。
---

# WeChat Publisher

Use this skill to create or publish WeChat Official Account articles. The normal output is a draft in the WeChat backend; it does not mass-send articles.

## First Run Checklist

If a user shares the GitHub link and asks how to use this project, guide them through `README.md` in this order: clone/install, create `wechat-publisher.yaml`, find Official Account AppID/AppSecret, configure IP whitelist, verify token, then create a test draft. Prefer the manual `git clone` path for first-time users because the config file location is obvious.

1. Ensure dependencies are installed:

```bash
python3 -m pip install requests pyyaml
```

On externally managed Python installs, use:

```bash
python3 -m pip install --user requests pyyaml --break-system-packages
```

2. Create a local config file in the skill root:

```bash
cp wechat-publisher.yaml.example wechat-publisher.yaml
```

3. Fill `wechat-publisher.yaml` with the user's WeChat Official Account credentials. Never commit this file.

4. Ask the user to add the current outbound IP to the WeChat Official Account IP whitelist:

```bash
curl ifconfig.me
```

5. Verify config:

```bash
python3 scripts/wechat_api.py list-accounts
```

6. Verify access token:

```bash
cd scripts
python3 -c "from wechat_token import get_access_token; print('OK:', get_access_token()[:10] + '...')"
```

Common errors:

- `40164`: outbound IP is not in the WeChat whitelist.
- `40001` / `40002`: AppID or AppSecret is wrong.
- `48001`: the Official Account lacks the required API permission, commonly because it is not verified or the interface is unavailable.

## Configuration

The only runtime config file is:

```text
wechat-publisher.yaml
```

It is loaded from the skill root by `scripts/config.py`. The public repo must contain only `wechat-publisher.yaml.example`; the real `wechat-publisher.yaml` is ignored by `.gitignore`.

Minimal account config:

```yaml
default: main

accounts:
  main:
    name: "公众号名称"
    app_id: "wx..."
    app_secret: "your_app_secret_here"
    author: "作者名"
    theme: "refined-blue"
    image_style: "hand-drawn-blue"
    newspic_image_style: "infographic-warm"
    voice: |
      写作语气说明。可以写目标读者、口吻、常用表达和禁用表达。

image_generation:
  generator: "baoyu-image-gen"

integrations:
  wechatsync_mcp_token: ""
```

Credentials are obtained in the WeChat Official Account backend, not the Mini Program backend:

- AppID: WeChat Official Account backend -> `设置与开发` -> `开发接口管理`; look for `开发者ID(AppID)`.
- AppSecret and API IP whitelist:
  1. In the Official Account backend, open `设置与开发` -> `开发接口管理`.
  2. If development access is already enabled, use the development secret area on that page.
  3. If it is not enabled, click the blue `微信开发者平台` link.
  4. In WeChat Developer Platform, find `我的业务` near the bottom.
  5. Choose `公众号`, enter the target Official Account, then open `基础信息`.
  6. Under `开发密钥`, enable/generate the key to get `AppSecret` and configure `API IP 白名单`.
- IP whitelist value: ask the user to run `curl ifconfig.me`. If VPN/proxy/network changes, the outbound IP may change and `40164` means the whitelist must be updated.
- Never use Mini Program credentials for Official Account draft publishing.

For a first connectivity test, create a tiny `article.md` and publish with the bundled cover image:

```bash
python3 scripts/publish.py \
  --account main \
  --input article.md \
  --cover tech/2026-05-04-ouroboros/images/01.png \
  --title "发布测试稿" \
  --digest "这是一篇用于验证微信公众号草稿箱接口的测试稿。" \
  --skip-ai-score
```

## Writing Workflow

When asked to create a WeChat article:

For real long-form article writing, read `references/writing_workflow.md` before drafting. It contains the reusable article structures, opening hooks, styling markers, humanization checklist, and final gate.

1. Identify target account. Use the requested `--account` when provided; otherwise use `default` from `wechat-publisher.yaml`.
2. Read the account `voice` and write in that tone.
3. Gather sources if the topic depends on current facts. Use reliable sources and cite them in the working notes.
4. Create a Markdown article. The first `# Title` becomes the WeChat title and is removed from body HTML.
5. Prefer concrete openings: data, question, contradiction, quote, artifact, timeline, or real scene.
6. Keep paragraphs short for mobile reading.
7. Use WeChat styling markers when helpful:
   - `**text**` for primary emphasis
   - `==text==` for yellow highlight
   - `++text++` for blue highlight
   - `%%text%%` for warnings
   - `&&text&&` for positive recommendations
8. Add images when the article needs them. Local image paths are resolved relative to the Markdown file.
9. Run AI-score check unless the user explicitly requests a quick technical test:

```bash
python3 scripts/ai_score.py article.md --threshold 45
```

10. Publish to draft:

```bash
python3 scripts/publish.py \
  --account main \
  --input article.md \
  --cover cover.png \
  --title "标题" \
  --digest "摘要"
```

For a connectivity-only test draft, `--skip-ai-score` is acceptable.

## Command Reference

List accounts:

```bash
python3 scripts/wechat_api.py list-accounts
```

Publish Markdown to WeChat draft:

```bash
python3 scripts/publish.py --account main --input article.md --cover cover.png
```

Convert Markdown to WeChat-compatible HTML:

```bash
python3 scripts/html_converter.py article.md --theme refined-blue -o article.html
```

Check AI writing score:

```bash
python3 scripts/ai_score.py article.md --threshold 45
```

Generate an image:

```bash
python3 scripts/generate_image.py --account main --prompt "A clean hand-drawn infographic" --image images/01.png
```

## Images and Draft Limits

- Cover images use WeChat permanent material API and consume permanent material quota.
- Body images use `media/uploadimg` and do not consume permanent material quota.
- `publish.py` stops if referenced body images fail to upload unless `--allow-missing-images` is passed.
- Draft creation is the final step. The user still reviews and publishes manually in the WeChat backend.

## Files To Keep Private

Never commit:

- `wechat-publisher.yaml`
- `.env` or `.env.local`
- `scripts/.token_cache*.json`
- generated unpublished article drafts containing private content
- local browser cookie files used by Gemini Web workflows

Before publishing the skill repo, run a secret scan such as:

```bash
rg -n "app_secret|access_token|refresh_token|sk-|AIza|wx[0-9a-fA-F]{16,}|token_cache|cookie" .
```
