# Mango Windows 样例核验记录

日期：2026-11-05  
环境：Windows；本地 Mango Python 虚拟环境，Python 3.11  
目的：对 Mango 仓库自带的手工构造二进制样例进行多组功能核验，减少单个样例偶然成功的影响。

## 核验方法

从 Mango 仓库 package/tests/binaries 选取样例，使用本机 mango 命令分析，并按仓库测试文件 package/tests/test_basic_facts_for_handcrafted_binaries.py 中写明的预期值核对 JSON：候选闭包数量、sink 名称、调用地址、调用深度；对 execve/execlp 样例核对参数个数和可控参数索引。

这次是按官方测试用例逐个运行 CLI 并核对 JSON，**不是在 Windows 上运行了完整的官方测试套件**。

## 结果

| 样例 | 核验结果 | 说明 |
|---|---|---|
| simple | 通过 | system 与 execve sink、地址及深度符合预期 |
| looper | 通过 | system sink 地址及深度符合预期 |
| nested | 通过 | system sink 地址及深度符合预期 |
| sprintf_resolved_and_unresolved | 通过 | system sink 地址及深度符合预期 |
| layered | 通过 | 设置 --max-depth 10 后，深度 8 的结果符合预期 |
| off_shoot | 通过 | system sink 地址及深度符合预期 |
| heap | 通过 | system sink 地址及深度符合预期 |
| wrapper | 通过 | 两条预期闭包符合预期 |
| early_resolve | 通过 | 两条预期闭包符合预期 |
| execve | 通过 | 找到 3 个参数，可控参数索引为 2 |
| execlp | 通过 | 找到 3 个参数，可控参数索引为 2 |
| recursive | **不一致** | 作者测试期望两条深度均为 2；本机稳定得到深度 2 和 4。重复运行两次，结果相同 |
| nvram | **不一致／受 Windows 辅助脚本影响** | env_resolve 正确提取 ls -la 和 TOP；作者的 env_resolve --merge 调用了 Unix find，在本机 Windows 命令解析下失败。根据提取结果提供合并字典后，Mango 成功运行，但输出 2 条 system 闭包，而仓库预期 1 条 |

**汇总：13 个样例中，11 个与仓库预期一致，2 个存在差异。** 所有 Mango 进程均成功启动并产生分析 JSON；这能证明当前副本具备实际分析能力，但两项差异意味着不能据此宣称 Windows 结果与作者环境完全一致。

## 结论与范围

这组结果比单个冒烟测试更有说服力：它覆盖了普通调用链、嵌套调用、分层调用、堆数据、封装函数、早期解析以及 execve/execlp 参数识别等不同结构。

目前可表述为：**“Mango 已在 Windows + Python 3.11 环境安装并能运行；作者仓库手工样例中 11/13 项与预期一致，递归和 NVRAM 样例仍有差异。因此完成了多样例功能核验，但还没有完成作者指定环境下的完整复现。”**

作者的 Artifact Appendix 指定验证环境为 Ubuntu 22.04 x64、Python 3.11、pip 24 或更高。若要把差异归因到环境，下一步应在该 Linux 环境中复跑这两项及官方测试套件；这一步尚未完成。

## 结果文件

逐样例 Mango JSON 与日志保存在本目录的同名子目录中；递归复跑结果保存在 recursive_repeat_1 和 recursive_repeat_2。NVRAM 的原始 env_resolve 结果与 Mango 结果分别保存在 nvram/env 和 nvram/mango_with_env。