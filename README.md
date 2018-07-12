## 功能说明
Hadoop cosn 插件实现了以腾讯云 COS 作为底层存储文件系统运行上层计算任务的功能，使用 Hadoop 大数据处理引擎，如 MapReduce，Hive、Spark、Tez 等，可以处理存储在腾讯云对象存储 COS 上的数据。  

## 使用限制
只适用于 COS V5 版本

## 使用环境
### 系统环境
Linux 或 Windows 系统

### 软件依赖
Hadoop-2.7.2 及以上版本
#### 安装及配置
具体 Hadoop 安装与配置 请参考 [Hadoop 安装与测试](/doc/product/436/10867)。
## 使用方法
### 安装 Maven
- **Linux ：**
```
sudo apt-get install maven
```
- **Windows：**
下载链接：[Maven](http://maven.apache.org/download.html)
安装与配置请参考：[Windows 环境下 Maven 安装与环境变量配置](http://www.cnblogs.com/liuhongfeng/p/5057827.html) 

### 获取 cos-java-sdk
下载地址：[cos-java-sdk](https://github.com/tencentyun/cos-java-sdk-v5-hadoop)

进入存放路径，运行以下命令进行编译，获取 target 目录下的cos_hadoop_api-5.2.5.jar：
```
mvn clean package -Dmaven.test.skip=true
```
### 获取 hadoop-cos 插件
下载地址：[hadoop-cos 插件](https://github.com/tencentyun/hadoop-cosn-v5)

因为 cosn 依赖 SDK，请将上一步编译的 cos_hadoop_api-5.2.5.jar 拷贝到 `src/main/resources` 下，然后运行以下命令进行编译，获取 target 目录下的 hadoop-cos-2.7.2.jar：
```
mvn clean package -Dmaven.test.skip=true
```
### 插件安装方法
#### 修改 hadoop_env.sh
在 `$HADOOP_HOME/etc/hadoop`目录下，进入 hadoop_env.sh，增加如下内容，将 cosn 相关 jar 包加入 Hadoop 环境变量：
```
for f in $HADOOP_HOME/share/hadoop/tools/lib/*.jar; do
  if [ "$HADOOP_CLASSPATH" ]; then
    export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$f
  else
    export HADOOP_CLASSPATH=$f
  fi
done
```
将获取的 cos_hadoop_api-5.2.5.jar 和 hadoop-cos-2.7.2.jar 拷贝到 `$HADOOP_HOME/share/hadoop/tools/lib`下。

#### 修改配置文件使用插件
修改 $HADOOP_HOME/etc/hadoop/core-site.xml，增加 COS 相关用户和实现类信息，例如：
```
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/rabbitliu/work/hadoop/hadoop_test</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property> 
        <name>dfs.name.dir</name>           
        <value>/data/rabbitliu/work/hadoop/hadoop_test/name</value> 
    </property>
    <property> 
        <name>fs.cosn.userinfo.secretId</name>           
        <value>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</value> 
    </property>
    <property> 
        <name>fs.cosn.userinfo.secretKey</name>           
        <value>xxxxxxxxxxxxxxxxxxxxxxxx</value> 
    </property>
    <property>
        <name>fs.cosn.impl</name>
        <value>org.apache.hadoop.fs.cosnative.NativeCosFileSystem</value>
    </property>
    <property>
        <name>fs.cosn.buffer.dir</name>
        <value>/data/rabbitliu/work/hadoop/hadoop_test/cos_buf</value>
    </property>
    <property>
        <name>fs.cosn.userinfo.region</name>
        <value>ap-guangzhou</value>
    </property>
</configuration>
```

> <font color="#0000cc">**注意：** </font>
配置文件中含有 COS 的几个属性：
- fs.cosn.userinfo.secretId/secretKey 属性：填写您账户的API 密钥信息。可通过 [云 API 密钥 控制台](https://console.cloud.tencent.com/capi) 查看。
- fs.cosn.impl 为 cosn 的实现类，固定为 org.apache.hadoop.fs.cosnative.NativeCosFileSystem。
- fs.cosn.buffer.dir 请设置一个实际存在的目录，运行过程中产生的临时文件会暂时放于此处。
- fs.cosn.userinfo.region 请填写您的地域信息，枚举值为 [可用地域](https://cloud.tencent.com/document/product/436/6224) 中的地域简称，如 ap-beijing、ap-guangzhou 等。

### 使用软件（以 Linux 为例）
#### 使用 hadoop fs 常用命令
命令格式为：`hadoop fs- -ls cosn://Bucket 路径`，下例中以名称为 example-1252681929 的 Bucket 为例，可在其后面加上具体路径。
```
hadoop fs -ls -R cosn://hdfs-test-1252681929/
-rw-rw-rw-   1 root root       1087 2018-06-11 07:49 cosn://hdfs-test-1252681929/LICENSE
drwxrwxrwx   - root root          0 1970-01-01 00:00 cosn://hdfs-test-1252681929/hdfs
drwxrwxrwx   - root root          0 1970-01-01 00:00 cosn://hdfs-test-1252681929/hdfs/2018
-rw-rw-rw-   1 root root       1087 2018-06-12 03:26 cosn://hdfs-test-1252681929/hdfs/2018/LICENSE
-rw-rw-rw-   1 root root       2386 2018-06-12 03:26 cosn://hdfs-test-1252681929/hdfs/2018/ReadMe
drwxrwxrwx   - root root          0 1970-01-01 00:00 cosn://hdfs-test-1252681929/hdfs/test
-rw-rw-rw-   1 root root       1087 2018-06-11 07:32 cosn://hdfs-test-1252681929/hdfs/test/LICENSE
-rw-rw-rw-   1 root root       2386 2018-06-11 07:29 cosn://hdfs-test-1252681929/hdfs/test/ReadMe
```
#### 运行 MapReduce 自带的 wordcount
> <font color="#0000cc">**注意：** </font>
以下命令中 hadoop-mapreduce-examples-2.7.2.jar 是以 2.7.2 版本为例，如版本不同，请修改成对应的版本号。

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount cosn://example/mr/input cosn://example/mr/output3
```
执行成功会返回统计信息，示例如下：
```
File System Counters
        COSN: Number of bytes read=72
        COSN: Number of bytes written=40
        COSN: Number of read operations=0
        COSN: Number of large read operations=0
        COSN: Number of write operations=0
        FILE: Number of bytes read=547350
        FILE: Number of bytes written=1155616
        FILE: Number of read operations=0
        FILE: Number of large read operations=0
        FILE: Number of write operations=0
        HDFS: Number of bytes read=0
        HDFS: Number of bytes written=0
        HDFS: Number of read operations=0
        HDFS: Number of large read operations=0
        HDFS: Number of write operations=0
    Map-Reduce Framework
        Map input records=5
        Map output records=7
        Map output bytes=59
        Map output materialized bytes=70
        Input split bytes=99
        Combine input records=7
        Combine output records=6
        Reduce input groups=6
        Reduce shuffle bytes=70
        Reduce input records=6
        Reduce output records=6
        Spilled Records=12
        Shuffled Maps =1
        Failed Shuffles=0
        Merged Map outputs=1
        GC time elapsed (ms)=0
        Total committed heap usage (bytes)=653262848
    Shuffle Errors
        BAD_ID=0
        CONNECTION=0
        IO_ERROR=0
        WRONG_LENGTH=0
        WRONG_MAP=0
        WRONG_REDUCE=0
    File Input Format Counters 
        Bytes Read=36
    File Output Format Counters 
        Bytes Written=40
```