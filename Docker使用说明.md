# Auto_Symlink Docker Compose 使用说明

## 文件说明

### docker-compose.yaml
这是Docker Compose配置文件，用于定义和运行Auto_Symlink容器。

## 使用前准备

### 1. 修改路径配置
在使用前，您需要修改以下路径为您的实际路径：

```yaml
# 云盘路径映射（必须修改）
- /your/cloud/path:/your/cloud/path:rslave

# 媒体库路径映射（必须修改）
- /your/media/path:/media

# 配置文件路径（可选修改）
- ./config:/app/config
```

### 2. 创建必要的目录
```bash
# 创建配置目录
mkdir -p ./config

# 创建日志目录（可选）
mkdir -p ./logs
```

## 启动服务

### 1. 基本启动
```bash
# 启动服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f auto_symlink
```

### 2. 停止服务
```bash
# 停止服务
docker-compose down

# 停止并删除卷（谨慎使用）
docker-compose down -v
```

## 配置说明

### 重要配置项

#### 1. 云盘路径映射
```yaml
- /your/cloud/path:/your/cloud/path:rslave
```
- **必须使用绝对路径**
- **左右路径必须完全一致**
- **rslave模式确保软链接正确指向真实文件**

#### 2. 媒体库路径映射
```yaml
- /your/media/path:/media
```
- 这是软链接创建的目标位置
- 确保EMBY/Jellyfin等媒体服务器也使用相同的绝对路径

#### 3. 端口映射
```yaml
ports:
  - "8095:8095"
```
- 通过 http://localhost:8095 访问Web管理界面
- 默认账号：admin
- 默认密码：password

## 常见问题解决

### 1. 群晖用户注意事项
群晖用户需要使用命令行创建容器，因为GUI界面无法选择rslave模式：

```bash
# 在群晖的任务计划中添加开机任务
mount --make-shared /volume1/
mount --make-shared /volume2/
systemctl daemon-reload
```

### 2. 权限问题
如果遇到权限问题，确保：
- 容器以root用户运行（已配置：user: "0:0"）
- 宿主机路径有正确的读写权限

### 3. 路径问题
- 确保所有路径都是绝对路径
- 云盘路径和媒体库路径在EMBY中也要使用相同的绝对路径
- 软链接仅支持绝对路径，Windows下的路径与Linux不一致

## 高级配置

### 1. 自定义网络
```yaml
networks:
  custom_network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### 2. 资源限制
```yaml
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '0.5'
    reservations:
      memory: 256M
      cpus: '0.25'
```

### 3. 环境变量
```yaml
environment:
  - TZ=Asia/Shanghai
  - LOG_LEVEL=INFO
  - DEBUG=false
```

## 监控和维护

### 1. 查看服务状态
```bash
# 查看容器状态
docker-compose ps

# 查看资源使用情况
docker stats auto_symlink

# 查看详细日志
docker-compose logs --tail=100 -f auto_symlink
```

### 2. 更新服务
```bash
# 拉取最新镜像
docker-compose pull

# 重启服务
docker-compose up -d
```

### 3. 备份配置
```bash
# 备份配置文件
cp -r ./config ./config_backup_$(date +%Y%m%d)
```

## 故障排除

### 1. 容器无法启动
- 检查路径是否正确
- 检查端口是否被占用
- 查看容器日志：`docker-compose logs auto_symlink`

### 2. Web界面无法访问
- 检查端口映射是否正确
- 检查防火墙设置
- 确认容器正在运行：`docker-compose ps`

### 3. 软链接创建失败
- 检查云盘路径是否正确
- 确认使用了rslave模式
- 检查权限设置

## 联系支持

- Telegram群组：https://t.me/autosymlink_channel
- 项目文档：https://github.com/shenxianmq/Auto_Symlink/wiki
- GitHub Issues：https://github.com/shenxianmq/Auto_Symlink/issues
