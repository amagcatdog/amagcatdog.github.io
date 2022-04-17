---
layout: wiki
title: c++ input output stream
cate1: c++
cate2: stream
description: c++ input output stream
keywords: c++, stream
type:
link:
---

c++ 标准的 I/O 库的类全景图：

![iostream](/images/wiki/iostream.gif)

其中，istream/ifstream/istringstream/cin 对应输入流，用于读出数据；ostream/ofstream/ostringstream 对应输出流，用于写入数据。iostream/fstream/stringstream对应输入输出流，既可以用于写入数据，也可以用于读出数据。这些输入输出都是带缓冲区的。

这里以 fstream 读写文件为例，记录使用过程中遇到的一些问题和注意事项：

```c++
#include <fstream>
using std::fstream;
using std::ios;
int main() {
    fstream file("test.txt", ios::in|ios::out|ios::binary);
    if (!file.is_open()) exit(1);
    // 读取开头4字节
    file.seekg(0, ios::beg);
    char file_head[4];
    file.read(file_head, 4);
    if (!file.fail())
    {
        std::cout << file_head << std::endl;
    }
    else
    {
        file.clear(); // 清除失败标记，否则后续操作都会失败
    }
    
    // 覆盖开头4字节
    file.seekp(0, ios::beg);
    for (int i = 0; i < 4; ++i)
        file_head[i] += 1;
    file.write(file_head, 4);
    file.flush();
    file.close();
    
    return 0;
}
```

## 文件打开方式

- ios::ate 表示打开文件并定位到末尾
- ios::trunc 表示打开文件并清空文件内容
- 默认以 ios::text 方式打开，二进制打开需要用 ios::binary
- fstream **修改**文件必须以 ios::in|ios::out 方式打开，如果文件不存在则打开失败
- fstream 读文件或文件末尾追加用 ios::in|ios::out|ios::app 方式打开，如果文件不存在则打开失败
- ifstream 读文件默认即为 ios::in 方式，如果文件不存在则打开失败
- ofstream 写文件默认即为 ios::out 方式，文件不存在则自动创建文件，如果文件存在则默认清空文件，需要保留文件内容且尾部追加用 ios::app

## 判断操作是否成功

- is_open() 用于检测文件是否打开成功
- good()/eof()/fail()/bad() 用于判断文件读写操作是否成功、是否到达文件末尾
- 如果文件操作失败，需要调用clear()清除文件操作状态才能继续进行读写

## 文件定位

- tellp() 输出流的当前位置
- seekp() 移动输出流当前位置
- tellg() 读入流的当前位置
- seekg() 移动读入流当前位置
- seekg(0, ios::beg) 移动读入流位置到开头
- seekp(0, ios::end) 移动写入流位置到末尾
- seekg(-1, ios::cur) 向文件开头位置移动读入流一个字符
- seekg(2, ios::beg) 移动读入流位置到开头的2个字符后