---
layout: post
title:  C++ リストの分割
date:   2024-11-17 01:00:00 +0900
categories: snipet
tags: cpp
mermaid: false
---

C++の仕事を始めました。

リストを分割するコードを記載します。

``` cpp
#include <list>

template<typename T>
std::list<std::list<T>> splitList(std::list<T> datas, int oneListSize) {
    const size_t dataNum = datas.size();
    const size_t listNum = (dataNum / oneListSize) + 1;
    std::list<std::list<T>> splitedList;

    size_t processedNum = 0;
    for (size_t i = 0; i < listNum; i++) {
        std::list<T> newList;

        processedNum += oneListSize;
        if (processedNum >= dataNum) {
            splitedList.push_back(datas);
            break;
        }

        auto itSplitEnd = datas.begin();
        std::advance(itSplitEnd, oneListSize);
        newList.splice(newList.begin(), datas, datas.begin(), itSplitEnd);
        splitedList.push_back(newList);
    }

    return splitedList;
}
```

listのメンバ関数splice()でリストの分割と結合ができます。

以上です。
