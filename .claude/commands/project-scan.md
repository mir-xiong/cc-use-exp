---
description: 扫描项目生成配置（CLAUDE.md/restart.sh/ignore/Docker）
---

扫描当前项目，自动识别技术栈并生成全套配置文件。

## ⚠️ 必须生成以下所有文件

| 序号 | 文件 | 用途 | 模板位置 |
|------|------|------|---------|
| 1 | `.claude/CLAUDE.md` | 项目配置 | 直接生成 |
| 2 | `restart.sh` | 前后端打包+启动脚本 | `templates/restart-*.sh.tmpl` |
| 3 | `.claudeignore` | Claude Code 忽略 | `templates/claudeignore.tmpl` |
| 4 | `.geminiignore` | Gemini CLI 忽略 | 与 `.claudeignore` 内容相同 |
| 5 | `.gitignore` | Git 忽略 | `templates/gitignore-*.tmpl` |
| 6 | `.dockerignore` | Docker 忽略 | `templates/dockerignore.tmpl` |
| 7 | `Dockerfile` | 容器构建 | `templates/Dockerfile-*.tmpl` |
| 8 | `docker-compose.yml` | 容器编排 | `templates/docker-compose-*.yml.tmpl` |

---

## 执行步骤

### 步骤 1：扫描项目类型

检测文件确定技术栈：

| 检测文件 | 项目类型 |
|---------|---------|
| `go.mod` | Go |
| `pom.xml` | Java + Maven |
| `build.gradle` | Java + Gradle |
| `package.json` | Node.js / 前端 |
| `web/` 或 `frontend/` 目录 | 前后端分离 |

检测数据库依赖（在 go.mod 或代码中搜索）：

| 关键词 | 依赖 |
|--------|------|
| `sqlite` | SQLite |
| `mysql` | MySQL |
| `postgres` | PostgreSQL |
| `redis`, `go-redis` | Redis |

### 步骤 2：显示扫描结果并确认

```
## 项目扫描结果

**项目类型**: [检测结果]
**数据库**: [检测结果]
**前后端分离**: [是/否]

即将生成 8 个文件，是否继续？[Y/n]
```

### 步骤 3：逐个生成文件

对于每个文件：

1. **检查是否存在**
2. **如已存在** → 询问用户：
   ```
   文件 [filename] 已存在，请选择：
   1. 覆盖
   2. 跳过
   ```
3. **如不存在** → 读取对应模板，根据项目类型调整后生成

**模板选择规则**：

| 项目类型 | restart.sh | Dockerfile | docker-compose.yml |
|---------|------------|------------|-------------------|
| Go | `restart-go.sh.tmpl` | `Dockerfile-go.tmpl` | 根据数据库选择 |
| Go + 前端 | `restart-go.sh.tmpl` | `Dockerfile-go-frontend.tmpl` | 根据数据库选择 |
| Java | `restart-java.sh.tmpl` | 自行生成 | 根据数据库选择 |

| 数据库 | docker-compose 模板 |
|--------|-------------------|
| SQLite | `docker-compose-sqlite.yml.tmpl` |
| MySQL | `docker-compose-mysql.yml.tmpl` |
| Redis | `docker-compose-redis.yml.tmpl` |
| MySQL + Redis | 合并两个模板 |

### 步骤 4：设置执行权限

```bash
chmod +x restart.sh
```

### 步骤 5：输出摘要

```
## 生成完成

已生成文件：
✅ .claude/CLAUDE.md
✅ restart.sh
✅ .claudeignore
✅ .geminiignore
✅ .gitignore
✅ .dockerignore
✅ Dockerfile
✅ docker-compose.yml

跳过文件：
⏭️ [用户选择跳过的文件]

下一步：
1. 检查生成的文件是否符合项目实际情况
2. 修改 docker-compose.yml 中的默认密码
3. 测试 ./restart.sh 是否正常工作
```

---

## restart.sh 使用说明

```bash
# 默认管理员账密
./restart.sh

# 自定义管理员账密
ADMIN_USER=myuser ADMIN_PASS=mypass ./restart.sh
```

---

## 注意事项

- 生成后务必检查文件内容是否符合项目实际
- `docker-compose.yml` 中的密码应在生产环境修改
- 如项目结构特殊，可能需要手动调整模板
