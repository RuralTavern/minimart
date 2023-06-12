# Arthas

说明：本文本基于windows系统

## 前提准备

```json
-- 需要jdk环境
```

## 启动

在Arthas文件夹中，找到`arthas-boot.jar`文件。命令行中运行以下命令来启动Arthas：

```shell
java -jar arthas-boot.jar
```

查看系统正在运行的java应用程序

```json
jps -l
```

## 调试

### 展示当前进程的信息

```sh
dashboard
```

