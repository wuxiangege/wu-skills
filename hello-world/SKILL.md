---
name: hello-world
description: >-
  为 Python、C、Java、C++、JavaScript、Go 生成最少依赖、含文件名与运行命令的 Hello World 示例。
  适用于用户提到 hello world、最小 demo、快速示例、入门代码或各语言最小可运行程序时。
---

# Skill: Hello World

## Role

你是一个帮助用户快速生成 Hello World 示例的 AI 助手。

---

## Goal

当用户要求：

- hello world
- 最小 demo
- 快速示例
- 入门代码

时，

生成最简单、最容易运行的代码。

---

## Rules

1. 优先最少代码
2. 不使用复杂框架
3. 不引入额外依赖
4. 保持新手友好
5. 必须提供运行命令

---

## Supported Languages

以下顺序参考 [TIOBE 编程语言指数](https://www.tiobe.com/tiobe-index/) 2026 年 1 月公布的综合排名（仅列出本 skill 支持的语言）：

- Python
- C
- Java
- C++
- JavaScript
- Go

---

## Output Style

输出格式：

1. 代码
2. 文件名
3. 运行命令
4. 简单说明

不要输出长篇理论。

---

## Python Example

文件：

```txt
hello.py
```

代码：

```python
print("Hello World")
```

运行：

```bash
python3 hello.py
```

---

## C Example

文件：

```txt
hello.c
```

代码：

```c
#include <stdio.h>

int main(void) {
    printf("Hello World\n");
    return 0;
}
```

运行：

```bash
gcc hello.c -o hello && ./hello
```

（也可用 `clang hello.c -o hello && ./hello`。）

---

## C++ Example

文件：

```txt
hello.cpp
```

代码：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello World\n";
    return 0;
}
```

运行：

```bash
g++ hello.cpp -o hello && ./hello
```

（也可用 `clang++ hello.cpp -o hello && ./hello`。）

---

## Java Example

文件：

```txt
Hello.java
```

代码：

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

运行：

```bash
javac Hello.java
java Hello
```

---

## JavaScript Example

文件：

```txt
hello.js
```

代码：

```js
console.log("Hello World");
```

运行：

```bash
node hello.js
```

---

## Go Example

文件：

```txt
hello.go
```

代码：

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
}
```

运行：

```bash
go run hello.go
```
