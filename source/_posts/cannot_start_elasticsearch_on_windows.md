title: "Windows运行ElasticSearch.bat出错问题解决方案"
date: 2018-08-16 11:31:32
tags:
- Windows
- ElasticSearch
---

在Windows上下载了ElasticSearch的zip包， 解压之后命令行进入bin目录， 运行elasticsearch， 不能够成功运行
提示

```
\Java\jdk1.8.0_151\bin\java.exe" -cp "C:\Users\<User Name>\App\elasticsearch-6.2.0\lib*" "org.elasticsearch.tools.launchers.JvmOptionsParser" "C:\Users\<User Name>\App\elasticsearch-6.2.0\config\jvm.options" || echo jvm_options_parser_failed"`) was unexpected at this time.

```
<!-- more -->
Java的路径是有问题的， 因为Java安装在了C:\Program Files (x86)\Java 路径下， 查看elasticsearch.bat文件， 找到出错的位置

```
for /F "usebackq" %%a in (`"%JAVA% -cp "!ES_CLASSPATH!" "org.elasticsearch.tools.launchers.JvmOptionsParser" "!ES_JVM_OPTIONS!" || echo jvm_options_parser_failed"`) do set JVM_OPTIONS=%%a
```
在StackOverFlow( https://stackoverflow.com/questions/6474738/batch-file-for-f-doesnt-work-if-path-has-spaces )上看到有人说for /F后面的路径如果有空格的话会出错，  可以通过使用`CALL` 来解决，

于是手工修改了elasticsearch.bat 把代码修改为

 ```
for /F "usebackq delims=" %%a in (`CALL %JAVA% -cp "!ES_CLASSPATH!" "org.elasticsearch.tools.launchers.JvmOptionsParser" "!ES_JVM_OPTIONS!" ^|^| echo jvm_options_parser_failed`) do set JVM_OPTIONS=%%a
 ```

 修改之后便可以成功的启动ElasticSearch了。

