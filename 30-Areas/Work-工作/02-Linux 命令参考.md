---
title: Linux 命令参考
created: 2026-03-11
tags:
  - work
  - linux
  - reference
aliases:
  - Linux 命令速查
---
## 💡 常用组合命令

<!-- 记录实际工作中用到的组合命令 -->
后端发版
```
docker ps 
```

```
docker stop hems-backend 
```

```
cd /usr/local/helianHome/hems-backend/ 
```

```
rm -f hems-0.0.1-SNAPSHOT.jar.bak 
```

```
mv hems-0.0.1-SNAPSHOT.jar hems-0.0.1-SNAPSHOT.jar.bak 
```

```
上传新包至/usr/local/helianHome/hems-backend/ 
```

```
docker start hems-backend 
```

```
docker logs -f --tail 300 hems-backend
```
前端发版
```
docker ps

docker stop hems-nginx 

cd /usr/local/helianHome/hems-nginx 

上传前端提供的压缩包（如 `舟山_0917.zip`）到该目录。 

unzip 武汉光谷0107.zip "dist/*" -d ./tem 

rm -rf ./html-bak mv ./html ./html-bak 

mv ./tem/dist/ ./html 

docker start hems-nginx
```
接口发版
```
docker stop hems-interface

cd /usr/local/helianHome/hems-interface

mv hems-interface.jar hems-interface.jar.bak

上传新的 jar 包

docker restart hems-interface
```
Redis 缓存清理
```
docker exec -it helian-redis bash

redis-cli -h 172.66.202.152 -p 6379 -a Helian@2022 -n 8

KEYS "*model_result_cache_key*"

EVAL "return redis.call('DEL', unpack(redis.call('KEYS', ARGV[1])))" 0 "*model_result_cache_key*"
```
Curl导报告
```
拉报告-linux 
sudo curl -L -X POST "http://192.168.100.10:8090/api/open/download/report/zip" -H "token:eyJhbGciOiJIUzUxMiJ9.eyJleHAiOjE3NzQzNjY1MjIsInN1YiI6IueuoeeQhuWRmCIsImlhdCI6MTc3MzM2NjUyMjc5Mn0.-h_EW6TJKBWSfvcHrWGKPnGDyHgXNXHB-qweavx0X5UKihGFt2UghpOYy4WVy8G1pyrernXCqZvOnixnBfsIfw" -H "Content-Type: application/json" -d '["228974_1"]' -o /usr/local/helianHome/staticfiles/report.zip

推报告-powershell 
curl.exe -X POST "http://10.20.3.2:9942/api/open/upload/report/zip" -F "file=@D:\myfiles\报告\光谷\report.zip"
```
测试服务器手动发包
```
nohup /usr/local/lib/jdk1.8.0_172/bin/java -Xms512m -Xmx512m -jar /opt/hems/server/znzj-wuhanguanggu-dev/hems-0.0.1-SNAPSHOT.jar -Dlog4j2.formatMsgNoLookups=true --server.port=9511 --spring.profiles.active=wuhanguanggudev  --his.feeItem.compareUrl=http://127.0.0.1:9510/compare/getMockData >output.log 2>&1 &
```
# Linux 命令参考

---

## 📂 文件操作

| 命令        | 说明     | 示例                     |
| --------- | ------ | ---------------------- |
| `ls -lah` | 详细列出文件 | `ls -lah /var/log`     |
| `cp -r`   | 复制目录   | `cp -r src/ dst/`      |
| `mv`      | 移动/重命名 | `mv old.txt new.txt`   |
| `rm -rf`  | 强制删除目录 | `rm -rf tmp/`          |
| `find`    | 查找文件   | `find . -name "*.log"` |
| `grep -r` | 递归搜索内容 | `grep -r "error" .`    |

---

## 🔍 文本处理

```bash
# 查看文件前 N 行
head -n 20 file.log

# 查看文件后 N 行
tail -f file.log          # 实时追踪
tail -n 100 file.log      # 最后 100 行

# 文本搜索
grep -i "pattern" file    # 忽略大小写
grep -v "pattern" file    # 反向匹配
grep -A 3 "pattern" file  # 显示匹配后 3 行
grep -B 3 "pattern" file  # 显示匹配前 3 行

# 管道组合
cat app.log | grep "ERROR" | wc -l    # 统计错误数
```

---

## 🔧 系统信息

```bash
# CPU/内存
top                         # 进程监控
htop                        # 增强版 top
free -h                     # 内存使用
df -h                       # 磁盘空间
du -sh *                    # 目录大小

# 网络
netstat -tulpn              # 端口占用
ss -tulpn                   # 新版端口查看
curl -I https://example.com # 测试请求

# 系统
uname -a                    # 系统信息
uptime                      # 运行时间
last                        # 登录历史
```

---

## 📦 压缩解压

```bash
# tar
tar -czvf archive.tar.gz dir/     # 压缩
tar -xzvf archive.tar.gz          # 解压

# zip
zip -r archive.zip dir/           # 压缩
unzip archive.zip                 # 解压
```
---

