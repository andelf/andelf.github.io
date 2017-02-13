---
layout: post
title: "Fix Ansible Tower: stdout capture is missing"
date: 2016-12-07 16:33:20 +0800
comments: true
categories: linux ansible
---
Ansible Tower will report ``stdout capture is missing`` when restoring from previous backup.

Or run from docker?

(得，不装 B 英语了)

长话短说，之前要把 Ansible Tower 拆到 Docker 里，结果发现总不能正常执行。任务界面会提示：

    stdout capture is missing

检查发现是 celery 进程出错，用 root 启动 celery 倒是正常的。

最后发现是 docker 中的 supervisord 启动时缺乏部分环境变量，解决方法：

```
change supervisor/conf.d/tower.conf
ADD:
[program:awx-celeryd]
......
environment=HOME="/var/lib/awx",USER="awx"
......
```

是的，为找到原因，逆向了整个 Ansible Tower。

Ref: [GitHub Issue](https://github.com/ansible/ansible/issues/13904)