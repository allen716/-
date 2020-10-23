# 1.概述

## 1.1概念

Maven是一款服务于Java平台的自动化构建工具，也是用Java写的

发展轨迹：Make--Ant--Maven--Gradle

## **1.2构建**

**什么是构建？**

构建：指的是项目从编译、测试、运行、打包、安装 ，部署整个过程都交给 maven 进行管理，这个过程称为构建

**构建过程中的各个环节：**

- 清理：将以前编译得到的旧的字节码文件删除，为下一次编译做准备
- 编译：将Java源程序编译成class字节码文件
- 测试：自动测试，自动调用 junit 程序
- 报告：测试程序执行的结果
- 打包：动态web工程打成war包，Java工程打成jar包
- 安装：maven特定的概念：将打包得到的文件复制到”仓库“中的指定位置
- 部署：将动态web工程生成的war包复制到servlet容器的指定目录下，使其可以运行

**注意：**

引入Maven后，Maven会自动将当前jar包所依赖的其他所有jar包导入进来



![image-20201022193112894](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20201022193112894.png)



# 2.常用命令

还剩一个mvn deploy



# 3.依赖的范围

<scope> </scope>

有几个属性：

**（1）compile**

- compile

<img src="C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20201023184534906.png" alt="image-20201023184534906" style="zoom:80%;" />

- 对主程序是否有效：有效
- 对测试程序是否有效：有效
- 是否参与打包：参与

**（2）provide**

举例：如servlet-api 这个jar包，在开发的时候需要，但是在tomcat上运行的时候不需要（tomcat里面已经由这个jar包了）

<img src="C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20201023185130894.png" alt="image-20201023185130894" style="zoom:80%;" />

- 对主程序是否有效：有效
- 对测试程序是否有效：有效
- 是否参与打包：不参与

**（3）test**

​	典型例子：Junit

- 对主程序是否有效：无效
- 对测试程序是否有效：有效
- 是否参与打包：不参与



# 4.生命周期

1. 各个构建环节执行的顺序：不能打乱顺序，必须按照既定的正确顺序来执行。
2. maven的核心程序中定义了抽象的生命周期，生命周期中的各个阶段的具体任务是由插件来完成的
3. maven核心程序为了更好地实现自动化构建，按照这一特点执行生命周期的各个阶段：不论现在要执行生命周期的哪一个阶段，都是从这个生命周期最初的位置开始执行



