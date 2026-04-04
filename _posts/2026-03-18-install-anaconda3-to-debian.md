---
title: 在 Debian(13) 中安装 Anaconda (3)
key: 2026-03-18
tags: Anaconda Debian Linux install
---

需要先下载

<!--more-->

## 1. 下载

## 2. 安装

下载完是一个巨大的文件, 扩展名居然是 `.sh`, 就运行了.

```sh
file Anaconda3-2025.12-1-Linux-x86_64.sh 
Anaconda3-2025.12-1-Linux-x86_64.sh: POSIX shell script executable (binary data)

chmod u+x Anaconda3-2025.12-1-Linux-x86_64.sh 

./Anaconda3-2025.12-1-Linux-x86_64.sh 

...

installation finished.
Do you wish to update your shell profile to automatically initialize conda?
This will activate conda on startup and change the command prompt when activated.
If you'd prefer that conda's base environment not be activated on startup,
   run the following command when conda is activated:

conda config --set auto_activate_base false

Note: You can undo this later by running `conda init --reverse $SHELL`

Proceed with initialization? [yes|no]
[no] >>> yes
no change     /home/chengchao/anaconda3/condabin/conda
no change     /home/chengchao/anaconda3/bin/conda
no change     /home/chengchao/anaconda3/bin/conda-env
no change     /home/chengchao/anaconda3/bin/activate
no change     /home/chengchao/anaconda3/bin/deactivate
no change     /home/chengchao/anaconda3/etc/profile.d/conda.sh
no change     /home/chengchao/anaconda3/etc/fish/conf.d/conda.fish
no change     /home/chengchao/anaconda3/shell/condabin/Conda.psm1
no change     /home/chengchao/anaconda3/shell/condabin/conda-hook.ps1
no change     /home/chengchao/anaconda3/lib/python3.13/site-packages/xontrib/conda.xsh
no change     /home/chengchao/anaconda3/etc/profile.d/conda.csh
modified      /home/chengchao/.bashrc

==> For changes to take effect, close and re-open your current shell. <==

Thank you for installing Anaconda3!

```

## 3 开发环境

```sh
conda create -n env_name
pip(conda) install package_name

If you'd prefer that conda's base environment not be activated on startup,

run the following command when canda is activated:

conda config --set auto_activate_base false

Note: You can undo this later by running :
conda init --reverse $SHELL

```

## 4 配置

[知乎](https://zhuanlan.zhihu.com/p/454069514)

```sh
## 创建一个新的开发环境：
source .bashrc
conda create -n sklearn

Retrieving notices: done
Channels:
 - defaults
Platform: linux-64
Collecting package metadata (repodata.json): done
Solving environment: done


==> WARNING: A newer version of conda exists. <==
    current version: 25.11.0
    latest version: 26.1.1

Please update conda by running

    $ conda update -n base -c defaults conda



## Package Plan ##

  environment location: /home/chengchao/anaconda3/envs/sklearn

Proceed ([y]/n)? y


Downloading and Extracting Packages:

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate sklearn
#
# To deactivate an active environment, use
#
#     $ conda deactivate

```

配置


```sh
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```



### 安装 jupyter notebook

```sh
conda install jupyter notebook
conda install numpy
conda install scikit-learn


jupyter notebook --generate-config
Writing default config to: '/home/chengchao/.jupyter/jupyter_notebook_config.py'    

jupyter server list

## 设置jupyter配置文件

$ vim /home/chengchao/.jupyter/jupyter_notebook_config.py

# 在配置文件最后面增加
c.NotebookApp.allow_remote_access = True #允许远程连接
c.NotebookApp.ip='*'                     # 设置所有ip皆可访问  
c.NotebookApp.password = 'sha1:4b2678fa7669:037692fc089b07c56f10b5b50e11e00e5a87c4b3'     # 上面复制的那个密钥'  
c.NotebookApp.open_browser = False       # 禁止自动打开浏览器  
c.NotebookApp.port = 8899                 # 设置打开端口
c.NotebookApp.notebook_dir = '/home/hadoop/'  #设置Notebook启动进入的目录

## 版本不一样了， 现在可能需要看看注释
## 根据情况修改参数

# c.ServerApp.allow_remote_access = False                                       
c.ServerApp.allow_remote_access = True 
# c.ServerApp.ip = 'localhost'
c.ServerApp.ip = '*'
# c.ServerApp.port = 0
c.ServerApp.port = 8899
# c.ServerApp.notebook_dir = ''

# c.ServerApp.password = ''


# 服务器端口对外开放，才能远程访问
$ nohup jupyter notebook --allow-root&

chengchao@mac15d:~/notebooks$ cat start.sh 
#!/usr/bin/env bash

/home/chengchao/anaconda3/envs/sklearn/bin/jupyter \
    --ip=0.0.0.0 \
    --port=8899 \
    --no-browser \
    --allow-roo \
     > jupyter.log 2>&1 &
echo "$!" > pid.txt
cat pid.txt

chengchao@mac15d:~/notebooks$ cat stop.sh 
#!/usr/bin/env bash

pid=$(cat pid.txt)
echo "execute kill -9 $pid"
kill -9 "$pid"

```


## Jupyter notebook 界面优化

[jupyter-themes](https://github.com/dunovank/jupyter-themes)

```sh
# install jupyterthemes
conda install -c conda-forge jupyterthemes

# update to latest version
conda update jupyterthemes

jt -t oceans16 -f fira -fs 17 -cellw 90% -ofs 14 -dfs 14 -T

```


## Appendix

- [https](https://scikit-learn.org/stable/)
- [https](https://jupyter.org/)
- [Python](https://www.python.org)
- [Anaconda](https://www.anaconda.com)