**1. 表面现象：** 明明电脑已经开启了科学上网工具（梯子），浏览器也能正常打开 GitHub 网页，但在 PowerShell / CMD 中执行 `git clone` 却报错 `Recv failure: Connection was reset`。

**2. 底层原因：浏览器与命令行的“温差”**

- **浏览器的流量：** 大多数科学上网工具在开启后，会自动接管系统的“浏览器流量”（HTTP/HTTPS），所以能顺畅地看网页。
    
- **终端命令行的流量：** 终端工具（如 Git、npm 等）默认是非常“死板”的。**它们默认不会自动走系统代理，而是头铁地去尝试直连外部网络。**
    
- **冲突爆发：** Git 尝试直连 GitHub，但在国内的网络环境下，直连 GitHub 极不稳定甚至被直接阻断（墙），于是 Git 的连接请求就被强制切断了，最终报出 `Connection was reset`。
    

**3. 解决思路：给命令行“牵线搭桥”** 必须通过代码显式地告诉 Git：“不要直连了，请把你的流量转发到我本地代理软件的端口上”。

**4. 解决代码备份：**

Bash

```
# 本地代理软件的端口是 6984
# 设置 Git 走代理
git config --global http.proxy http://127.0.0.1:6984
git config --global https.proxy http://127.0.0.1:6984

# 如果以后关了代理，或者需要直连国内仓库（如 Gitee），记得取消代理设置
git config --global --unset http.proxy
git config --global --unset https.proxy
```

---

**总结一下：** 开了梯子只是给**浏览器**修好了路，但没有给**Git** 指路。后来执行的那两行 `git config` 命令，就是专门给 Git 指路的“导航仪”。
[为Obsidian接入Gemini_CLI时遇到的问题](为Obsidian接入Gemini_CLI时遇到的问题.md)关于网络配置方面的其他问题