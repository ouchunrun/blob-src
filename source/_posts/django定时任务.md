---
title: django定时任务
date: 2016-06-19
tags: [python, django] 
---

## django-crontab安装

    pip install django-crontab

## django-crontab配置

只需要将django-crontab加入到settings.py的INSTALLED_APPS即可。如下代码：

      INSTALLED_APPS = (
          'django_crontab',
          ...
      )

<!--more-->

django-crontab可以定时运行自定义命令和函数两种方式，因为之前尝试用command+crontab时已经实现了自定义command，所以自然而然使用了自定义命令这种形式。

## django-crontab定时运行命令

在settings.py中加入了django-crontab的命令：

    CRONJOBS = [
       ('47 11 * * *', 'django.core.management.call_command', ['aizhan_5domain_visits']),
    ]

意思就是每天11点47分运行aizhan_5domain_visits这个命令。接下来就剩最后一步任务加载了。

## django-crontab定时运行函数

django-crontab也可以定时运行函数，只是在CRONJOBS配置时有差别。CRONJOBS关于函数的配置如下：

    CRONJOBS = (
        # 初级模式
        ('*/5 * * * *', 'myproject.myapp.cron.my_scheduled_job'),

        # 中级模式
        ('0   0 1 * *', 'myproject.myapp.cron.my_scheduled_job', '> /tmp/last_scheduled_job.log'),

        #高级模式
        ('0   0 * * 0', 'django.core.management.call_command', ['dumpdata', 'auth'], {'indent': 4}, '> /home/john/backups/last_sunday_auth_backup.json'),
    )

分析结果：

- 初级模式很直观，意思就是每五分钟执行一次my_scheduled_job这个程序；
- 中级模式有个后缀，意思是将程序my_scheduled_job的结果输出到文件/tmp/last_scheduled_job.log中；
- 高级模式加入了参数，其中['dumpdata', 'auth']和{'indent': 4}都是参数，只是[]中的参数是按照顺序代入，而{}中的参数指定了变量名称，最后一个也是输出结果的后缀。

## django-crontab任务加载

django-crontab任务加载比较简单，只需要运行`python manage.py crontab add`即可。如果你运行`crontab -e`可以看到crontab中多了一行：

    47 11 * * * /home/aizhan/bin/python /home/aizhan/aizhan/manage.py crontab run c27d1050fb7f87225bcff587ef5a35a3 # django-cronjobs for aizhan

这是django-crontab自动生成的。

如果要移除所有的任务，则运行`python manage.py crontab remove`;
当你修改了任务，需要再次运行`python manage.py crontab add`。
