
# Android 6.0 Changes
:toc:
:toc-placement: preamble
:toclevels: 3

官方文档：[http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html](http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html)

本文进行简单的翻译和总结。

在我看来，需要重点关注如下方面：

* 运行时权限带来的影响，考虑如何去适应和调整代码。
* 评估打盹模式和应用待机给产品带来的影响。
* 是时候完全移除对Apache HTTP Client的依赖。（发现国内好多SDK都在用，难！）
* 确保能够使用Android SDK r23编译你的代码。



:numbered:

## 运行时权限

在Android 6.0之前，对于应用所申请的权限，用户只能在安装应用时被动接受，
没有任何选择权利（除非放弃安装）。

Android 6.0引入了新的权限模型，允许用户在应用运行期间授予或者拒绝权限请求。
用户还可以通过权限管理界面来授予或撤销应用所申请的权限。
有了这个新的权限机制，在应用安装和升级时，不再需要用户确认权限或者权限变更，
使整个用户体验更加顺畅。

对于开发者，如果你打算把应用的Target SDK变更为23或者更高（Android 6.0+），
那么需要首先修改代码，在运行时去检查和请求所需权限。具体做法，请参考
http://developer.android.com/training/permissions/index.html 。

## 打盹模式和应用待机 (Doze & App Standby)

Android 6.0在省电优化方面引入了两个新的机制：
打盹模式（Doze）和应用待机（App Standby）。

### 打盹模式（Doze）

当设备未充电，且屏幕也是关闭状态，只要过上一小段时间，
设备就会进入打盹模式(Doze Mode)，以便让系统尽可能地保持在休眠状态。
在这种模式下，设备会周期性地苏醒过来一小会，以便应用和系统执行一些待执行的操作。

### 应用待机（App Standby）

应用待机（App Standby）机制允许系统决定，当用户很少使用一个应用时，其是否处于闲置状态。
当用户一段时间不去使用该应用时，系统会把其标记为闲置状态。

当设备未充电时，对于处于闲置状态的应用，系统会禁用它们的网络访问，
也会挂起它们的同步等其它任务。

关于 Doze & App Standby的更详细介绍，请参考
http://developer.android.com/training/monitoring-device-state/doze-standby.html 。

## Apache HTTP Client

Android 6.0已经移除了对Apache HTTP Client的支持，请使用HttpURLConnection来代替。

如果需要继续使用Apache HTTP，需要在build.gradle文件中添加一行（不建议再使用）：
```
android {
    useLibrary 'org.apache.http.legacy'
}
```
#（注：需要Gradle plugin v1.3.0+）#

## BoringSSL

在2014年OpenSSL的Heartbleed事件（参考 http://heartbleed.com ）发生之后，
忍了OpenSSL很久的那帮人终于找到了理由，开始了分裂OpenSSL的行动：

* fork版本 |描述

* LibreSSL
* OpenBSD的fork版本。据称，在fork后的第一星期，
OpenBSD团队首先清理掉了9万多行OpenSSL的老旧代码。

* BoringSSL
* Google的fork版本。



> 在BoringSSL项目启动一年多之后，#Android 6.0已正式使用BoringSSL代替OpenSSL。#

> 如果你的native代码直接使用了系统的OpenSSL库（libcrypto.so & libssl.so），
那么需要调整你的代码，确保在Android 6.0上也能工作。

> Android NDK一直未把OpenSSL纳入NDK API中。在native代码中，如果想要获取加密能力，
> 需要通过JNI调用Java的加密API；或者，自己引入其它加密库。

> 或许，将来有一天，BoringSSL有机会被纳入NDK API。
毕竟这是掌握在Google手中的棋子，想怎么玩就怎么玩。

## 获取硬件标识

* 为了保护用户的隐私，Android 6.0移除了WiFi和蓝牙的硬件标识的程序访问接口。
现在，WifiInfo.getMacAddress() 和 BluetoothAdapter.getAddress() 两个方法均
返回一个常量值 “02:00:00:00:00:00”。

* 通过蓝牙和Wi-Fi扫描时，为了获取附近外部设备的硬件标识，你的应用必须拥有相应的权限：

##
## 接口所需权限

* WifiManager.getScanResults()
* 需要拥有 ACCESS_FINE_LOCATION 和 ACCESS_COARSE_LOCATION 两个权限。

* BluetoothDevice.ACTION_FOUND
* 需要拥有 ACCESS_COARSE_LOCATION 权限。

* BluetoothLeScanner.startScan()
* 需要拥有 ACCESS_FINE_LOCATION 和 ACCESS_COARSE_LOCATION 两个权限。


* TIP: 当运行Android 6.0的设备发起后台Wi-Fi或者蓝牙扫描时，
外部的设备是可以感知到这个动作的，但外部设备看到的是一个随机产生的MAC地址。

## 通知（Notification）

> Android 6.0移除了 Notification.setLatestEventInfo() 接口。
现在，需要使用 Notification.Builder 来构建 Notification 对象。
当需要反复更新一个通知时，应该重用 Notification.Builder 对象，
通过调用 build() 方法来获取更新后的Notification对象。

> TIP: 通过 adb shell dumpsys notification 命令，不再打印出通知的文本信息。
可通过 adb shell dumpsys notification --noredact 命令打印出一个通知对象的文本。

## AudioManager

> 通过 AudioManager 去直接设置音量或者关闭声音的操作不再被被支持。
setStreamSolo() 方法已经被废弃，请改用 requestAudioFocus() 方法。
类似地，setStreamMute() 方法也已经被废弃，请改用 adjustStreamVolume() 方法，
传入参数 ADJUST_MUTE 或者 ADJUST_UNMUTE。

## 文本选择

> 当用户选中一段文本时，我们可以在展示一个悬浮工具条（floating toolbar），
里面有剪切、复制、粘贴等操作。悬浮工具条的用户交互实现类似于上下文action bar。

> 关于这里的文本选择设计规范可参考
http://www.google.com/design/spec/patterns/selection.html#selection-text-selection 。

> 具体用法可参考
http://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-test-selection 。

> 目前仅支持Android 6.0，Support库也还不支持悬浮工具条。

## Android Keystore

> Android keystore provider不再支持DSA，但ECDSA依然被支持。

> 当安全锁屏被禁用或者重置时，那些不需要加密的Key将不再被删除，
而那些需要加密的Key将被删除。（不理解这些Key具体指什么Key）

## Wi-Fi和网络

> 现在，只有你自己创建的WifiConfuration，你才能够去修改它的状态。
你不再被允许去修改或者删除其他用户或者应用创建的WifiConfiguration。

> 之前，应用可以通过 enableNetwok() 和 “disableAllOthers=true”
来强迫设备去连接特定的Wi-Fi网络，并从其它网络断开。
现在，设备不再从其它网络断开。
可以通过 multinetwork API（Network#openConnection() & Network#bindSocket()
& ConnectivityManager#bindProcessToNetwork()）来确保使用特定的网络。

## Camera Service

> 之前，在Camera Service中访问共享资源是按照“先来，先服务”的策略。
现在，改为了按优先级处理的策略。具体情况如下：

* 访问Camera子系统资源，包括打开、配置Camera设备，是基于Camara client应用进程的优先级
来服务的。用户可见或者有前台Activity的应用进程，被赋予了高优先级。

* 当一个高优先级的应用试图访问Camera时，低优先级的活跃Camera client可能会被踢掉。
这个时候，被踢掉的Camera client的onError()（老Camera API）
或者onDisconnected()（Camera2 API）会被回调。

* 当有适当的Camera硬件时，不同的应用进程能够独自且同时打开和使用不同的Camera设备。
然而，对于多进程的使用场景（同时访问可能引起Camera设备的严重性能下降或者服务问题），
现在能够被Camera service检测到并禁止。这个改动可能会导致踢掉低优先级client，
尽管没有其它应用试图访问Camera设备。

* 切换当前用户，会导致前一个用户账号拥有的应用的活跃Camera client被踢开。
访问Camera仅限于当前设备用户所拥有的user profiles。

## Runtime

>ART Runtime已经正确实现了Constructor#newInstance()的访问规则。
这个改动修复了之前Dalvik没有正确检测访问规则的问题。
如果你的应用调用了Constructor#newInstance()，并且想重写访问规则，
请调用setAccessible并传递参数true。
如果你的应用使用了 v7-appcompat库 或者 v7-recycleview库，
你必须更新你的应用使用最新的support库。
否则，请确保所有XML中引用的自定义类都已更新，以便其constructors可以被访问。

> Android 6.0也更新了动态链接器的行为。现在，动态链接器已经理解一个库的so名字和路径的差异
（参见bug https://code.google.com/p/android/issues/detail?id=6670 ），
通过so名字搜索已经被正确实现。之前有错误的DT_NEEDED条目（通常是编译机文件系统中的绝对路径）
的应用是可以工作的，现在在加载时可能会失败。

> dlopen(3) RTLD_LOCAL 参数已经被正确实现。由于 RTLD_LOCAL 是默认的，
因此对那些没有显示使用 RTLD_LOCAL 的 dlopen(3) 调用也是有影响的。
通过 RTLD_LOCAL，后面通过 dlopen(3) 调用加载的库将不能够使用之前加载的库中的符号。

> 在之前的Android版本中，如果你的应用请求系统加载一个有text relocations的共享库时，
系统会显示一个警告但依然允许加载那个库。
从Android 6.0开始，如果你的应用Target SDK是23（或者更高），那么系统会拒绝加载该库。
关于text relocations，可参考此文档
https://wiki.gentoo.org/wiki/Hardened/Textrels_Guide 。

## APK包验证

> 现在，系统执行更严格的APK包验证。如果清单文件中声明了一个文件，但却不存在于APK中，
那么系统会认为这个APK包是破损的。如果APK中的任何文件被移除，必须对APK包进行重签名。

## USB连接

> 现在，通过USB端口的设备连接被设置为仅充电模式。为了通过USB连接来访问设备和其中的内容，
用户必须显示授予权限。

## 企业Android（Android for Work）

* Android 6.0包含了许多企业Android的改动。

### 企业联系人

> 当用户查看通话记录时，Google Dialer Call Log能够展示企业联系人。
可调用并传递参数 true 给 DevicePolicyManager#setCrossProfileCallerIdDisabled()，
来阻止企业联系人出现在Google Dialer Call Log中。
仅当调用并传递参数 false 给 DevicePolicyManager.setBluetoothContactSharingDisabled() 时，
企业联系人才可以被蓝牙设备所访问（默认不可访问）。

### Wi-Fi配置

> 如果一个企业档案（Work Profile）被删除时，那么其所添加的Wi-Fi配置也会被一同删除。

> 如果 Settings.Global#WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN 设置为非零，
那么由已激活的 Device Owners 所创建的任何Wi-Fi配置都不可以被用户修改或删除。
用户依然可以创建和修改他们自己的Wi-Fi配置。已激活的 Device Owners
有特权编辑或删除任何Wi-Fi配置，包括那些别人创建的。

### WPC安装提示

> 在非 managed context 情况下添加Google帐户时，如果Google账户需要通过
Work Policy Controller 应用 (WPC) 来管理，那么添加过程中会提示用户安装合适的WPC。
同样，通过 “Settings > Accounts” 和 设备初次使用引导程序
发起的添加帐户也会有这个提示。

### DevicePolicyManager APIs 变化情况

* 调用 setCameraDisabled() 仅影响当前调用用户使用相机。
在 managed profile 中调用该方法，不会影响主帐户中应用的相机使用。

* setKeyguardDisabledFeatures()方法对 Profile Owners 和 Device Owners 都可用了。

* Profile Owner 现在可以设置如下的keyguard限制：
> * KEYGUARD_DISABLE_TRUST_AGENTS 和 KEYGUARD_DISABLE_FINGERPRINT，
能够影响该 profile 的parent user。
> * KEYGUARD_DISABLE_UNREDACTED_NOTIFICATIONS，
仅影响 managed profile 中应用产生的通知。

* createAndInitializeUser() 和 createUser() 两个方法已被废弃。

* （理解不了这句话）The setScreenCaptureDisabled() method now also blocks the assist structure when an app of the given user is in the foreground.

* EXTRA_PROVISIONING_DEVICE_ADMIN_PACKAGE_CHECKSUM 已默认使用SHA-256，
依然支持SHA-1以便向后兼容（但将来会放弃支持）。
EXTRA_PROVISIONING_DEVICE_ADMIN_SIGNATURE_CHECKSUM 现在仅支持SHA-256。

* （这句话写错了？）Device initializer APIs which existed in the Android 6.0 (API level 23) are now removed.

* EXTRA_PROVISIONING_RESET_PROTECTION_PARAMETERS 已被移除，
以便 NFC dump provisioning 不可以用编程方式去解锁一个有恢复出厂设置保护的设备。

* 在 managed device 的 NFC provisioning过程中，
现在可以使用 EXTRA_PROVISIONING_ADMIN_EXTRAS_BUNDLE 来传递数据给 device owner app，

* Android企业API都为Android 6.0运行时权限做了优化，包括 Work profile、assist layer等。
DevicePolicyManager中新的权限API不会影响 pre-M 应用。

* 当用户退出由 ACTION_PROVISION_MANAGED_PROFILE 或者 ACTION_PROVISION_MANAGED_DEVICE
发起的设置流程时，系统会返回返回码 Activity#RESULT_CANCELED。

### 其它API变化情况

* Data usage：类 android.app.usage.NetworkUsageStats 改名为 NetworkStats。

### Global settings中的变化情况

* 如下设置不再通过 DevicePolicyManager#setGlobalSettings() 来设置：
** BLUETOOTH_ON
** DEVELOPMENT_SETTINGS_ENABLED
** MODE_RINGER
** NETWORK_PREFERENCE
** WIFI_ON

* 如下设置可通过 DevicePolicyManager#setGlobalSettings()来设置：
** Settings.Global#WIFI_DEVICE_OWNER_CONFIGS_LOCKDOWN
