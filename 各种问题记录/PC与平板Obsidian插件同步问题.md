**1. 表面现象：** 桌面端 Obsidian（Windows）和平板端 Obsidian（Android）共用同一个 GitHub 仓库来同步笔记，但双方安装的插件不同。如果直接推送包含插件文件的仓库，会导致 `.obsidian/plugins/` 目录在设备间相互覆盖，且插件文件体积大（尤其是 Excalidraw、Obsidian Git 等动辄几十 MB），拖慢每次的 git 操作。

**2. 底层原因：插件目录不该被同步**

- **插件的本质：** `.obsidian/plugins/` 下的每个插件都是通过 Obsidian 社区插件市场独立安装的，不同设备安装的插件版本、配置可能不同。
- **git 的"无差别追踪"：** 如果不加处理，git 会追踪插件目录下所有文件，包括编译后的 JS 包（`.js`）、样式文件（`.css`）、JSON 配置（`data.json`）。这些文件既大又频繁变动，且**跨设备没有意义**——平板的插件配置不应该覆盖 PC 的，反之亦然。
- **冲突风险：** 如果 PC 和平板对同一个插件的 `data.json` 做了不同修改，git pull 时会报合并冲突，需要手动解决，非常麻烦。

**3. 解决思路：用 `.gitignore` 把插件目录踢出 git 追踪**

核心原则：**插件各自装，笔记一起同步。**

- `.obsidian/plugins/` 目录整体加入 `.gitignore`，不再被 git 追踪。
- 各设备通过 Obsidian 社区插件市场各自安装所需插件，互不干扰。
- 笔记内容（`.md` 文件和其他资产）照常通过 GitHub 同步。

**4. 操作步骤（已执行）：**

```bash
# 第一步：在 .gitignore 中添加插件目录
# 写入 .obsidian/plugins（注意用正斜杠，相对路径）
# 不要写 Windows 绝对路径（如 .\obsidian\plugins 或 D:\...）

# 第二步：如果插件文件之前已被 git 追踪，需要从追踪中移除
# ⚠️ 注意：执行后其他设备 pull 会删除本地插件文件！
# git rm -r --cached .obsidian/plugins/

# 本仓库已执行方案：只加 .gitignore，不清除已有追踪
# 这样新插件不会被追踪，已安装的插件不受影响
```

**.gitignore 最终内容：**

```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
.obsidian/plugins
```

**5. 注意事项：**

- **已有追踪 vs 新增忽略：** `.gitignore` 只对**未追踪**的文件生效。如果插件文件之前已经被 git 追踪过，即使加了 `.gitignore`，它们仍然会出现在 `git status` 中。要彻底清除需要执行 `git rm --cached`（见第 4 步），但这个操作会导致其他设备 pull 时删除本地插件文件。
- **本仓库的处理方式：** 选择了**温和方案**——只加 `.gitignore`，不清除已有追踪。好处是现有插件在设备间保持同步不会被删，坏处是之前已追踪的插件文件变动仍会显示在 `git status` 中，不过不影响使用。
- **新安装的插件：** 所有**未来**安装的新插件都不会被 git 追踪，各设备独立管理。

**6. 现场还原（执行过程中的一次波折）：**

```
第一次操作：错误地执行了 git rm --cached + commit + push
          结果：远程仓库删除了插件文件追踪
          问题：平板 GitSync 如果 pull，会删除本地所有插件
          解决：用 git revert 撤销了该 commit，改用温和方案

关键教训：git rm --cached 会删除远程追踪记录，
          其他设备 pull 时会删除本地对应文件。
          在有多设备的仓库中操作务必谨慎。
```

**7. 相关工具：**

- **桌面端：** 直接使用系统 git 或 Obsidian Git 插件 + GitHub 远程仓库
- **平板端（Android）：** [GitSync](https://github.com/ViscousPot/GitSync) —— Android 上的 git 同步工具，用于平板 Obsidian 仓库的拉取和推送
- **GitSync 常见问题：** 如果报错 `Software caused connection abort (errno=103)`，通常是平板网络问题（WiFi 不稳定、VPN 干扰、GitHub API 被阻断），切换网络或重试即可，与仓库配置无关。

---

**总结一下：** PC 和平板共用 GitHub 仓库同步笔记，但**插件各自独立安装**。`.gitignore` 中的 `.obsidian/plugins` 就是这条"分界线"——笔记走 git 共享，插件各管各的。
