# PaiSmart 本地启动说明

本文档记录当前 Windows 本地环境已经跑通的启动方式。

## 1. 基础服务

基础服务使用 `docs/docker-compose.yaml` 启动，包括：

- MySQL
- Redis
- Kafka
- Elasticsearch
- MinIO

如果当前终端识别 `docker`，执行：

```powershell
docker compose -f docs\docker-compose.yaml up -d
```

如果 IDEA/PowerShell 提示 `docker` 不存在，使用 Docker Desktop 的完整路径：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" compose -f docs\docker-compose.yaml up -d
```

查看服务状态：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" compose -f docs\docker-compose.yaml ps
```

当前本地端口：

```text
MySQL:         localhost:3306
Redis:         localhost:16379
Kafka:         localhost:9092
Elasticsearch: http://localhost:9200
MinIO:         http://localhost:19000
MinIO Console: http://localhost:19001
```

说明：Windows 当前环境不能绑定宿主机 `6379`，所以 Redis 映射为 `16379:6379`，后端 `.env` 里也要使用 `SPRING_DATA_REDIS_PORT=16379`。

## 2. 初始化 MySQL 和 MinIO

如果是第一次启动，确认数据库存在：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" exec mysql mysql -uroot -pPaiSmart2025 -e "CREATE DATABASE IF NOT EXISTS PaiSmart DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

确认 MinIO bucket 存在：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" exec minio sh -c "mc alias set local http://127.0.0.1:19000 admin PaiSmart2025 >/dev/null && mc mb --ignore-existing local/uploads"
```

## 3. 后端配置

项目根目录需要有 `.env`。该文件被 `.gitignore` 忽略，不提交到仓库。

关键配置：

```env
SPRING_PROFILES_ACTIVE=dev
SERVER_PORT=8081

SPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/PaiSmart?useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
SPRING_DATASOURCE_USERNAME=root
SPRING_DATASOURCE_PASSWORD=PaiSmart2025

SPRING_DATA_REDIS_HOST=localhost
SPRING_DATA_REDIS_PORT=16379
SPRING_DATA_REDIS_PASSWORD=PaiSmart2025

SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092

ELASTICSEARCH_HOST=localhost
ELASTICSEARCH_PORT=9200
ELASTICSEARCH_SCHEME=http
ELASTICSEARCH_USERNAME=elastic
ELASTICSEARCH_PASSWORD=PaiSmart2025

MINIO_ENDPOINT=http://localhost:19000
MINIO_PUBLIC_URL=http://localhost:19000
MINIO_ACCESS_KEY=admin
MINIO_SECRET_KEY=PaiSmart2025
MINIO_BUCKET_NAME=uploads
```

Embedding API key 放在：

```env
EMBEDDING_API_KEY=你的 text-embedding-v4 key
```

不要把真实 key 写入文档或提交到 Git。

## 4. 启动后端

如果 `mvn` 已加入 PATH：

```powershell
mvn spring-boot:run
```

如果 PowerShell 提示 `mvn` 不存在，使用 IntelliJ 自带 Maven：

```powershell
& "D:\IntelliJ IDEA 2025.2.2\plugins\maven\lib\maven3\bin\mvn.cmd" spring-boot:run
```

后端启动成功标志：

```text
Tomcat started on port 8081
Started SmartPaiApplication
```

访问地址：

```text
http://localhost:8081
```

根路径返回 `403` 属于正常现象，表示 Spring Security 已生效。

## 5. 启动前端

进入前端目录：

```powershell
cd frontend
```

如果 `pnpm` 可用：

```powershell
pnpm install
pnpm dev
```

如果 `pnpm` 不在 PATH，使用 corepack：

```powershell
& "C:\Program Files\nodejs\corepack.cmd" pnpm install
& "C:\Program Files\nodejs\corepack.cmd" pnpm dev
```

如果遇到 pnpm 11 拦截构建脚本：

```powershell
& "C:\Program Files\nodejs\corepack.cmd" pnpm approve-builds --all
& "C:\Program Files\nodejs\corepack.cmd" pnpm install
```

前端启动成功标志：

```text
VITE v6.3.5 ready
Local: http://localhost:9527/
```

访问地址：

```text
http://localhost:9527
```

## 6. 默认登录账号

当前本地数据库已创建管理员账号：

```text
账号：admin
密码：Admin@123456
```

如果数据库被清空，需要重新创建管理员账号：

1. 修改根目录 `.env`

```env
ADMIN_BOOTSTRAP_ENABLED=true
ADMIN_BOOTSTRAP_USERNAME=admin
ADMIN_BOOTSTRAP_PASSWORD=Admin@123456
```

2. 重启后端。

3. 登录成功后改回：

```env
ADMIN_BOOTSTRAP_ENABLED=false
```

## 7. 常见问题

### docker 命令不存在

说明 Docker Desktop 已安装但没有进入当前终端 PATH。使用完整路径：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" version
```

长期解决：把下面目录加入系统 PATH，然后重启 IDEA：

```text
C:\Program Files\Docker\Docker\resources\bin
```

### mvn 命令不存在

使用 IntelliJ 自带 Maven：

```powershell
& "D:\IntelliJ IDEA 2025.2.2\plugins\maven\lib\maven3\bin\mvn.cmd" -v
```

长期解决：把下面目录加入系统 PATH：

```text
D:\IntelliJ IDEA 2025.2.2\plugins\maven\lib\maven3\bin
```

### Redis 6379 端口无法绑定

当前 Windows 环境拒绝绑定宿主机 `6379`，已经改为：

```yaml
16379:6379
```

后端配置应使用：

```env
SPRING_DATA_REDIS_PORT=16379
```

### MinIO bucket 不存在

错误类似：

```text
The specified bucket does not exist
```

执行：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" exec minio sh -c "mc alias set local http://127.0.0.1:19000 admin PaiSmart2025 >/dev/null && mc mb --ignore-existing local/uploads"
```

### 前端缺少 @iconify/utils

错误类似：

```text
Cannot find package '@iconify/utils'
```

执行：

```powershell
cd frontend
& "C:\Program Files\nodejs\corepack.cmd" pnpm add -D @iconify/utils
```

## 8. 停止服务

停止 Docker 基础服务：

```powershell
& "C:\Program Files\Docker\Docker\resources\bin\docker.exe" compose -f docs\docker-compose.yaml down
```

停止后端或前端：

- 如果是在终端前台启动，按 `Ctrl + C`
- 如果是后台启动，根据端口查 PID 后停止：

```powershell
Get-NetTCPConnection -LocalPort 8081,9527 -State Listen
Stop-Process -Id <PID> -Force
```
