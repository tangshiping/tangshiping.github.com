---
layout: page
title: "Grails的测试"
description: ""
---
{% include JB/setup %}

以下内容参考Grails文档：[12 Testing 2.3.8](http://grails.org/doc/latest/guide/testing.html)

在使用`create-*`和`generate-*`命令时，会自动创建单元或集成测试，例如，运行`create-controller`命令：

```bash
grails create-controller com.acme.app.simple
```

创建内容包括：

- `grails-app/controllers/com/acme/app/SimpleController.groovy`
- 单元测试: `test/unit/com/acme/app/SimpleControllerTests.groovy`

# 运行测试

使用`test-app`命令：

```
grails test-app
```

会得到如下输出：

```
-------------------------------------------------------
Running Unit Tests…
Running test FooTests...FAILURE
Unit Tests Completed in 464ms …
-------------------------------------------------------
Tests failed: 0 errors, 1 failures
```

同时在`target/test-reports`目录下生成文本报告和HTML报告。

**推荐**：在Grails的交互模式下执行测试，有以下好处：

- 后续的测试执行速度会明显加快
- 在浏览器中快速打开测试报告：

```bash
open test-report
```

## 目标测试

运行名为"SimpleController"的控制器的所有测试：

```
grails test-app SimpleController
```

运行所有控制器的所有测试，可使用通配符`*`：

```
grails test-app *Controller
```

指定包名：

```
grails test-app some.org.*Controller
```

或者运行指定包中的所有测试：

```
grails test-app some.org.*
```

也可以运行包中的所有测试，包括子包：

```
grails test-app some.org.**.*
```

指定特定的测试方法：

```
grails test-app SimpleController.testLogin
```

更复杂的示例：


```
grails test-app some.org.* SimpleController.testLogin BookController
```

## 指定测试种类和阶段

语法格式：`phase:type`

Grails支持4个测试阶段：unit, integration, functional和other，及针对unit和integration的JUnit测试种类。

执行integration（集成）测试：

```
grails test-app integration:integration
```

phase和type都是可选的，以下命令运行unit阶段的所有测试种类：

```
grails test-app unit:
```

Grails的[Spock Plugin](http://grails.org/plugin/spock)增加了新的测试种类，针对unit, integration和functional阶段增加了`spock`测试，运行这些测试的命令如下：

```
grails test-app :spock
```

运行functional阶段的所有spock测试：

```
grails test-app functional:spock
```

也可以指定多个模式：

```
grails test-app unit:spock integration spock
```

## 在测试种类和阶段中指定目标

如下所示：

```
grails test-app integration: unit: some.org.**.*
```

将执行integration和unit两个阶段中所有在some.org包或子包下的测试。

# 单元测试

单元测试中不涉及物理资源如数据库。

## 测试混入(Test Mixins)

