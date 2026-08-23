# qmc-decrypt

> 纯 Python、零依赖的 QQ 音乐 QMC 加密文件离线解密工具，单文件即用，为 Hi-Res 场景设计。

## 这是什么

`qmc_decrypt.py` 是一个单文件 Python 工具（约 1300 行，仅标准库），把 QQ 音乐下载的 QMC 加密音频（`.mflac` / `.mgg` / `.qmc0` 等）还原为可播放的原始文件。它是 [AuralDesk](https://github.com/HowenXu/AuralDesk) 的 Hi-Res 全链路中「解密」这一环的独立版本。

## 与其他解密算法有什么不同

市面上已有的 QMC 解密实现大致是这几类：

- **unlock-music**（前端 JS / WebAssembly）：在浏览器里跑，交互为主，不适合作为本地程序的内置组件；
- **qmcdump**（Rust）：需要编译或下载对应平台的二进制，对 Windows 老机器不算友好；
- **各类 C# / 脚本小工具**：大多只覆盖旧版 v1 静态密钥，或需要 Node/Python 环境外加一堆依赖。

**本工具的特点：**

| 对比项 | 本工具 | 多数同类工具 |
| --- | --- | --- |
| 依赖 | 纯标准库，零第三方依赖 | 需要 Rust / Node / npm 包或编译器 |
| 运行 | 单文件 `python qmc_decrypt.py xxx.mflac`，Windows / macOS / Linux 直接跑 | 需先构建或安装 |
| 格式覆盖 | v1 静态密钥 + v2 内嵌 EKey（`QQMusic EncV2,Key:` 双层 TEA 与单层 V1）+ Map（短密钥）/ RC4（长密钥）两种流密码 | 多只支持其中一两种 |
| 密钥来源 | 内嵌 EKey 直接离线解；无内嵌密钥时支持 `--ekey` 手动提供、`--ekey-db` 读安卓 `player_process_db`、`--frida-fallback` 借壳兜底 | 多数只认内嵌密钥 |
| 自检 | 内置与 unlock-music 官方 Rust 实现逐一对拍的测试向量，`--self-test` 一键自检 | 多数没有 |
| 场景 | 为「下载裸数据 + 服务器取 ekey 拼尾 → 离线解密出 Hi-Res FLAC」这条自动化链路设计 | 多为单文件手动解密 |

其中「从服务器取 ekey 再离线解密」这条路径，是本工具在 Hi-Res（96kHz）场景下比单纯依赖客户端内嵌密钥更实用的地方：PC 新版客户端下载的文件已不含内嵌密钥（MusicEx），而通过旧版下载链路拿到的文件带 ekey 尾，配合本工具即可稳定离线解出 Hi-Res 原始文件。

## 使用方法

```bash
# 单个文件（自动识别格式）
python qmc_decrypt.py song.mflac

# 批量解密整个目录
python qmc_decrypt.py ./music_dir -o ./decrypted

# 无内嵌密钥时手动提供 ekey
python qmc_decrypt.py song.mflac --ekey "eyJ..."

# 从安卓端 player_process_db 取密钥
python qmc_decrypt.py song.mflac --ekey-db player_process_db

# 自检（内置 Rust 对拍测试向量）
python qmc_decrypt.py --self-test
```

支持的格式：v1（`.tkm` / `.bkc*` / 十六进制扩展名）与 v2（`.mflac` / `.mgg` / `.mgg0` / `.mgg1` / `.mflac0` / `.mmp4` / `.qmcflac` / `.qmcogg` / `.qmc0` / `.qmc2` / `.qmc3` / `.qmc4` / `.qmc6` / `.qmc8`），含 `QQMusic EncV2,Key:` 双层 TEA 与单层 V1 两种 EKey 形式，以及 Map 与 RC4 两种流密码。

## 集成

AuralDesk 桌面端将本工具随包分发（`qqapi/app/qmc_decrypt.py`），流程为：QQ 音乐接口取下载地址 → 下载裸数据 → 服务器取 ekey 拼接文件尾 → 调用本工具离线解密 → 得到完整无损文件（实测 96000Hz / 24bit / 2ch）→ 交给 HQPlayer 升频。

## 许可证

[AGPL-3.0](LICENSE)。算法以 [unlock-music](https://github.com/rong6/unlock-music) 的官方 Rust 实现为参照逐一对拍移植。

仅用于解密你自己合法下载、有权使用的音频文件。音乐平台不易，请尊重版权，支持正版。
