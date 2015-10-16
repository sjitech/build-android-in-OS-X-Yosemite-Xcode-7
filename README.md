# build-android-in-OS-X-Yosemite-Xcode-7
build whole android (AOSP) in Mac OS X Yosemite + Xcode 7.0.1(v10.5 SDK)


##My environment:
	Mac OX X 10.10.5(14F27)
	Xcode 7.0.1(7A1001)
	jdk1.7.0_80
	jdk1.8.0_51 (Default)

First of course, download AOSP by official <a href="https://source.android.com/source/downloading.html">instructions</a> except that i use android-5.1.1_r14 branch instead of android-4.0.1_r1.  

Then what i did especically are: 

##1. To avoid check error of OS X SDK version, run following command first:
	export build_mac_version=`sw_vers -productVersion`  #for me, result is 10.10.5
	export mac_sdk_version=10.9  #this is the biggest version AOSP build system support
	export mac_sdk_root=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk
	export gcc_darwin_version=11 

##2. To skip disk case sensitive check and java version check, create a file TOP_AOSP_DIR/out/versions_checked.mk with contents:
	VERSIONS_CHECKED := 5
	BUILD_EMULATOR ?= false

##3. To avoid error of some header files not found, create a symbol link MacOSX10.11.sdk/...c++/v1 => xctoolchain/...c++/v1: 

	sudo ln -s /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1 /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/usr/include/c++/v1

##4. AOSP's prebuild clang++ or g++ can not handle some latest header file of OS X SDK, i have no way, so modify following two files:

###  $mac_sdk_root/System/Library/Frameworks/CoreGraphics.framework/Versions/A/Headers/CGFont.h:

	53c53,54
	< static const CGFontIndex kCGGlyphMax = kCGFontIndexMax;
	---
	> //static const CGFontIndex kCGGlyphMax = kCGFontIndexMax;
	> static const CGFontIndex kCGGlyphMax = ((1 << 16) - 2);

###  $mac_sdk_root/System/Library/Frameworks/CoreServices.framework/Versions/A/Frameworks/FSEvents.framework/Versions/A/Headers/FSEvents.h:

	489a490
	> #if MAC_OS_X_VERSION_MIN_REQUIRED > MAC_OS_X_VERSION_10_9
	495a497
	> #endif



-------------------------
OK, now you can <a href="https://source.android.com/source/building.html">make</a>, as a tip, you can add "showcommands" option to make, and even combine more like following commands to save output with timestamp prefix to log file.

    make -j4 -k showcommands 2>&1 | (while read line; do echo `date +"%Y-%m-%d %H:%M:%S"` $line; done) | tee out/make.log

Good luck
