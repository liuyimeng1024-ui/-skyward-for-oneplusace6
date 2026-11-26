# 一加 Ace 6（PLQ110）官方全量包与防回滚状态

数据来源：RomCloud 的 OTA 索引（`roms.lian86.top`），直链指向官方 CDN
`gauss-compota-c-cn.allawnfs.com`。ARB 值由 `xbl_config` 分区读出，
见 `scripts/ota-part.py` 与 `scripts/ota-arb.py`。

## 防回滚（ARB）机制

OPPO/一加的防回滚**不在 vbmeta 的 `rollback_index` 里**，而在 **`xbl_config`
分区**的 OEM metadata 中。用 vbmeta 判断会得出错误结论。

检测方法：

```sh
# 设备当前状态
adb shell su -c "dd if=/dev/block/by-name/xbl_config_a of=/data/local/tmp/x.img"
adb pull /data/local/tmp/x.img
arbextract x.img

# 某个 OTA 包（只下载几 KB）
python scripts/ota-part.py "<OTA直链>" xbl_config x.img
arbextract x.img
```

`arbextract` 来自 <https://github.com/koaaN/arbextract>。

规则：**设备 ARB 值 ≥ 包 ARB 值**才允许刷入。ARB 不可逆，一旦升上去，
装 ARB 更低的包必然变砖（无 fastboot、无 9008，只能换主板）。

## 实测数据

设备当前：`PLQ110_17.0.0.104(CN01)`，**ARB = 1**

| 版本 | ARB | 来源 | 能否刷入 |
| --- | --- | --- | --- |
| ColorOS PLQ110_17.0.1.200(CN01) | 1 | 实测 | ✅ |
| ColorOS PLQ110_17.0.0.104(CN01) | 1 | 设备当前 | ✅ |
| ColorOS PLQ110_16.0.10.500(CN01) | 1 | 实测 | ✅ |
| ColorOS PLQ110_16.0.9.402(CN01) | 1 | 推断 | ✅ |
| ColorOS PLQ110_16.0.9.401(CN01) | 1 | 推断 | ✅ |
| ColorOS PLQ110_16.0.8.300(CN01) | 1 | 推断 | ✅ |
| — 分界线 — | | | |
| ColorOS PLQ110_16.0.7.200(CN01) | 1 | 实测 | ✅ |
| ColorOS PLQ110_16.0.5.700(CN01) | 1 | 实测 | ✅ |
| ColorOS PLQ110_16.0.3.502(CN01) | **0** | 实测 | ❌ 变砖 |
| ColorOS PLQ110_16.0.0.205(CN01) | **0** | 实测 | ❌ 变砖 |

**分界线在 16.0.5.700**：从这一版起带 ARB。更早的包（16.0.3.502 及以下）
ARB 为 0，刷到本机（ARB=1）上会硬砖。

## 恢复用包的选择

恢复（救砖）时只能选 **ARB = 1** 的包，即 `16.0.5.700` 及更新的版本。

Recovery 里默认只列 **最新版 + 前 5 个**，共 6 个：

```
1. ColorOS PLQ110_17.0.1.200(CN01)
2. ColorOS PLQ110_17.0.0.104(CN01)
3. ColorOS PLQ110_16.0.10.500(CN01)
4. ColorOS PLQ110_16.0.9.402(CN01)
5. ColorOS PLQ110_16.0.9.401(CN01)
6. ColorOS PLQ110_16.0.8.300(CN01)
```

这 6 个全部 ARB = 1，都可以安全刷入。

## 完整版本列表

| # | 版本 | ARB |
| --- | --- | --- |
| 1 | PLQ110_17.0.1.200(CN01) | 1 |
| 2 | PLQ110_17.0.0.104(CN01) | 1 |
| 3 | PLQ110_16.0.10.500(CN01) | 1 |
| 4 | PLQ110_16.0.9.402(CN01) | 1 |
| 5 | PLQ110_16.0.9.401(CN01) | 1 |
| 6 | PLQ110_16.0.8.300(CN01) | 1 |
| 7 | PLQ110_16.0.7.201(CN01) | 1 |
| 8 | PLQ110_16.0.7.200(CN01) | 1 |
| 9 | PLQ110_16.0.5.702(CN01) | 1 |
| 10 | PLQ110_16.0.5.700(CN01) | 1 |
| 11 | PLQ110_16.0.3.502(CN01) | 0 |
| 12 | PLQ110_16.0.2.403(CN01) | 0 |
| 13 | PLQ110_16.0.2.402(CN01) | 0 |
| 14 | PLQ110_16.0.1.302(CN01) | 0 |
| 15 | PLQ110_16.0.0.207(CN01) | 0 |
| 16 | PLQ110_16.0.0.205(CN01) | 0 |

「推断」指按分界线推定：ARB 随版本单调不减，16.0.5.700 起为 1，
其后的版本不会回落到 0。
