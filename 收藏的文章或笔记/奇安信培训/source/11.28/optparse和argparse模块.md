**注意在 Python 中，`optparse` 模块用于解析命令行参数。然而，从 Python 2.7 开始，推荐使用更为强大和灵活的 `argparse` 模块，因为 `optparse` 模块在较新的 Python 版本中已经被标记为过时。`argparse` 提供了更多的功能，并且易于使用。**
### 一、optparse
```python
from optparse import OptionParser

def main():
    # 创建 OptionParser 实例
    parser = OptionParser("usage: %prog [options] arg1 arg2")

    # 添加命令行选项
    parser.add_option("-f", "--file", dest="filename", help="write report to FILE", metavar="FILE")
    parser.add_option("-q", "--quiet", action="store_false", dest="verbose", default=True, help="don't print status messages to stdout")

    # 解析命令行参数
    (options, args) = parser.parse_args()

    # 处理解析后的参数
    if options.verbose:
        print("Verbose mode enabled")
    if options.filename:
        print(f"Output file specified: {options.filename}")

    # 处理位置参数
    if len(args) > 0:
        print(f"Positional arguments: {args}")

if __name__ == "__main__":
    main()
```
在这个例子中，`OptionParser` 被用于定义命令行选项。然后通过 `add_option` 方法添加选项，其中 `-f` 和 `--file` 是选项的短格式和长格式，`dest` 指定了在解析后存储选项值的属性名称，`help` 提供了关于选项的描述信息。
在 `optparse` 中，`parser.parse_args()` 返回的是一个元组，其中包含两个元素：`options` 和 `args`。
- `options` 是一个包含所有选项值的对象，可以通过点运算符 `.` 来访问每个选项的值。这些选项值对应于你在 `OptionParser` 中使用 `add_option` 添加的选项。
- `args` 是一个列表，包含所有未被解析为选项的剩余的位置参数。这里的 "位置参数" 指的是那些没有带有 `-` 或 `--` 标志的参数。
**注：**`optparse` 模块主要关注处理命令行选项和可选参数，**而不提供直接定义位置参数的机制**。如果你需要处理位置参数，通常建议使用 `argparse` 模块
### 二、argparse
`argparse` 是 Python 标准库中用于解析命令行参数的模块。它提供了一种简单而灵活的方式来处理命令行输入，包括选项（可选参数）和参数（位置参数）。

以下是一个简单的使用 `argparse` 模块的例子：
```python
import argparse

def main():
    # 创建 ArgumentParser 实例
    parser = argparse.ArgumentParser(description='A script with command line arguments.')

    # 添加位置参数
    parser.add_argument('arg1', type=int, help='The first positional argument')
    parser.add_argument('arg2', type=float, help='The second positional argument')

    # 添加可选参数
    parser.add_argument('-o', '--optional', type=str, default='default_value', help='An optional argument')

    # 解析命令行参数
    args = parser.parse_args()

    # 处理位置参数
    print(f'arg1: {args.arg1}')
    print(f'arg2: {args.arg2}')

    # 处理可选参数
    print(f'optional: {args.optional}')

if __name__ == "__main__":
    main()

```
在这个例子中，`ArgumentParser` 实例是主要的命令行解析器。通过 `add_argument` 方法，你可以添加位置参数和可选参数。对于位置参数，你需要指定它的名称、类型、帮助信息等。对于可选参数，你可以指定短格式和长格式，还可以设置默认值、类型、帮助信息等。
运行脚本时，用户可以通过命令行提供相应的参数：
```python
python script.py 10 3.14 -o custom_value
```
在这个例子中，`10` 是 `arg1`，`3.14` 是 `arg2`，而 `-o custom_value` 是可选参数。`argparse` 会自动解析这些参数，并将它们存储在 `args` 对象中。
`argparse` 还提供了许多其他功能，如互斥组、子命令等，以满足更复杂的命令行参数需求。查看[官方文档](https://docs.python.org/3/library/argparse.html)以获取更详细的信息。