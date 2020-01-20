---
title: MapReduce工程(IDEA) 
tags: hadoop
---


----------


## 1. maven工程

### 1.1 创建maven工程

1. 选择创建工程。

![创建工程](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564214418666.png)

2. 选择Maven工程，不选模板。

![maven选项](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564214502971.png)


3. 填好坐标，选择项目存放地址，创建工程。

![坐标](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564214578980.png)

### 1.2 修改配置文件

1. 修改pom.xml，mainClass选择自己的入口类如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>per.hao</groupId>
    <artifactId>MapReduceTest</artifactId>
    <version>1.0</version>


    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <hadoop.version>2.7.2</hadoop.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
    </dependencies>

    <!-- 构建打包插件, mainClass选择自己的入口类 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <!-- 指定入口函数 -->
                            <mainClass>per.hao.mapreduce.MRMainClass</mainClass>
                            <!-- 是否添加依赖的jar路径配置 -->
                            <addClasspath>false</addClasspath>
                            <!-- 依赖的jar包存放位置，和生成的jar放在同一级目录下 -->
                            <!--<classpathPrefix>lib/</classpathPrefix>-->
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass>per.hao.mapreduce.MRMainClass</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

2. 在项目的src/main/resources目录下，新建一个文件，命名为log4j.properties，在文件中填入

``` verilog
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n

```

### 1.3 Mapper类

``` java
package per.hao.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * 输入:
 *      行读取偏移量:LongWritable
 *      每行内容:Text
 * 输出:
 *      单词:Text
 *      单词计数:IntWritable
 * */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private Text k = new Text();
    private static final IntWritable ONE = new IntWritable(1);

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        // 获取一行数据
        String line = value.toString();

        // 切分
        String[] words = line.split("\\s");

        // 输出
        for (String word : words) {
            k.set(word);
            context.write(k, ONE);
        }
    }
}

```


### 1.4 Reduces类

``` java
package per.hao.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordCountReduce extends Reducer<Text, IntWritable, Text, IntWritable> {

    private int sum;
    private IntWritable v = new IntWritable();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 累加求和
        sum = 0;
        for (IntWritable count : values) {
            sum += count.get();
        }

        // 输出
        v.set(sum);
        context.write(key, v);
    }
}

```


### 1.5 Driver类

``` java
package per.hao.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 获取配置并根据配置获取任务实例
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //设置jar加载路径
        job.setJarByClass(WordCountDriver.class);

        // 设置Mapper、Reduce类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReduce.class);

        // 设置Mapper输出
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 设置最终输出
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 设置输入输出路径
        if (args.length < 2) {
            System.out.println("需要指定输入输出路径");
            System.exit(1);
        } else {
            FileInputFormat.setInputPaths(job, new Path(args[0]));
            FileOutputFormat.setOutputPath(job, new Path(args[1]));
        }

        // 提交任务
        boolean result = job.waitForCompletion(true);
        System.exit(result ? 0 : 1);

    }
}

```


### 1.6 入口类

``` java
package per.hao.mapreduce;

import org.apache.hadoop.util.ProgramDriver;
import per.hao.mapreduce.wordcount.WordCountDriver;

public class MRMainClass {
    public static void main(String[] args) {
        int exitCode = -1;
        ProgramDriver pd = new ProgramDriver();

        try {
            pd.addClass("wordcount", WordCountDriver.class, "我的MapReduce测试程序-WordCount");

            exitCode = pd.run(args);
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }

        System.exit(exitCode);
    }
}

```

### 1.7 测试

1. 打包jar包

``` shell
mvn clean test package
```

![打包好的jar](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564296828315.png)

2. 上传jar到服务器

3. 创建文件word.txt，内容如下：

``` txt
export	HADOOP_CLUSTER_NAME	myhadoop
export	HADOOP_TMP_DIR	hdata	hadoop
hdata	export
HADOOP_TMP_DIR	myhadoop	export
```


4. 创建文件到指定路径

``` shell
# 创建路径
/opt/cluster/hadoop/bin/hadoop fs -mkdir -p /mapreduce/test/input/20180702;
# 上传
/opt/cluster/hadoop/bin/hadoop fs -put ./word.txt /mapreduce/test/input/20180702;
```

5. 测试运行wordcount

``` shell
/opt/cluster/hadoop/bin/hadoop jar ./MapReduceTest-1.0.jar wordcount /mapreduce/test/input/20180702 /mapreduce/test/output/20180702;

```

6. 结果

![输出结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564298770677.png)


## 2. 普通工程

**注：** 相比maven的通过pom.xml配置文件配置依赖与打包；普通工程 **==手动添加依赖 #E91E63==** 与 **==打包 #E91E63==**

### 2.1 添加依赖

1. 点击File -> Project Structure
2. 点击Modules -> 选择项目 -> Dependencies -> JARs or dir...

![依赖添加界面](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299019180.png)

### 2.2 打包

1. 点击File -> Project Structure。
2. 依次点击图片所示蓝色部分。

![添加打包](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299209821.png)

3. 选择mainClass与依赖打包选项，点击OK。

![打包选项](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299500875.png)

![配置完成](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299571099.png)

4. 选择打包，弹出窗口选择build，rebuild....

![打包](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299809509.png)


5. 输出目录，找到输出的jar

![输出目录](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299894558.png)

![输出的jar](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1564299929577.png)
