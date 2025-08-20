# "B" 位操作扩展，版本 1.0.0

## 目录

- [Zb* 概述](#zb-概述)
- [字指令](#字指令)
- [扩展详述](#扩展详述)
  - [Zba: 地址生成指令](#zba-地址生成指令)
  - [Zbb: 基础位操作](#zbb-基础位操作)
  - [Zbc: 进位无关乘法](#zbc-进位无关乘法)
  - [Zbs: 单位操作](#zbs-单位操作)

## 摘要

B 标准扩展由 Zba、Zbb 和 Zbs 扩展提供的指令组成。位操作扩展集合包含对基础 RISC-V 架构的多个组件扩展，旨在提供代码大小减少、性能改进和能耗降低的某种组合。本扩展为密码学、图像处理、数字信号处理等领域提供了重要的位操作原语。

### 涉及的指令格式

位操作扩展主要使用 R-type 指令格式：

#### R-type 格式（位操作）
```
[31:25] funct7 | [24:20] rs2 | [19:15] rs1 | [14:12] funct3 | [11:7] rd | [6:0] opcode
   7位         |    5位      |    5位      |    3位        |   5位   |   7位

示例: ANDN x1, x2, x3  (按位与非)
funct7=0100000 | rs2=x3 | rs1=x2 | funct3=111 | rd=x1 | opcode=0110011
结果: x1 = x2 & (~x3)
```

#### I-type 格式（立即数位操作）
```
[31:20] imm[11:0] | [19:15] rs1 | [14:12] funct3 | [11:7] rd | [6:0] opcode
    12位         |    5位      |    3位        |   5位   |   7位

示例: SLLI.UW x1, x2, 5  (无符号字左移)
imm=000000000101 | rs1=x2 | funct3=001 | rd=x1 | opcode=0011011
结果: x1 = (x2[31:0] << 5)  // 零扩展32位后左移
```

### 扩展指令列表

| 扩展 | 指令 | 格式 | RV32 | RV64 | 功能描述 |
|------|------|------|------|------|----------|
| **Zba** | ADD.UW | R-type | - | ✓ | rd = rs1[31:0] + rs2 (无符号字加法) |
| Zba | SH1ADD | R-type | ✓ | ✓ | rd = (rs1 << 1) + rs2 |
| Zba | SH2ADD | R-type | ✓ | ✓ | rd = (rs1 << 2) + rs2 |
| Zba | SH3ADD | R-type | ✓ | ✓ | rd = (rs1 << 3) + rs2 |
| Zba | SLLI.UW | I-type | - | ✓ | rd = rs1[31:0] << imm |
| **Zbb** | ANDN | R-type | ✓ | ✓ | rd = rs1 & (~rs2) |
| Zbb | ORN | R-type | ✓ | ✓ | rd = rs1 \| (~rs2) |
| Zbb | XNOR | R-type | ✓ | ✓ | rd = rs1 ^ (~rs2) |
| Zbb | CLZ | R-type | ✓ | ✓ | rd = 前导零计数 |
| Zbb | CTZ | R-type | ✓ | ✓ | rd = 尾随零计数 |
| Zbb | CPOP | R-type | ✓ | ✓ | rd = 人口计数(置位数) |
| Zbb | MAX | R-type | ✓ | ✓ | rd = max(rs1, rs2) 有符号 |
| Zbb | MAXU | R-type | ✓ | ✓ | rd = max(rs1, rs2) 无符号 |
| Zbb | MIN | R-type | ✓ | ✓ | rd = min(rs1, rs2) 有符号 |
| Zbb | MINU | R-type | ✓ | ✓ | rd = min(rs1, rs2) 无符号 |
| Zbb | SEXT.B | R-type | ✓ | ✓ | rd = SignExt(rs1[7:0]) |
| Zbb | SEXT.H | R-type | ✓ | ✓ | rd = SignExt(rs1[15:0]) |
| Zbb | ZEXT.H | R-type | ✓ | ✓ | rd = ZeroExt(rs1[15:0]) |
| Zbb | ROL | R-type | ✓ | ✓ | rd = rs1 循环左移 rs2 位 |
| Zbb | ROR | R-type | ✓ | ✓ | rd = rs1 循环右移 rs2 位 |
| Zbb | RORI | I-type | ✓ | ✓ | rd = rs1 循环右移 imm 位 |
| Zbb | REV8 | R-type | ✓ | ✓ | rd = 字节顺序反转 |
| Zbb | ORC.B | R-type | ✓ | ✓ | rd = 字节或归约 |
| **Zbc** | CLMUL | R-type | ✓ | ✓ | rd = rs1 ⊗ rs2 (低位无进位乘法) |
| Zbc | CLMULH | R-type | ✓ | ✓ | rd = (rs1 ⊗ rs2) >> XLEN |
| Zbc | CLMULR | R-type | ✓ | ✓ | rd = (rs1 ⊗ rs2) >> (XLEN-1) |
| **Zbs** | BCLR | R-type | ✓ | ✓ | rd = rs1 & ~(1 << rs2) |
| Zbs | BCLRI | I-type | ✓ | ✓ | rd = rs1 & ~(1 << imm) |
| Zbs | BEXT | R-type | ✓ | ✓ | rd = (rs1 >> rs2) & 1 |
| Zbs | BEXTI | I-type | ✓ | ✓ | rd = (rs1 >> imm) & 1 |
| Zbs | BINV | R-type | ✓ | ✓ | rd = rs1 ^ (1 << rs2) |
| Zbs | BINVI | I-type | ✓ | ✓ | rd = rs1 ^ (1 << imm) |
| Zbs | BSET | R-type | ✓ | ✓ | rd = rs1 \| (1 << rs2) |
| Zbs | BSETI | I-type | ✓ | ✓ | rd = rs1 \| (1 << imm) |

---

## Zb* 概述

位操作（bitmanip）扩展集合包含对基础 RISC-V 架构的多个组件扩展，旨在提供代码大小减少、性能改进和能耗降低的某种组合。虽然指令旨在通用，但某些指令在某些领域比其他领域更有用。因此，提供了几个较小的 bitmanip 扩展。每个这些较小的扩展按通用功能和用例分组，每个都有自己的 Zb* 扩展名称。

每个 bitmanip 扩展包含一组具有相似目的并且通常可以共享相同逻辑的 bitmanip 指令。一些指令仅在一个扩展中可用，而其他指令在多个扩展中可用。指令具有独立于它们出现的扩展的助记符和编码。因此，当实现具有重叠指令的扩展时，逻辑或编码中没有冗余。

## 字指令

bitmanip 扩展遵循 RV64 中的约定，_w_ 后缀指令（w 前面没有点）忽略其输入的高 32 位，将最低有效 32 位作为有符号值操作，并产生符号扩展到 XLEN 的 32 位有符号结果。

带有后缀 _.uw_ 的 Bitmanip 指令有一个操作数，它是从指定寄存器的最低有效 32 位提取的无符号 32 位值。除此之外，这些执行完整的 XLEN 操作。

带有后缀 _.b_、_.h_ 和 _.w_ 的 Bitmanip 指令只查看输入的最低有效 8 位、16 位和 32 位（分别），并产生基于特定指令符号扩展或零扩展的 XLEN 宽结果。

## 扩展详述

第一组发布进行公开评议的 bitmanip 扩展是：

- **Zba**: 地址生成指令
- **Zbb**: 基础位操作  
- **Zbc**: 进位无关乘法
- **Zbs**: 单位操作

### Zba: 地址生成指令

地址生成指令对于计算有效地址是有用的。全加器是很多处理器实现中非常常见的组件，并且地址生成通常在处理器执行的早期就需要。这些指令允许在地址生成单元内实现，减少延迟和能量。

虽然移位和加法指令限制为最大左移 3，但基础 ISA 中的 slli 指令可以用于执行类似的移位以索引更宽元素的数组。当索引要被解释为无符号字时，可以使用在此扩展中添加的 slli.uw。

#### Zba 指令详述

**SH1ADD, SH2ADD, SH3ADD**：移位加法指令
```
SH1ADD x1, x2, x3   # x1 = (x2 << 1) + x3
SH2ADD x1, x2, x3   # x1 = (x2 << 2) + x3  
SH3ADD x1, x2, x3   # x1 = (x2 << 3) + x3

用例：数组索引
- SH1ADD: 访问16位数组 (元素大小 = 2字节)
- SH2ADD: 访问32位数组 (元素大小 = 4字节)
- SH3ADD: 访问64位数组 (元素大小 = 8字节)
```

**ADD.UW** (仅RV64)：无符号字加法
```
ADD.UW x1, x2, x3   # x1 = x2[31:0] + x3
                     # 将x2的低32位作为无符号数与x3相加
```

**SLLI.UW** (仅RV64)：无符号字左移立即数
```
SLLI.UW x1, x2, 5   # x1 = x2[31:0] << 5
                     # 将x2的低32位零扩展后左移5位
```

### Zbb: 基础位操作

基础位操作扩展提供指令来操作RISC-V寄存器中的位。这些指令设计为位操作、算术和某些密码算法的构建块。

#### 逻辑与否定

**ANDN, ORN, XNOR**：逻辑与否定操作
```
ANDN x1, x2, x3    # x1 = x2 & (~x3)    按位与非
ORN x1, x2, x3     # x1 = x2 | (~x3)    按位或非  
XNOR x1, x2, x3    # x1 = x2 ^ (~x3)    按位异或非

用例示例：
ANDN - 清除特定位：x1 = x2 & (~mask)
ORN  - 设置特定位的补：x1 = x2 | (~mask)
XNOR - 比较操作：if (x2 XNOR x3) == all_ones then x2 == x3
```

#### 计数前导/尾随零位

**CLZ, CTZ**：零位计数
```
CLZ x1, x2     # x1 = 计算x2中前导零的数量
CTZ x1, x2     # x1 = 计算x2中尾随零的数量

示例：
x2 = 0x00F00000 (二进制: 00000000111100000000000000000000)
CLZ x1, x2  => x1 = 8  (前导零数量)
CTZ x1, x2  => x1 = 20 (尾随零数量)

用例：
- 找到最高有效位位置
- 找到最低有效位位置  
- 计算2的幂次
```

#### 人口计数

**CPOP**：计算置位数
```
CPOP x1, x2    # x1 = 计算x2中设置为1的位数

示例：
x2 = 0x0F0F0F0F (二进制: 00001111000011110000111100001111)
CPOP x1, x2  => x1 = 16  (共16个1位)

用例：
- 汉明重量计算
- 错误检测和纠正
- 密码学应用
```

#### 整数最小值/最大值

**MIN, MAX, MINU, MAXU**：最值比较
```
MAX x1, x2, x3     # x1 = max(x2, x3) 有符号比较
MIN x1, x2, x3     # x1 = min(x2, x3) 有符号比较  
MAXU x1, x2, x3    # x1 = max(x2, x3) 无符号比较
MINU x1, x2, x3    # x1 = min(x2, x3) 无符号比较

用例：
- 范围限制
- 饱和算术
- 条件选择而无分支
```

#### 符号和零扩展

**SEXT.B, SEXT.H, ZEXT.H**：扩展操作
```
SEXT.B x1, x2    # x1 = SignExt(x2[7:0])   8位符号扩展
SEXT.H x1, x2    # x1 = SignExt(x2[15:0])  16位符号扩展
ZEXT.H x1, x2    # x1 = ZeroExt(x2[15:0])  16位零扩展

替代实现：
SEXT.B ≡ 使用 slli + srai 的组合
SEXT.H ≡ 使用 slli + srai 的组合  
ZEXT.H ≡ 使用 slli + srli 的组合
```

#### 位旋转

**ROL, ROR, RORI**：循环移位
```
ROL x1, x2, x3     # x1 = x2 循环左移 x3 位
ROR x1, x2, x3     # x1 = x2 循环右移 x3 位  
RORI x1, x2, imm   # x1 = x2 循环右移 imm 位

示例：
x2 = 0x12345678, 循环右移4位
ROR x1, x2, 4  => x1 = 0x81234567

用例：
- 密码学算法
- 哈希函数
- 数据打包/解包
```

#### 字节操作

**REV8, ORC.B**：字节级操作
```
REV8 x1, x2    # x1 = 字节顺序反转(字节交换)
ORC.B x1, x2   # x1 = 每字节或归约

REV8示例 (RV32):
x2 = 0x12345678  => x1 = 0x78563412

ORC.B示例：
如果任何字节非零，则该字节变为0xFF，否则为0x00
```

### Zbc: 进位无关乘法

进位无关乘法对于错误检测代码和密码学是有用的。这些操作将操作数视为二进制多项式并在 GF(2) 上执行乘法。

**CLMUL, CLMULH, CLMULR**：进位无关乘法
```
CLMUL x1, x2, x3   # x1 = (x2 ⊗ x3)[XLEN-1:0]      低位部分
CLMULH x1, x2, x3  # x1 = (x2 ⊗ x3)[2*XLEN-1:XLEN] 高位部分  
CLMULR x1, x2, x3  # x1 = (x2 ⊗ x3)[2*XLEN-2:XLEN-1] 反向高位

算法：
result = 0
for i in 0 to XLEN-1:
    if (x2[i]):
        result ^= (x3 << i)

用例：
- CRC计算
- 有限域乘法
- 错误检测码
- 密码学基元
```

### Zbs: 单位操作

单位操作提供设置、清除、反转或提取寄存器中单个位的机制。位由其索引指定。

**BSET, BCLR, BINV, BEXT**：单位操作
```
BSET x1, x2, x3    # x1 = x2 | (1 << x3)     设置位
BCLR x1, x2, x3    # x1 = x2 & ~(1 << x3)    清除位
BINV x1, x2, x3    # x1 = x2 ^ (1 << x3)     反转位
BEXT x1, x2, x3    # x1 = (x2 >> x3) & 1     提取位

立即数版本：
BSETI x1, x2, imm  # x1 = x2 | (1 << imm)
BCLRI x1, x2, imm  # x1 = x2 & ~(1 << imm)  
BINVI x1, x2, imm  # x1 = x2 ^ (1 << imm)
BEXTI x1, x2, imm  # x1 = (x2 >> imm) & 1

用例：
- 位字段操作
- 标志设置/清除
- 位掩码操作
- 硬件寄存器控制
```

### 实现提示

**逻辑与否定指令** 可以通过反转基础要求的 AND、OR 和 XOR 逻辑指令的 _rs2_ 输入来实现。在某些实现中，用于减法的 rs2 上的反相器可以为此目的重用。

**地址生成指令** 可以在地址生成单元中实现，通过重用现有的全加器逻辑，移位操作可以通过简单的布线实现。

**单位操作** 可以通过移位器和简单逻辑门的组合有效实现。移位量可以重用现有移位指令的移位器。

---

位操作扩展为 RISC-V 处理器提供了强大的位级操作能力，特别适合密码学、数字信号处理、图像处理和系统编程等应用领域。这些指令通过专门的硬件支持可以显著提高性能并减少代码大小。