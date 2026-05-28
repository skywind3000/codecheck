# CodeCheck — 注释驱动的本地单元测试工具

![Python](https://img.shields.io/badge/Python-3.x-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Languages](https://img.shields.io/badge/Supports-C%20%7C%20C++%20%7C%20Python-orange)

> 测试数据随代码，告别反复复制粘贴

做 Online Judge 题目时，你是否也在反复做这些事？

- 复制粘贴测试数据到 stdin
- 手动维护 `input.txt` / `output.txt`
- 每次改完代码再跑一遍，逐个核对输出

CodeCheck 让测试数据直接写在源代码注释里，一键编译 + 验证，零额外文件。

## 为什么用 CodeCheck？

| 方式 | 操作步骤 | 数据管理 |
|------|---------|---------|
| 手动复制粘贴 | 每次运行都复制 | 分散，容易丢 |
| input.txt 重定向 | 需额外维护文件 | 数据和代码分离 |
| **CodeCheck** | **一条命令** | **数据就在代码里** |

## Quick Start

安装：

```bash
# 无需安装，直接运行
python codecheck.py hello.c
```

编写一个带嵌入式测试的 C 程序：

```c
#include <stdio.h>

int main() {
    int a, b;
    scanf("%d%d", &a, &b);
    printf("%d\n", a + b);
    return 0;
}

// @input: test1
// 10 20
// @output:
// 30

// @input: test2
// 5 7
// @output:
// 12
```

运行和验证：

```bash
# 编译并运行（默认模式，不比对输出）
python codecheck.py hello.c

# 验证所有测试用例
python codecheck.py -c hello.c

# 只验证第1个测试用例
python codecheck.py -c -1 hello.c

# 调试模式：用 @input 作为 stdin 运行，直接显示输出而不比对
python codecheck.py -d hello.c

# 调试第2个测试用例
python codecheck.py -d -2 hello.c
```

样例输出：

```
Compiling hello.c ...
[1/2] Running unit test: test1 ... PASS
[2/2] Running unit test: test2 ... PASS
[result] All 2 unit tests passed!
```

这是 `-c` 命令的运行结果，首先检测是否需要编译（源代码比可执行新），然后用多组测试数据来验证程序是否输出满足预期。

## 运行模式

| 模式 | 选项 | 说明 |
|------|------|------|
| **start** | （默认） | 编译并运行，不做测试验证 |
| **debug** | `-d` | 用 `@input` 作为 stdin 运行，直接显示输出内容而不对比 |
| **check** | `-c` | 逐个运行测试用例，比对实际输出与 `@output` |

## 测试指令

在源代码注释中嵌入以下指令：

### `@input` — 定义标准输入

指定测试名称及输入数据，后续连续注释行作为 stdin 内容：

```c
// @input: test1
// 10 20
```

Python 中使用 `#` 注释：

```python
# @input: test1
# 10 20
```

也可以不给名称，自动命名为 `test1`、`test2` ...：

```c
// @input:
// 10 20
```

### `@output` — 定义预期输出

紧接 `@input` 后的连续注释行作为预期输出：

```c
// @input: test1
// 10 20
// @output:
// 30
```

### `@args` — 定义命令行参数

为程序指定命令行参数：

```c
// @args: --verbose --count=10
```

使用 `-a` 选项运行：

```bash
python codecheck.py -a hello.c
```

### `@timeout` — 设置超时（秒）

```c
// @input: slow_test timeout=5
// 100000
// @output:
// done
// @timeout: 5
```

也支持在 `@input` 名称后用 `key=value` 指定：

```c
// @input: slow_test timeout=5
```

## 支持的语言

| 语言 | 扩展名 | 处理方式 | 注释格式 |
|------|--------|----------|----------|
| C | `.c` | 编译后运行 | `//` 和 `/* */` |
| C++ | `.cpp`, `.cc`, `.cxx` | 编译后运行 | `//` 和 `/* */` |
| Python | `.py`, `.pyw` | 直接运行 | `#` 和 `"""` |

## IDE 集成

### Dev-C++ 一键运行

1. 打开 Dev-C++，菜单 **工具 → 配置工具**
2. 添加一个新工具：
   - **标题**：CodeCheck 验证
   - **程序**：`python`（或 `python3`）
   - **参数**：`codecheck.py -c "$(FILENAME)"`
   - **工作目录**：`$(DIRECTORY)`
3. 保存后即可在 Dev-C++ 中一键运行验证

> 提示：将 `codecheck.py` 放到 PATH 环境变量中，或填写完整路径。

## 完整示例

### C 语言

```c
#include <stdio.h>

// @input: add_two_numbers
// 10 20
// @output:
// 30

// @input: another_case
// 5 7
// @output:
// 12

int main() {
    int a, b;
    scanf("%d%d", &a, &b);
    printf("%d\n", a + b);
    return 0;
}
```

### Python

```python
# @input: add_two_numbers
# 10 20
# @output:
# 30

# @input: another_case
# 5 7
# @output:
# 12

a, b = map(int, input().split())
print(a + b)
```

### 多行注释中的测试（C/C++）

```c
/*
@input: block_test
5 7

@output:
12
*/
```

## 命令行选项

```
Usage: python codecheck.py [options] <source-file>

  -h, --help     显示帮助信息
  -c, --check    验证嵌入式单元测试的输出
  -d, --debug    调试模式运行（不比对输出）
  -a, --args     使用 @args 指定的命令行参数运行
  -{num}         选择指定测试用例（1-based，配合 -c/-d 使用）
```

## 注释提取细节

- C/C++：正确识别 `//` 单行注释和 `/* */` 多行注释，忽略字符串内的注释符号
- Python：正确识别 `#` 注释，忽略字符串（包括三引号字符串、raw string、f-string、byte string）内的 `#` 符号
- 测试数据必须连续排列在指令后面，中间如果有其他注释或代码行则会中断数据收集

## 配置文件

配置文件路径：`~/.config/codecheck.ini`

```ini
[default]
cc = /usr/bin/gcc          # C/C++ 编译器路径
python = /usr/bin/python3  # Python 解释器路径
flags = -O2 -g -Wall       # 默认编译选项
cflags = ...               # C 专用编译选项
cxxflags = ...             # C++ 专用编译选项
ldflags = ...              # 链接选项
timeout = 10               # 默认超时时间（秒）
```

如果不指定 `flags`，编译时默认使用 `-O2 -g -Wall -lm`。

编译时会自动定义宏 `_CODECHECK=1`，可用于区分本地调试和正式提交：

```c
#ifdef _CODECHECK
    // 本地调试专用代码
#endif
```

## 许可证

[MIT License](LICENSE) © skywind3000