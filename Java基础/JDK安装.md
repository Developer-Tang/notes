## JDK安装

> 下载地址：[JDK](https://www.oracle.com/java/technologies/downloads/#java8)

## 安装步骤

> 此处省略，注意记下安装位置，
>
> PS：JDK8在安装JDK的最后会弹出一个JRE的单独安装界面，该安装程序直接取消即可

## 配置环境变量

> 此电脑 >> 右键 >> 属性 >> 高级系统设置 >> 环境变量

**新增JAVA_HOME**

> **`JAVA_HOME`：** 填写JDK的安装目录

**新增CLASS_PATH**

> **`CLASS_PATH`：** `.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar`

**修改PATH**

> **`PATH`：** 新增如下
> - `%JAVA_HOME%\bin`
> - `%JAVA_HOME%\jre\bin`
>
> PS：且将加入环境变量上移至最开头

**测试环境是否生效**
> `Win+R` 唤起运行输入 `cmd` 回车，分别执行如下命令
> - `java` / `java -version`
> - `javac`