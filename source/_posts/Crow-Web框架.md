title: Crow Web框架
categories: 日志
tags: [crow]
date: 2021-05-09 13:05:00
---
[https://crowcpp.org][1]

Crow is C++ microframework for web. (inspired by Python Flask)

## 入门

``` c++
#include "crow.h"

int main()
{
    crow::SimpleApp app;

    CROW_ROUTE(app, "/")([](){
        return "Hello world";
    });

    app.port(18080).multithreaded().run();
}
```

### JSON Response

``` c++
CROW_ROUTE(app, "/json")
([]{
    crow::json::wvalue x;
    x["message"] = "Hello, World!";
    return x;
});
```

### Arguments

``` c++
CROW_ROUTE(app,"/hello/<int>")
([](int count){
    if (count > 100)
        return crow::response(400);
    std::ostringstream os;
    os << count << " bottles of beer!";
    return crow::response(os.str());
});
```

Handler arguments type check at compile time

``` c++
// Compile error with message "Handler type is mismatched with URL paramters"
CROW_ROUTE(app,"/another/<int>")
([](int a, int b){
    return crow::response(500);
});
```

### Handling JSON Requests

``` c++
CROW_ROUTE(app, "/add_json")
.methods("POST"_method)
([](const crow::request& req){
    auto x = crow::json::load(req.body);
    if (!x)
        return crow::response(400);
    int sum = x["a"].i()+x["b"].i();
    std::ostringstream os;
    os << sum;
    return crow::response{os.str()};
});
```

<!--more-->

## 进阶

!!!
<iframe style="width:100%; height:600px; border:0px" src="https://crowcpp.org/guides/app/"></iframe>
!!!

  [1]: https://crowcpp.org/