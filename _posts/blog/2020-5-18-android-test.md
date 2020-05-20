---
layout: post
title: Android自动化测试
categories: [Blog,Cat]
description: Android自动化测试
keywords: Android, Auto，Test，Junit，Expresso，UIAutomator，Appium, Robolectric
---

# Android自动化测试

测试是应用开发必不可少的一环，服务端开发大部分会有自动化测试，因为服务端是接口类型，按照约定的格式测试即可，而且服务端每个接口都是独立的，更容易集成测试。

但是客户端的测试包含APP、网页却很难落地。纵观国内应用开发，几乎很少有真正落地客户端自动化测试的。因为客户端是面向用户的，用户可能会点击屏幕任意位置，或者任意长的流程才能走完一个事情。

## 测试的周期

![在测试开发周期中，先编写失败的单元测试，再编写代码以使其通过测试，然后重构。整个功能开发周期存在于一个基于界面的更大周期的一个步骤中。](https://tva1.sinaimg.cn/large/007S8ZIlly1gewtctl79xj30go0btgmx.jpg)

测试周期可以是很小的改动，频繁测试，抑或是大的重构，需要完整测试，无外乎以上两种。



## 测试实施的原则



![包含三层的金字塔](https://tva1.sinaimg.cn/large/007S8ZIlly1gewtenoxxij30go0a9wf8.jpg)





测试应包含单元测试，集成测试，UI测试。三者参照金字塔原则70%，20%，10%。沿着金字塔逐级向上，从小型测试到大型测试，各类测试的还原度逐级提高，但维护和调试工作所需的执行时间和工作量也逐级增加。



## 单元测试

客户端的单元测试与服务端没有太大差异，使用JUnit即可，断言可以使用google提供的Truth库。如果需要模拟Android环境，可以使用 [Robolectric](http://robolectric.org/)，能够测试以下行为：

- 组件生命周期

- 事件循环

- 所有资源

  

  

如果想在真机测试，可以使用AndroidJUnitTest(也称为插桩测试）默认在插桩测试线程运行，如果需要在主线程，使用  [`@UiThreadTest`](https://developer.android.google.cn/reference/androidx/test/annotation/UiThreadTest) 注解即可



## 集成测试

 Espresso 是很好的集成测试框架，能够完成此任务



## UI测试（大型测试）



Espresso可以满足需求。

同时如果是测试其他应用，或者没有源代码，可以使用[UI Automator](https://developer.android.google.cn/training/testing/ui-testing/uiautomator-testing)。

如果想跨android,ios,h5可以使用Appium



Appium环境搭建： https://www.cnblogs.com/Jack-cx/p/9968518.html

测试代码：

`

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
# coding:utf-8

from appium import webdriver
from selenium.webdriver.support.ui import WebDriverWait

from selenium.webdriver.support import expected_conditions as EC

from time import sleep

import unittest
import os
import time


class AndroidTest(unittest.TestCase):

    @classmethod
    def setUpClass(self):
        desired_caps = {}

        desired_caps['platformName'] = 'Android'  # 设备系统

        desired_caps['platformVersion'] = '10.0'  # 设备系统版本

        desired_caps['deviceName'] = '2KE0219B16021414'  # 设备名称

        desired_caps['appPackage'] = 'com.example.learnandroidtest'  # 上面提到获取的参数

        desired_caps['appActivity'] = 'com.example.learnandroidtest.MainActivity'  # 上面说到获取的参数

        self.driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)


    @classmethod
    def tearDownClass(self):
        self.driver.quit()


    def leftSwipe(self):
        window_size = self.driver.get_window_size()
        self.driver.swipe(start_x=window_size["width"] * 0.8,
                          start_y=window_size["height"] * 0.5,
                          end_x=window_size["width"] * 0.1,
                          end_y=window_size["height"] * 0.5)


    def wait_for_element(self, xpath=None, id=None, index=None, timeOut=20):
        startTime = time.time()
        nowTime = time.time()
        while nowTime - startTime < timeOut:
            try:
                if xpath is not None:
                    el = self.driver.find_element_by_xpath(xpath)
                    return el
            except:
                pass

            try:
                if id is not None:
                    if index is not None:
                        return self.driver.find_element_by_id(id)[index]
                    else:
                        return self.driver.find_element_by_id(id)
            except:
                pass

            sleep(1)

            nowTime = time.time()

        raise Exception("Element xpath[%s] id[%s] index[%s] is not found" % (xpath, id, index))


    def test_a_utFrame(self):
        print(self.driver.current_activity)


        time.sleep(8)
        circulation = 2

        while circulation > 0:
            time.sleep(1)
            self.leftSwipe()
            self.leftSwipe()

            self.wait_for_element(id="com.example.learnandroidtest:id/editText").clear().send_keys("Espresso")
            time.sleep(1)

            # 系统返回键
            self.wait_for_element(id="com.example.learnandroidtest:id/myButton").click()
            time.sleep(1)

            self.assertEqual(self.wait_for_element(id="com.example.learnandroidtest:id/textView").text, "Espresso")

            circulation -= 1


if __name__ == '__main__':
    suite = unittest.TestSuite()
    suite.addTest(AndroidTest("test_a_utFrame"))
    unittest.TextTestRunner(verbosity=2).run(suite)
```

`

