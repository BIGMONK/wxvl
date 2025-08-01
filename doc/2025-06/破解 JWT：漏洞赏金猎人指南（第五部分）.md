#  破解 JWT：漏洞赏金猎人指南（第五部分）  
haidragon  安全狗的自我修养   2025-06-27 03:56  
  
# 官网：http://securitytech.cc/  
#   
## 通过头部路径遍历绕过 JWT 身份验证kid——一种隐秘的管理员接管方法  
  
“有时只需要一个空字节和一些路径遍历就能破解管理面板。”~ Aditya Bhatt  
# ✨ 前言  
  
这是我的“破解 JWT：漏洞赏金猎人指南”系列文章的第五部分，我将使用 PortSwigger 实验室深入研究现实世界中的 JWT 错误配置。如果你错过了之前的系列文章，请务必查看：  
-   
- 第一部分 — JWT HS256 None 绕过  
- 第二部分 —— JWT 密钥混淆利用  
- 第 3 部分 — JWTjwk标头注入  
- 第 4 部分 — JWTjku标头注入  
本系列文章揭示了不安全的 JWT 处理如何导致完全身份验证绕过、权限提升，甚至完全管理员接管 💀  
  
  
  
JWT 文章索引  
  
  
  
  
# 🔐 快速入门 — JWT 和kidHeader  
  
JWT（JSON Web 令牌）是用于身份验证的无状态令牌。它们使用对称（HS256）或非对称（RS256）密钥进行签名，并在授予访问权限之前由服务器验证。  
  
（密钥 ID）标kid头提示服务器在验证 JWT 签名时应使用哪个密钥。在安全设置中，此值应与受信任密钥标识符的静态列表进行匹配。  
  
但是，当开发人员直接使用该kid值构建路径并从文件系统读取密钥时，攻击者可以通过路径遍历来滥用它——甚至可以指向它/dev/null并使用空字节进行验证。这正是此次攻击所利用的。  
  
  
  
JWT 第 5 条  
  
  
  
  
# 🚨 TL;DR  
  
在本文中，我们利用了一个实现不佳的 JWT 验证机制，该机制使用kid标头从服务器的文件系统加载密钥。通过使用路径遍历将 指向 ，kid并/dev/null使用空字节 ( ) 对令牌进行签名AA==，我们冒充了管理员用户并删除了其他用户。🧨  
  
  
  
# 🛠️ 实验室设置 — PortSwigger 的游乐场  
- 实验室：通过kid标头路径遍历绕过 JWT 身份验证  
- 账户：wiener:peter  
- 目标：/admin以管理员身份访问并删除用户carlos  
# 🔬 漏洞利用演练（逐步 PoC）  
  
这可不是一般的 PoC。我们要完全进入“角斗士”模式——一步步来，纯粹的黑客攻击，绝无虚言。  
> 📌 首选工具：Burp Suite + JWT 编辑器扩展 — 又名黑客的瑞士军刀 🗿  
  
  
  
  
## 1.进入竞技场  
  
启动实验室并使用您信任的组合登录：wiener:peter。  
  
  
  
屏幕截图 1  
  
  
🪪 资历够了？ 有。实验室装备齐全了？ 有。肾上腺素飙升？ 太棒了。  
## 2. 触摸禁忌之门  
  
尝试访问/admin。您将在门口被拦住：  
> “只有以管理员身份登录才能使用管理界面。”🛑 但你猜怎么着？我们成为了管理员。  
  
  
  
  
屏幕截图 2  
  
## 3. 拦截甜蜜包裹  
  
在 Burp 中捕获/admin请求并将其发送到 Repeater。  
  
⏳ 马上就要变得辣了。  
  
  
  
屏幕截图 3  
  
## 4. 铁匠锻造：空字节密钥  
- 转到 JWT 编辑器 > 密钥 > 新建对称密钥  
- 像网络铁匠一样锻造一把新钥匙。  
- 手动将k值设置为AA==（Base64 表示空字节）☠️没错——我们什么都没签署这个坏男孩。  
屏幕截图 4  
  
## 5.准备有效载荷  
  
回到 Repeater 中的 JWT 选项卡。选中该令牌。你现在手里拿着一颗数字手榴弹。  
  
  
  
屏幕截图 5  
  
## 6.塑造末日的标题  
- 设定sub为"administrator"——这就是我们现在的样子  
- 设置kid为：../../../../../../../dev/null  
🎯 精准路径遍历——深入深渊。  
  
  
  
屏幕截图 6  
  
## 7.像传奇一样签名  
  
点击“签名”，选择空字节密钥，然后勾选Don’t modify the header。你不仅仅是在修改令牌，更是在改写命运。🗿🔥  
  
  
  
屏幕截图 7  
  
## 8. 发送。粉碎。拥有。  
  
点击“发送”——然后 BOOM 💥 您已绕过身份验证。您已进入/admin。  
  
没有钥匙。没有门。只有大脑和字节。  
  
  
  
屏幕截图 8  
  
## 9. 向卡洛斯开枪  
  
寻找：/admin/delete?username=carlos。  
  
把它粘贴到 URL 栏里——像精确制导的网络导弹一样发射出去。🧨卡洛斯？淘汰了。  
  
  
  
屏幕截图 9  
  
## 10.确认击杀🏁  
  
右键单击→在浏览器中显示。  
  
粘贴最后一个 URL —任务完成，黑客。🧠💀  
  
  
  
  
  
  
# 🧩 为什么有效  
  
该应用会根据kid值动态构建文件路径，并使用它来读取密钥。当我们将其指向 时/dev/null，它会获取一个空文件，该文件与我们使用空字节签名的 JWT 完美匹配。服务器看到签名后，不会抛出错误，并授予我们管理员权限。💥  
  
  
  
# 🐞 漏洞赏金计划影响  
  
在实际场景中，此错误可能导致：  
- 🚨 完全身份验证绕过  
- 🔐 管理员冒充  
- 👻 隐形利用（未记录错误）  
- ⚡ 快速且可自动化的攻击  
这是一个关键问题，尤其是在使用 JWT 库而没有验证kid值或限制文件访问时。  
  
  
  
# 🧪 有效载荷摘要  
  
// JWT Header  
  
{  
  
  
"alg"  
:   
"HS256"  
,  
  
  
"kid"  
:   
"../../../../../../../dev/null"  
  
}  
  
// JWT Payload  
  
{  
  
  
"sub"  
:   
"administrator"  
  
}  
  
// Signing Key (Base64 null byte)  
  
AA==  
  
  
  
# 🧠 经验教训  
- 🚫 永远不要相信用户控制的kid标头  
- ✅ 清理所有接触文件系统的输入  
- 🔐 不要动态加载机密 - 使用内存中的静态密钥  
- 💀/dev/null不仅无害，在脆弱的环境中，它更是致命的  
# 🎉 最后的话  
  
这个实验是一个优雅的例子，展示了如何利用空字节和路径遍历轻松突破你的身份验证壁垒。从用户到管理员——无需暴力破解，也无需复杂的漏洞利用链——纯粹的 JWT 配置错误乐趣 🧠🗿  
  
如果你是漏洞赏金猎人或渗透测试人员，请务必仔细查看 JWT 标头，尤其是在kid涉及漏洞时。你永远不知道会发现什么样的漏洞。  
  
  
  
# 🔓 祝您黑客愉快！  
  
至此，我们又一次深入探索了 JWT 错误配置的世界。从空字节到路径遍历，我们彻底颠覆了服务器逻辑，最终戴上了管理员的桂冠👑。  
  
记住——黑客攻击不仅仅关乎有效载荷和标头。它关乎心态、创造力，以及一点点可控的混乱。  
  
因此，无论您是熬夜处理 Burp 请求还是边喝茶边扫描令牌……  
  
继续突破界限，继续打破常规，继续黑客攻击。下次再见，黑客。到那时再见……  
  
#HappyHacking 🗿🔥💥  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vBZcZNVQERFwEstUwtm1vDJuUhUe81vt6swbj15W6qnwhpKrXqFcarBHtqmnyKNKJoGsr5lOq3BN4uUgV8jic5Q/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "")  
  
-   
- 公众号:安全狗的自我修养  
  
- vx:2207344074  
  
- http://  
gitee.com/haidragon  
  
- http://  
github.com/haidragon  
  
- bilibili:haidragonx  
  
- ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vBZcZNVQERHYgfyicoHWcBVxH85UOBNaPMJPjIWnCTP3EjrhOXhJsryIkR34mCwqetPF7aRmbhnxBbiaicS0rwu6w/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
-   
- ![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/vBZcZNVQERHYgfyicoHWcBVxH85UOBNaPZeRlpCaIfwnM0IM4vnVugkAyDFJlhe1Rkalbz0a282U9iaVU12iaEiahw/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
