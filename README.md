## 安卓 基于ANT的 Multi-Dex 构建支持
公司一直采用ant方式构建各个渠道包, 近来一些由于一些海外渠道需要支持非常多的第三方库, 导致单个dex中method数量超出64k, 引发
构建时TOP-LEVEL异常. 
经过一番研究, 修改标准build.xml, 支持了Multi-Dex.

#### 优点
- 基于ADT工程, 简单易用.
- 基于标准google android ant修改, 可应对多用项目需求:(多模块依赖, aidl, 多资源包叠加 ...), 并且稳定可靠. 

#### 准备

```
├── README.md
├── adt
│   ├── build_rule_isyuu.xml
│   ├── build.xml
│   ├── libs
│   │   └── android-support-multidex-1.0.1.jar
│   ├── main-dex-rule.txt
│   └── project.properties
└── sdk
    └── tools
        └── lib
            └── ant-contrib-1.0.jar
```

- 将sdk中 "ant-contrib-1.0.jar" 拷贝至本机 $ANDROID-SDK-ROOT/tools/lib 中
- 将adt中 build_rule_isyuu.xml, build.xml 拷贝至本地ADT项目中.
- 将adt中libs/android-support-multidex-1.0.1.jar 拷贝至本地ADT项目的libs中, 若ADT中已经有其他版本可以省略.
- 在ADT项目中project.properties 定义 support.multi.dex=true
- 拷贝adt中 main-dex-rule.txt 到 ADT项目下, 并在末尾追加启动Application类名, 或启动Activity 类名.
- 若apk在运行时尚未支持multi-dex, 请修改启动代码以支持, 否则会引发各种ClassNotFound 异常, 具体修改如下:
	- 若项目 Manifest中没有显示指定Application类, 则指定 Application name="android.support.multidex.MultiDexApplication" 即可.
	
	```
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
	```
	- 若项目自定义实现了 Application 类
		- 修改该类继承自 android.support.multidex.MultiDexApplication
	
	```
		//继承MultiDex Application 类
		public class MyApplication extends android.support.multidex.MultiDexApplication {
			...
		}

	```
	
	- 若无法修改继承关系(例如: 第三方要求必须继承他们的Application) 则在 attachBaseContext 重载中进行dex的安装.
		
	```
		public class MyApplication extends Application {
		    @Override
		    protected void attachBaseContext(Context base) {
		       super.attachBaseContext(base);
		       //安装dex
		       android.support.multidex.MultiDex.install(this);
		    }
		}
		
	```
	
	- 若项目没有自定义Application,但指定了第三方的Application, 则添加 自定义Application(例如MyApplication.java),进行如上修改, 然后在Manifest中指定自定义Application类名.
	

#### 构建

```
ant clean && ant release
```