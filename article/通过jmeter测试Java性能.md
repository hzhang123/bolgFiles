---
title: 通过jmeter测试Java性能
tags: jmeter
grammar_cjkRuby: true
---

> 对java与scala等代码或客户端的性能测试，不能直接发起http请求，需要实例化对象发送请求实现性能测试。
> 1. jmeter提供的Java Request取样器可以控制实现JavaSamplerClient接口的类，输入参数并获取响应，利用多线程进行性能测试。
> 2. 通过maven插件启动jmeter，配合部分插件管理依赖，简化每次测试环境的准备工作。

[toc]

## 环境搭建

### 1. 搭建方式

**手动配置环境：**

1. 创建测试项目
2. 复制jmeter lib与lib\ext目录下的依赖文件添加到当前工程，如eclipse：==鼠标选中工程点击右键 -> Build Path -> Configure Build Path -> 弹出框Libraries Tab -> 点击 Add External JARs -> 切换到待导入路径选取依赖jar==。
3. 创建测试类实现JavaSamplerClient接口或继承AbstractJavaSamplerClient，并重写。 ==并不推荐使用实现接口的方式，使用集成AbstractJavaSamplerClient的方式比较好。 #F44336==
4. 编译源码，导出为Runnable Jar File
5. 将**编译后jar包**放入jmeter lib\ext目录，**项目依赖jar包**放入jmeter lib目录，复制项目依赖配置文件
6. 启动jmeter，创建测试计划进行测试。

**通过maven构建配置环境：**

1. 创建测试项目
2. 添加jmeter dependency
3. 添加并配置插件：jmeter-maven-plugin（maven启动jmeter）、maven-dependency-plugin（copy依赖）、maven-dependency-plugin（打包项目并移动到制定目录）、maven-resources-plugin（copy项目依赖文件）
4. 创建测试类实现JavaSamplerClient接口或继承AbstractJavaSamplerClient，并重写所需测试方法。
5. 启动jmeter，创建测试计划进行测试。

上面两中方式启动jmeter测试都可以：
- 手动配置环境：从命令行启动jmeter，将所有依赖都copy进jmeter项目，对于需要较大并发的分布式测试比较好，但是每次修改变动都需要重新打包copy。
- 通过maven构建配置环境：在项目内启动jmeter，只需要在pom.xml中配置依赖，每次有修改重新构建并启动便可，但是分布式的测试仍然需要在分布机器上发送依赖包。

调试脚本与小并发的测试通过maven比较方便，在调试好之后将依赖分发到分布式机器上通过命令行启动agent。

### 2. maven配置

在maven的pom.xml文件中如下根据自己需要配置依赖文件和变量便可。
- 依赖：org.apache.jmeter组织下添加的这三个依赖是编写Java Request扩展的必备。
- jmeter-maven-plugin：maven项目中启动jmeter的插件。详细使用方法：[插件配置官方文档](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin/wiki/Advanced-Configuration)
- maven-dependency-plugin：maven构建依赖copy插件。
- maven-antrun-plugin：ant插件，对编译后生成的文件再操作。
- maven-resources-plugin：有些不是jmeter的配置依赖，在项目test/jmeter目录下无法自动copy到jmeter/bin目录下，使用这个插件copy。

注意：maven版本需要与插件兼容，如过不兼容jmeter插件是无法运行的，如==jmeter-maven-plugin:2.9.0需要maven 3.6.0+==

``` xml?linenums
	<!-- 变量配置 -->
    <properties>
        <!-- version -->
        <jmeter.version>5.0</jmeter.version>
        <jmeter.maven.plugin.version>2.9.0</jmeter.maven.plugin.version>
        <maven.dependency.plugin.version>3.1.0</maven.dependency.plugin.version>
        <!-- 测试结果存放路径 -->
        <jmeter.result.jtl.dir>${project.build.directory}/jmeter/results</jmeter.result.jtl.dir>
        <!-- 测试报表存放路径 -->
        <jmeter.result.html.dir>${project.build.directory}/jmeter/html</jmeter.result.html.dir>
	</properties>
	<!-- Java Request Sample依赖添加 -->
	<dependencies>
        <dependency>
            <groupId>org.apache.jmeter</groupId>
            <artifactId>ApacheJMeter_core</artifactId>
            <version>${jmeter.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.jmeter</groupId>
            <artifactId>ApacheJMeter_java</artifactId>
            <version>${jmeter.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.jmeter</groupId>
            <artifactId>jorphan</artifactId>
            <version>${jmeter.version}</version>
        </dependency>
    </dependencies>
	<!-- 构建插件 -->
	<build>
        <plugins>
			<!-- jmeter插件 -->
            <plugin>
                <groupId>com.lazerycode.jmeter</groupId>
                <artifactId>jmeter-maven-plugin</artifactId>
                <version>${jmeter.maven.plugin.version}</version>
                <executions>
                    <execution>
                        <id>jmeter-tests</id>
                        <goals>
                            <goal>jmeter</goal>
                        </goals>
                    </execution>
                    <!-- 设置ignoreResultFailures，必须把 jmeter-check-results加上-->
                    <execution>
                        <id>jmeter-check-results</id>
                        <goals>
                            <goal>results</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!-- 指定jmeter版本 -->
                    <jmeterVersion>${jmeter.version}</jmeterVersion>
                    <!-- 聚合报告会覆盖generateReports，指定生成CSV结果 -->
                    <generateReports>true</generateReports>
                    <resultsFileFormat>csv</resultsFileFormat>
                    <!-- 设置忽略失败是否停止运行 -->
                    <ignoreResultFailures>true</ignoreResultFailures>
                    <!-- 设置结果文件末尾时间戳 -->
                    <testResultsTimestamp>true</testResultsTimestamp>
                    <!-- 时间戳格式 -->
                    <resultsFileNameDateFormat>Y-M-D H:m:s</resultsFileNameDateFormat>
                    <testFilesIncluded>
                        <!-- 指定运行的jmeter脚本 -->
                        <jMeterTestFile>JmeterTest.jmx</jMeterTestFile>
                    </testFilesIncluded>
                    <!-- 指定结果生成目录 -->
                    <resultsDirectory>${project.build.directory}/jmeter/results</resultsDirectory>
                    <!-- JVM参数 -->
                    <jMeterProcessJVMSettings>
                        <xms>2048</xms>
                        <xmx>4096</xmx>
                        <arguments>
                            <argument>-Xprof</argument>
                            <argument>-Xfuture</argument>
                        </arguments>
                    </jMeterProcessJVMSettings>
                </configuration>
            </plugin>
            <!--复制copy依赖文件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <encoding>UTF-8</encoding>
                            <!--输出路径-->
                            <outputDirectory>
                                ${project.build.directory}/jmeter/bin
                            </outputDirectory>
                            <resources>
                                <resource>
                                    <!--项目中的路径-->
                                    <directory>${project.basedir}/src/test/jmeter/</directory>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
			<!-- copy被测项目依赖到指定目录 -->
			<plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>${maven.dependency.plugin.version}</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <overWriteSnapshots>true</overWriteSnapshots>
                    <overWriteReleases>true</overWriteReleases>
                    <overWriteIfNewer>true</overWriteIfNewer>
                    <outputDirectory>${project.build.directory}/jmeter/lib</outputDirectory>
                </configuration>
            </plugin>
			<!-- copy编译打包后jar文件到指定目录：由于jar是在编译完成之后才能拿到，所以用ant插件在编译后copy jar -->
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <configuration>
                            <tasks>
                                <mkdir dir="${project.build.directory}/jmeter/lib/ext"/>
                                <copy todir="${project.build.directory}/jmeter/lib/ext" overwrite="true" >
                                    <fileset dir="${project.build.directory}" erroronmissingdir="false">
                                        <include name="*.jar"/>
                                    </fileset>
                                </copy>
                            </tasks>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

## 测试例子

### 1. 项目编写

tess4j是Tesseract OCR API的Java JNA封装，除了准确性也可以测试一下处理速度。

1. 新建maven项目。
2. 添加tess4j maven依赖，jmeter依赖参考上面的maven配置添加

``` xml?linenums
        <dependency>
            <groupId>net.sourceforge.tess4j</groupId>
            <artifactId>tess4j</artifactId>
            <version>${tess4j.version}</version>
        </dependency>
```

3. 创建工程目录如下：

``` shell
JavaRequestDemo
├── JavaRequestDemo.iml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── Tess4jTest.java
    │   └── resources
    └── test
        ├── java
        └── jmeter
            ├── JmeterTest.jmx
            ├── jmeter.properties
            ├── reportgenerator.properties
            ├── saveservice.properties
            ├── system.properties
            ├── tessdata
            │   └── chi_sim.traineddata
            ├── upgrade.properties
            └── user.properties

```

4. 下载训练库，[下载地址](https://github.com/tesseract-ocr/tessdata)， 不同语言的识别需要下载对应的训练库，这里下载中文的训练库chi_sim.traineddata，放置到src/test/jmeter/tessdata下
5. ==test/jmeter可以放置项依赖的各种配置文件和jmeter的配置文件，会自动copy到编译后jmeter的bin目录 #F44336==。
6. 编写测试类Tess4jTest.java

``` java?linenums
import net.sourceforge.tess4j.ITesseract;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;
import org.apache.jmeter.config.Arguments;
import org.apache.jmeter.protocol.java.sampler.AbstractJavaSamplerClient;
import org.apache.jmeter.protocol.java.sampler.JavaSamplerContext;
import org.apache.jmeter.samplers.SampleResult;
import org.apache.jmeter.testelement.TestElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.util.Iterator;


/**
 * public Arguments getDefaultParameters();设置可用参数及的默认值；
 * public void setupTest(JavaSamplerContext arg0)：每个线程测试前执行一次，做一些初始化工作；
 * public SampleResult runTest(JavaSamplerContext arg0)：开始测试，从arg0参数可以获得参数值；
 * public void teardownTest(JavaSamplerContext arg0)：测试结束时调用；
 * */
public class Tess4jTest extends AbstractJavaSamplerClient {

    public static void main(String[] args) {
        Arguments arguments = new Arguments();
        arguments.addArgument("dataPath", "src/test/jmeter/tessdata1");
        arguments.addArgument("language", "chi_sim");
        arguments.addArgument("imagePath", "/tmp/20200119154923.jpg");
        JavaSamplerContext context = new JavaSamplerContext(arguments);
        Tess4jTest test = new Tess4jTest();
        test.setupTest(context);
        test.runTest(context);
        test.teardownTest(context);
    }

    private static final Logger LOG = LoggerFactory.getLogger(Tess4jTest.class);
    // OCR 客户端
    private ITesseract instance = new Tesseract();
    // The name of the sampler
    private String name;
    // Java Request参数
    private String dataPath;
    private String language;
    private File image;

    /*
    * setup操作，初始化参数
    * */
    @Override
    public void setupTest(JavaSamplerContext context) {
        if (LOG.isDebugEnabled()) {
            LOG.debug(whoAmI() + "\tsetupTest()");
            listParameters(context);
        }
        dataPath = context.getParameter("dataPath");
        language = context.getParameter("language");
        image = new File(context.getParameter("imagePath"));
        name = context.getParameter(TestElement.NAME);
    }

    /**
     * 开始测试
     * */
    @Override
    public SampleResult runTest(JavaSamplerContext context) {
        SampleResult sample = new SampleResult();
        sample.setSampleLabel(name);
        //
        /*
        * dataPath一定要能访问否则jvm会异常退出，可以指定相对路径与绝对路径，可能只是提示异常：
        * Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
        * 1. main函数中测试相对路径是相对于src的平级目录
        * 2. jmeter中是相对于jmeter启动目录，maven插件是从bin目录启动，所以如果使用相对路径一般是copy到bin目录下
        * */
        instance.setDatapath(dataPath);
        instance.setLanguage(language);

        sample.setSamplerData(image.getAbsolutePath());
        try {
            // 开始计时
            sample.sampleStart();
            // 测试的主方法
            String result = instance.doOCR(image);
            // 设置结果
            sample.setSuccessful(true);
            sample.setResponseData(result, "UTF-8");
        } catch (TesseractException e) {
            LOG.error("Tess4jTest error", e);
            sample.setSuccessful(false);
            sample.setResponseMessage(e.toString());
        } finally {
            // 计时结束
            sample.sampleEnd();
        }

        if (LOG.isDebugEnabled()) {
            LOG.debug(whoAmI() + "\trunTest()" + "\tTime:\t" + sample.getTime());
            listParameters(context);
        }
        return sample;
    }

    /*
    * 参数设置获取
    * */
    @Override
    public Arguments getDefaultParameters() {
        Arguments params = new Arguments();
        params.addArgument("dataPath", "tessdata");
        params.addArgument("language", "chi_sim");
        params.addArgument("imagePath", "/tmp/20200119154923.jpg");
        return params;
    }

    /**
     * 打印参数列表
     */
    private void listParameters(JavaSamplerContext context) {
        Iterator<String> argsIt = context.getParameterNamesIterator();
        while (argsIt.hasNext()) {
            String lName = argsIt.next();
            LOG.debug(lName + "=" + context.getParameter(lName));
        }
    }

    private String whoAmI() {
        return new StringBuilder()
                .append(Thread.currentThread().toString())
                .append("@")
                .append(Integer.toHexString(hashCode()))
                .toString();
    }
}

```

7. 测试运行，直接运行Tess4jTest.java main方法

![运行结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579430030659.png)

### 2. 运行测试

1. 构建项目：mvn clean package
2. 启动jmeter：mvn jmeter:gui
3. 编写测试计划，单线程运行一下

![测试代码](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579431584156.png)

![测试结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579431681562.png)

![多添加几个线程的聚合报告](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1579431737657.png)







