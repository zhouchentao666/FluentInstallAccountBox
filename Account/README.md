# Account 目录：云端账号源（可安全公开）

| 文件 | 内容 |
| --- | --- |
| `steam_accounts.enc` | 账号容器密文（账号与密码均为字段级密文） |
| `manifest.json` | 版本清单，只有密文地址 |
| `devices.json` | 已授权设备的 **RSA-3072 公钥** |

这里**没有明文账号、没有私钥**，客户端每次启动现拉，本地不留。

## 授权一台新设备

对方执行：

```bash
python tools/init_device.py --export
```

把公钥发给你，你执行：

```bash
python tools/grant_device.py --key-file 对方.pub.pem --name "小明的电脑"
```

这会按新名单**重新封装数据密钥**（账号密文本身不变），然后推送 `Account/` 生效。

## 撤销一台设备

```bash
python tools/grant_device.py --list
python tools/grant_device.py --revoke <device_id>
```

重新封装后，被撤销设备手里的旧容器**立即失效**——因为数据密钥不再为它封装。
