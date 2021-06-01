---
title: C++ 文件读写
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-24 18:10:29
password:
summary:
tags:
  - C/C++
categories:
---

一个读入文件的方法，这里调用了`iostream` 中的 `getline` 方法

```c++
#include <fstream>
#include <iostream>

using namespace std;

int main() {
    char buff[100];
    ifstream infile("t.txt");
    cout << "reading from t.txt" << endl;

    cout << "the data is: " << endl;
    while (!infile.eof()) {
        infile.getline(buff, 100);
        cout << buff << endl;
    }
    cout << "END!!" << endl;
    infile.close();
    return 0;
}
```
