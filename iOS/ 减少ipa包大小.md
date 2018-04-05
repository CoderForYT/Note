# 减少ipa包大小

## 资源文件的优化

1. 删除不使用的图片，文件，md文件等等。删除图片可以使用：LSUnusedResources进行删除 

   LSUnusedResources下载链接： http://blog.lessfun.com/blog/2015/09/02/find-unused-resources-in-xcode-project/

2. 压缩图片可以使用ImageOptim对png无损压缩，甚至可以进行有损压缩。

   ImageOptim 下载链接：https://imageoptim.com/mac

3. 图片都是用Assets.xcassets进行管理



## 代码优化

1. 删减未被使用的代码，和库类。可以使用AppCode进行分析

2. 使用只支持arm64, armv7, armv7s 的静态库（arm64, armv7, armv7s 真机， i386, x86_64 模拟器）

3. 减少冗余字符串

   代码上定义的所有静态字符串都会记录在在可执行文件的__cstring段，如果项目里Log非常多，这个空间占用也是可观的，也有几百K的大小，可以考虑清理所有冗余的字符串。另外如果有特别长的字符串，建议抽离保存成静态文件，因为AppStore对可执行文件加密导致压缩率低，特别长的字符串抽离成静态资源文件后压缩率会比在可执行文件里高很多。

4. 缩短类/方法名长度

   观察linkmap可以发现每个类和方法名都在__cstring段里都存了相应的字符串值，所以类和方法名的长短也是对可执行文件大小是有影响的，原因还是object-c的动态特性，因为需要通过类/方法名反射找到这个类/方法进行调用，object-c对象模型会把类名，方法名列表都保存下来。

   实际上这部分占用的长度比较小，中型项目也就几百K，对安全性要求高的情况可以试试。



## 编译优化(xcode9.1)

相关文章 ： 

 * https://baijiahao.baidu.com/s?id=1561897201123027&wfr=spider&for=pc
 * https://www.jianshu.com/p/11710e7ab661

主要设置：

1. Optimization Level ：smallest, space

2. Deployment Postprocessing ：YES

3. Strip Linked Product：DEBUG下设为NO，RELEASE下设为YES，用于RELEASE模式下缩减app的大小；

   ​

## 思维导图

```
http://aliyunzixunbucket.oss-cn-beijing.aliyuncs.com/jpg/b3c5bb04e7919c897f3923f41a2100f2.jpg?x-oss-process=image/resize,p_100/auto-orient,1/quality,q_90/format,jpg/watermark,image_eXVuY2VzaGk=,t_100,g_se,x_0,y_0
```

