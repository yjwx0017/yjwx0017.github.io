title: nlohmann-json
categories: 日志
tags: [JSON,nlohmann/json]
date: 2022-04-10 14:06:00
---
[https://github.com/nlohmann/json/][1]
[https://json.nlohmann.me/features/arbitrary_types/][2]

## 创建 json 对象

``` c++
// create an empty structure (null)
json j;

// add a number that is stored as double (note the implicit conversion of j to an object)
j["pi"] = 3.141;

// add a Boolean that is stored as bool
j["happy"] = true;

// add a string that is stored as std::string
j["name"] = "Niels";

// add another null object by passing nullptr
j["nothing"] = nullptr;

// add an object inside the object
j["answer"]["everything"] = 42;

// add an array that is stored as std::vector (using an initializer list)
j["list"] = { 1, 0, 2 };

// add another object (using an initializer list of pairs)
j["object"] = { {"currency", "USD"}, {"value", 42.99} };

// instead, you could also write (which looks very similar to the JSON above)
json j2 = {
  {"pi", 3.141},
  {"happy", true},
  {"name", "Niels"},
  {"nothing", nullptr},
  {"answer", {
    {"everything", 42}
  }},
  {"list", {1, 0, 2}},
  {"object", {
    {"currency", "USD"},
    {"value", 42.99}
  }}
};
```

<!--more-->


## 明确指定类型

``` c++
// a way to express the empty array []
json empty_array_explicit = json::array();

// ways to express the empty object {}
json empty_object_implicit = json({});
json empty_object_explicit = json::object();

// a way to express an _array_ of key/value pairs [["currency", "USD"], ["value", 42.99]]
json array_not_object = json::array({ {"currency", "USD"}, {"value", 42.99} });
```

## 通过字符串字面构造 json 对象

``` c++
// create object from string literal
json j = "{ \"happy\": true, \"pi\": 3.141 }"_json;

// or even nicer with a raw string literal
auto j2 = R"(
  {
    "happy": true,
    "pi": 3.141
  }
)"_json;

// parse explicitly
auto j3 = json::parse(R"({"happy": true, "pi": 3.141})");
```

## json 对象转为字符串

``` c++
// explicit conversion to string
std::string s = j.dump();    // {"happy":true,"pi":3.141}

// serialization with pretty printing
// pass in the amount of spaces to indent
std::cout << j.dump(4) << std::endl;
// {
//     "happy": true,
//     "pi": 3.141
// }
```

## 输入输出流

``` c++
// read a JSON file
std::ifstream i("file.json");
json j;
i >> j;

// write prettified JSON to another file
std::ofstream o("pretty.json");
o << std::setw(4) << j << std::endl;
```

## STL风格 API

``` c++
// create an array using push_back
json j;
j.push_back("foo");
j.push_back(1);
j.push_back(true);

// also use emplace_back
j.emplace_back(1.78);

// iterate the array
for (json::iterator it = j.begin(); it != j.end(); ++it) {
  std::cout << *it << '\n';
}

// range-based for
for (auto& element : j) {
  std::cout << element << '\n';
}

// getter/setter
const auto tmp = j[0].get<std::string>();
j[1] = 42;
bool foo = j.at(2);

// comparison
j == R"(["foo", 1, true, 1.78])"_json;  // true

// other stuff
j.size();     // 4 entries
j.empty();    // false
j.type();     // json::value_t::array
j.clear();    // the array is empty again

// convenience type checkers
j.is_null();
j.is_boolean();
j.is_number();
j.is_object();
j.is_array();
j.is_string();

// create an object
json o;
o["foo"] = 23;
o["bar"] = false;
o["baz"] = 3.141;

// also use emplace
o.emplace("weather", "sunny");

// special iterator member functions for objects
for (json::iterator it = o.begin(); it != o.end(); ++it) {
  std::cout << it.key() << " : " << it.value() << "\n";
}

// the same code as range for
for (auto& el : o.items()) {
  std::cout << el.key() << " : " << el.value() << "\n";
}

// even easier with structured bindings (C++17)
for (auto& [key, value] : o.items()) {
  std::cout << key << " : " << value << "\n";
}

// find an entry
if (o.contains("foo")) {
  // there is an entry with key "foo"
}

// or via find and an iterator
if (o.find("foo") != o.end()) {
  // there is an entry with key "foo"
}

// or simpler using count()
int foo_present = o.count("foo"); // 1
int fob_present = o.count("fob"); // 0

// delete an entry
o.erase("foo");
```

## STL 容器转为 json 对象

``` c++
std::vector<int> c_vector {1, 2, 3, 4};
json j_vec(c_vector);
// [1, 2, 3, 4]

std::deque<double> c_deque {1.2, 2.3, 3.4, 5.6};
json j_deque(c_deque);
// [1.2, 2.3, 3.4, 5.6]

std::list<bool> c_list {true, true, false, true};
json j_list(c_list);
// [true, true, false, true]

std::forward_list<int64_t> c_flist {12345678909876, 23456789098765, 34567890987654, 45678909876543};
json j_flist(c_flist);
// [12345678909876, 23456789098765, 34567890987654, 45678909876543]

std::array<unsigned long, 4> c_array {{1, 2, 3, 4}};
json j_array(c_array);
// [1, 2, 3, 4]

std::set<std::string> c_set {"one", "two", "three", "four", "one"};
json j_set(c_set); // only one entry for "one" is used
// ["four", "one", "three", "two"]

std::unordered_set<std::string> c_uset {"one", "two", "three", "four", "one"};
json j_uset(c_uset); // only one entry for "one" is used
// maybe ["two", "three", "four", "one"]

std::multiset<std::string> c_mset {"one", "two", "one", "four"};
json j_mset(c_mset); // both entries for "one" are used
// maybe ["one", "two", "one", "four"]

std::unordered_multiset<std::string> c_umset {"one", "two", "one", "four"};
json j_umset(c_umset); // both entries for "one" are used
// maybe ["one", "two", "one", "four"]

std::map<std::string, int> c_map { {"one", 1}, {"two", 2}, {"three", 3} };
json j_map(c_map);
// {"one": 1, "three": 3, "two": 2 }

std::unordered_map<const char*, double> c_umap { {"one", 1.2}, {"two", 2.3}, {"three", 3.4} };
json j_umap(c_umap);
// {"one": 1.2, "two": 2.3, "three": 3.4}

std::multimap<std::string, bool> c_mmap { {"one", true}, {"two", true}, {"three", false}, {"three", true} };
json j_mmap(c_mmap); // only one entry for key "three" is used
// maybe {"one": true, "two": true, "three": true}

std::unordered_multimap<std::string, bool> c_ummap { {"one", true}, {"two", true}, {"three", false}, {"three", true} };
json j_ummap(c_ummap); // only one entry for key "three" is used
// maybe {"one": true, "two": true, "three": true}
```

## JSON Pointer

以路径方式访问元素：

``` c++
// a JSON value
json j_original = R"({
  "baz": ["one", "two", "three"],
  "foo": "bar"
})"_json;

// access members with a JSON pointer (RFC 6901)
j_original["/baz/1"_json_pointer];
// "two"
```

## 任意类型转换

对于自定义类型，通常会这么做：

``` c++
namespace ns {
    // a simple struct to model a person
    struct person {
        std::string name;
        std::string address;
        int age;
    };
}

ns::person p = {"Ned Flanders", "744 Evergreen Terrace", 60};

// convert to JSON: copy each value into the JSON object
json j;
j["name"] = p.name;
j["address"] = p.address;
j["age"] = p.age;

// ...

// convert from JSON: copy each value from the JSON object
ns::person p {
    j["name"].get<std::string>(),
    j["address"].get<std::string>(),
    j["age"].get<int>()
};
```

更好的方式：

``` c++
// create a person
ns::person p {"Ned Flanders", "744 Evergreen Terrace", 60};

// conversion: person -> json
json j = p;

std::cout << j << std::endl;
// {"address":"744 Evergreen Terrace","age":60,"name":"Ned Flanders"}

// conversion: json -> person
auto p2 = j.get<ns::person>();

// that's it
assert(p == p2);
```

只需要实现两个函数：

``` c++
using json = nlohmann::json;

namespace ns {
    void to_json(json& j, const person& p) {
        j = json{{"name", p.name}, {"address", p.address}, {"age", p.age}};
    }

    void from_json(const json& j, person& p) {
        j.at("name").get_to(p.name);
        j.at("address").get_to(p.address);
        j.at("age").get_to(p.age);
    }
} // namespace ns
```

两个函数的定义可使用更加方便的宏实现：

``` c++
namespace ns {
    NLOHMANN_DEFINE_TYPE_NON_INTRUSIVE(person, name, address, age)
}
```

侵入式宏：

``` c++
namespace ns {
    class address {
      private:
        std::string street;
        int housenumber;
        int postcode;

      public:
        NLOHMANN_DEFINE_TYPE_INTRUSIVE(address, street, housenumber, postcode)
    };
}
```

## 二进制格式（BSON, CBOR, MessagePack, and UBJSON）

``` c++
// create a JSON value
json j = R"({"compact": true, "schema": 0})"_json;

// serialize to BSON
std::vector<std::uint8_t> v_bson = json::to_bson(j);

// 0x1B, 0x00, 0x00, 0x00, 0x08, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0x00, 0x01, 0x10, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00

// roundtrip
json j_from_bson = json::from_bson(v_bson);

// serialize to CBOR
std::vector<std::uint8_t> v_cbor = json::to_cbor(j);

// 0xA2, 0x67, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0xF5, 0x66, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x00

// roundtrip
json j_from_cbor = json::from_cbor(v_cbor);

// serialize to MessagePack
std::vector<std::uint8_t> v_msgpack = json::to_msgpack(j);

// 0x82, 0xA7, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0xC3, 0xA6, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x00

// roundtrip
json j_from_msgpack = json::from_msgpack(v_msgpack);

// serialize to UBJSON
std::vector<std::uint8_t> v_ubjson = json::to_ubjson(j);

// 0x7B, 0x69, 0x07, 0x63, 0x6F, 0x6D, 0x70, 0x61, 0x63, 0x74, 0x54, 0x69, 0x06, 0x73, 0x63, 0x68, 0x65, 0x6D, 0x61, 0x69, 0x00, 0x7D

// roundtrip
json j_from_ubjson = json::from_ubjson(v_ubjson);
```

  [1]: https://github.com/nlohmann/json/
  [2]: https://json.nlohmann.me/features/arbitrary_types/