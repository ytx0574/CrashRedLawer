###为App添加dylib:

- 1.自己编译或从越狱机提取dylib;
- 2.使用命令查看:otool -L kwmusic.dylib

###
	OPPO-X27:Desktop johnson$ otool -L kwmusic.dylib 
	kwmusic.dylib (architecture armv7):
	/Library/MobileSubstrate/DynamicLibraries/kwmusic.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
	/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1450.14.0)
	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1450.14.0)
	@loader_path/libsubstrate.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
	kwmusic.dylib (architecture arm64):
	/Library/MobileSubstrate/DynamicLibraries/kwmusic.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
	/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1450.14.0)
	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1450.14.0)
	@loader_path/libsubstrate.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
###
如上所示, 里面有包含对CydiaSubstrate的依赖, 这是一般越狱机特有的环境依赖, 那么非越狱机就需要修改此依赖

* 3.修改依赖, 提取出libsubstrate.dylib与即将使用的dylib, 并放在同一目录, 通过如下命令修改依赖

###
	install_name_tool -change /Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate @loader_path/libsubstrate.dylib kwmusic.dylib
###
之后可以再次使用上面的命令(otool -L kwmusic.dylib)进行查看, 发现依赖已发生变化, 如下:
###
	OPPO-X27:Desktop johnson$ otool -L kwmusic.dylib 
	kwmusic.dylib (architecture armv7):
	/Library/MobileSubstrate/DynamicLibraries/kwmusic.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
	/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1450.14.0)
	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1450.14.0)
	@loader_path/libsubstrate.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
	kwmusic.dylib (architecture arm64):
	/Library/MobileSubstrate/DynamicLibraries/kwmusic.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
	/System/Library/Frameworks/Foundation.framework/Foundation (compatibility version 300.0.0, current version 1450.14.0)
	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1450.14.0)
	@loader_path/libsubstrate.dylib (compatibility version 0.0.0, current version 0.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 104.2.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
###
* 4.重新签名kwmusic.dylib和libsubstrate.dylib

###
	codesign -f -s "证书名称" 签名文件路径
###

* 5.为可执行文件添加依赖[insert_dylib](https://github.com/Tyilo/insert_dylib), 编译后将得到的insert_dylib文件和另外两个.dylib文件放在同一目录, 执行下面的命令:


###
	./insert_dylib @executable_path/kwmusic.dylib /KWPlayer.app/KWPlayer
###
你会发现KWPlayer.app里面增加了一个加上后缀的同名文件"KWPlayer_patched", 把原来的KWPlayer改为KWPlayer_backup, 然后再把KWPlayer_patched改为
KWPlayer

* 6.重签名安装

安装失败的原因检查: 

1.使用如下命令匹配各个可执行文件的arch类型
###
	OPPO-X27:Desktop johnson$ lipo -info kwmusic.dylib 
	Architectures in the fat file: kwmusic.dylib are: armv7 arm64 
### 
如果类型不一致, 也会导致崩溃; 一般情况为iPhone5之类的机型, 找了一个iPhone5s提取出来的arm64, 安装肯定崩溃;

2.自行查看设备日志...








