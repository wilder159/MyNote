
### _**问题描述**_

#### _**环境**_

- **操作系统**：Windows 10/11
- **终端**：Windows Terminal、Windows PowerShell

#### _**问题**_

Windows[](https://so.csdn.net/so/search?q=&spm=1001.2101.3001.7020) 环境下执行

ssh-add ~/.ssh/id_rsa

报错如下

Error connecting to agent: No such file or directory

![[../_resources/未命名/103550ac374467bda017d9c86fe6b327_MD5.png]]

### _**解决步骤**_

#### _**1. 启动 PowerShell**_

管理员身份启动 「Windows PowerShell」。

#### _**2. 检查服务**_

运行以下指令，检查 `ssh-agent` 服务是否启动成功。

get-service ssh*

若输出如下，显示 `Status` 结果为`Stopped`，则表示服务未启动。

Status   Name               DisplayName
------   ----               -----------
Stopped  ssh-agent          OpenSSH Authentication Agent

#### _**3. 启动服务**_

使用以下两条指令启动 ssh-agent 服务

Set-Service -Name ssh-agent -StartupType Manual
Start-Service ssh-agent

再次使用 `get-service ssh*` 查看服务是否启动，如下图所示显示 `Status` 为 `Running` 则表示， `ssh-agent` 服务已启动。

![[../_resources/未命名/936c5852bd7ac9b2e80d38b65490bb75_MD5.png]]

#### _**4. 检查**_

运行以下指令查看 `ssh-agent` 已经添加的秘钥：

ssh-add -l

![[../_resources/未命名/e51f88c580c44b713a7c8ebf23cd49a4_MD5.png]]

如果未未添加过秘钥，则输出 `The agent has no identities`

测试
全文完

本文由 [简悦 SimpRead](http://ksria.com/simpread) 优化，用以提升阅读体验

使用了 全新的简悦词法分析引擎 beta，[点击查看](http://ksria.com/simpread/docs/#/%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90%E5%BC%95%E6%93%8E)详细说明

[问题描述](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-0)[环境](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-1)[问题](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-2)[解决步骤](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-3)[1. 启动 PowerShell](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-4)[2. 检查服务](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-5)[3. 启动服务](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-6)[4. 检查](https://blog.csdn.net/m0_63969219/article/details/124650073#sr-toc-7)