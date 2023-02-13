## 1. lambda expressions are not supported a language level '1.5'

**使用IDEA配置JDK1.8版本使用lambda表达式报错**

**解决方法**

* 在“File -> Settings -> Build, Execution, Deployment -> Compiler”中找到“Java Compiler”，更改“Project bytecode version”和“Target bytecode version”。

![lambda_01](https://user-images.githubusercontent.com/49053144/218421348-46935098-6604-4b8d-a9a4-c5fbacc9b163.PNG)

* 在“File -> Settings -> Build, Execution, Deployment -> Compiler”中找到“Java Compiler”，更改“Project bytecode version”和“Target bytecode version”。

![lambda_02](https://user-images.githubusercontent.com/49053144/218421394-625fac26-b3c8-4291-832e-28903f47d2e9.PNG)

* 在“File -> Settings -> Build, Execution, Deployment -> Compiler”中找到“Java Compiler”，更改“Project bytecode version”和“Target bytecode version”。

![lambda_03](https://user-images.githubusercontent.com/49053144/218421275-0b8004b2-7e8c-4db5-9bde-9fd036d5a812.PNG)

* 若仍报错 “-source 1.5 中不支持 lambda 表达式”，则在pom.xml文件中增加如下内容：

```java
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    <build>


