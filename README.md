# qmc-decrypt

> 中文版 README：[README.zh-CN.md](README.zh-CN.md)

> Pure-Python, zero-dependency offline decryptor for QQ Music's QMC-encrypted files — a single-file tool built for the Hi-Res use case.

## What it is

`qmc_decrypt.py` is a single-file Python tool (≈1300 lines, standard library only) that restores QQ Music's encrypted downloads (`.mflac`, `.mgg`, `.qmc0`, …) to playable originals. It is the standalone release of the "decrypt" stage inside [AuralDesk](https://github.com/HowenXu/AuralDesk)'s Hi-Res pipeline.

## How it differs from other decryptors

Existing QMC tools mostly fall into a few buckets:

- **unlock-music** (browser JS / WebAssembly): interactive-first, awkward to embed into a local program;
- **qmcdump** (Rust): needs a compiled binary per platform — not friendly on older Windows boxes;
- assorted **C# / script tools**: usually only cover the legacy v1 static-key scheme, or need Node/Python plus a pile of dependencies.

**This tool's angle:**

| Aspect | This tool | Most peers |
| --- | --- | --- |
| Dependencies | Standard library only, zero third-party deps | Need Rust / Node / npm packages or a compiler |
| Running | `python qmc_decrypt.py xxx.mflac`, works on Windows / macOS / Linux as-is | Requires a build or install first |
| Format coverage | v1 static key + v2 embedded EKey (`QQMusic EncV2,Key:` two-layer TEA and single-layer V1) + both Map (short-key) and RC4 (long-key) stream ciphers | Usually only one or two of these |
| Key sources | Embedded EKey decrypts offline; for key-less files: `--ekey`, `--ekey-db` (Android `player_process_db`), `--frida-fallback` | Most only accept embedded keys |
| Self-test | Ships test vectors ported 1:1 from the official unlock-music Rust implementation; `--self-test` verifies | Rarely present |
| Use case | Built for the automated "download raw data + fetch ekey from server → offline decrypt to Hi-Res FLAC" chain | Single-file manual decryption |

The "fetch an ekey from the server and decrypt offline" path is what makes this practical for Hi-Res (96 kHz): current PC clients no longer embed a key (MusicEx), whereas files from the legacy download chain carry an ekey trailer — which this tool turns into the original Hi-Res file offline.

## Usage

```bash
# Single file (format auto-detected)
python qmc_decrypt.py song.mflac

# Batch a whole folder
python qmc_decrypt.py ./music_dir -o ./decrypted

# Manual ekey for key-less files
python qmc_decrypt.py song.mflac --ekey "eyJ..."

# Pull keys from an Android player_process_db
python qmc_decrypt.py song.mflac --ekey-db player_process_db

# Self-test (Rust-ported vectors)
python qmc_decrypt.py --self-test
```

Supported formats: v1 (`.tkm`, `.bkc*`, hex extensions) and v2 (`.mflac`, `.mgg`, `.mgg0`, `.mgg1`, `.mflac0`, `.mmp4`, `.qmcflac`, `.qmcogg`, `.qmc0`, `.qmc2`, `.qmc3`, `.qmc4`, `.qmc6`, `.qmc8`), including the `QQMusic EncV2,Key:` two-layer TEA and single-layer V1 EKey forms, and both Map and RC4 stream ciphers.

## Integration

AuralDesk ships this tool with the app (`qqapi/app/qmc_decrypt.py`): it fetches the download URL from the QQ Music API → downloads the raw data → appends the server-provided ekey → runs this tool to decrypt offline → gets the complete lossless file (verified 96000 Hz / 24 bit / 2 ch) → hands it to HQPlayer for upsampling.

## License

[AGPL-3.0](LICENSE). The algorithms are ported 1:1 from the official [unlock-music](https://github.com/rong6/unlock-music) Rust implementation.

Copyright © 2026 [Howen_Xu](https://github.com/HowenXu). All rights reserved.

For learning purposes only — delete downloaded content within 24 hours.
