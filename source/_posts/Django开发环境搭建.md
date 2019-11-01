---
title: Django开发环境搭建
date: 2019/11/1
updated: 2019/11/1
author: qfxl
catalog: true
tags:
    - Django
    - Python
---

# 环境搭建

## Django

### pyenv安装与使用

任意切换python版本，pyenv的原理是劫持环境变量

1. 安装

* 下载pyenv

```shell
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```

* 配置环境变量（~/.bashrc）

```shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
```

* 添加pyenv初始化，实现命令自动补全

```shell
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
```

* 重启当前shell，使环境配置生效

```shell
exec "$SHELL"
source ~/.bash_profile
```

2. 使用

* 查看有哪些python版本可以安装

```shell
pyenv install --list
```

* 安装某个python版本

```shell
pyenv install -v 3.6.6
```

* 查看当前python版本

```shell
pyenv versions
```

* 切换python版本

```shell
pyenv global 3.6.6
```

* 卸载某个python版本

```shell
pyenv uninstall 3.6.6
```

### virtualenv安装与使用

virtualenv用来创建一个完全隔离的python环境，各个工程的依赖相互隔离

1. 安装

```shell
pip install virtualenv
```

2. 使用

```shell
mkdir myproject
cd myproject
# 使用系统当前的python版本创建虚拟环境
virtualenv venv
# 使用指定python版本创建虚拟环境
virtualenv -p ~/.pyenv/versions/3.6.6/bin/python venv
# 激活虚拟环境
source venv/bin/activate
# 退出虚拟环境
deactivate
```

### django安装与使用

1. 安装

```shell
source venv/binactivate
pip isntall django==2.1.7
```

2. 使用

* 创建django项目

```shell
django-admin startproject myproject
```

* 运行django项目

```shell
cd myproject
python manage.py runserver
```

* 生成requirements.txt

```shell
pip freeze > requirements.txt
```

* 使用requirements.txt

```shell
pip install -r requirements.txt
```

