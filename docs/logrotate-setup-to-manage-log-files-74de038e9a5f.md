# 日志旋转安装程序以管理日志文件

> 原文：<https://itnext.io/logrotate-setup-to-manage-log-files-74de038e9a5f?source=collection_archive---------3----------------------->

![](img/54d64a4778002269f904f706a17df72e.png)

马库斯·斯皮斯克在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的照片

一个晴朗的夜晚，您接到警报系统的电话，称其中一台生产服务器没有响应。您立即打开笔记本电脑并开始调试，以找到可能出错的地方。过了一段时间后，你发现它的磁盘是满的，因为日志没有被清除。由于日志对于调试和识别生产中的错误至关重要，所以您保留了它们。

如果日志管理不当，这可能就是你的故事。在当今世界，我们的应用程序会生成大量日志数据，并且可以创建 GB 大小的文件。大型日志文件很难调试，占用大量空间，并且会降低定期备份服务器映像和数据的速度。将日志文件保持在可管理的大小并删除无用的旧日志始终是一个好主意。为了实现这一点，我们需要一个工具来帮助我们所有的需求，这可以使用 logrotate 来完成。

根据 Logrotate [Github 页面](https://github.com/logrotate/logrotate):

> logrotate 实用程序旨在简化生成大量日志文件的系统上的日志文件管理。Logrotate 允许自动循环压缩、删除和邮寄日志文件。Logrotate 可以设置为每天、每周、每月或当日志文件达到一定大小时处理日志文件。

Logrotate 可以用来做以下事情:

1.  基于大小和时间旋转文件
2.  将日期添加到日志文件中，这有助于调试
3.  创建具有所需权限的新日志文件
4.  删除旧的日志文件
5.  在日志轮转前后运行自定义命令
6.  压缩日志文件以节省空间

在我们开始使用 logrotate 之前，让我们来看看一些用于设置的常见配置参数。

**日志文件**:我们通过列出日志文件路径，后跟一组所需的参数来定义旋转参数。在一个配置文件中，我们可以放置多个这样的配置。

**循环计数**:定义在开始删除日志文件之前，保留多少个存档日志。

**循环间隔**:定义循环日志文件的频率。这包括每日、每周、每月和每年。如果未定义，则每当 logrotate 运行时，它都会旋转。

**Size** :我们可以定义 Size 参数来指定旋转文件时的文件大小。

**压缩**:如果我们希望我们的归档日志被压缩，我们添加这个参数。

**命令**:我们可以为日志旋转运行旋转前和旋转后命令。这可以使用参数*预旋转*和*后旋转*来完成。

**missingok** :该参数指定如果没有日志文件，则不输出错误。

让我们从使用 Ansible 命令安装 logrotate 开始。

```
- name: Install logrotate
  package: 
    name: logrotate
    state: present
  when: logrotate_scripts is defined
```

一旦安装了 logrotate，它将每天使用 cron 作业运行，该作业执行位于“/etc/cron.daily/logrotate”的脚本。我们可以在“/etc/logrotate.conf”中找到默认的 logrotate 配置文件，该文件包含了“/etc/logrotate.d/”中的所有配置。我们将在这个文件夹中创建所有需要的 logrotate 配置文件，这样当 logrotate 执行时，它可以为所有定义的日志文件进行旋转。在本例中，我们将使用 nginx 日志旋转配置。

```
- name: Setup logrotate scripts
  template:
    src: logrotate.d.j2
    dest: "{{ logrotate_conf_dir }}{{ item.name }}"
  with_items: "{{ logrotate_scripts }}"
  when: logrotate_scripts is defined
```

在我们定义的文件夹中设置 nginx 的配置变量:

```
---
  logrotate_conf_dir: "/etc/logrotate.d/"
  logrotate_scripts:
    - name: nginx
      log_dir: '/var/log/nginx'
      log_extension: 'log'
      options:
        - rotate 7
        - daily
        - size 10M
        - missingok
        - compress
        - create 0644 nginx nginx
      scripts:
          postrotate: "/bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true"
```

关于 nginx postscript 的细节可以在[这里](https://www.nginx.com/resources/wiki/start/topics/examples/logrotation/)找到。创建的最终配置文件添加如下:

```
/var/log/nginx/*.log {
  rotate 7
  weekly
  size 10M
  missingok
  compress
  delaycompress
  create 0644 nginx nginx
  postrotate
    /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
  endscript
}
```

完整的代码可以在这个 git 资源库中找到:[https://github.com/MiteshSharma/LogrotateUsingAnsible](https://github.com/MiteshSharma/LogrotateUsingAnsible)

***PS:如果你喜欢这篇文章，请用掌声支持它*** 👏 ***。欢呼***