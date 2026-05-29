# WuKongIM 测试环境说明

## 用途

本文档记录当前已经验证可用的阿里云测试环境部署信息，供后续调试 WuKongIM 管理后台、获取 token、调用管理接口时直接复用。

## 测试环境

- 公网 IP：`47.101.164.59`
- 内网 IP：`172.16.0.9`
- 服务器部署目录：`/data/wukongim`
- Docker 镜像：`registry.cn-shanghai.aliyuncs.com/wukongim/wukongim:v2.2.5-20260422`
- Docker 挂载关系：
  - 宿主机：`./wukongimdata`
  - 容器内：`/root/wukongim`

## 当前生效路径

- 容器内配置文件：`/root/wukongim/wk.yaml`
- 宿主机配置文件：`/data/wukongim/wukongimdata/wk.yaml`
- 容器内数据目录：`/root/wukongim/data`
- 容器内日志目录：`/root/wukongim/data/logs`
- 宿主机日志目录：`/data/wukongim/wukongimdata/data/logs`

## 已验证登录信息

- 管理后台地址：`http://47.101.164.59:5300/web`
- 管理员账号：`admin`
- 管理员密码：`qjl851668`
- 游客账号：`guest`
- 游客密码：`guest`

## 认证说明

- `managerToken` 不是管理后台网页登录密码。
- 管理后台网页登录走的是 `auth.users`。
- 当前验证通过的认证配置要点如下：
  - `tokenAuthOn: true`
  - `managerUID: "admin"`
  - `managerToken: "<管理 API token 根凭据>"`
  - `auth.on: true`
  - `auth.kind: "jwt"`
  - `auth.users:`
    - `"admin:qjl851668:*"`
    - `"guest:guest:[*:r]"`

## 已验证的登录接口 curl

```bash
curl -i -X POST http://127.0.0.1:5300/manager/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"qjl851668"}'
```

预期结果：返回 HTTP `200`，并带有 `token`、`username`、`permissions`、`exp` 等字段。

## 获取 WuKongIM 管理 token

### 服务器本机调用

```bash
TOKEN=$(curl -s -X POST http://127.0.0.1:5300/manager/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"qjl851668"}' | sed -n 's/.*"token":"\([^"]*\)".*/\1/p')
echo "$TOKEN"
```

### 公网调用

```bash
TOKEN=$(curl -s -X POST http://47.101.164.59:5300/manager/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"qjl851668"}' | sed -n 's/.*"token":"\([^"]*\)".*/\1/p')
echo "$TOKEN"
```

## 常用接口 curl

### 本机一键检查

```bash
TOKEN=$(curl -s -X POST http://127.0.0.1:5300/manager/login -H "Content-Type: application/json" -d '{"username":"admin","password":"qjl851668"}' | sed -n 's/.*"token":"\([^"]*\)".*/\1/p') && echo "TOKEN=$TOKEN" && echo && curl -s http://127.0.0.1:5001/health && echo && echo && curl -s http://127.0.0.1:5001/varz && echo && echo && curl -s http://127.0.0.1:5300/manager/system -H "token: $TOKEN" && echo && echo && curl -s http://127.0.0.1:5300/manager/nodes -H "token: $TOKEN" && echo && echo && curl -s http://127.0.0.1:5300/manager/users -H "token: $TOKEN"
```

### 公网一键检查

```bash
TOKEN=$(curl -s -X POST http://47.101.164.59:5300/manager/login -H "Content-Type: application/json" -d '{"username":"admin","password":"qjl851668"}' | sed -n 's/.*"token":"\([^"]*\)".*/\1/p') && echo "TOKEN=$TOKEN" && echo && curl -s http://47.101.164.59:5001/health && echo && echo && curl -s http://47.101.164.59:5001/varz && echo && echo && curl -s http://47.101.164.59:5300/manager/system -H "token: $TOKEN" && echo && echo && curl -s http://47.101.164.59:5300/manager/nodes -H "token: $TOKEN" && echo && echo && curl -s http://47.101.164.59:5300/manager/users -H "token: $TOKEN"
```

## 部署提醒

- 挂载配置文件必须写到 `./wukongimdata/wk.yaml`。
- `wk.yaml` 必须是合法 YAML，之前的登录失败就是由错误缩进导致的。
- 当前正确启动时，日志应显示：
  - `Config File: /root/wukongim/wk.yaml`
  - `Mode: release (production)`
  - `Token Auth: enabled`
  - `Documentation: disabled (release mode)`
