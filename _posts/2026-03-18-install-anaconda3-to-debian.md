---
title: 在 Debian(13) 中安装 Anaconda (3)
key: 2026-03-18
tags: Anaconda Debian Linux install
---

需要先下载

<!--more-->

## 1. 下载

地址: 

- https://www.anaconda.com/
- https://www.anaconda.com/download/success?reg=skipped

## 2. 安装

下载完是一个巨大的文件, 扩展名居然是 `.sh`.

好吧, 我们运行它.

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

## 3 创建开发环境

```sh
conda create -n env_name
pip(conda) install package_name

If you'd prefer that conda's base environment not be activated on startup,

run the following command when canda is activated:

conda config --set auto_activate_base false

Note: You can undo this later by running :
conda init --reverse $SHELL

```

上面是配置命令示例, 我的配置是:

```sh
conda create -n sklearn
Please update conda by running

    $ conda update -n base -c defaults conda



## Package Plan ##

  environment location: /home/chengchao/anaconda3/envs/sklearn

conda activate sklearn
## 需要先激活 base 环境.
## 才能激活自定义的开发环境
```

## 4 配置

我参考了这里: [知乎](https://zhuanlan.zhihu.com/p/454069514)

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
# 编辑配置文件，添加以下内容
echo "c.ServerApp.token = ''" >> ~/.jupyter/jupyter_notebook_config.py
echo "c.ServerApp.allow_unauthenticated_access = True" >> ~/.jupyter/jupyter_notebook_config.py
echo "c.ServerApp.allow_origin = '*'" >> ~/.jupyter/jupyter_notebook_config.py


# 服务器端口对外开放，才能远程访问
$ nohup jupyter notebook --allow-root&

chengchao@mac15d:~/notebooks$ cat start.sh 
#!/usr/bin/env bash

/home/chengchao/anaconda3/envs/sklearn/bin/jupyter notebook \
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

jt -t oceans16 -f 'fira' -fs 15 -cellw 90% -ofs 15 -dfs 15 -T

```


## 5 调用 scikit-learn

### 5.1 What is iris data set?

统计学和 ML 的示例. (petal ) + (speal)

| 花萼长度 | 花萼宽度 | 花瓣长度 | 花瓣宽度 | 种类 |
| ------- | -------- | ------- | ------- | ---  |
| 5.1     | 3.5      | 1.4     | 0.2     | xxx  |


使用 scikit-learn 进行数据处理的四个关键点

1. 区分开数据属性和数据结果
2. 属性数据和结果数据都是可以**量化**的
3. 运算过程中, 属性数据与结果数据的类型都是 NumPy 数组
4. 属性数据与结果数据的维度是对应的. (输入 10 行, 输出也是 10 行)



### 5.2 load iris data set

```python

from sklearn imiport datasets
iris = datasets.load_iris()

print(type(iris.data))
print(iris.data.shape)
print(type(iris.target))
print(iris.target.shape)


```

### 5.3 分类问题

根据数据集目标的特征或者属性, 划分到已有的类别中.


常用算法:

K近邻(KNN), 逻辑回归, 决策树, 朴素贝叶斯


### 5.3 KNN

```python
## 加载 Iris 数据

from sklearn import datasets
iris = datasets.load_iris()

## 赋值
X = iris.data
y = iris.target

print(X.shape)
print(y.shape)

## 建模
# 1. 引入模型
from sklearn.neighbors import KNeighborsClassifier
# 2. 创建实例
knn = KNeighborsClassifier(n_neighbors=1)
print(f"knn => {knn}")

# 3. 模型训练
knn.fit(X, y)

# 4. 预测
knn.predict([
    [1, 2, 3, 4]
])

x_test = [[1, 2,3, 4],[2, 4, 1, 2]]
knn.predict(x_test)

knn_5 = KNeighborsClassifier(n_neighbors=5)
knn_5.fit(X, y)
knn_5.predict(x_test)

# 确认模型结构
print(knn_5)
```

## 模型评估准确率

- 最大化训练准确率通常会导致模型**复杂化**(比如增加维度), 通常会降低模型的通用性.
- 过度复杂模型容易导致训练数据的**过度拟合**.
- 模型训练的目标是为了预测**新数据**.

评估流程: 分离训练数据与测试数据

1. 把数据分成两部分: 训练集/测试集
2. 用训练集训练数据
3. 用测试集测试模型.
4. (KNN) k 越小模型越复杂.

```python
from sklearn.metrics import accuracy_score
y_pred = knn_5.predict(X)
print(accuracy_score(y, y_pred))




## 数据分离
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size = 0.4)
print(X_train.shape, X_test.shape, y_train.shape, y_test.shape)


## K： 1 ~ 25
# 遍历
# 建立 Model / 训练 / 预测 / 对测试集准确率的计算。
k_range = list(range(1, 26))
score_train = []
score_test = []
for k in k_range:
    knn = KNeighborsClassifier(n_neighbors=k)
    knn.fit(X_train, y_train)
    y_train_pred = knn.predict(X_train)
    y_test_pred = knn.predict(X_test)
    score_train.append(accuracy_score(y_train, y_train_pred))
    score_test.append(accuracy_score(y_test, y_test_pred))


import matplotlib.pyplot as plt
%matplotlib inline

plt.plot(k_range, score_train)
plt.xlabel('K(KNN model)')
plt.ylabel('Training Accuracy')
```

## 逻辑回归

用于解决分类问题的一种模型. 根据数据特征或属性, 计算其归属于某一类别的概率 `P(x)` . 处理例如二分类问题.


```sh
P(x)= 1 / 1 + e ^-(ax =b)

y = {

1, p(x) >= 0.5
0, p(x) < 0.5

# y 为类别结果, P 为概率, x 为特征值, a, b 为常量
```

皮马印第安人糖尿病


## Appendix

- [scikit-learn.org](https://scikit-learn.org/stable/)
- [httpjupyter.orgs](https://jupyter.org/)
- [Python](https://www.python.org)
- [Anaconda](https://www.anaconda.com)