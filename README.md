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

##### 3.PLA 3.3.12   Advertising Identifier 审核被拒解决方法 。
  分析如下：
    由于Apple修改了审核标准，苹果禁止不使用广告而采集IDFA的APP上架，IDFA只能用于广告服务。
    "You and Your Applications (and any third party with whom you have contracted to serve advertising) may us the Advertising Identifier, and any information obtained through the use of the Advertising Identifier, only for the purpose of serving advertising. If a user resets the Advertising Identifier, then You agree not to combine, correlate, link or otherwise associate, either directly or indirectly, the prior Advertising Identifier and any derived information with the reset Advertising Identifier.We also found that your app uses the Advertising Identifier but does not include ad functionality. This does not comply with the terms of the Apple Developer Program License Agreement, as required by the App Store Review Guidelines.If your app does not serve ads, please check your code - including any third-party libraries - to remove any instances of:class: ASIdentifierManager
selector: advertisingIdentifier framework: AdSupport.framework ......

报这条错误的原因如下

1 使用了第三方的库，第三方的库根据IDFA进行跟踪用户，同时APP没有加载广告。

2 使用了第三方的库，第三方的库根据IDFA进行跟踪用户，同时加载了iAD广告。

3 同时使用了iAD+ADMOB等广告

对应的解决方法：

第一种情况解决方法：
需要把和IDFA相关的代码和接口去除，因为IDFA只可以用于广告服务。

第二种情况解决方法：
iAD不使用IDFA，具体怎么实现的，iOS内部搞的，所以要解决这个问题需要把iAD换成类似Admob一类的广告服务，或者按照第一种情况来解决，就是去除第三方中IDFA相关的代码和接口。

第三种情况解决方法:
大概比较费解，明明加了Admob等广告，为啥还是给我拒绝了呢，这种情况要看广告的加载机制，一般开发者会优先加载iAD,如果没有广告源，则加载Admob（Admob是使用了IDFA），问题就出现了在这里，审核人员一般在美国，那里是有iAD的，或者现在app的状态还没有上线，iad属于测试状态，所以iAD的广告是可以获取，这样就给审核人员一个印象：app使用了IDFA（admob中），但是只是展示了iAD的广告，没有看到其他的广告服务，他们会怀疑你使用IDFA做了其他的事情，所以拒了！！！

解决方式：1.用终端命令在项目中查找那个文件中带有advertisingIdentifier、ASIdentifierManager等字样的字符串 strings LangQin/libMobClickLibrary.a | grep advertisingIdentifier ，在友盟统计中找到了带有advertisingIdentifier标识的字符串，而我们的应用没有加载任何广告，显然属于第一种情况，对应这种情况，在友盟官方提供了两套的SDK（即有无获取IDFA版的）。

解决方式:2.另外，官方还提供另外一种方法，正确填写在Appstore上填写IDFA选项。IDFA选项有四个（汉字是对这个四个选项的说明）

1.serve advertisements within the app
服务应用中的广告。如果你的应用中集成了广告的时候，你需要勾选这一项。

√2.Attribute this app installation to a previously served advertisement.
跟踪广告带来的安装。

√3.Attribute an action taken within this app to a previously served advertisement
跟踪广告带来的用户的后续行为。

√4.Limit Ad Tracking setting in iOS
这一项下的内容其实就是对你的应用使用idfa的目的做下确认，只要你选择了采集idfa，那么这一项都是需要勾选的。

如果应用没有使用广告而采集IDFA，则2，3，4项必须选上，但官方最后还有一句话“如果您仍因为采集IDFA被Appstore审核拒绝，建议您集成任意一家广告或选用友盟无IDFA版SDK”。这么做还是有可能被拒，即使没这么做过，我最后还是替换了无IDFA版的友盟SDK。



##### 4、 后台播放模式没录视屏审核被拒之解决之道-----问题盘查之比较分析法。

  问题描述：
  
  朗琴项目迭代版本提交审核报了这样的问题 “We began the review of your app but are not able to continue because we need access to a video that demonstrates your app in use on an iOS device. Specifically, since your app utilizes the background audio mode, please include the demonstration of such background mode(s) in use on an actual iOS device within your demo video (app running in the background)”.需要我们提供带有后台播放的Video再上传上去。
    
    朗琴中性版本首版本提交审核以后报了这样的问题“Multitasking Apps may only use background services for their intended purposes: VoIP, audio playback, location, task completion, local notifications, etc.Your app declares support for audio in the UIBackgroundModes key in your Info.plist but did not include features that require persistent audio.The audio key is intended for use by applications that provide audible content to the user while in the background, such as music player or streaming audio applications. Please revise your app to provide audible content to the user while the app is in the background or remove the "audio" setting from the UIBackgroundModes key.
”

  问题分析：
    
    这两个问题，虽然说法不一样，给人的映像却是一样的问题，只是暂时说不出个所以然，令我感到奇怪的是，为什么朗琴的首版本不报这个问题，非要到迭代版本了，才说要带有后台播放的视频，既然它要，那么给它就是了，对于第二个问题，通过百度、bing\stackOverFlow等等网页式搜索法也还是没找到确切的问题所在。很苦恼，“咋办”两字一直浮现在脑海里面，想过很多，一开始我还想他不是想证明一下能后台播放吗？索性我一打开应用，就播歌，它播歌回到后台以后看到了，就行了，但这种方法是个程序员都知道不好，.....看字面上的问题好像我的后台播放好像还没有配置好的意思，如果是这个原因，找其他的项目来看看，看看他们是不是和中性版本设置的一样，如果一样并且成上架了，那就不是这个问题，反之就是，如果不是，那就再找找之前的应用是不是都没有录后台播放的视频。如果没有录的也上传失败了，那就是这个原因。重录视屏。
    
  问题解决：
    
    把车载项目找来了看后台配置的代码，确实不一样，车载项目，在启动成功的方法里面，设置了后他播放的代码AVAudioSession *session = [AVAudioSession sharedInstance]; [session setActive:YES error:nil];[session setCategory:AVAudioSessionCategoryPlayback error:nil];而朗琴项目一个也没有，有点欣喜若狂的感觉，又找来了蜗灯项目，蜗灯项目和朗琴一样，没有在启动成功的方法里面设置，也成功了，在找了其它的几个项目，不管设置与否，都成功了。否决了这一命题，接下来就是有没有视频的事儿了，确实，问了下测试，之前的项目都有录后台播放视屏的事，但是后来的这几个视频没录，之前有几个项目也因为没有录后台播放视频，也是审核不过，被驳回了，朗琴合并版和中性版的一样，都没有录后台播放的视频，其它被驳回的APP后面录了视屏以后才成功上传。经过一番思考，还说不定就是这样的问题，貌似也只有这个可能了，录下视频。把应用传上去了再继续修改这里所写的东西。









