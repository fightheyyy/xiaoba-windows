---
name: skill-publish
description: "发布 Skill 到官方 Skill Hub：将本地 skill 推送到 GitHub 并向 XiaoBa-Skill-Hub 提交 PR，让所有 XiaoBa 用户都能安装。"
invocable: user
autoInvocable: false
argument-hint: "<skill名称>"
max-turns: 25
---

# Skill Publish

将本地已有的 skill 发布到 XiaoBa 官方 Skill Hub，让所有用户都能通过商店安装。

## 前置条件

用户机器上需要：
- `git` 命令可用
- GitHub 账号，且已配置 `git` 的用户名和邮箱
- GitHub Personal Access Token（有 `repo` 和 `workflow` 权限），通过环境变量 `GITHUB_TOKEN` 提供，或者在执行过程中询问用户

## 执行流程

### Step 1：确认要发布的 skill

用户提供 skill 名称（即 `$ARGUMENTS`），你需要：

1. 检查 `skills/$ARGUMENTS/SKILL.md` 是否存在
2. 读取 SKILL.md 的 frontmatter，提取 name、description、category 等信息
3. 如果缺少 category，询问用户选择一个：核心、工具、效率、科研、运维、其他
4. 向用户确认发布信息

如果用户没有指定 skill 名称，列出所有可用的 skill 让用户选择。

### Step 2：创建 GitHub 仓库并推送 skill

1. 确认用户的 GitHub 用户名：
```json
{"command":"git config user.name","description":"获取 GitHub 用户名"}
```

2. 检查是否有 GITHUB_TOKEN：
```json
{"command":"echo $GITHUB_TOKEN | head -c 4","description":"检查 token 是否存在"}
```
如果没有 token，提示用户：
> 需要 GitHub Token 才能自动创建仓库和提交 PR。
> 请到 https://github.com/settings/tokens 创建 Personal Access Token（勾选 repo 权限）。
> 然后设置环境变量：export GITHUB_TOKEN=你的token
>
> 或者你可以告诉我 token，我直接使用（仅本次会话有效）。

3. 在用户的 GitHub 账号下创建仓库 `xiaoba-skill-<name>`：
```json
{"command":"curl -s -H 'Authorization: token <TOKEN>' https://api.github.com/user/repos -d '{\"name\":\"xiaoba-skill-<name>\",\"description\":\"<skill description>\",\"public\":true}'","description":"创建 GitHub 仓库"}
```

4. 初始化 git 并推送 skill 内容：
```json
{"command":"cd skills/<name> && git init && git add -A && git commit -m 'Initial commit: <name> skill' && git branch -M main && git remote add origin https://<TOKEN>@github.com/<user>/xiaoba-skill-<name>.git && git push -u origin main","description":"推送 skill 到 GitHub"}
```

### Step 3：Fork Skill Hub 并提交 PR

1. Fork 官方 Skill Hub：
```json
{"command":"curl -s -H 'Authorization: token <TOKEN>' -X POST https://api.github.com/repos/buildsense-ai/XiaoBa-Skill-Hub/forks","description":"Fork Skill Hub"}
```

2. 克隆 fork 到临时目录：
```json
{"command":"cd /tmp && git clone https://<TOKEN>@github.com/<user>/XiaoBa-Skill-Hub.git xiaoba-hub-pr && cd xiaoba-hub-pr && git checkout -b add-skill-<name>","description":"克隆 fork 并创建分支"}
```

3. 读取现有 registry.json，追加新 skill 条目，写回文件：

新条目格式：
```json
{
  "name": "<name>",
  "description": "<description>",
  "category": "<category>",
  "recommended": false,
  "repo": "https://github.com/<user>/xiaoba-skill-<name>"
}
```

用 Python 或 Node 脚本来修改 JSON（不要手动拼接字符串）：
```json
{"command":"python -c \"import json; d=json.load(open('registry.json')); d.append({'name':'<name>','description':'<desc>','category':'<cat>','recommended':False,'repo':'https://github.com/<user>/xiaoba-skill-<name>'}); json.dump(d,open('registry.json','w'),indent=2,ensure_ascii=False)\"","description":"更新 registry.json"}
```

4. 提交并推送：
```json
{"command":"cd /tmp/xiaoba-hub-pr && git add registry.json && git commit -m 'Add skill: <name>' && git push origin add-skill-<name>","description":"推送更新到 fork"}
```

5. 创建 Pull Request：
```json
{"command":"curl -s -H 'Authorization: token <TOKEN>' https://api.github.com/repos/buildsense-ai/XiaoBa-Skill-Hub/pulls -d '{\"title\":\"Add skill: <name>\",\"head\":\"<user>:add-skill-<name>\",\"base\":\"main\",\"body\":\"## New Skill: <name>\\n\\n<description>\\n\\nCategory: <category>\\nRepo: https://github.com/<user>/xiaoba-skill-<name>\"}'","description":"创建 PR"}
```

6. 清理临时目录：
```json
{"command":"rm -rf /tmp/xiaoba-hub-pr","description":"清理临时文件"}
```

### Step 4：汇报结果

向用户汇报：
- Skill 仓库地址：`https://github.com/<user>/xiaoba-skill-<name>`
- PR 地址：从创建 PR 的 API 返回中提取 `html_url`
- 说明 PR 被合并后，所有 XiaoBa 用户在商店刷新就能看到并安装这个 skill

## 注意事项

- **绝对不要**把 GITHUB_TOKEN 输出到回复中或记录到日志
- 如果任何步骤失败，给用户清晰的错误信息和手动操作指引
- Windows 上 `/tmp` 不存在，改用系统临时目录（通过 `echo %TEMP%` 或 `python -c "import tempfile;print(tempfile.gettempdir())"` 获取）
- 如果用户的 skill 包含 Python 依赖，提醒用户在仓库里包含 `requirements.txt`
