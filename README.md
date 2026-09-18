# oppo_oplus_realme_5x-hwid（HWID 校验版）

欧加真（OPPO / OnePlus / Realme）**5.10 / 5.15** OKI 内核，fastbuild 快速构建 + **HWID 白名单校验**。
基于 `BailinT/oppo_oplus_realme_5x`（非 hwid 版，12/12 已跑绿），仅允许 `hwid/allowlist.txt` 中的设备启动。

## 与 5x 非 hwid 版的差异

1. **Checkout 步骤**：首个 step 拉取本仓（私有资源本地 cp，不再 wget 自身）
2. **HWID 注入步骤**：初始化源码后注入 `hwid_lock.c` + `hwid_allowlist.h`（12 棵树锚点全部实测存在）
3. **CCACHE_KEY 加 `-hwid` 后缀**：缓存与非 hwid 仓互不干扰
4. 其余与 5x 完全一致（三类修复：老树 inotify helper / strict-prototypes / sm8550 ThinLTO+swap 全部保留）

## HWID 机制（5-key 版）

读取 `androidboot.chipid` → `androidboot.cpuid` → `androidboot.emmcid` → `androidboot.serialno` → `oplusboot.serialno`，
任一 key 命中 allowlist 即放行（`hwid_match` 先 strcasecmp 精确比对，再尝试 kstrtoull 数值比对）。
不在白名单 / 读不到任何 key → `panic()` 重启。

## 加设备流程

1. 设备端（原厂内核）跑 `sh hwid/probe_hwid.sh [预期id]`，拿 `hwid_probe.log`
2. 取原始值（**不要**用识别脚本格式化过的输出），确认来源 key
3. 填入 `hwid/allowlist.txt`（`^[0-9a-z._:-]{4,128}$`，小米 cpuid 等超长 ID 必须原值逐字符一致）
4. 提交后触发构建；刷入前先确认设备已在 allowlist

## 状态

- [x] 12 个 x workflow 已生成（改造自 5x 非 hwid 版，YAML + bash -n 全过）
- [x] 注入脚本 5.10/5.15 树兼容性实测（main.c/Makefile 锚点 + strscpy/kstrtoull/xbc_find_value 全有）
- [ ] 首次构建验证

## 参考

- 非 hwid 版：`BailinT/oppo_oplus_realme_5x`（12/12 绿，ccache 全存，复跑 10 分钟）
- 交接文档：`docs/交接-5x内核链-20260918.md`
