# ios-app-store-review-guidelines-tips

Tips:
1.个人App Store的审核似乎没有企业App Store账号审核严格。

2.上传app store时，提示兼容64位失败。   
  奇怪的是，虽然上传app store时失败并提示兼容64位设备失败，但是在真机调试运行时，无论是64位的iphone 6还是32位的iphone 5设备，都能正常运行，我也不知道什么原因。   
  后来修改了配置中的other linker Flags 中的参数，把 -all load ,改成 -all load -lz 却能成功上传app store。   
  虽然问题解决了，但是感觉还是瞎蒙的，上网找了下资料，有对上述参数的描述，顺便贴出来，方便大家了解。
  
  下面逐个介绍几个常用参数：
－ObjC：加了这个参数后，链接器就会把静态库中所有的Objective-C类和分类都加载到最后的可执行文件中

－all_load :会让链接器把所有找到的目标文件都加载到可执行文件中，但是千万不要随便使用这个参数！假如你使用了不止一个静态库文件，
然后又使用了这个参数，那么你很有可能会遇到ld: duplicate symbol错误，因为不同的库文件里面可能会有相同的目标文件，所以建议在遇到-ObjC失效的情况下使用-force_load参数。

-force_load：所做的事情跟-all_load其实是一样的，但是-force_load需要指定要进行全部加载的库文件的路径，这样的话，你就只是完全加载了一个库文件，不影响其余库文件的按需加载   






