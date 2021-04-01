---
title: Cache 一致性协议
top: false
cover: false
toc: true
mathjax: true
date: 2021-04-01 17:30:45
password:
summary:
tags:
  - 体系结构
categories:
---

## MESI

处理器上有一套完整的协议，来保证 Cache 一致性。比较经典的 Cache 一致性协议当属 MESI 协议，奔腾处理器有使用它，很多其他的处理器都是使用它的变种。

单核 Cache 中每个 Cache line 有 2 个标志：dirty 和 valid 标志，它们很好的描述了 Cache 和 Memory(内存)之间的数据关系(数据是否有效，数据是否被修改)，而在多核处理器中，多个核会共享一些数据，MESI 协议就包含了描述共享的状态。

在 MESI 协议中，每个 Cache line 有 4 个状态，可用 2 个 bit 表示，它们分别是：

| 状态         | 描述                                                                        |
| ------------ | --------------------------------------------------------------------------- |
| M(Modified)  | 这行数据有效，数据被修改了，和内存中的数据不一致，数据只存在于本 Cache 中。 |
| E(Exclusive) | 这行数据有效，数据和内存中的数据一致，数据只存在于本 Cache 中。             |
| S(Shared)    | 这行数据有效，数据和内存中的数据一致，数据存在于很多 Cache 中。             |
| I(Invalid)   | 这行数据无效。                                                              |

M(Modified)和 E(Exclusive)状态的 Cache line，数据是独有的，不同点在于 M 状态的数据是 dirty 的(和内存的不一致)，E 状态的数据是 clean 的(和内存的一致)。

S(Shared)状态的 Cache line，数据和其他 Core 的 Cache 共享。只有 clean 的数据才能被多个 Cache 共享。

I(Invalid)表示这个 Cache line 无效。

当内核需要访问的数据不在本 Cache 中，而其它 Cache 有这份数据的备份时，本 Cache 既可以从内存中导入数据，也可以从其它 Cache 中导入数据，不同的处理器会有不同的选择。

MESI 协议为了使自己更加通用，没有定义这些细节，只定义了状态之间的迁移，下面的描述假设本 Cache 从内存中导入数据。

### Invalid:

| 事件         | 行为                                                                                                                                                                                                                                                                                                                   | next state |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------: |
| local Read   | 如果其它 Cache 没有这份数据，本 Cache 从内存中取数据，Cache line 状态变成 E；如果其它 Cache 有这份数据，且状态为 M，则将数据更新到内存，本 Cache 再从内存中取数据，2 个 Cache 的 Cache line 状态都变成 S；如果其它 Cache 有这份数据，且状态为 S 或者 E，本 Cache 从内存中取数据，这些 Cache 的 Cache line 状态都变成 S |    E/S     |
| Local Write  | 从内存中取数据，在 Cache 中修改，状态变成 M；如果其它 Cache 有这份数据，且状态为 M，则要先将数据更新到内存；如果其它 Cache 有这份数据，则其它 Cache 的 Cache line 状态变成 I                                                                                                                                           |     M      |
| Remote Read  | 既然是 Invalid，别的核的操作与它无关                                                                                                                                                                                                                                                                                   |     I      |
| Remote Write | 既然是 Invalid，别的核的操作与它无关                                                                                                                                                                                                                                                                                   |     I      |

### Exclusive:

| 事件         | 行为                                             | next state |
| ------------ | ------------------------------------------------ | :--------: |
| Local Read   | 从 Cache 中取数据，状态不变                      |     E      |
| Local Write  | 修改 Cache 中的数据，状态变成 M                  |     M      |
| Remote Read  | 数据和其它核共用，状态变成了 S                   |     S      |
| Remote Write | 数据被修改，本 Cache line 不能再使用，状态变成 I |     I      |

### Shared:

| 事件         | 行为                                                                | next state |
| ------------ | ------------------------------------------------------------------- | :--------: |
| Local Read   | 从 Cache 中取数据，状态不变                                         |     S      |
| Local Write  | 修改 Cache 中的数据，状态变成 M，其它核共享的 Cache line 状态变成 I |     M      |
| Remote Read  | 状态不变                                                            |     S      |
| Remote Write | 数据被修改，本 Cache line 不能再使用，状态变成 I                    |     I      |

### Modified:

| 事件         | 行为                                                                                   | next state |
| ------------ | -------------------------------------------------------------------------------------- | :--------: |
| Local Read   | 从 Cache 中取数据，状态不变                                                            |     M      |
| Local Write  | 修改 Cache 中的数据，状态不变                                                          |     M      |
| Remote Read  | 这行数据被写到内存中，使其它核能使用到最新的数据，状态变成 S                           |     S      |
| Remote Write | 这行数据被写到内存中，使其它核能使用到最新的数据，由于其它核会修改这行数据，状态变成 I |     I      |

AMD 的 Opteron 处理器使用从 MESI 中演化出的 MOESI 协议，O(Owned)是 MESI 中 S 和 M 的一个合体，表示本 Cache line 被修改，和内存中的数据不一致，不过其它的核可以有这份数据的拷贝，状态为 S。

Intel 的 core i7 处理器使用从 MESI 中演化出的 MESIF 协议，F(Forward)从 Share 中演化而来，一个 Cache line 如果是 Forward 状态，它可以把数据直接传给其它内核的 Cache，而 Share 则不能。
