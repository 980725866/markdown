### 下载 repo 工具:
```java
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

### 导入到环境变量
```java
export PATH=~/bin:$PATH
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```


### 初始化repo仓库
```java
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-1.6_r1
    -u：指定从中检索清单代码库的网址。常见清单位于 https://android.googlesource.com/platform/manifest。
    -m：选择代码库中的清单文件。如果未选择清单名称，则默认为 default.xml。
    -b：指定修订版本，即特定的 manifest-branch。
```

### 下载源码
```java
repo sync -c -j16
    -c, --current-branch  fetch only current branch from server 当前分支
    -j JOBS, --jobs=JOBS  projects to fetch simultaneously (default 4) 多线程
```

### 显示manifest文件内容
```java
repo manifest
```



### 在每个项目中运行指定的 shell 命令(迭代器)
```java
repo forall -c <COMMAND>
    -c：要运行的命令和参数。
    COMMAND: shell指令
```

### 显示提交与工作树之间的更改
```java
repo diff
```

### 将工作树与临时区域（索引）以及此分支 (HEAD) 上的最近一次提交进行比较。
```java
// 在这三种状态存在差异之处显示每个文件的摘要行
repo status
```


### 参考博客
https://source.android.com/source/using-repo#top_of_page
https://wiki.jikexueyuan.com/project/android-source/using-repo.html
https://source.android.google.cn/setup/develop/repo
