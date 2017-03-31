---
title: 记Selenium WebDriver在IE chorme firefox浏览器运行
date: 2015-1-4 15:35:07
tags: [test,selenium]
categories: test
---

### InternetExplorer
```
    WebDriver driver = new InternetExplorerDriver();
    driver.get("http://www.baidu.com");
```
直接运行会报错，的错处理如下：
需要设置IE的Driver到“webdriver.ie.driver”变量中，否则可能遇到报错信息：
```
 The path to the driver executable must be set by the webdriver.ie.driver system property; for more information, see http://code.google.com/p/selenium/wiki/InternetExplorerDriver. The latest version can be downloaded from http://code.google.com/p/selenium/downloads/list
```
因为在IE中测试需要IEDriverServer.ext的支持，从官网下载，区分64位和32分，将这个路径赋值给webdriver.ie.driver
```
 System.setProperty("webdriver.ie.driver", "D:\\web\\test\\selenium\\IEDriverServer_x64_2.44.0\\IEDriverServer.exe");
```
如果IE浏览器设置安全性较高，在“Internet Options”中都不要选择“Enable Protected Mode”（保护模式），否则可能遇到如下的错误提示
```
 Started InternetExplorerDriver server (64-bit)
 2.25.2.0
 Listening on port 40961
 Exception in thread "main" org.openqa.selenium.WebDriverException: Unexpected error launching Internet Explorer. Protected Mode settings are not the same for all zones. Enable Protected Mode must be set to the same value (enabled or disabled) for all zones. (WARNING: The server did not provide any stacktrace information)
 Command duration or timeout: 1.18 seconds
 Build info: version: '2.25.0', revision: '17482', time: '2012-07-18 22:18:01'
 System info: os.name: 'Windows 7', os.arch: 'x86', os.version: '6.1', java.version: '1.6.0_29'
 Driver info: driver.version: InternetExplorerDriver
 Session ID: 01e30b64-e403-440c-bed8-4859ef2128f9
     at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
     at sun.reflect.NativeConstructorAccessorImpl.newInstance(Unknown Source)
     at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(Unknown Source)
     at java.lang.reflect.Constructor.newInstance(Unknown Source)
     at org.openqa.selenium.remote.ErrorHandler.createThrowable(ErrorHandler.java:188)
     at org.openqa.selenium.remote.ErrorHandler.throwIfResponseFailed(ErrorHandler.java:145)
     at org.openqa.selenium.remote.RemoteWebDriver.execute(RemoteWebDriver.java:498)
     at org.openqa.selenium.remote.RemoteWebDriver.startSession(RemoteWebDriver.java:182)
     at org.openqa.selenium.remote.RemoteWebDriver.startSession(RemoteWebDriver.java:167)
     at org.openqa.selenium.ie.InternetExplorerDriver.startSession(InternetExplorerDriver.java:133)
     at org.openqa.selenium.ie.InternetExplorerDriver.setup(InternetExplorerDriver.java:106)
     at org.openqa.selenium.ie.InternetExplorerDriver.<init>(InternetExplorerDriver.java:52)
     at com.selenium.test.TempGoogle.main(TempGoogle.java:15)
```
解决方法有两种，一种是修改掉IE的设置，不要在任何情况下使用保护模式（protected mode），另一种即是在运行时设置IE的Capabilities。
最终代码如下
```
 System.setProperty("webdriver.ie.driver", "D:\\web\\test\\selenium\\IEDriverServer_x64_2.44.0\\IEDriverServer.exe");
 DesiredCapabilities ieCapabilities = DesiredCapabilities.internetExplorer();
 ieCapabilities.setCapability(InternetExplorerDriver.INTRODUCE_FLAKINESS_BY_IGNORING_SECURITY_DOMAINS, true);
 WebDriver driver = new InternetExplorerDriver(ieCapabilities);
    driver.get("http://www.baidu.com");
    WebElement element = driver.findElement(By.name("wd"));
    element.sendKeys("ztesoft");
    element.submit();
    System.out.println("Page Title is: " + driver.getTitle());

    (new WebDriverWait(driver,10)).until(new ExpectedCondition<Boolean>() {
        public Boolean apply(WebDriver d) {
            return d.getTitle().toLowerCase().startsWith("ztesoft");
        }
    });

    System.out.println("page title is: " + driver.getTitle());
    driver.quit();
```

### Chrome
在Chrome浏览器上运行测试脚本，首先需要下载ChromeDriver.exe
按操作系统区分
chromedriver_win32
chromedriver_mac32
chromedriver_linux64
设置变量 “webdriver.chrome.driver”为对应的路径

### Firefox
Firefox是默认安装路径，webdriver可以正常访问找到它。
如果非默认路径，需要设置下
```
System.setProperty("webdriver.firefox.bin", "C:\\Program Files\\Mozilla Firefox\\firefox.exe");
WebDriver webDriver = new FirefoxDriver();
```

### 常见问题
#### sendKeys在IE下输入英文或者数字会很慢
现象：sendKeys在IE下输入英文或者数字会很慢，中文者不会。而在Firefox下则完全没有这个问题
处理： 参考 http://stackoverflow.com/questions/8790420/how-to-speed-up-sendkeys-in-internet-explorer-when-using-selenium-webdriver-2
采用第二种方法用javascript方式 解决
```java
  String setEffDate = "document.getElementById(\"effectiveDate\").value=\"2013-08-02\"";
  ((JavascriptExecutor)driver).executeScript(setEffDate);

  String setExpireDate = "document.getElementById(\"expireDate\").value=\"2013-08-03\"";
  ((JavascriptExecutor)driver).executeScript(setExpireDate);
```

#### 对于日期型选择输入控件不可以直接输入的处理
```java
 //现在这个地方不可以直接输入了，可以通过js来输入
    String str = "document.getElementById(\"startdatepicker\").readonly=false";
    String strDate = "document.getElementById(\"startdatepicker\").value=\"2013-08-02\"";
    ((JavascriptExecutor)wd).executeScript(str);
    ((JavascriptExecutor)wd).executeScript(strDate);
```

#### 报错ElementNotVisibleException
driver.findElement(By.id("inlineCheckbox2")).click();
在IE下无法对隐藏元素调用click()方法，否则会报错ElementNotVisibleException

解决的方法有两种：
1. 把隐藏元素先给显示出来再赋值
2. 用javascript方式

注：在已加载了JQuery的页面上可以直接使用JQuery选择器，进行元素的操作
