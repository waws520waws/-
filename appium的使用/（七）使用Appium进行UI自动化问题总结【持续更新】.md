# （七）使用Appium进行UI自动化问题总结【持续更新】

##  前言

在使用appium进行批量跑case或单个测试的时候经常出现许多问题，现作出一些总结与回应。

![img](https://img-blog.csdnimg.cn/20190430171704530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2N4MjQzNjk4,size_16,color_FFFFFF,t_70)

## 问题

### 1、Appium健壮性

**描述**：由于现在appium并不是特别成熟，在运行case的时候经常会出现appium识别不了元素（元素实际存在）、appium运行一段时间后自动退出。

**建议**：鉴于此原因，建议每次跑case的时候，重启appium。（有时甚至需要连续重启两次，第一次重启刚跑没几分钟就挂了，第二次相对稳定）

### 2、手机连接问题

**描述**：使用手机连接电脑运行case时，容易出现appium读取不到设备信息。甚至在正在运行过程中手机突然中断也是会出现的。

**建议**：使用连接良好的数据线，保持网络稳定。

## 总结

总之进行自动化测试的时候需要考虑appium、手机连接、网络环境等等问题的尽可能规避。