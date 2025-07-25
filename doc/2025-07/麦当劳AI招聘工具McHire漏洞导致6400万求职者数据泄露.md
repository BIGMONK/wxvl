#  麦当劳AI招聘工具McHire漏洞导致6400万求职者数据泄露  
 船山信安   2025-07-19 16:00  
  
麦当劳旗下AI招聘工具McHire存在重大安全缺陷，暴露了6400万份求职申请。调查发现，不安全的直接对象引用（IDOR）漏洞与弱默认凭证共同导致了大规模个人数据泄露，开发方Paradox.ai已迅速完成修复。  
## 漏洞详情曝光  
  
麦当劳绝大多数特许经营商使用的AI招聘平台McHire存在安全漏洞，导致超过6400万求职者的个人信息遭暴露。安全研究人员伊恩·卡罗尔和萨姆·柯里发现，该漏洞允许未授权访问包含姓名、电子邮箱、电话号码和家庭住址在内的敏感数据。  
  
事件起因是Reddit用户报告由Paradox.ai开发的McHire聊天机器人"Olivia"出现异常响应。研究人员随即发现两处关键缺陷：首先，餐厅业主的管理登录界面接受极易猜测的默认凭证——用户名和密码均为"123456"，通过该凭证可直接获取系统内测试餐厅账户的管理权限。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/7nIrJAgaibicN4yhViaVjzXUXr2hvGEDJK6XefibTrFnqibZbRNSXQl1DBZ6Bic4O9Hth7nib7D7HQNlhF5mIhO4uc4mg/640?wx_fmt=jpeg&from=appmsg "")  
  
图片来源：Reddit  
## IDOR漏洞加剧风险  
  
更严重的问题在于内部API存在不安全的直接对象引用（IDOR）漏洞。攻击者只需修改网址中的数字参数（本案中与申请人聊天记录关联的lead_id），任何McHire账户持有者都能查看其他申请人的聊天交互机密信息。  
  
研究人员在博客中指出，通过此漏洞可查阅数百万份求职申请的详细信息，包括未脱敏的联系方式，甚至能获取用于冒充申请人登录的身份验证令牌，查看原始聊天记录。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/7nIrJAgaibicN4yhViaVjzXUXr2hvGEDJK68WnQoYwcMlDs74fZ8Z9WreRPebqXguliaEC8g5RZnV429hKqqVs3jZw/640?wx_fmt=jpeg&from=appmsg "")  
  
图片来源：伊恩·卡罗尔  
## 平台运作机制与修复过程  
  
通过https://jobs.mchire.com/  
访问的McHire平台会引导求职者完成自动化流程，包括Traitify.com的性格测试。申请者与Olivia交互时需提供联系方式及班次偏好。  
  
研究人员从餐厅业主端测试申请时发现存在漏洞的API，注意到获取候选人信息的PUT/api/lead/cem-xhr  
请求可通过修改lead_id  
参数查看他人数据。  
  
意识到潜在数据暴露规模后，研究团队立即启动披露程序，于2025年6月30日美国东部时间17:46联系Paradox.ai和麦当劳。麦当劳迅速确认报告，当日19:31即禁用默认管理凭证。Paradox.ai确认所有问题在2025年7月1日22:18前彻底解决，两家公司均表示将加强数据安全防护。  
## 行业专家警示AI安全风险  
  
全球数据隐私管理公司MineOS联合创始人兼CEO科比·尼桑评论称："此事件警示企业，若在缺乏适当监督的情况下仓促部署面向客户的AI工作流，将使自身和数百万用户暴露于不必要的风险中。"  
  
"问题不在于AI技术本身，而是缺乏基本的安全防护与治理机制。任何收集或处理个人数据的AI系统，都应与企业核心业务系统遵循相同的隐私保护、安全及访问控制标准。"科比强调，"这意味着需要建立身份验证、审计追踪机制，并将其整合至整体风险工作流，而非放任其成为监管盲区。随着AI应用加速普及，企业必须将其视为受监管资产而非新奇工具，从初始阶段就构建确保责任归属的框架。"  
  
转载来源：【  
https://www.freebuf.com/articles/database/438725.html】【  
https://hackread.com/mcdonalds-ai-hiring-tool-mchire-leaked-job-seekers-data/】  
  
