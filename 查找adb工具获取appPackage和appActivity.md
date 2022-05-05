https://zhuanlan.zhihu.com/p/144737398





### 查找adb工具获取appPackage和appActivity

```
adb shell
dumpsys window windows | grep -E 'App'
dumpsys window windows | grep -E 'mTop'

```

snapshot=TaskSnapshot{ mId=1650511716455 mTopActivityComponent=com.tencent.mm/.ui.LauncherUI mSnapshot=android.hardware.HardwareBuffer@3d9f752 (1080x2376) mColorSpace=sRGB IEC61966-2.1 (id=0, model=RGB) mOrientation=1 mRotation=0 mTaskSize=Point(1080, 2376) mContentInsets=[0,96][0,0] mIsLowResolution=false mIsRealSnapshot=true mWindowingMode=1 mAppearance=24 mVivoBarAppearance=24 mIsTranslucent=false mHasImeSurface=false



可以看出

```
  appPackage='com.tencent.mm',
            appActivity='.ui.LauncherUI',
```

### 加上noreset

```
noReset='True'
```

不会重新登录





最近调试 脚本，发现运行过程中，开始的时候一两条都运行OK，后面一直报错Connection aborted.', RemoteDisconnected('Remote end closed connection without response'

  中间经历很多，有类似经验的伙伴 说的 VIVO 手机 电池优化把appium  进程杀掉了，！！！！ 后面把电池设置一下就OK 了，巨坑，都找不到啥原因

appium  settings  已经安装的另外两个apk包都需要设置允许后台高耗电才行，设置其中一个进程也会被杀掉

appium  踩坑集
https://blog.51cto.com/u_15127517/4046518