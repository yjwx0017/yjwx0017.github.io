title: pugixml
categories: 日志
tags: [pugixml]
date: 2021-05-07 15:14:00
---
[https://pugixml.org/docs/quickstart.html][1]

## 加载XML文件

``` c++
pugi::xml_document doc;

pugi::xml_parse_result result = doc.load_file("tree.xml");

std::cout << "Load result: " << result.description() << ", mesh name: " << doc.child("mesh").attribute("name").value() << std::endl;
```

## 加载错误处理

``` c++
pugi::xml_document doc;
pugi::xml_parse_result result = doc.load_string(source);

if (result)
{
    std::cout << "XML [" << source << "] parsed without errors, attr value: [" << doc.child("node").attribute("attr").value() << "]\n\n";
}
else
{
    std::cout << "XML [" << source << "] parsed with errors, attr value: [" << doc.child("node").attribute("attr").value() << "]\n";
    std::cout << "Error description: " << result.description() << "\n";
    std::cout << "Error offset: " << result.offset << " (error at [..." << (source + result.offset) << "]\n\n";
}
```

## 从内存加载XML

``` c++
const char source[] = "<mesh name='sphere'><bounds>0 0 1 1</bounds></mesh>";
size_t size = sizeof(source);

// You can use load_buffer_inplace to load document from mutable memory block; the block's lifetime must exceed that of document
char* buffer = new char[size];
memcpy(buffer, source, size);

// The block can be allocated by any method; the block is modified during parsing
pugi::xml_parse_result result = doc.load_buffer_inplace(buffer, size);

// You have to destroy the block yourself after the document is no longer used
delete[] buffer;
```

<!--more-->

## 从 stream 中加载

``` c++
std::ifstream stream("weekly-utf-8.xml");
pugi::xml_parse_result result = doc.load(stream);
```

## 遍历子元素

``` c++
for (pugi::xml_node tool = tools.child("Tool"); tool; tool = tool.next_sibling("Tool"))
{
    std::cout << "Tool " << tool.attribute("Filename").value();
    std::cout << ": AllowRemote " << tool.attribute("AllowRemote").as_bool();
    std::cout << ", Timeout " << tool.attribute("Timeout").as_int();
    std::cout << ", Description '" << tool.child_value("Description") << "'\n";
}
```

## 使用迭代器遍历子元素

``` c++
for (pugi::xml_node_iterator it = tools.begin(); it != tools.end(); ++it)
{
    std::cout << "Tool:";

    for (pugi::xml_attribute_iterator ait = it->attributes_begin(); ait != it->attributes_end(); ++ait)
    {
        std::cout << " " << ait->name() << "=" << ait->value();
    }

    std::cout << std::endl;
}
```

## C++11 遍历子元素

``` c++
for (pugi::xml_node tool: tools.children("Tool"))
{
    std::cout << "Tool:";

    for (pugi::xml_attribute attr: tool.attributes())
    {
        std::cout << " " << attr.name() << "=" << attr.value();
    }

    for (pugi::xml_node child: tool.children())
    {
        std::cout << ", child " << child.name();
    }

    std::cout << std::endl;
}
```

## 使用 xml_tree_walker

``` c++
struct simple_walker: pugi::xml_tree_walker
{
    virtual bool for_each(pugi::xml_node& node)
    {
        for (int i = 0; i < depth(); ++i) std::cout << "  "; // indentation

        std::cout << node_types[node.type()] << ": name='" << node.name() << "', value='" << node.value() << "'\n";

        return true; // continue traversal
    }
};

simple_walker walker;
doc.traverse(walker);
```

## 使用 XPATH

``` c++
pugi::xpath_node_set tools = doc.select_nodes("/Profile/Tools/Tool[@AllowRemote='true' and @DeriveCaptionFrom='lastparam']");

std::cout << "Tools:\n";

for (pugi::xpath_node_set::const_iterator it = tools.begin(); it != tools.end(); ++it)
{
    pugi::xpath_node node = *it;
    std::cout << node.node().attribute("Filename").value() << "\n";
}

pugi::xpath_node build_tool = doc.select_node("//Tool[contains(Description, 'build system')]");

if (build_tool)
    std::cout << "Build tool: " << build_tool.node().attribute("Filename").value() << "\n";

// CAUTION
// XPath functions throw xpath_exception objects on error; 
// the sample above does not catch these exceptions.
```

## 修改数据

``` c++
pugi::xml_node node = doc.child("node");

// change node name
std::cout << node.set_name("notnode");
std::cout << ", new node name: " << node.name() << std::endl;

// change comment text
std::cout << doc.last_child().set_value("useless comment");
std::cout << ", new comment text: " << doc.last_child().value() << std::endl;

// we can't change value of the element or name of the comment
std::cout << node.set_value("1") << ", " << doc.last_child().set_name("2") << std::endl;
pugi::xml_attribute attr = node.attribute("id");

// change attribute name/value
std::cout << attr.set_name("key") << ", " << attr.set_value("345");
std::cout << ", new attribute: " << attr.name() << "=" << attr.value() << std::endl;

// we can use numbers or booleans
attr.set_value(1.234);
std::cout << "new attribute value: " << attr.value() << std::endl;

// we can also use assignment operators for more concise code
attr = true;
std::cout << "final attribute value: " << attr.value() << std::endl;

// CAUTION
// attribute() and child() functions do not add attributes or nodes
// to the tree, so code like node.attribute("id") = 123; will not do 
// anything if node does not have an attribute with name "id". 
// Make sure you’re operating with existing attributes/nodes by 
// adding them if necessary.
```

## 添加新属性或节点

``` c++
// add node with some name
pugi::xml_node node = doc.append_child("node");

// add description node with text child
pugi::xml_node descr = node.append_child("description");
descr.append_child(pugi::node_pcdata).set_value("Simple node");

// add param node before the description
pugi::xml_node param = node.insert_child_before("param", descr);

// add attributes to param node
param.append_attribute("name") = "version";
param.append_attribute("value") = 1.1;
param.insert_attribute_after("type", param.attribute("name")) = "float";
```

## 删除属性或节点

``` c++
// remove description node with the whole subtree
pugi::xml_node node = doc.child("node");
node.remove_child("description");

// remove id attribute
pugi::xml_node param = node.child("param");
param.remove_attribute("value");

// we can also remove nodes/attributes by handles
pugi::xml_attribute id = param.attribute("name");
param.remove_attribute(id);
```

## 保存到文件

``` c++
// save document to file
std::cout << "Saving result: " << doc.save_file("save_file_output.xml") << std::endl;
```

## 保存到 stream

``` c++
// save document to standard output
std::cout << "Document:\n";
doc.save(std::cout);
```

## 使用 xml_writer

``` c++
struct xml_string_writer: pugi::xml_writer
{
    std::string result;

    virtual void write(const void* data, size_t size)
    {
        result.append(static_cast<const char*>(data), size);
    }
};
```

``` c++
xml_string_writer writer;
doc.save(writer);
```

``` c++
xml_string_writer writer;
node.save(writer);
```

  [1]: https://pugixml.org/docs/quickstart.html