title: 探索PyGithub
categories: 日志
tags: [Python,PyGithub]
date: 2016-11-02 12:53:00
---
PyGithub是GitHub API的Python版，玩了一下，还是蛮有趣的。最初是在chromium-source-tarball项目中一个Python脚本里发现的，竟然能通过Python操作GitHub，再一次证明Python太强大了，TM简直啥都能做，哈哈。

写个小工具，GitHub里Follow我的，我也Follow回敬一下，礼尚往来。

## 安装PyGithub：

`pip install pygithub`

## 完整脚本

``` python
#!/usr/bin/env python
# coding: utf-8

# GitHub
# Follow我的, Follow之
# 未Follow我的，从Following中删除之
# 即同步互Follow关系

import getpass
from github import Github

# 始终Follow的大神
ALWAYS_FOLLOW = [
    'torvalds',
    'michaelliao',
    'zcbenz',
    'ruanyf',
]

def sync():
    username = raw_input('Username:')
    password = getpass.getpass()
    print 'Please wait...'

    g = Github(username, password)
    me = g.get_user()

    followers = me.get_followers()
    following = me.get_following()
    followers_names = [x.login for x in followers]
    following_names = [x.login for x in following]

    all_is_ok = True  # 意思是已是同步状态，无需操作
    for user in following_names:
        is_my_follower = user in followers_names
        in_always_list = user in ALWAYS_FOLLOW

        if (not is_my_follower) and (not in_always_list):
            me.remove_from_following(g.get_user(user))
            print 'User \'%s\' has been removed from following list!' % user
            all_is_ok = False

    for user in followers_names:
        if user not in following_names:
            me.add_to_following(g.get_user(user))
            print 'New user \'%s\' has been followed!' % user
            all_is_ok = False

    if all_is_ok:
        print 'All is OK!'

if __name__ == "__main__":
    sync()
```

还可以再完善一下，比如还没有加上异常处理。