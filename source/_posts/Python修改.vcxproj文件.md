title: Python修改.vcxproj文件
categories: 日志
tags: [Python]
date: 2016-11-21 13:53:00
---
工作需要，修改一些项目配置，重复、枯燥、乏味，所以用Python写了个自动化脚本，记录一下：

``` python
#!/usr/bin/env python
# coding: utf-8

import os
import re

files = []
current_config = '' # e.g. Debug2010

def main():
    global current_config

    get_files()
    for filepath in files:
        print filepath
        tmp_file = filepath + '.tmp'
        try:
            os.remove(os.path.abspath(tmp_file))
        except:
            pass

        os.rename(filepath, tmp_file)
        with open(tmp_file, 'r') as tmp_f:
            with open(filepath, 'w') as f:
                for line in tmp_f.readlines():
                    # 确定当前配置
                    m = re.search(r'''<PropertyGroup Condition=.*=='(.*)\|''', line)
                    if m is not None:
                        current_config = m.group(1)
                        f.write(line)
                        continue

                    m = re.search(r'''<ItemDefinitionGroup Condition=.*=='(.*)\|''', line)
                    if m is not None:
                        current_config = m.group(1)
                        f.write(line)
                        continue

                    # 替换
                    line = replace(line)
                    if line:
                        f.write(line)

        os.remove(os.path.abspath(tmp_file))

def get_files():
    for dirpath, dirnames, filenames in os.walk('.'):
        for filename in filenames:
            _, ext = os.path.splitext(filename)
            if ext != '.vcxproj':
                continue
            filepath = os.path.join(dirpath, filename)
            files.append(filepath)

def replace(line):
    arx_version = ''
    lib_suffix = ''
    ret = line
    is_debug = False

    if current_config == 'Debug2010':
        arx_version = '2013'
        lib_suffix = '19'
        is_debug = True
    elif current_config == 'Release2010':
        arx_version = '2013'
        lib_suffix = '19'
    elif current_config == 'Debug2012':
        arx_version = '2015'
        lib_suffix = '20'
        is_debug = True
    elif current_config == 'Release2012':
        arx_version = '2015'
        lib_suffix = '20'

    if arx_version == '':
        return ret

    # 附加包含目录
    regex_str = r'<AdditionalIncludeDirectories>.*OBJECTARX\\.+</AdditionalIncludeDirectories>'
    m = re.search(regex_str, line)

    if m is not None:
        ret = re.sub(r'(OBJECTARX\\)([0-9]+?)(\\)', r'OBJECTARX\\%s\\' % arx_version, line)
        return ret

    # 附加依赖项
    regex_str = r'<AdditionalDependencies>.+</AdditionalDependencies>'
    m = re.search(regex_str, line)
    if m is not None:
        if is_debug:
            ret = re.sub(r'ZW([a-zA-Z]+?)d\.lib', r'ZW\g<1>%sd.lib' % lib_suffix, line)
        else:
            ret = re.sub(r'ZW([a-zA-Z]+?)\.lib', r'ZW\g<1>%s.lib' % lib_suffix, line)

    return ret

if __name__ == '__main__':
    main()
```