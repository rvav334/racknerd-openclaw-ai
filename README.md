# racknerd openclaw：选对套餐跑 AI 助手，一键安装和 SSH 脚本两条路都讲清楚，附全套餐价格与避坑点

想在 RackNerd 的 VPS 上跑 OpenClaw，核心问题其实就三个：买哪个套餐、怎么装、怎么连上 Telegram 这类聊天软件。先给结论：RackNerd 官方已经把 OpenClaw 做进了 ctrl 控制面板的一键应用目录，官方 FAQ 明确说 OpenClaw 至少要 8 GB 内存，对应年付 $119.99 的 8 GB KVM 套餐；社区里也有人在 2 GB 小内存机器上跑通过，但那是省钱打法，求稳还是按官方建议来。

## 选购速览

> 跑 OpenClaw 的主流选择是 **8 GB KVM VPS，$119.99/年，续费同价**，官方 ctrl 面板支持一键安装 OpenClaw，下单时选 Ubuntu 或 Debian 即可。预算紧、只想先试试的，2 GB 套餐（$35.99/年）在社区有小内存跑通的先例，但别对流畅度期望太高。
>
> 👉 查看官方推荐跑 OpenClaw 的 8 GB 年付套餐

## RackNerd 全套餐对比：OpenClaw 该选哪一档

RackNerd 当前的 Specials 特惠线共五档，全部是 KVM 虚拟化、年付计费，都带 1 个独立 IPv4、1Gbps 端口、完整 root 权限和 ctrl 面板。这是官网 Specials 页面现行展示的全部套餐：

| 套餐 | 配置（vCPU/内存/SSD） | 月流量 | 年付价格 | 购买入口 |
| --- | --- | --- | --- | --- |
| 1 GB KVM | 1 核 / 1 GB / 20 GB | 3 TB | $21.99 | [ 订购 1 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=952) |
| 2 GB KVM | 2 核 / 2 GB / 35 GB | 5 TB | $35.99 | [ 订购 2 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=953) |
| 4 GB KVM | 3 核 / 4 GB / 60 GB | 7 TB | $59.99 | [ 订购 4 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=954) |
| 6 GB KVM | 6 核 / 6 GB / 100 GB | 12 TB | $89.99 | [ 订购 6 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=955) |
| 8 GB KVM | 7 核 / 8 GB / 150 GB | 20 TB | $119.99 | [ 订购 8 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=956) |

对照官方给出的其他应用门槛：Hermes Agent、AnythingLLM、Ollama、LibreChat 要 4 GB 起步，n8n 和 Coolify 2 GB 就够。所以如果你之后还想在同一台机器上多跑几个自托管应用，8 GB 的余量才有意义；只跑 OpenClaw 一个，6 GB 也未尝不可，但官方 FAQ 的原话是 OpenClaw "wants at least 8 GB of RAM"，差价 $30/年，多数人没必要为了省这点钱去赌内存余量。

## 内存之争：官方说 8 GB，社区说 2 GB 也能跑

这是市面上大部分教程没讲透的地方，两边说法都真实存在，但口径完全不同。

官方 ai-vps 页面给 OpenClaw 标的配置是 4+ vCPU、8+ GB RAM，FAQ 建议直接选 8 GB 套餐。这是保守且面向生产场景的口径。

社区这边：有博主在 2 核 2.5 GB 内存、洛杉矶 DC-02 机房的 RackNerd 机器上用 Debian 13 完整装好了 OpenClaw 并接通 Telegram；也有教程推荐 1 核 1–2 GB、年付十多美元的入门套餐起步；NodeLoc 论坛上还有人提到 2 GB 内存的 VPS 现在能用。另外市面上确实有更便宜的同类方案，比如 DediRock 有 2 GB 内存、30 GB 存储、2 TB 流量的年付套餐，约 $9.88–$10.88/年，纯从价格看更低。

怎么理解这个分歧？OpenClaw 本体是个 Node.js 网关程序，小内存装上、跑起来、接到聊天软件，确实可以。但它真正的玩法是挂技能插件、跑自动化任务、长年在线，内存一紧张，最先崩的就是这些附加功能。所以更实际的分法是：先用最小成本验证自己是不是真的会用它 → 2 GB 试水；确定长期用、要挂插件和自动化 → 按官方建议上 8 GB。中间档 4 GB 的位置比较尴尬，官方应用目录里它只是其他 AI 应用的及格线。

## 两条安装路线：一键部署和 SSH 脚本各适合谁

RackNerd 官方给了两种装法，耗时和代价不一样。

**路线一：ctrl 面板一键安装。** 在 Specials 下单后登录 ctrl 面板，打开 One-Click Apps 选 OpenClaw。官方说明重装系统并部署大约需要 15–45 分钟，完成后 SSH root 密码和应用登录凭据直接显示在面板里。要注意的是，一键安装会永久清除 VPS 上现有的系统和数据，所以只适合在全新机器上操作。

**路线二：SSH 手动跑官方脚本。** 开源教程里通用的做法，一条命令：

bash
curl -fsSL https://openclaw.ai/install.sh | bash


脚本会自动装好 Node.js 22+ 和全部依赖，2–5 分钟完成，之后运行 `openclaw onboard` 进入配置向导。装完 `openclaw start` 看到 "Gateway running" 就算成功，开机自启会自动设置。日常就这几条命令：`openclaw status` 看状态、`openclaw logs` 查日志、`openclaw update` 更新、`openclaw configure` 换模型。

配置向导里有几个具体选择值得提前知道：

1. **模型提供商**：向导里可以选 Qwen，安装脚本会提示它是免费的，浏览器 OAuth 认证（用 Google 账号登录即可）之后就能先跑起来，之后再换成自己的模型。中文教程里更推荐 Minimax M2.1，需要先去平台注册拿 API Key，免费额度可测试，正式用建议充值。用 Docker 部署的进阶玩家也有接 DeepSeek 的先例。
2. **网关端口**：默认 18789，后面所有聊天软件的连接都走这个网关。
3. **权限配置**：向导会问是否允许控制浏览器、执行系统命令等，新手先按默认。
4. **技能（skill）**：可以先选 No 尽快跑通，之后让 OpenClaw 自己按需安装。

## 连上 Telegram 只差最后几步

OpenClaw 装好后本身没有聊天入口，要通过消息渠道接入，Telegram 是最顺的一条路。

先在 Telegram 里找 @BotFather 发送 `/newbot`，按提示设置 bot 名称和用户名（用户名必须以 "bot" 结尾），拿到 API Token。这个 Token 相当于你 bot 的密码，务必保密。找官方 bot 时认准名称后面的蓝色对勾和账号 ID 93372553，别接上假冒的。回到 VPS 的配置向导，messaging channel 选 TG (Bot API)，粘贴 Token，然后给你的 bot 发一条 `/start`，把返回的配对码在终端里执行 `openclaw pairing approve` 加配对码批准。配对完成后只有被批准的账号能使用这个 bot。

连接失败先查两样：Token 是否粘贴正确、服务是否在运行（`openclaw start`）。WhatsApp（扫码）、Discord、Slack 也都支持，步骤类似。微信要单独说清楚：**个人微信官方不支持**，因为微信个人版没有开放 Bot API；只有企业微信能通过第三方插件接入，且需要企业认证，折腾程度完全不同。指望在个人微信里使唤它的，这个环节可以直接放弃。

## 安全面：18789 端口千万别裸奔公网

OpenClaw 的权限很大，能操作系统、跑命令、控制浏览器，网关一旦暴露公网，等于把你整台 VPS 交了出去。这不是危言耸听——确实有第三方网站在做 OpenClaw 暴露实例的公开监测，上面挂着不少直接暴露 18789 端口的机器记录。

几个能落地的做法：

- 不在防火墙放行 18789 端口，改用 Tailscale、ZeroTier 或 Cloudflare Tunnel 做内网穿透，只有你自己的设备能连进网关。
- ctrl 面板自带 Network Firewall 入站过滤规则，可以在面板层面先挡一层。
- 改强 root 密码，最好换成 SSH 密钥登录，这两步在装任何东西之前就该做。
- Bot Token 和网关 Token 都别外泄，配对批准机制意味着没被你批准的账号本来就用不了，前提是 Token 没被拿走。

## 续费价格与真实总成本

RackNerd 的 Specials 套餐是终身循环价，官方 FAQ 明确确认续费价格与首次购买相同，不会像很多主机商那样首年低价、续费翻倍。这意味着成本可以直接按年数乘出来：8 GB 套餐跑三年是 $359.97，折合每月约 $10；如果 2 GB 试水方案能撑住你的用量，三年 $107.97，折合每月约 $3。

付款支持支付宝、微信和 PayPal，付款成功后等几分钟到半小时会收到邮件，里面有 VPS 的 IP、root 密码和控制面板链接，即时开通，不用等人工审核。下单时系统可选 CentOS、Rocky Linux、AlmaLinux、Fedora、Debian 或 Ubuntu，跑 OpenClaw 建议选 Ubuntu 22.04 或 24.04，Node.js 22+ 的兼容性最好；想用自定义 ISO 可以开工单申请，面板里还有 HTML5 VNC 控制台兜底。

## 常见问题

**RackNerd 跑 OpenClaw 需要备案或特殊配置吗？** 不需要。它是海外 VPS，开箱即用，完整 root 权限，想装什么自己决定。

**小内存套餐装不上怎么办？** 先确认脚本是否完整跑完、`openclaw start` 是否报错。有博主在小内存机器上的替代建议是改试 Hermes Agent 这个更轻的 AI agent，同样是 ctrl 一键应用目录里的成员，4 GB 起步。

**一台机器能同时跑 OpenClaw 和别的东西吗？** 可以，但内存要算总账。Ollama、AnythingLLM 这类应用各自要 4 GB 起步，8 GB 套餐同时跑 OpenClaw 加一个轻量应用是合理的上限，再多就该分开部署了。

**重装系统会丢什么？** 机器上所有数据。ctrl 的一键安装本身就是重装系统加部署应用，重装前该备份的先备份。

## 按你的情况做最后决定

只是好奇想玩玩 OpenClaw，或者不确定自己会不会长期用：从便宜的档位试水就行，2 GB 年付 $35.99，跑不动再升级，损失有限。已经想清楚要把它当 24 小时在线的 AI 助手长期用、要挂技能插件的：直接按官方建议上 8 GB，$119.99/年续费同价，内存余量够你折腾。顺手把 Ollama、n8n 这些也玩起来的，同样是 8 GB 这一个答案。

下单前最后确认两件事：机房选离你近的线路，系统选 Ubuntu 22.04 或 24.04。选好了就可以👉 用年付方案开通 8 GB 套餐部署 OpenClaw，装完记得第一件事是改 root 密码，第二件事是把 18789 藏好。
