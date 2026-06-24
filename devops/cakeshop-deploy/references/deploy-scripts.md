# CakeShop 部署脚本参考

## deploy_final.py — 一键部署（推荐）

```python
import paramiko, os, time

HOST = "112.124.53.155"
USER = "root"
PASSWORD=*** = r"D:\JAVAEE\demo\demo-deploy.tar.gz"
REMOTE_DIR = "/opt/app/demo"

def run(client, cmd, timeout=300):
    stdin, stdout, stderr = client.exec_command(cmd, timeout=timeout)
    out = stdout.read().decode('utf-8', errors='replace')
    if out: print(out[-3000:])
    return out

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect(HOST, 22, USER, PASSWORD, timeout=30)

sftp = client.open_sftp()
sftp.put(TAR_FILE, f"/opt/app/demo-deploy.tar.gz")
sftp.close()

run(client, "docker stop cakeshop-app cakeshop-mysql 2>/dev/null; docker rm cakeshop-app cakeshop-mysql 2>/dev/null")
run(client, "cd /opt/app && rm -rf demo.bak 2>/dev/null; mv demo demo.bak 2>/dev/null; mkdir -p demo")
run(client, "cd /opt/app && tar -xzf demo-deploy.tar.gz -C demo/ && rm -f demo-deploy.tar.gz")
run(client, "cd /opt/app/demo && docker compose up -d --build", timeout=900)

time.sleep(15)
run(client, "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'")
run(client, "docker logs --tail 20 cakeshop-app 2>&1")
print(f"Done! http://{HOST}:8090/")
client.close()
```

## check.py — 服务器状态检查

```python
import paramiko, re
exec(open(r"D:\JAVAEE\demo\deploy2.py").read().split("def run_ssh")[0])
c = paramiko.SSHClient()
c.set_missing_host_key_policy(paramiko.AutoAddPolicy())
c.connect("112.124.53.155", 22, "root", PASSWORD, timeout=15)

def r(cmd, t=30):
    stdin, stdout, stderr = c.exec_command(cmd, timeout=t)
    out = stdout.read().decode()
    if out: print(out)
    err = stderr.read().decode()
    if err: print("E:", err[-300:])

r("docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'")
r("docker logs --tail 50 cakeshop-app 2>&1")
r("netstat -tlnp | grep 8090 || ss -tlnp | grep 8090")
c.close()
```

## fix_db.py — 数据库修复

```python
import paramiko, re, time
exec(open(r"D:\JAVAEE\demo\deploy2.py").read().split("def run_ssh")[0])
c = paramiko.SSHClient()
c.set_missing_host_key_policy(paramiko.AutoAddPolicy())
c.connect("112.124.53.155", 22, "root", PASSWORD, timeout=15)

def r(cmd, t=30):
    stdin, stdout, stderr = c.exec_command(cmd, timeout=t)
    out = stdout.read().decode()
    if out: print(out)

r("docker exec -i cakeshop-mysql mysql -uroot -proot cookieshop < /opt/app/demo/sql/init.sql 2>&1")
r("docker restart cakeshop-app")
time.sleep(10)
r("docker logs --tail 15 cakeshop-app 2>&1")
c.close()
```

## 注意

- 所有脚本依赖 deploy2.py 中的 PASSWORD 变量（避免密码硬编码在多个文件中）
- check.py 和 fix_db.py 通过 `exec(open(...).read().split("def run_ssh")[0])` 动态加载密码
- 执行时用 Hermes venv Python：`C:\Users\Loze\AppData\Local\hermes\hermes-agent\venv\Scripts\python.exe`
