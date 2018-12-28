# linux清空java环境
#### 转发https://www.cnblogs.com/jiftle/p/9660865.html

linux清理Java环境
1.清理Java环境
```
rm -f /usr/bin/java
rm -f /etc/alternatives/java

rm -f /usr/bin/javac
rm -f /etc/alternatives/javac

rm -f /usr/bin/jar
rm -f /etc/alternatives/jar
```

说明：清理掉安装的Java环境到系统的软连接，执行过程中报错，也不影响，继续。

2.清理系统
vi /etc/profile

删除以下三行内容
```
export JAVA_HOME=/usr/java/jdk1.8.0_181
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH
```

```
source /etc/profile
```

3.删除Java安装目录
```
rm -rf /usr/java
```
清理完成！