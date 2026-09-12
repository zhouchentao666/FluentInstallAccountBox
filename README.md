# Steam 共享账号盒 · 云端账号源

这个仓库**只是账号云端**，不含任何源代码。

客户端每次启动从这里拉取账号，本地不保存任何账号数据。

## 目录里有什么

```
Account/
├── steam_accounts.enc   账号容器（AES-256-GCM 密文）
├── manifest.json        版本清单（只有密文地址）
└── devices.json         已授权设备的 RSA 公钥
```

| 文件 | 内容 | 能不能被看懂 |
| --- | --- | --- |
| `steam_accounts.enc` | 账号容器的密文 | 不能 |
| `manifest.json` | 一个 URL 和版本号 | 能，但没有信息量 |
| `devices.json` | 一堆 RSA **公钥** | 能，但公钥只能加密不能解密 |

## 为什么把账号放在公开仓库里也是安全的

加密分两层：

1. **容器层**：随机数据密钥用 AES-256-GCM 加密整个账号列表，
   这把密钥再用**每台已授权设备的 RSA-3072 公钥**分别做 OAEP 封装；
2. **字段层**：每条账号的「账号」和「密码」再用同一把密钥单独加密。

所以：

- 克隆这个仓库 = 拿到密文 + 公钥，**没有私钥就解不开**；
- 私钥只存在于每一台被授权设备的本机（`%LOCALAPPDATA%\SteamAccountBox\device_key.pem`），从不上传；
- 客户端打开容器后，账号密码**仍然是密文**，只有用户点击上号的那一刻才解出这一条；
- 容器带 GCM 校验，任何篡改都会被拒绝。

## 客户端怎么用

```bash
pip install -r requirements.txt
python main.py
```

- 新设备：设置页「导出公钥」→ 发给管理员 → 登记后即可解密；
- 每次启动自动同步，也可点「云端同步」手动拉取；
- 双击卡片一键上号。

> 若 GitHub raw 访问不畅，把 `Account/` 下这三个文件放到任意对象存储 / CDN，
> 在设置里把「云端 manifest 地址」改成你的地址即可。

## 管理员（本机执行，不在此仓库）

```bash
python tools/import_accounts.py 明文.txt --dry-run     # 脱敏预览
python tools/import_accounts.py 明文.txt --shred       # 生成容器并销毁明文
python tools/grant_device.py --key-file 对方.pub.pem --name "小明的电脑"
python tools/grant_device.py --revoke <device_id>      # 撤销后旧设备立刻失效
python tools/set_game.py --all "剑星"                   # 改游戏名，不动凭据
```

改动后把 `Account/` 提交推送即对所有客户端生效（版本号递增才会覆盖）。
