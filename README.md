# wechat-publisher

一个把 Markdown / 选题 / 参考资料发布到微信公众号草稿箱的 Skill。它会帮你做公众号排版、上传封面和正文图片,最后在公众号后台创建一篇草稿。

> 只创建草稿,不会自动群发。你仍然需要登录公众号后台检查后手动发布。

## 适合谁

- 想把 Markdown 一键转成公众号草稿的人
- 想让 Codex / Claude Code / 其他 Agent 帮你写公众号文章的人
- 想复用一套已经验证过的公众号 AppID、AppSecret、IP 白名单发布流程的人

## 先准备这些

1. 一个微信公众号后台账号,不是小程序后台。
2. 公众号的 `AppID` 和 `AppSecret`。
3. 本机已安装 `python3`。
4. 本机已安装 `git`。
5. 一张用于测试的封面图。仓库里也自带了一张测试图,可以先用。

AppID / AppSecret 获取位置:

1. 打开 [微信公众平台](https://mp.weixin.qq.com)。
2. 进入你的公众号后台,确认当前账号是公众号,不是小程序。
3. 在左侧菜单打开 `设置与开发`。
4. 点击 `开发接口管理`。
5. 在这个页面可以看到 `开发者ID(AppID)`,这就是配置里的 `app_id`。
6. 如果你之前已经开启过开发接口,页面里会直接看到开发密钥相关区域,里面有 `AppSecret` 和 `API IP 白名单`。
7. 如果你之前没有开启过,页面会提示你前往微信开发者平台。点击页面里的蓝色文字 `微信开发者平台`。
8. 跳转到微信开发者平台后,在页面下方找到 `我的业务`。
9. 在 `我的业务` 里选择 `公众号`,点进你的公众号。
10. 进入公众号板块后,找到 `基础信息`。
11. 在 `基础信息` 下方找到 `开发密钥`。
12. 点击开启 / 生成开发密钥后,就能看到本项目需要的 `AppSecret` 和 `API IP 白名单`。
13. 把 `AppID` 填到 `wechat-publisher.yaml` 的 `app_id`,把 `AppSecret` 填到 `app_secret`。

注意:小程序的 AppID / AppSecret 不能用于公众号发文。

## 推荐安装方式:手动安装

这个方式最适合第一次使用,因为配置文件就在你 clone 下来的目录里,不容易找丢。

```bash
git clone https://github.com/Wayne-bear3/wechat-publisher.git
cd wechat-publisher
python3 -m pip install requests pyyaml
```

如果安装依赖时报 `externally-managed-environment`,改用:

```bash
python3 -m pip install --user requests pyyaml --break-system-packages
```

把这个目录注册成 Skill。按你用的客户端选一个:

```bash
# Codex
mkdir -p ~/.codex/skills
ln -s "$(pwd)" ~/.codex/skills/wechat-publisher

# Claude Code
mkdir -p ~/.claude/skills
ln -s "$(pwd)" ~/.claude/skills/wechat-publisher
```

然后重启 Codex / Claude Code,或者新开一个会话。

## 快捷安装方式:npx

熟悉 skills 工具的话,也可以直接安装:

```bash
npx skills add Wayne-bear3/wechat-publisher
```

安装后需要找到 skill 的安装目录,因为后面要在里面创建 `wechat-publisher.yaml`。如果你不确定目录在哪里,优先使用上面的手动安装方式。

## 配置公众号

在项目根目录复制配置模板:

```bash
cp wechat-publisher.yaml.example wechat-publisher.yaml
```

打开 `wechat-publisher.yaml`,把下面几项换成你自己的:

```yaml
default: main

accounts:
  main:
    name: "你的公众号名称"
    app_id: "wx..."
    app_secret: "你的公众号 AppSecret"
    author: "作者名"
    theme: "refined-blue"
    image_style: "hand-drawn-blue"
    newspic_image_style: "infographic-warm"
    voice: |
      这里写你的公众号语气。
      比如:面向 AI 产品经理,口吻直接一点,少用套话。

image_generation:
  generator: "baoyu-image-gen"

integrations:
  wechatsync_mcp_token: ""
```

`wechat-publisher.yaml` 里有密钥,不要提交到 GitHub。仓库的 `.gitignore` 已经默认忽略它。

## 配置 IP 白名单

微信要求调用接口的公网 IP 必须在白名单里。

运行:

```bash
curl ifconfig.me
```

把输出的 IP 添加到公众号后台的 `IP 白名单`。如果你开了 VPN、代理、公司网络或云服务器,实际出口 IP 可能会变。报 `40164` 时,重新运行 `curl ifconfig.me`,再把新 IP 加进去。

## 验证配置

先确认脚本能读到账号:

```bash
python3 scripts/wechat_api.py list-accounts
```

正常会看到类似:

```text
已配置 1 个账号:
  main  你的公众号名称  AppID: wx123456...  作者: 作者名  (默认)
```

再验证能拿到微信 access_token:

```bash
cd scripts
python3 -c "from wechat_token import get_access_token; print('OK:', get_access_token()[:10] + '...')"
cd ..
```

看到 `OK: ...` 就说明 AppID、AppSecret、IP 白名单已经通了。

## 发布第一篇测试稿

创建一篇最小测试稿:

```bash
cat > article.md <<'EOF'
# 发布测试稿

> 这是一篇用于验证微信公众号草稿箱接口的测试稿。

如果你能在公众号后台草稿箱看到这篇文章,说明 AppID、AppSecret、IP 白名单和草稿接口已经连通。

## 测试内容

- Markdown 已成功读取
- 微信排版已成功生成
- 封面图已成功上传
- 草稿已成功创建

## 写在最后

这篇文章只是测试,不用正式发布。
EOF
```

仓库自带了一张测试封面图,可以直接用:

```bash
python3 scripts/publish.py \
  --account main \
  --input article.md \
  --cover tech/2026-05-04-ouroboros/images/01.png \
  --title "发布测试稿" \
  --digest "这是一篇用于验证微信公众号草稿箱接口的测试稿。" \
  --skip-ai-score
```

成功后会看到类似:

```text
API连接正常,token已获取
封面图上传成功: media_id=...
草稿创建成功! media_id=...
发布成功! 文章已保存到草稿箱。
```

然后登录公众号后台,打开草稿箱,查看《发布测试稿》。

## 日常使用

发布你自己的 Markdown:

```bash
python3 scripts/publish.py \
  --account main \
  --input your-article.md \
  --cover cover.png \
  --title "你的标题" \
  --digest "你的摘要"
```

只做排版预览,不发布:

```bash
python3 scripts/html_converter.py your-article.md --theme refined-blue -o article.html
```

检查文章是否太像 AI 写的:

```bash
python3 scripts/ai_score.py your-article.md --threshold 45
```

在 Agent 里可以这样说:

```text
使用 wechat-publisher,帮我把这篇 Markdown 发布到公众号草稿箱。
```

## 常见问题

| 问题 | 原因 | 解决 |
|---|---|---|
| `ConfigError: 未找到 wechat-publisher.yaml` | 没有复制配置文件 | 运行 `cp wechat-publisher.yaml.example wechat-publisher.yaml` |
| `40164 invalid ip` | 当前公网 IP 不在白名单 | 运行 `curl ifconfig.me`,把 IP 加到公众号后台白名单 |
| `40001` / `40002` | AppID 或 AppSecret 不对 | 确认填的是公众号凭证,不是小程序凭证 |
| `48001` | 接口权限不可用 | 检查公众号类型、认证状态和接口权限 |
| `40009` | 图片格式或大小不符合要求 | 换 jpg/png,压缩到 10MB 以内 |
| 草稿箱没看到文章 | 可能登录错公众号或发布失败 | 看终端最后是否有 `草稿创建成功` |
| VPN 打开后又失败 | 出口 IP 变了 | 重新查 IP 并更新白名单 |

## 安全提醒

不要把这些文件提交到 GitHub:

- `wechat-publisher.yaml`
- `.env` / `.env.local`
- `scripts/.token_cache*.json`
- 浏览器 cookie 文件
- 未公开文章、客户资料、内部文档

发布前可以扫一下:

```bash
rg -n "app_secret|access_token|refresh_token|sk-|AIza|wx[0-9a-fA-F]{16,}|token_cache|cookie" .
```

## 目录结构

```text
wechat-publisher/
├── SKILL.md
├── README.md
├── wechat-publisher.yaml.example
├── scripts/
├── assets/
├── references/
└── tests/
```

## License

MIT
