title: RapidJSON
categories: 日志
tags: [JSON,RapidJSON]
date: 2022-04-10 15:07:00
---
[http://rapidjson.org/zh-cn/][1]

## 用法一览

``` c++
// rapidjson/example/simpledom/simpledom.cpp
#include "rapidjson/document.h"
#include "rapidjson/writer.h"
#include "rapidjson/stringbuffer.h"
#include <iostream>
 
using namespace rapidjson;
 
int main() {
    // 1. Parse a JSON string into DOM.
    const char* json = "{\"project\":\"rapidjson\",\"stars\":10}";
    Document d;
    d.Parse(json);
 
    // 2. Modify it by DOM.
    Value& s = d["stars"];
    s.SetInt(s.GetInt() + 1);
 
    // 3. Stringify the DOM
    StringBuffer buffer;
    Writer<StringBuffer> writer(buffer);
    d.Accept(writer);
 
    // Output {"project":"rapidjson","stars":11}
    std::cout << buffer.GetString() << std::endl;
    return 0;
}
```

<!--more-->


## JSON Pointer

``` json
{
    "foo" : ["bar", "baz"],
    "pi" : 3.1416
}
```

``` c++
#include "rapidjson/pointer.h"
 
// ...
Document d;
 
// 使用 Set() 创建 DOM
Pointer("/project").Set(d, "RapidJSON");
Pointer("/stars").Set(d, 10);
 
// { "project" : "RapidJSON", "stars" : 10 }
 
// 使用 Get() 访问 DOM。若该值不存在则返回 nullptr。
if (Value* stars = Pointer("/stars").Get(d))
    stars->SetInt(stars->GetInt() + 1);
 
// { "project" : "RapidJSON", "stars" : 11 }
 
// Set() 和 Create() 自动生成父值（如果它们不存在）。
Pointer("/a/b/0").Create(d);
 
// { "project" : "RapidJSON", "stars" : 11, "a" : { "b" : [ null ] } }
 
// GetWithDefault() 返回引用。若该值不存在则会深拷贝缺省值。
Value& hello = Pointer("/hello").GetWithDefault(d, "world");
 
// { "project" : "RapidJSON", "stars" : 11, "a" : { "b" : [ null ] }, "hello" : "world" }
 
// Swap() 和 Set() 相似
Value x("C++");
Pointer("/hello").Swap(d, x);
 
// { "project" : "RapidJSON", "stars" : 11, "a" : { "b" : [ null ] }, "hello" : "C++" }
// x 变成 "world"
 
// 删去一个成员或元素，若值存在返回 true
bool success = Pointer("/a").Erase(d);
assert(success);
 
// { "project" : "RapidJSON", "stars" : 10 }
```

辅助函数：

``` c++
Document d;
 
SetValueByPointer(d, "/project", "RapidJSON");
SetValueByPointer(d, "/stars", 10);
 
if (Value* stars = GetValueByPointer(d, "/stars"))
    stars->SetInt(stars->GetInt() + 1);
 
CreateValueByPointer(d, "/a/b/0");
 
Value& hello = GetValueByPointerWithDefault(d, "/hello", "world");
 
Value x("C++");
SwapValueByPointer(d, "/hello", x);
 
bool success = EraseValueByPointer(d, "/a");
assert(success);
```



  [1]: http://rapidjson.org/zh-cn/