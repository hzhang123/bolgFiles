---
title: Java自动化环境搭建笔记（2）
tags: 自动化测试
---


----------

> 在笔记一中已经完成了一键构建项目、xml指定规划测试集、数据解耦与allure报告生成的开发。接下来便是：
> - 浏览器驱动通过配置启动
> - 页面元素定位解耦，通过配置文件映射JavaBean定位器集合对象
> - 关键字命令，测试基类新增方法，提供基础关键字(这里给部分常用方法，后续扩充)
> - 测试失败截图
> 1. 基础依赖
>     - 笔记一种的项目已搭建完成
> 2. 测试依赖待开发
>     - 浏览器驱动对象获取工具类
>     - 定位器对象与定位器初始化与获取工具类
>     - 关键字方法开发
>     - 测试失败截图监听类

[toc]

# 1. 项目结构

对比笔记1中新增了一些目录

![当前结构变化](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563587366041.png)

# 2. 代码开发

## 2.1 浏览器启动

浏览器启动需要写一个浏览器驱动管理类从xml文件中读取数据初始化驱动，对外提供静态获取方法。

### 2.1.1 driver.xml 浏览器驱动配置文件

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!--  driverIndex标识获取对应与name节点的index相同的浏览器驱动 -->
<driver driverIndex="0">

    <!-- 谷歌浏览器配置文件 -->
    <name value="org.openqa.selenium.chrome.ChromeDriver" index="0">
        <!-- name驱动类，value驱动名，{default.driver.path}会自动替换为当前项目src/test/resources -->
        <properties>
            <property name="webdriver.chrome.driver" value="{default.driver.path}/driver/chromedriver.exe" />
        </properties>
        <!-- chrome的路径 -->
        <options>
            <option type="binary">C:\Program Files (x86)\Google\Chrome\Application\chrome.exe</option>
        </options>
    </name>

    <!-- 火狐浏览器 对应的selenium3.x版本 的配置文件 -->
    <name value="org.openqa.selenium.firefox.FirefoxDriver"
          seleniumVersion="3.x" index="1">
        <properties>
            <property name="SystemProperty.BROWSER_BINARY"
                      value="C:\Program
        Files (x86)\Mozilla Firefox\firefox.exe" />
            <property
                    name="webdriver.gecko.driver"
                    value="E:/driver/geckodriver.exe" />
        </properties>
    </name>

    <!-- 火狐浏览器 对应的selenium2.x版本 的配置文件 -->
    <name value="org.openqa.selenium.firefox.FirefoxDriver"
          seleniumVersion="2.x" index="2">
        <properties>
            <property name="SystemProperty.BROWSER_BINARY"
                      value="C:\Program
        Files (x86)\Mozilla Firefox\firefox.exe" />
        </properties>
    </name>

    <!--IE浏览器配置文件 -->
    <name value="org.openqa.selenium.ie.InternetExplorerDriver"
          index="3">
        <properties>
            <property
                    name="webdriver.ie.driver"
                    value="{default.driver.path}\driver\IEDriverServer.exe" />
        </properties>
        <capabilities>
            <!-- name能力，value能力的值，type能力值的数据类型 -->
            <capability
                    name="InternetExplorerDriver.IGNORE_ZOOM_SETTING"
                    type="boolean"
                    value="true" />
            <capability
                    name="InternetExplorerDriver.INTRODUCE_FLAKINESS_BY_IGNORING_SECURITY_DOMAINS"
                    type="boolean"
                    value="true" />
        </capabilities>
    </name>
</driver>
```

### 2.1.2 WebDriverUtil 驱动管理类

对外提供getDriver()来获取浏览器对象。

``` java
package per.hao.utils;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;

public class WebDriverUtil {

    private static final Logger log = LoggerFactory.getLogger(WebDriverUtil.class);
    /* WebDriver class */
    private static Class clazz;
    /* WebDriver Object */
    private static Object obj ;

    /**
     * 根据driver.xml文件初始化获取浏览器WebDriver对象
     *
     * @return WebDriver 浏览器WebDriver对象
     * */
    public static WebDriver getDriver() {
        Document document = null;
        Element driverElement= null;
        String driverClassName =null;

        SAXReader reader = new SAXReader();
        try {
            document = reader.read(WebDriverUtil.class.getResourceAsStream("/driver.xml"));
        } catch (DocumentException e) {
            log.error("read driver.xml failed", e);
        }

        /** 获取驱动类名 */
        Element rootElement = document.getRootElement();
        // driver index
        int index = Integer.parseInt(rootElement.attributeValue("driverIndex"));
        /** 遍历name节点 */
        List<Element> driverNameElements = rootElement.elements("name");
        for (Element driverNameElement : driverNameElements) {
            if (index == Integer.parseInt(driverNameElement.attributeValue("index"))) {
                driverClassName = driverNameElement.attributeValue("value");
                driverElement = driverNameElement;
            }
        }

        /** 获取驱动class */
        try {
            clazz = Class.forName(driverClassName);
            log.info("get class:" + driverClassName);
        } catch (ClassNotFoundException e) {
            log.error("get class error", e);
        }

        /**
         * 下面是解析XML系统参数并设置
         */
        Element propertiesElement = driverElement.element("properties");
        List<Element> propertyElements = propertiesElement.elements("property");
        //设置系统参数(driver路径等)
        for (Element property : propertyElements) {
            System.setProperty(property.attributeValue("name"), formatPath(property.attributeValue("value")));
            log.info("set property:" + property.attributeValue("name") + "=" +formatPath(property.attributeValue("value")));
        }

        //设置能力（ie的话，需要设置忽略域设置等级 以及忽略页面百分比的能力）
        Element capabilitiesElement = driverElement.element("capabilities");
        if (capabilitiesElement != null) {
            DesiredCapabilities realCapabilities = new DesiredCapabilities();
            List<Element> capabilitiesElements = capabilitiesElement.elements("capability");
            for (Element capability : capabilitiesElements) {
                //遍历能力列表，并给能力赋值
                if ("boolean".equals(capability.attributeValue("type"))) {
                    realCapabilities.setCapability(capability.attributeValue("name"),
                            Boolean.parseBoolean(capability.attributeValue("value")));
                } else if ("string".equals(capability.attributeValue("type"))) {
                    realCapabilities.setCapability(capability.attributeValue("name"),
                            capability.attributeValue("value"));
                }
                log.info("set capability:" + capability.attributeValue("name") + "=" + capability.attributeValue("value"));
            }
        }

        /** 如果是chrome设置chrome的一些属性 */
        Element optionsElement = driverElement.element("options");
        if (optionsElement != null) {
            ChromeOptions chromeOptions = new ChromeOptions();
            for (Element optionElement : (List<Element>) optionsElement.elements()) {
                if ("binary".equals(optionElement.attributeValue("type"))) {
                    String binary = formatPath(optionElement.getStringValue());
                    chromeOptions.setBinary(binary);
                    log.info("set chrome Binary:" + binary);
                }
            }

            WebDriver driver = new ChromeDriver(chromeOptions);
            log.info("get driver succeed:" + driverClassName);
            return driver;
        }


        /*
         * 通过反射，创建驱动对象
         */
        try {
            obj = clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        log.info("get driver succeed:" + driverClassName);
        return (WebDriver) obj;
    }

    /**
     * 替换xml中出现的需要替换的变量
     *
     * @param path String路径
     *
     * @return String 替换之后的路径
     * */
    private static String formatPath(String path) {
        // chrome不一定安装在用户目录下，去掉
//        if (path.contains("{user.home}")) {
//            path = path.replace("{user.home}", System.getProperty("user.home"));
//        }
        if (path.contains("{default.driver.path}")) {
            path = path.replace("{default.driver.path}", "src/test/resources");
        }

        return path;
    }

}

```

## 2.2 定位器获取

界面上的所有的元素按照界面 -> 元素的结构写到xml中，实现定位界面元素方法、界面元素属性、等待时间与代码的解耦

### 2.2.1 定位器文件示例

<h2 id="定位器文件示例"></h2>

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<pages>
    <!-- 百度查询 -->
    <!-- pageName: 页面名称，代码中每次跳转到新界面使用pageName来切换到本界面的定位器集合，或者使用pageName打开界面等等 -->
    <!-- url: 为这个界面的url，open("pageName")的时候回查找该url并打开 -->
    <!-- defaultTimeOut: 默认查找元素超时时间，必填 -->
    <!--<page pageName="" url="https://www.baidu.com/" defaultTimeOut="2">-->
    <page pageName="百度首页" url="https://www.baidu.com/" defaultTimeOut="2">
        <!-- name: 定位器的名称，代码中，使用定位器名称去指定操作界面上根据定位器查找到的元素 -->
        <!-- by: 定位方法。可选 id linkText name xpath className cssSelector partialLinkText tagName
       为By类中方法的名称，通过这个属性反射获取并调用的方法 -->
        <!-- vanle: -->
        <!--  timeOut: 如果不填，默认查找超时时间使用page节点的 defaultTimeOut，defaultTimeOut必填-->
        <!-- <locator name="" by="" value="" timeOut=""/> -->
        <!-- <locator name="" by="" value="" /> -->
        <locator name="查询框" by="id" value="kw" timeOut="3"/>
        <locator name="百度一下" by="id" value="su" timeOut="3"/>
    </page>
    <!-- 禅道登录界面 -->
    <page pageName="LoginPage" url="http://192.168.2.3/zentao/user-login.html" defaultTimeOut="2">
        <locator name="用户名" by="id" value="account" timeOut="3"/>
        <locator name="密码" by="name" value="password"/>
        <locator name="登录" by="id" value="submit" timeOut="3"/>
        <locator name="忘记密码" by="linkText" value="忘记密码"/>
    </page>

    <!-- 禅道首页 -->
    <page pageName="HomePage" url="http://192.168.2.3/zentao/my/" defaultTimeOut="2">
        <locator name="用户姓名" by="xpath" value="//*[@id='userNav']/li/a/span[1]" timeOut="3"/>
        <locator name="系统下拉框" by="xpath" value="//*[@id='userNav']/li/a"/>
        <locator name="退出" by="xpath" value="//*[@id='userNav']/li/ul/li[13]/a" timeOut="3"/>
    </page>

</pages>
```

![定位器xml与界面关系](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563593173884.png)


### 2.2.2 LocatorInfo 定位器对象

``` java
package per.hao.beans;

import java.io.Serializable;

/**
 * locator映射对象
 *
 * */
public class LocatorInfo implements Serializable {
    private String name;
    private String by;
    private String value;
    private int timeOut;

    public LocatorInfo(String name, String by, String value, int timeOut) {
        this.name = name;
        this.by = by;
        this.value = value;
        this.timeOut = timeOut;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setBy(String by) {
        this.by = by;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public void setTimeOut(int timeOut) {
        this.timeOut = timeOut;
    }

    public String getName() {
        return name;
    }

    public String getBy() {
        return by;
    }

    public String getValue() {
        return value;
    }

    public int getTimeOut() {
        return timeOut;
    }

    @Override
    public String toString() {
        return "LocatorInfo{" +
                "name='" + name + '\'' +
                ", by='" + by + '\'' +
                ", value='" + value + '\'' +
                ", timeOut=" + timeOut +
                '}';
    }
}

```

### 2.2.3 PageInfo 页面对象

``` java
package per.hao.beans;

import java.io.Serializable;
import java.util.HashMap;
import java.util.Map;

/**
 * page映射对象
 * */
public class PageInfo implements Serializable {
    private String pageName;
    private String url;
    private Map<String, LocatorInfo> locatorInfoMap= new HashMap<>();

    public PageInfo(String pageName, String url, Map<String, LocatorInfo> locatorInfoMap) {
        this.pageName = pageName;
        this.url = url;
        this.locatorInfoMap = locatorInfoMap;
    }

    public void setPageName(String pageName) {
        this.pageName = pageName;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setLocatorInfoMap(Map<String, LocatorInfo> locatorInfoMap) {
        this.locatorInfoMap = locatorInfoMap;
    }

    public String getPageName() {
        return pageName;
    }

    public String getUrl() {
        return url;
    }

    public Map<String, LocatorInfo> getLocatorInfoMap() {
        return locatorInfoMap;
    }

    @Override
    public String toString() {
        return "PageInfo{" +
                "pageName='" + pageName + '\'' +
                ", url='" + url + '\'' +
                ", locatorInfoMap=" + locatorInfoMap +
                '}';
    }
}

```

### 2.2.4 LocatorUtil 定位器解析工具类

``` java
package per.hao.utils;

import org.apache.commons.collections.map.HashedMap;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import per.hao.beans.LocatorInfo;
import per.hao.beans.PageInfo;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class LocatorUtil {

    private static final Logger log = LoggerFactory.getLogger(LocatorUtil.class);
    private static String filePath;
    // Map<filePath, Map<pageName, PageInfo>>
    private static Map<String, Map<String, PageInfo>> pageInfoMaps = new HashMap<>();

    /**
     * 重写构造函数
     *
     * @param filePath locator文件在resources下的路径
     *                 相对于配置文件目录，如：/locator/locator.xml
     *                 即为resources下的locator目录中的locator.xml
     * */
    public LocatorUtil(String filePath) {
        this.filePath = filePath;
    }

    /**
     * 确定只解析一次
     * */
    private static void sureInit() {
        if (pageInfoMaps.get(filePath) == null) {
            loadXML();
        }
    }

    /**
     * 解析locator定位xml文件
     * */
    private static void loadXML() {
        SAXReader saxReader = new SAXReader();
        Document document = null;

        try {
            document = saxReader.read(LocatorUtil.class.getResourceAsStream(filePath));
        } catch (DocumentException e) {
            log.error("open locator file failed:", e);
        }

        if (document != null) {
            log.info("open locator file succeed:" + filePath);
        }

        // 页面page遍历
        Element root = document.getRootElement();
        List<Element> pageElements = root.elements("page");
        Map<String, PageInfo> pageInfoMap = new HashMap<>();

        for (Element pageElement : pageElements) {
            String pageName = pageElement.attributeValue("pageName");
            String url = pageElement.attributeValue("url");
            String defaultTimeOut = pageElement.attributeValue("defaultTimeOut");

            // 定位器locator遍历
            List<Element> locatorElements = pageElement.elements("locator");
            Map<String, LocatorInfo> locatorInfoMap = new HashedMap();
            for (Element locatorElement : locatorElements) {
                String name = locatorElement.attributeValue("name");
                String by = locatorElement.attributeValue("by");
                String value = locatorElement.attributeValue("value");
                String timeOut = locatorElement.attributeValue("timeOut");
                if (timeOut == null || "".equals(timeOut)) {
                    timeOut = defaultTimeOut;
                }

                locatorInfoMap.put(name,
                        new LocatorInfo(name, by, value, Integer.parseInt(timeOut)));
            }

            pageInfoMap.put(pageName, new PageInfo(pageName, url, locatorInfoMap));
        }
        pageInfoMaps.put(filePath, pageInfoMap);
        log.info("parse locator file succeed:" + filePath);
    }

    /**
     * 获取测试界面url
     *
     * @param filePath locator文件在resources下的路径
     *                 相对于配置文件目录，如：/locator/locator.xml
     *                 即为resources下的locator目录中的locator.xml
     *
     * @param pageName 页面名称，locator.xml中page节点pageName元素属性
     *
     * @return 页面url
     * */
    public static String getURL(String filePath, String pageName) {
        new LocatorUtil(filePath);
        sureInit();
        String url = LocatorUtil.pageInfoMaps.get(filePath).get(pageName).getUrl();
        log.debug("get url:" + url);
        return url;
    }

    /**
     * 获取定位器
     *
     * @param filePath locator文件在resources下的路径
     *                 相对于配置文件目录，如：/locator/locator.xml
     *                 即为resources下的locator目录中的locator.xml
     * @param pageName 页面名称，locator.xml中page节点元素属性
     * @param name     定位器名称 locator.xml中locator节点name元素属性
     *
     * @return 定位器
     * */

    public static LocatorInfo getLocator(String filePath, String pageName, String name) {
        new LocatorUtil(filePath);
        sureInit();
        LocatorInfo locatorInfo = LocatorUtil.pageInfoMaps
                .get(filePath)
                .get(pageName)
                .getLocatorInfoMap().get(name);
        log.debug("get locator:" + locatorInfo);
        return locatorInfo;
    }
}

```

### 2.2.5 LocatorSource 定位器文件源注解

``` java
package per.hao.annotations;


import java.lang.annotation.*;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface LocatorSource {
    /**
     * locator文件在resources下的路径
     *         相对于配置文件目录，如：/locator/locator.xml
     *         即为resources下的locator目录中的locator.xml
     * */
    String filePath();
}
```

### 2.2.6 测试 LocatorUtil 方法

![LocatorUtil方法返回](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563590298809.png)

## 2.3 测试基类完善

1. 测试基类中添加测试集启动前通过WebDriverUtil获取驱动、测试集运行完成运行环境清理工作。
2. 添加getWebElement方法根据当前测试方法LocatorSource注解指定的定位器文件来定位元素。
3. 添加内部类GetLocatorFilePath，用来获取当前运行方法的LocatorSource注解，返回filePath标注的路径，用来在作为getWebElement参数获取元素。
4. 完善浏览器操作方法供继承类调用。

### 2.3.1 BaseTester 测试基类

``` java
package per.hao;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.WebDriverWait;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.testng.Assert;
import org.testng.ITestNGMethod;
import org.testng.Reporter;
import org.testng.annotations.AfterSuite;
import org.testng.annotations.BeforeSuite;
import org.testng.annotations.DataProvider;
import per.hao.annotations.DataSource;
import per.hao.annotations.LocatorSource;
import per.hao.beans.LocatorInfo;
import per.hao.utils.DataSourceType;
import per.hao.utils.ExcelReader;
import per.hao.utils.LocatorUtil;
import per.hao.utils.WebDriverUtil;

import java.io.File;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Iterator;
import java.util.regex.Matcher;


public class BaseTester<T> {

    public static final Logger log =
            LoggerFactory.getLogger(BaseTester.class);
    /** driver */
    public static WebDriver driver;

    /** 当前定位器指针, 用来指向PageInfo的pageName */
    private static String pageName;

    /** 浏览器导航栏对象，封装导航栏方法用 */
    private static WebDriver.Navigation navigation;

    /**
     * 测试集启动前初始化
     * */
    @BeforeSuite
    protected void beforeSuit() {
        driver = WebDriverUtil.getDriver();
        // 窗口最大化
        driver.manage().window().maximize();
        navigation = driver.navigate();
    }

    /**
     * 测试集后置操作
     * */
    @AfterSuite
    protected void afterSuit() {
        driver.quit();
    }
    //-----------------------------浏览器管理-----------------------------------
    /**
     * 清空cookies
     * */
    protected void deleteAllCookies() {
        driver.manage().deleteAllCookies();
    }

    // ------------------------------断言-----------------------------------
    /**
     * 判断expected是否包含actual
     *
     * @param expected 文本
     * @param actural 文本
     * */
    protected void assertTextContain(String expected, String actural) {
        Assert.assertTrue(expected.contains(actural),
                expected + " [not contain] " + actural);
    }

    /**
     * 判断expected是否等于actual
     *
     * @param expected T expected
     * @param actural T actural
     * */
    public void assertTextEqual(T expected, T actural) {
        Assert.assertEquals(expected, actural);
    }

    /**
     * 判断是否为true
     *
     * @param b 参数b
     * */
    public void assertTrue(boolean b) {
        Assert.assertTrue(b);
    }

    // ------------------------------获取指定属性-----------------------------------
    /**
     * 获取页面title标题
     *
     * @return String
     * */
    protected String getTitle() {
        return driver.getTitle();
    }

    /**
     * 获取定位器定位到元素的文本内容
     *
     * @param locatorName 定位器名称 locator.xml中locator节点name元素属性
     *
     * @return String
     * */
    protected String getText(String locatorName) {
        return getWebElement(locatorName).getText();
    }

    /**
     * 获取定位器定位到元素是否被勾选
     *
     * @param locatorName 定位器名称 locator.xml中locator节点name元素属性
     *
     * @return boolean
     * */
    protected boolean isSelected(String locatorName) {
        return getWebElement(locatorName).isSelected();
    }

    // ------------------------------元素基本操作-----------------------------------
    /**
     * WebElement   根据定位器定位并点击
     *
     * @param locatorName 定位器名称 locator.xml中locator节点name元素属性
     * */
    protected void click(String locatorName) {
        getWebElement(locatorName).click();
    }

    /**
     * WebElement   根据定位器定位并输入指定内容
     *
     * @param locatorName 定位器名称 locator.xml中locator节点name元素属性
     * @param input 输入内容
     * */
    protected void input(String locatorName, String input) {
        getWebElement(locatorName).sendKeys(input);
    }

    /**
     * WebElement   根据定位器定位并清空
     *
     * @param locatorName 定位器名称 locator.xml中locator节点name元素属性
     * */
    protected void clear(String locatorName) {
        getWebElement(locatorName).clear();
    }

    // ------------------------------导航栏基本操作-----------------------------------
    /**
     * navigation   浏览器后退操作
     * */
    protected void back() {
        navigation.back();
    }

    /**
     * navigation   浏览器前进操作
     * */
    protected void forward() {
        navigation.forward();
    }

    /**
     * navigation   浏览器刷新操作
     * */
    protected void refresh() {
        navigation.refresh();
    }

    /**
     * navigation   打开指定页面
     *
     * @param pageName 页面名称，locator.xml中page节点pageName元素属性
     * */
    protected void open(String pageName) {
        String url =
                LocatorUtil.getURL(new GetLocatorFilePath().invok().getFilePath(), pageName);
        go(url);
    }

    /**
     * navigation   打开指定url
     *
     * @param url 连接串
     * */
    protected void go(String url) {
        navigation.to(url);
    }

    /**
     * 睡眠指定时间
     *
     * @param millis 毫秒
     * */
    protected void sleep(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            log.error("sleep error", e);
        }
    }

    /**
     * 将定位器指针移动到指定界面集合下
     * */
    protected void locate(String pageName) {
        this.pageName = pageName;
    }

    /**
     * 根据定位器名称获取界面元素
     *
     * @param locatorName 定位器名称 locator.xml locator节点name属性
     *
     * @return WebElement
     * */
    protected WebElement getWebElement(String locatorName) {
        String filePath = new GetLocatorFilePath().invok().getFilePath();
        return getWebElement(filePath, locatorName);
    }

    /**
     * 根据locater文件的路径、页面名称，定位器名称获取界面元素
     *
     * @param filePath locator文件在resources下的路径
     *                 相对于配置文件目录，如：/locator/locator.xml
     *                 即为resources下的locator目录中的locator.xml
     *
     * @param locatorName 定位器名称 locator.xml locator节点name属性
     *
     * @return WebElement
     * */
    protected WebElement getWebElement(String filePath, String locatorName) {
        LocatorInfo locator = LocatorUtil.getLocator(filePath, pageName, locatorName);
        String byMethodName = locator.getBy();
        String value = locator.getValue();
        int timeOut = locator.getTimeOut();

        Class<By> byClass = By.class;
        Method declaredMethod;

        try {
            declaredMethod = byClass.getDeclaredMethod(byMethodName, String.class);
            By by = (By) declaredMethod.invoke(null, value);

            WebDriverWait wait = new WebDriverWait(driver, timeOut);
            WebElement until = wait.until(new ExpectedCondition<WebElement>() {
                public WebElement apply(WebDriver input) {
                    return input.findElement(by);
                }
            });
            return until;
        } catch (NoSuchMethodException e) {
            log.error("locator by may be error in " + locatorName, e);
        } catch (IllegalAccessException e) {
            log.error("", e);
        } catch (InvocationTargetException e) {
            log.error("", e);
        }
        return null;
    }

    /**
     * 内部工具类，用来获取LocatorSource注解中的locator filePath
     * */
    private class GetLocatorFilePath {
        private String filePath;

        ITestNGMethod iTestNGMethod = Reporter.getCurrentTestResult().getMethod();
        Method method = iTestNGMethod.getMethod();
        LocatorSource locatorSource = method.getAnnotation(LocatorSource.class);

        /**
         * 初始化
         * */
        public GetLocatorFilePath invok() {
            filePath = locatorSource.filePath();
            return this;
        }

        /**
         * 获取LocatorSource注解中的locator filePath
         * */
        public String getFilePath() {
            return filePath;
        }
    }

    /**
     * 数据提供公共接口
     * */
    @DataProvider(name = "getData")
    public static Iterator<Object[]> getData(Method method) {

        DataSource dataSource = null;

        /** 数据源注解存在判断 */
        if (method.isAnnotationPresent(DataSource.class)) {
            dataSource = method.getAnnotation(DataSource.class);
        } else {
            log.error("未指定@DataSource注解却初始化了dataProvider");
        }

        /** 根据数据源类型返回对应数据迭代器 */
        if (DataSourceType.CSV
                .equals(dataSource.dataSourceType())) {

            // CSVReader

        } else if (DataSourceType.POSTGRESQL
                .equals(dataSource.dataSourceType())) {

            // PostgresqlReader

        }

        /* 默认读取excel */
        // 根据名称
        if (!"".equals(dataSource.name())) {

            return ExcelReader.getDataByName(
                    dealFilePath(dataSource.filePath()), dataSource.name());
            // 根据锚点
        } else if (!"".equals(dataSource.locate())) {

            return ExcelReader.getDataByLocate(
                    dealFilePath(dataSource.filePath()), dataSource.sheetName(), dataSource.locate());
           // 读取整个sheet页
        } else {

            return ExcelReader.getDataBySheetName(
                    dealFilePath(dataSource.filePath()), dataSource.sheetName());

        }
    }

    /**
     * 如果只存在文件名，则拼接默认读取目录，否则使用指定的路径
     *
     * @param filePath 文件路径
     * */
    private static  String dealFilePath(String filePath) {
        if (!filePath.matches(".*[/\\\\].*")) {
            filePath = "src/test/resources/data/" + filePath;
        }

        return new File(filePath.replaceAll("[/\\\\]+",
                Matcher.quoteReplacement(File.separator))).getAbsolutePath();
    }
}

```

## 2.4 失败用例截图

### 2.4.1 TestFaildListener 用例截图监听类

截图覆盖了TestListenerAdapter的onTestFailure方法，在测试失败的时候进行截图并添加为附件，添加附件使用了Allure的注解方法Attachment，详细用法可以看:[testNG官方文档](https://testng.org/doc/documentation-main.html#testng-listeners)，[Allure官方文档](https://docs.qameta.io/allure/#_testng)

``` java
package per.hao.listener;

import io.qameta.allure.Attachment;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.testng.ITestResult;
import org.testng.TestListenerAdapter;
import per.hao.BaseTester;

public class TestFaildListener extends TestListenerAdapter {
    @Override
    public void onTestFailure(ITestResult iTestResult) {
        screenShot();
    }

    /**
     * 截图并添加为附件
     * 
     * @return byte[]附件
     * */
    @Attachment(value = "Page screenshot",type = "image/png")
    public byte[] screenShot() {
        return ((TakesScreenshot) BaseTester.driver).getScreenshotAs(OutputType.BYTES);
    }
}
```

### 2.4.2 testNG.xml 中添加监听

``` xml
<!-- 失败用例截图 -->
<listener class-name="per.hao.listener.TestFaildListener"/>
```

# 3. 需修改文件

## 3.1 pom.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>per.hao</groupId>
    <artifactId>selenium-project-hzhang</artifactId>
    <version>1.0.0</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <allure.version>2.10.0</allure.version>
        <java.version>1.8</java.version>
        <aspectj.version>1.9.2</aspectj.version>
    </properties>

    <dependencies>

        <!-- xml解析：https://mvnrepository.com/artifact/dom4j/dom4j -->
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
            <scope>test</scope>
        </dependency>

        <!-- selenium：https://mvnrepository.com/artifact/org.seleniumhq.selenium/selenium-java -->
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.141.59</version>
            <scope>test</scope>
        </dependency>

        <!-- testNG框架 -->
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>6.14.3</version>
            <scope>test</scope>
        </dependency>

        <!-- allure报告依赖 -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-maven</artifactId>
            <version>${allure.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-all</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>

        <!-- 日志依赖 -->
        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
            <scope>test</scope>
        </dependency>

        <!-- excel读取插件 -->
        <!-- https://mvnrepository.com/artifact/org.apache.poi/poi -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>4.1.0</version>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.poi/poi-ooxml -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>4.1.0</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
                <configuration>
                    <argLine>
                        -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar"
                    </argLine>
                    <!-- 测试失败继续运行 -->
                    <testFailureIgnore>true</testFailureIgnore>
                    <!-- testNG用例集xml设置 -->
                    <suiteXmlFiles>
                        <suiteXmlFile>
                            src/test/resources/testng/testNG.xml
                        </suiteXmlFile>
                    </suiteXmlFiles>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjweaver</artifactId>
                        <version>${aspectj.version}</version>
                    </dependency>
                </dependencies>
            </plugin>

            <!-- 构建插件, 指定版本与编码 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>utf-8</encoding>
                </configuration>
            </plugin>

        </plugins>
    </build>

    <reporting>
        <excludeDefaults>true</excludeDefaults>
        <plugins>
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>${allure.version}</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                </configuration>
            </plugin>
        </plugins>
    </reporting>
</project>
```

## 3.2 testNG.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="MyProjectSuite" parallel="classes" thread-count="1">
    <test verbose="2" preserve-order="true" name="登录测试">
        <classes>
            <class name="per.hao.cases.zendao.login.LoginTest" />
        </classes>
    </test>
    <test name="百度测试">
        <classes>
            <class name="per.hao.cases.BaiDuExampleTest" />
        </classes>
    </test>

    <listeners>
        <!-- 数据源监听(修改@Test注解中的部分配置) -->
        <listener class-name="per.hao.listener.DataSourceListener"/>
        <!-- 失败用例截图 -->
        <listener class-name="per.hao.listener.TestFaildListener"/>
    </listeners>
</suite>
```

# 4. 测试

## 4.1 测试数据

对应testNG.xml的两个测试类，测试用到的定位器文件元素已经放到[定位器文件示例](#定位器文件示例)中了

2. 1. 百度查询测试

![百度查询测试](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563595664620.png)

![百度查询测试数据](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563594616280.png)

2. 登录测试

![登录测试](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563594733031.png)

![登录测试](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563594754734.png)

![登录测试测试数据](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563595752354.png)

## 4.2 测试运行

项目根目录下打开命令行：

``` shell
# 1. 运行测试
mvn clean test site
# 2. 生成报告
mvn io.qameta.allure:allure-maven:serve
```

## 4.3 测试结果

![测试统计](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563596607684.png)

![测试结果](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563596572059.png)

![测试失败截图](https://www.github.com/hzhang123/bolgFiles/raw/master/xiaoshujiang/1563596636379.png)
