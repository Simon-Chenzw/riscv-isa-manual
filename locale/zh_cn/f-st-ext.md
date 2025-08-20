# "F" 单精度浮点扩展，版本 2.2

## 目录

- [F寄存器状态](#f寄存器状态)
- [浮点控制状态寄存器](#浮点控制状态寄存器)
- [NaN装箱](#nan装箱)
- [浮点加载存储指令](#浮点加载存储指令)
- [浮点算术指令](#浮点算术指令)
- [浮点转换指令](#浮点转换指令)
- [浮点比较指令](#浮点比较指令)
- [浮点分类指令](#浮点分类指令)

## 摘要

本章描述单精度浮点的标准指令集扩展，名为"F"，添加符合IEEE 754-2008算术标准的单精度浮点计算指令。F扩展依赖于"Zicsr"扩展进行控制和状态寄存器访问。F扩展为RISC-V架构提供了完整的IEEE 754兼容的单精度浮点运算能力。

### 涉及的指令格式

#### 浮点指令格式

**R-type格式（浮点算术）：**
```
[31:27] funct5 | [26:25] fmt | [24:20] rs2 | [19:15] rs1 | [14:12] rm | [11:7] rd | [6:0] opcode
   5位        |   2位       |    5位      |    5位      |   3位     |   5位   |   7位

示例: FADD.S f1, f2, f3
funct5=00000 | fmt=00 | rs2=f3 | rs1=f2 | rm=dyn | rd=f1 | opcode=1010011
结果: f1 = f2 + f3 (单精度浮点加法)
```

**R4-type格式（融合乘加）：**
```
[31:27] rs3 | [26:25] fmt | [24:20] rs2 | [19:15] rs1 | [14:12] rm | [11:7] rd | [6:0] opcode
   5位     |   2位       |    5位      |    5位      |   3位     |   5位   |   7位

示例: FMADD.S f1, f2, f3, f4
rs3=f4 | fmt=00 | rs2=f3 | rs1=f2 | rm=dyn | rd=f1 | opcode=1000011
结果: f1 = (f2 * f3) + f4 (融合乘加)
```

**I-type格式（浮点加载/转换）：**
```
[31:20] imm[11:0] | [19:15] rs1 | [14:12] funct3 | [11:7] rd | [6:0] opcode
    12位         |    5位      |    3位        |   5位   |   7位

示例: FLW f1, 8(x2)
imm=000000001000 | rs1=x2 | funct3=010 | rd=f1 | opcode=0000111
结果: f1 = M[x2 + 8] (从内存加载单精度浮点数)
```

### 主要指令列表

| 类型 | 指令 | 格式 | 功能描述 |
|------|------|------|----------|
| **加载存储** | FLW | I-type | rd = M[rs1 + imm] (加载字到浮点寄存器) |
| 加载存储 | FSW | S-type | M[rs1 + imm] = rs2 (存储浮点字) |
| **算术** | FADD.S | R-type | rd = rs1 + rs2 (单精度加法) |
| 算术 | FSUB.S | R-type | rd = rs1 - rs2 (单精度减法) |
| 算术 | FMUL.S | R-type | rd = rs1 × rs2 (单精度乘法) |
| 算术 | FDIV.S | R-type | rd = rs1 ÷ rs2 (单精度除法) |
| 算术 | FSQRT.S | R-type | rd = √rs1 (单精度平方根) |
| **融合乘加** | FMADD.S | R4-type | rd = (rs1 × rs2) + rs3 |
| 融合乘加 | FMSUB.S | R4-type | rd = (rs1 × rs2) - rs3 |
| 融合乘加 | FNMSUB.S | R4-type | rd = -(rs1 × rs2) + rs3 |
| 融合乘加 | FNMADD.S | R4-type | rd = -(rs1 × rs2) - rs3 |
| **符号注入** | FSGNJ.S | R-type | rd = rs1的值，rs2的符号位 |
| 符号注入 | FSGNJN.S | R-type | rd = rs1的值，rs2符号位取反 |
| 符号注入 | FSGNJX.S | R-type | rd = rs1的值，符号位异或 |
| **最值** | FMIN.S | R-type | rd = min(rs1, rs2) |
| 最值 | FMAX.S | R-type | rd = max(rs1, rs2) |
| **转换** | FCVT.W.S | R-type | rd = (int32_t)rs1 |
| 转换 | FCVT.WU.S | R-type | rd = (uint32_t)rs1 |
| 转换 | FCVT.S.W | R-type | rd = (float)rs1 |
| 转换 | FCVT.S.WU | R-type | rd = (float)rs1 (无符号) |
| **移动** | FMV.X.W | R-type | rd = rs1 (位模式不变) |
| 移动 | FMV.W.X | R-type | rd = rs1 (位模式不变) |
| **比较** | FEQ.S | R-type | rd = (rs1 == rs2) ? 1 : 0 |
| 比较 | FLT.S | R-type | rd = (rs1 < rs2) ? 1 : 0 |
| 比较 | FLE.S | R-type | rd = (rs1 <= rs2) ? 1 : 0 |
| **分类** | FCLASS.S | R-type | rd = 分类结果位向量 |

---

## F寄存器状态

F扩展添加32个浮点寄存器，`f0-f31`，每个32位宽，以及浮点控制和状态寄存器`fcsr`，它包含浮点单元的操作模式和异常状态。我们使用术语FLEN来描述RISC-V ISA中浮点寄存器的宽度，对于F单精度浮点扩展，FLEN=32。

### 浮点寄存器文件

| 寄存器 | ABI名称 | 描述 | 保存者 |
|--------|---------|------|--------|
| f0-f7 | ft0-ft7 | 浮点临时寄存器 | 调用者 |
| f8-f9 | fs0-fs1 | 浮点保存寄存器 | 被调用者 |
| f10-f11 | fa0-fa1 | 浮点参数/返回值寄存器 | 调用者 |
| f12-f17 | fa2-fa7 | 浮点参数寄存器 | 调用者 |
| f18-f27 | fs2-fs11 | 浮点保存寄存器 | 被调用者 |
| f28-f31 | ft8-ft11 | 浮点临时寄存器 | 调用者 |

> **注释：**
> 我们考虑了整数和浮点值的统一寄存器文件，因为这简化了软件寄存器分配和调用约定，并减少了总用户状态。然而，分离组织增加了给定指令宽度可访问的总寄存器数量，简化了为宽超标量发射提供足够寄存器文件端口，支持解耦浮点单元架构，并简化了内部浮点编码技术的使用。

## 浮点控制状态寄存器

浮点控制和状态寄存器`fcsr`是一个RISC-V控制和状态寄存器（CSR）。它是一个32位读/写寄存器，选择浮点算术操作的动态舍入模式并保持累积的异常标志。

### FCSR寄存器格式

```
[31:8] Reserved | [7:5] frm | [4:0] fflags
     24位      |   3位     |    5位

frm (浮点舍入模式):
000 = RNE (舍入到最近偶数)
001 = RTZ (舍入到零)  
010 = RDN (舍入向下)
011 = RUP (舍入向上)
100 = RMM (舍入到最近，远离零)
101-111 = 保留

fflags (浮点异常标志):
[4] NV = 无效操作
[3] DZ = 除零
[2] OF = 溢出
[1] UF = 下溢
[0] NX = 不精确
```

### 舍入模式详述

**RNE（舍入到最近偶数）：**
- 舍入到最近的可表示值
- 当恰好在中间时，选择最低有效位为0的值
- IEEE 754默认模式

**RTZ（舍入到零）：**
- 总是朝零方向舍入
- 等效于截断

**RDN（舍入向下）：**
- 总是朝负无穷方向舍入
- 等效于向下取整

**RUP（舍入向上）：**
- 总是朝正无穷方向舍入  
- 等效于向上取整

**RMM（舍入到最近，远离零）：**
- 舍入到最近的可表示值
- 当恰好在中间时，远离零舍入

### CSR访问指令

```
FRCSR rd        # rd = fcsr (读取完整FCSR)
FSCSR rd, rs1   # swap rd ↔ fcsr
FRFLAGS rd      # rd = fcsr[4:0] (读取异常标志)
FSFLAGS rd, rs1 # swap rd ↔ fcsr[4:0]  
FRRM rd         # rd = fcsr[7:5] (读取舍入模式)
FSRM rd, rs1    # swap rd ↔ fcsr[7:5]
```

## NaN装箱

当FLEN > 32时，单精度浮点值必须进行NaN装箱在浮点寄存器中以保持与其他浮点扩展的兼容性。

### NaN装箱规则

```
单精度值在64位寄存器中的存储：
[63:32] = 0xFFFFFFFF | [31:0] = 单精度值

检查有效性：
if (reg[63:32] == 0xFFFFFFFF):
    值是有效的单精度数
else:
    值被视为标准NaN (0x7FC00000)
```

## 浮点加载存储指令

浮点加载和存储指令在浮点寄存器和内存之间传输浮点值。

### 浮点加载

**FLW（浮点加载字）：**
```
FLW rd, offset(rs1)
功能: rd = M[rs1 + imm]

内存格式: IEEE 754单精度（32位）
- [31] 符号位
- [30:23] 指数 (偏置127)
- [22:0] 尾数（隐含前导1）

示例: FLW f1, 16(x2)  # f1 = M[x2 + 16]
```

### 浮点存储

**FSW（浮点存储字）：**
```
FSW rs2, offset(rs1)
功能: M[rs1 + imm] = rs2[31:0]

示例: FSW f3, 20(x4)  # M[x4 + 20] = f3
```

## 浮点算术指令

### 基本算术

**FADD.S（单精度加法）：**
```
FADD.S rd, rs1, rs2
功能: rd = rs1 + rs2

异常: 无效操作, 溢出, 下溢, 不精确

示例: FADD.S f1, f2, f3  # f1 = f2 + f3
```

**FSUB.S（单精度减法）：**
```
FSUB.S rd, rs1, rs2
功能: rd = rs1 - rs2

示例: FSUB.S f1, f2, f3  # f1 = f2 - f3
```

**FMUL.S（单精度乘法）：**
```
FMUL.S rd, rs1, rs2
功能: rd = rs1 × rs2

示例: FMUL.S f1, f2, f3  # f1 = f2 * f3
```

**FDIV.S（单精度除法）：**
```
FDIV.S rd, rs1, rs2
功能: rd = rs1 ÷ rs2

异常: 无效操作, 除零, 溢出, 下溢, 不精确

示例: FDIV.S f1, f2, f3  # f1 = f2 / f3
```

**FSQRT.S（单精度平方根）：**
```
FSQRT.S rd, rs1
功能: rd = √rs1

异常: 无效操作（负数）, 不精确

示例: FSQRT.S f1, f2  # f1 = sqrt(f2)
```

### 融合乘加指令

融合乘加指令执行乘法然后加法，作为单个操作进行舍入，提供更高的精度。

**FMADD.S（融合乘加）：**
```
FMADD.S rd, rs1, rs2, rs3
功能: rd = (rs1 × rs2) + rs3

优势: 只进行一次舍入，精度更高

示例: FMADD.S f1, f2, f3, f4  # f1 = (f2 * f3) + f4
```

**FMSUB.S（融合乘减）：**
```
FMSUB.S rd, rs1, rs2, rs3
功能: rd = (rs1 × rs2) - rs3

示例: FMSUB.S f1, f2, f3, f4  # f1 = (f2 * f3) - f4
```

**FNMSUB.S（融合负乘加）：**
```
FNMSUB.S rd, rs1, rs2, rs3
功能: rd = -(rs1 × rs2) + rs3

示例: FNMSUB.S f1, f2, f3, f4  # f1 = -(f2 * f3) + f4
```

**FNMADD.S（融合负乘减）：**
```
FNMADD.S rd, rs1, rs2, rs3
功能: rd = -(rs1 × rs2) - rs3

示例: FNMADD.S f1, f2, f3, f4  # f1 = -(f2 * f3) - f4
```

### 符号注入指令

符号注入指令操作浮点数的符号位，而不影响其他位。

**FSGNJ.S（符号注入）：**
```
FSGNJ.S rd, rs1, rs2
功能: rd = |rs1| × sign(rs2)

示例: FSGNJ.S f1, f2, f3  # f1 = abs(f2) with sign of f3
```

**FSGNJN.S（符号注入取反）：**
```
FSGNJN.S rd, rs1, rs2
功能: rd = |rs1| × (-sign(rs2))

示例: FSGNJN.S f1, f2, f3  # f1 = abs(f2) with opposite sign of f3
```

**FSGNJX.S（符号注入异或）：**
```
FSGNJX.S rd, rs1, rs2
功能: rd[31] = rs1[31] ^ rs2[31], rd[30:0] = rs1[30:0]

用途: 实现浮点绝对值和取反
FSGNJX.S f1, f2, f2  # f1 = abs(f2)
FSGNJN.S f1, f2, f2  # f1 = -f2
```

### 最值指令

**FMIN.S, FMAX.S（最小值、最大值）：**
```
FMIN.S rd, rs1, rs2  # rd = min(rs1, rs2)
FMAX.S rd, rs1, rs2  # rd = max(rs1, rs2)

NaN处理:
- 如果任一操作数是NaN，返回另一个操作数
- 如果两个都是NaN，返回标准NaN
- -0.0 < +0.0

示例: FMIN.S f1, f2, f3  # f1 = min(f2, f3)
```

## 浮点转换指令

### 浮点到整数转换

**FCVT.W.S（浮点到有符号整数）：**
```
FCVT.W.S rd, rs1, rm
功能: rd = (int32_t)rs1

舍入模式可以是立即数或动态(dyn)
异常: 无效操作（NaN, 无穷, 溢出）, 不精确

示例: FCVT.W.S x1, f2, rtz  # x1 = (int)f2, 截断
```

**FCVT.WU.S（浮点到无符号整数）：**
```
FCVT.WU.S rd, rs1, rm
功能: rd = (uint32_t)rs1

示例: FCVT.WU.S x1, f2, rne  # x1 = (unsigned)f2
```

### 整数到浮点转换

**FCVT.S.W（有符号整数到浮点）：**
```
FCVT.S.W rd, rs1, rm
功能: rd = (float)rs1

示例: FCVT.S.W f1, x2, rne  # f1 = (float)x2
```

**FCVT.S.WU（无符号整数到浮点）：**
```
FCVT.S.WU rd, rs1, rm
功能: rd = (float)(uint32_t)rs1

示例: FCVT.S.WU f1, x2, rne  # f1 = (float)(unsigned)x2
```

### 位模式移动

**FMV.X.W（浮点寄存器到整数寄存器）：**
```
FMV.X.W rd, rs1
功能: rd = rs1 (位模式完全相同，无转换)

用途: 检查浮点位模式、实现特殊函数

示例: FMV.X.W x1, f2  # x1 = f2的位模式
```

**FMV.W.X（整数寄存器到浮点寄存器）：**
```
FMV.W.X rd, rs1
功能: rd = rs1 (位模式完全相同，无转换)

示例: FMV.W.X f1, x2  # f1 = x2的位模式解释为浮点数
```

## 浮点比较指令

浮点比较指令比较两个浮点值并将比较结果存储在整数寄存器中。

**FEQ.S（浮点相等）：**
```
FEQ.S rd, rs1, rs2
功能: rd = (rs1 == rs2) ? 1 : 0

NaN处理: 任何操作数是NaN时返回0
异常: 无效操作（信号NaN）

示例: FEQ.S x1, f2, f3  # x1 = (f2 == f3)
```

**FLT.S（浮点小于）：**
```
FLT.S rd, rs1, rs2
功能: rd = (rs1 < rs2) ? 1 : 0

NaN处理: 任何操作数是NaN时返回0
异常: 无效操作（任何NaN）

示例: FLT.S x1, f2, f3  # x1 = (f2 < f3)
```

**FLE.S（浮点小于等于）：**
```
FLE.S rd, rs1, rs2
功能: rd = (rs1 <= rs2) ? 1 : 0

示例: FLE.S x1, f2, f3  # x1 = (f2 <= f3)
```

## 浮点分类指令

**FCLASS.S（浮点分类）：**
```
FCLASS.S rd, rs1
功能: rd = 分类结果位向量

返回值位含义:
[0] = 负无穷
[1] = 负正常数
[2] = 负次正常数  
[3] = 负零
[4] = 正零
[5] = 正次正常数
[6] = 正正常数
[7] = 正无穷
[8] = 信号NaN
[9] = 静默NaN

示例: FCLASS.S x1, f2
if (x1 & 0x001): print("负无穷")
if (x1 & 0x200): print("静默NaN")
```

## 异常处理

### 浮点异常类型

1. **无效操作（NV）**：
   - NaN参与比较
   - 无穷 - 无穷
   - 0 × 无穷
   - 负数开平方根

2. **除零（DZ）**：
   - 有限非零数除以零

3. **溢出（OF）**：
   - 结果大小超过最大可表示值

4. **下溢（UF）**：
   - 结果大小小于最小可表示正常数且结果不精确

5. **不精确（NX）**：
   - 结果无法精确表示

### IEEE 754特殊值

```
正零:     0x00000000
负零:     0x80000000
正无穷:   0x7F800000
负无穷:   0xFF800000
静默NaN:  0x7FC00000 (及其他具有指数=255, 尾数≠0且MSB=1的值)
信号NaN:  0x7FA00000 (指数=255, 尾数≠0且MSB=0)
```

---

F扩展为RISC-V提供了完整的IEEE 754兼容单精度浮点运算能力，包括所有标准算术操作、转换、比较和分类功能。其设计强调性能、标准兼容性和实现灵活性，使其适用于从嵌入式系统到高性能科学计算的广泛应用。