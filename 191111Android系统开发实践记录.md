# Android 系统开发

## 系统源代码下载

```bash
git clone git@192.168.181.9:mtk/repo.git
chmod 777 repo/repo
mkdir alps
cd alps
../repo/repo init -u git@192.168.181.9:mtk/manifest.git -m mt8788_9.0.xml
../repo/repo sync
../repo/repo start master --all  # 本地自定义分支名字，master
../repo/repo forall -p -c git checkout -b T22 mt8788_9.0/T22  # 切换到 T22 分支
```



## 系统源代码编译

```bash
vim wisky_init.sh
chmod a+x wisky_init.sh
chmod 755 wisky_init.sh
./wisky_init.sh
```



一开始由于 wisky_init.sh 有错误，删除掉它生成的文件后，修改重新执行后成功。

正确的文件内容：

```shell
cd build/
ln -s make/target/ target
ln -s make/tools/ tools
ln -s make/core/ core
ln -s make/buildspec.mk.default buildspec.mk.default
ln -s make/CleanSpec.mk CleanSpec.mk
ln -s make/envsetup.sh envsetup.sh
cd ../
cp build/make/core/root.mk Makefile
ln -s build/soong/root.bp Android.bp
ln -s build/soong/bootstrap.bash bootstrap.bash
```



查看之前编译设置的信息：

`litao@grape:~/lt/alps/out/target/product/tb8788p1_64_bsp/system$ vim build.prop`



**每次新连终端的时候，都要执行一下 `source build/envsetup.sh` 并 `lunch`，不然 make 会报错**

`make installclean -j16`

`litao@grape:~/lt/alps$ . wisky/common_config_nota.sh wisky/customer/T22_tiepian/`



~~报错以后，做了以下修改：~~

 https://blog.csdn.net/yanglin222/article/details/100552803 



```bash
zhaosenlin@wisky11:~$ ls
mtk
zhaosenlin@wisky11:~$ cd mtk/
zhaosenlin@wisky11:~/mtk$ ls
apk  mt8768_9  repo  T22  vendor.img
zhaosenlin@wisky11:~/mtk$ cd T22/  
zhaosenlin@wisky11:~/mtk/T22$ source          
zhaosenlin@wisky11:~/mtk/T22$ source build/envsetup.sh 
zhaosenlin@wisky11:~/mtk/T22$ cd out/target/
common/  product/ 
zhaosenlin@wisky11:~/mtk/T22$ cd out/target/product/tb8788p1_64_bsp/system/
zhaosenlin@wisky11:~/mtk/T22/out/target/product/tb8788p1_64_bsp/system$ vim build.prop 
zhaosenlin@wisky11:~/mtk/T22/out/target/product/tb8788p1_64_bsp/system$ cd ../../../../
zhaosenlin@wisky11:~/mtk/T22/out$ cd ../
zhaosenlin@wisky11:~/mtk/T22$ lunch
zhaosenlin@wisky11:~/mtk/T22$ make installclean -j16
zhaosenlin@wisky11:~/mtk/T22$ . wisky/common_config_nota.sh wisky/customer/T22/

make snod
```

编译如果发生错误，可以查看根目录下的 `build.log` 文件，搜索 `failed` 字段。



常用的机器和代码地址：

`zhengjunhua@tomato:~/mtk/mt8788_9$`



## 系统安装

使用 `Flash_ool` 来刷入系统：

进入 `Download` 页面，将 `Scatter-loading File` 选中为 `./out/target/product/tb8788p1_64_bsp/MT6771_Android_scatter.txt`

模式选择 `Download Only`

点击 `Download` 按钮，然后将设备数据线连接到电脑。



另一个已经编译过的系统：

`Z:\mtk\mt8788_backup\pub\T22-images-v1.02_20191112user\images\MT6771_Android_scatter.txt` 



## 编辑器

`Source Insight`

>  打开项目：点击顶部的 Project -> New Project
>
> SourceInsight F3 向前搜索，F4 向后搜索
>
> 上一个位置 Alt + ,    下一个位置 Alt + .



## 工作任务

##### 更换系统原桌面壁纸

~~使用目标壁纸替换 `framework/base/core/res/res` 目录下 `drawable-nodpi/drawable-xhdpi/drawale-xxhdpi/drawable-xxxhdpi` 这四个文件夹下面的 `default_wallpaper` 文件。~~

文件位置：

`./lt/alps/frameworks/base/core/res/res/drawable-sw600dp-nodpi/default_wallpaper.png`



##### 隐藏移动网络的三角图标（内置 EMS 卡）

~~修改文件`framework/base/packages/SystemUI/src/com/android/systemui/statusbar/SignalClusterView.java`，修改 440 行 apply() 函数。~~

~~FAQ11708~~

~~`frameworks/base/core/res/res/values$ vim config.xml`~~

~~`frameworks\base\packages\SettingsProvider\res\values\defaults.xml` 第 49 行，改成 false，~~

~~` frameworks\base\packages\SettingsProvider\src\com\android\providers\settings\DatabaseHelper.java ` 2549 行，将 loadSetting 最后一个函数改为 false。~~

~~原来一直搞错了-_-，netstats_enabled 是流量统计 :)~~



11.19

~~修改 `vendor\mediatek\proprietary\packages\services\Telephony\src\com\android\phone\MobileNetworkSettings.java` 213 行为 false。~~

~~`<bool name="def_mobile_data_always_on">false</bool>`~~

~~` def_networks_available_notification_on" `~~



11.20（错误）

`/frameworks/base/packages/SystemUI/src/com/android/systemui/statusbar/policy`

```java
291 /*        IconState statusIcon = new IconState(mCurrentState.enabled && !mCurrentState.airplaneMode,
292                 getCurrentIconId(), contentDescription);*/
293         IconState statusIcon = new IconState(false && !mCurrentState.airplaneMode,
294                 getCurrentIconId(), contentDescription);
```



##### 出厂默认开放安装 apk 和使用 U 盘的权限

安装 apk：设置 `frameworks/base/packages/SettingsProvider/res/values ` 中的 ` def_install_non_market_apps ` 为 1，` def_package_verifier_enable ` 为 0。

使用 U 盘的权限：



##### 隐藏开发者模式中的“默认USB配置”

[参考文章]( https://blog.csdn.net/wxd_csdn_2016/article/details/84791287 )

 https://blog.csdn.net/qq_34149526?t=1 

[参考删除偏好设置的按钮]( https://blog.csdn.net/jydzm/article/details/85850087 )



##### 更换系统开机 logo

FAQ08771

判断当前设备使用的 logo 图片是哪一个尺寸的：

`./device/mediateksample/tb8788p1_64_bsp/ProjectConfig.mk` 文件中含有 Logo 的标记，在文件夹 `./vendor/mediatek/proprietary/bootable/bootloader/lk/dev/logo` 中，包含了 Logo 标记的文件夹，替换其中的图片文件即可。



# 11.20（未解决）

`vendor/mediatek/proprietary/packages/apps/SettingsProvider/res/values/defaults.xml`

修改了 install app，stats，always data



`./proprietary/packages/apps/SystemUI/src/com/android/systemui/statusbar/policy/MobileSignalController.java`

`587         boolean showDataIcon = false;`



# 11.21（未解决）

`<string name="def_location_providers_allowed" translatable="false">gps,network</string>`

删掉 network



proprietary/packages/apps/SystemUI/src/com/android/systemui/statusbar/policy/MobileSignalController.java

```
 893     private boolean isDataDisabled() {
 894         boolean isDataDisabled = !mPhone.getDataEnabled(mSubscriptionInfo.getSubscriptionId());
 895         if (DEBUG) {
 896             Log.d(mTag, "isDataDisabled = " + isDataDisabled);
 897         }
 898         return true;
 899         //return isDataDisabled;
 900     }
```



# 11.22（未解决）

```
`258     <string name="mobile_data_settings_summary" msgid="5087255915840576895">"通过移动网络访问数据"</string>`
```

`litao@grape:/orange/litao/alps/vendor/mediatek/proprietary/packages/services/Telephony/res/values-zh-rCN$ vim strings.xml`



```
<com.android.phone.MobileDataPreference
        android:key="mobile_data_enable"
        android:title="@string/mobile_data_settings_title"
        android:summary="@string/mobile_data_settings_summary"/>
```

`litao@grape:/orange/litao/alps/vendor/mediatek/proprietary/packages/services/Telephony/res/xml$ vim network_setting_fragment.xml`



`litao@grape:/orange/litao/alps/vendor/mediatek/proprietary/packages/services/Telephony/src/com/android/phone$ vim MobileNetworkSettings.java`

```
重要的变量
mMobileDataPref

 212         final boolean enabledEsimUiByDefault =
 213                 SystemProperties.getBoolean(KEY_ENABLE_ESIM_UI_BY_DEFAULT, true);


 310         static final int preferredNetworkMode = Phone.PREFERRED_NT_MODE;


1072             /*mMobileDataPref.setEnabled(hasActiveSubscriptions
1073                     && shouldEnableCellDataPrefForSimLock());*/
1074             mMobileDataPref.setEnabled(false);

```





找 android.provider.Settings.Global.PREFERRED_NETWORK_MODE

frameworks/base/core/java/android/provider/Settings.java`



`litao@grape:/orange/litao/alps/vendor/mediatek$ vim proprietary/packages/services/Telephony/src/com/android/phone/MobileDataPreference.java`



`./frameworks/base/core/java/android/preference/Preference.java`

```
114     private void updateChecked() {
115         //setChecked(mTelephonyManager.getDataEnabled(mSubId));
116         setChecked(false);
117     }


1288     /**
1289      * Should be called when the data of this {@link Preference} has changed.
1290      */
1291     protected void notifyChanged() {
1292         if (mListener != null) {
1293            // mListener.onPreferenceChange(this);
1294         }
1295     }
```



是 MobilePreferenceData 的父类，开机初始化在这个地方，setEnabled，将 mEnabled 改成 false

`./frameworks/base/core/java/android/preference/DialogPreference.java`



# 1123

## 移动图标的问题（未解决）

>思路：
>
>- 找到控制移动信号图标显示的代码（目前的思路是通过设置中的移动网络设置代码来查找）
>
>  ```java
>  <string name="data_usage_disable_mobile" msgid="3577275288809667615">"要关闭移动数据网络吗？"</string>
>  
>  // 最终的关闭网络实现在 DialogPreference.java 文件中，292 行
>  
>  MobileDataPreference.java 中的 232 实现了 dialog 的逻辑，这个地方实现了对移动数据的操作
>      
>      @Override
>      public void onClick(DialogInterface dialog, int which) {
>          if (which != DialogInterface.BUTTON_POSITIVE) {
>              return;
>          }
>          if (mMultiSimDialog) {
>              mSubscriptionManager.setDefaultDataSubId(mSubId);
>              //setMobileDataEnabled(true);
>              /// M: To support multiple IMS.
>              disableDataForOtherSubscriptions(mSubId);
>          } else {
>              // TODO: extend to modify policy enabled flag.
>              setMobileDataEnabled(false);
>          }
>  	}
>  ```
>
>  
>
>- 看哪些地方调用了显示的代码，从而找到系统初始化移动网络的代码
>
>  `framework/base/telephony/java/android/telephony/TelephonyManager.java` 文件对网络进行的操作
>
>  6412 行：
>
>  ```java
>  /**
>       * Turns mobile data on or off.
>       * If this object has been created with {@link #createForSubscriptionId}, applies to the given
>       * subId. Otherwise, applies to {@link SubscriptionManager#getDefaultDataSubscriptionId()}
>       *
>       * <p>Requires Permission:
>       * {@link android.Manifest.permission#MODIFY_PHONE_STATE MODIFY_PHONE_STATE} or that the calling
>       * app has carrier privileges (see {@link #hasCarrierPrivileges}).
>       *
>       * @param enable Whether to enable mobile data.
>       *
>       */
>      @SuppressAutoDoc // Blocked by b/72967236 - no support for carrier privileges
>      @RequiresPermission(android.Manifest.permission.MODIFY_PHONE_STATE)
>      public void setDataEnabled(boolean enable) {
>          setDataEnabled(getSubId(SubscriptionManager.getDefaultDataSubscriptionId()), enable);
>      }
>  
>      /**
>       * @hide
>       * @deprecated use {@link #setDataEnabled(boolean)} instead.
>      */
>      @SystemApi
>      @Deprecated
>      @RequiresPermission(android.Manifest.permission.MODIFY_PHONE_STATE)
>      public void setDataEnabled(int subId, boolean enable) {
>          try {
>              Log.d(TAG, "setDataEnabled: enabled=" + enable);
>              // 从 JNI 获取
>              ITelephony telephony = getITelephony();
>              if (telephony != null)
>                  telephony.setUserDataEnabled(subId, enable);
>          } catch (RemoteException e) {
>              Log.e(TAG, "Error calling ITelephony#setUserDataEnabled", e);
>          }
>      }
>  ```
>
>  
>
>- 修改初始化移动网络的代码
>
>  `./vendor/mediatek/proprietary/packages/apps/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java`

- `Preference.java` 中的 notifyChanged(setEnabled) 只会改变设置当中移动网络按钮的开关状态，所以将它的注释取消掉

- 改变移动网络状态的应该是在点击事件当中，~~但是并不是实现的接口，而是 Event 那个东西~~，不是改变 enabled 的事件，是 onClick 事件

- ~~修改 Preference 文件 1145 行~~

  ```
              /*if (preferenceScreen != null && listener != null
                      && listener.onPreferenceTreeClick(preferenceScreen, this)) {
                  return;
              }*/
              if (preferenceScreen != null && listener != null) {
                  return;
              }
  ```



## apk 安装弹窗的问题（已解决）

> 思路：
>
> - 先找到“来历不明的应用很可能会损害您的平板电脑和个人数据...”这一段文字，找到它的 id
>
>   位于 `./vendor/mediatek/proprietary/packages/apps/PackageInstaller` 中`name="anonymous_source_warning"`
>
> - 看哪里用到了文字
>
>   `PackageInstallerActivity` 中的自定义 View `AnonymousSourceDialog`
>
>   `createDialog` 方法通过 ` AnonymousSourceDialog.newInstance()` 创建了自定义 View
>
> - 找到控制 dialog 弹出的文件
>
> - 将弹出 dialog 的代码注释掉，只留有点击确定的代码，默认确定
>
>   在 PackageInstallerActivity 的第 245 行加入如下代码：
>
>   ```java
>   if (id == DLG_ANONYMOUS_SOURCE) {
>       PackageInstallerActivity activity = ((PackageInstallerActivity)
>                                            getActivity());
>       activity.mAllowUnknownSources = true;
>       activity.initiateInstall();
>       return ;
>   }
>   ```
>
>   - 编译后没有效果，改 `./packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java`
>
>     ```java
>     if (id == DLG_ANONYMOUS_SOURCE) {
>         mAllowUnknownSources = true;
>         initiateInstall();
>         return ;
>     }
>     ```
>
>     

- `./frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java` 
- `./vendor/mediatek/proprietary/packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java`



## T22 客制化固件更新

- 默认 apk 文件要放在 `wisky\customer\T22\system\preinstall` 中

- 编译之前先在 framework/base 中 pull 一下



# 1125

## 移动网络图标

> - 似乎找到了一个和问题相关的类
>
>   `vendor/mediatek/proprietary/packages/apps/MtkSettings/src/com/android/settings/network$ vim MobileNetworkPreferenceController.java`
>
>   - 测试发现这个类里边的 `mPreference.setEnabled` 是控制设置中移动网络选项的，当为 false 时，将无法进入移动网络的设置页面
>   - 最后确定了，这里边都是 Preference 控制类，即用来控制设置里边的选项的
>
> - 猜测出问题的地方
>
>   - 设置开机默认为关闭移动网络的地方，设置失效
>
>     失效原因猜测：
>
>     - 获取默认值的时候，没有获取到，而是直接设置的开启，所以改变默认值无法关闭
>
>   - 状态栏下拉菜单的加载有问题，判断移动网络是否开启的代码有误
>
> - 先从 StatusBar 着手，让下拉后的 item 能够正常显示
>
>   - 在文件 `vendor/mediatek/proprietary/packages/apps/SystemUI/res/values-zh-rCN$ vim strings.xml` 下找到 `<string name="accessibility_cell_data" msgid="5326139158682385073">"移动数据"</string>`
>
>   - `StatusBar.java` 中的 `createAndAddWindows();` 方法，创建了下拉窗口
>
>   - 通过跟踪代码，发现最后到了 `vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/SystemUIFactory.java` 的 ` createQSTileHost `  方法，里面 new 了一个 `vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/qs$ vim QSTileHost.java` 类
>
>   - 关键类 `vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/qs/tileimpl$ vim QSFactoryImpl.java`
>
>   - `android.provider.Settings.Secure.QS_TILES` 保存了需要显示的 Tile，路径 `/frameworks/base/core/java/android/provider/Settings.java`
>
>     ```java
>       985     /**
>       986      * Activity Action: Show settings for selecting the network operator.
>       987      * <p>
>       988      * In some cases, a matching Activity may not exist, so ensure you
>       989      * safeguard against this.
>       990      * <p>
>       991      * The subscription ID of the subscription for which available network operators should be
>       992      * displayed may be optionally specified with {@link #EXTRA_SUB_ID}.
>       993      * <p>
>       994      * Input: Nothing.
>       995      * <p>
>       996      * Output: Nothing.
>       997      */
>       998     @SdkConstant(SdkConstantType.ACTIVITY_INTENT_ACTION)
>       999     public static final String ACTION_NETWORK_OPERATOR_SETTINGS =
>      1000             "android.settings.NETWORK_OPERATOR_SETTINGS";
>     
>     ```
>
>     accessibility_quick_settings_data_saver_changed_on
>
>     `vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/qs/tiles$ vim DataSaverTile.java`
>
>     发现这是流量节省按钮
>
>   - `vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/qs/tiles$ vim CellularTile.java` 的 `handleClick` 是移动网络 item 的单击事件
>
>     ```java
>     public CellularTile(QSHost host) {
>         ...
>         // 这个地方默认为 false，改成 true 以后，原来的移动数据变成了流量使用情况
>         mDisplayDataUsage = mQuickSettingsPlugin.customizeDisplayDataUsage(true);
>         ...
>     }
>     ```
>
>     220 行：
>
>     ```java
>     /*boolean mobileDataEnabled = mDataController.isMobileDataSupported()
>                     && mDataController.isMobileDataEnabled();*/
>     boolean mobileDataEnabled = true;
>     ```
>
>     通过测试发现此处是每次改变移动数据都会判断的，包括初始化和单击事件，但是只改变按钮是否亮，不影响移动数据的开启和关闭



## 修改横竖屏桌面壁纸

将壁纸文件放入 `frameworks\base\core\res\res\drawable-sw600dp-nodpi` 文件夹中

修改文件 `frameworks\base\core\res\res\values\symbols.xml ` 1268 行 :

```xml
  ...
  <java-symbol type="drawable" name="default_wallpaper" />
  <java-symbol type="drawable" name="default_wallpaper_verticle" />
```

修改文件 `framework/base/core/java/android/app/WallpaperManager.java` 1860 行：

```java
if (which == FLAG_LOCK) {
    /* Factory-default lock wallpapers are not yet supported
            whichProp = PROP_LOCK_WALLPAPER;
            defaultResId = com.android.internal.R.drawable.default_lock_wallpaper;
            */
    return null;
} else if(context.getResources().getConfiguration().orientation == Configuration.ORIENTATION_PORTRAIT) {
    whichProp = PROP_WALLPAPER;
    defaultResId = com.android.internal.R.drawable.default_wallpaper_portrait;
} else {
    whichProp = PROP_WALLPAPER;
    defaultResId = com.android.internal.R.drawable.default_wallpaper;
}
```

有问题，切换成竖屏以后，原来正常的横屏就会变成竖屏的壁纸

# 1126

## 修改默认壁纸

将代码修改成这样：

```java
if (which == FLAG_LOCK) {
    /* Factory-default lock wallpapers are not yet supported
            whichProp = PROP_LOCK_WALLPAPER;
            defaultResId = com.android.internal.R.drawable.default_lock_wallpaper;
            */
    return null;
} else {
    whichProp = PROP_WALLPAPER;
    if(context.getResources().getConfiguration().orientation == Configuration.ORIENTATION_PORTRAIT) {
        defaultResId = com.android.internal.R.drawable.default_wallpaper_portrait;
    } else {
        defaultResId = com.android.internal.R.drawable.default_wallpaper;
    }
}
```

修改 ` .vendor/mediatek/proprietary/packages/apps/SystemUI/src/com/android/systemui/ImageWallpaper.java `

```java
294
/*if (!redrawNeeded && !mOffsetsChanged) {
                    if (DEBUG) {
                        Log.d(TAG, "Suppressed drawFrame since redraw is not needed "
                                + "and offsets have not changed.");
                    }
                    return;
                }*/
    
    326
    /*if (!redrawNeeded && xPixels == mLastXTranslation && yPixels == mLastYTranslation) {
                    if (DEBUG) {
                        Log.d(TAG, "Suppressed drawFrame since the image has not "
                                + "actually moved an integral number of pixels.");
                    }
                    return;
                }*/
```



## 移动数据图标问题

```java
boolean mobileDataEnabled = mDataController.isMobileDataSupported()
                && mDataController.isMobileDataEnabled();
```

这一段代码是对下拉状态栏移动网络图标的打开判断，所以要先检查是哪个方法出了问题，错误判断了当前移动网络数据。

`isMobileDataSupported()` 正常，所以应该是 `mDataController.isMobileDataEnabled()` 的判断有问题。

找到类 `com.android.settingslib.net.DataUsageController;`。

在 `.vendor/mediatek/proprietary/packages/apps/SettingsLib/src/com/android/settingslib/net/DataUsageController.java` 中。

确实 `mDataController.isMobileDataEnabled()` 为 false。

#### 是由于传给 TelephonyManager 的 subId 有问题，了解 MobileDataPreference.java 的 initialize 传的 subId

MobileNetworkSettings 中 

`final int phoneSubId = mPhone.getSubId();`

`import com.android.internal.telephony.Phone;`



##### 临时决定从状态栏信号图标着手

`vendor/mediate/proprietary/packages/apps/SystemUI/src/com/android/systemui/statusbar/policy/NetworkControllerImpl.java`

`vendor/mediate/proprietary/packages/apps/SystemUI/src/com/android/systemui/statusbar/policy/MobileSignalController.java`

```java
private boolean isDataDisabled() {
    //注释掉了这一行，改成了和下拉状态栏一样的判断
    //boolean isDataDisabled = !mPhone.getDataEnabled(mSubscriptionInfo.getSubscriptionId());
    boolean isDataDisabled = !mPhone.getDataEnabled();
    if (DEBUG) {
        Log.d(mTag, "isDataDisabled = " + isDataDisabled);
    }
    //return true;
    return isDataDisabled;
}
```

176 行：

```java
// Get initial data sim state.
updateDataSim();
mObserver = new ContentObserver(new Handler(receiverLooper)) {
    @Override
    public void onChange(boolean selfChange) {
        updateTelephony();
    }
};
```

注释掉第 669 行：

```java
//return mServiceState.getDataRegState() == ServiceState.STATE_IN_SERVICE;
```

参考文章： http://ask.openluat.com/article/936 



`vendor/mediatek$ vim proprietary/packages/apps/SystemUI/ext/src/com/mediatek/systemui/ext/ISystemUIStatusBarExt.java` 



##### 状态栏下拉的移动按钮始终还是存在问题

MobileNetworkSettings 的 mPhone.getSubId();



```java
 private void updatePhone(int slotId) {
            final SubscriptionInfo sir = mSubscriptionManager
                    .getActiveSubscriptionInfoForSimSlotIndex(slotId);
            if (sir != null) {
                int phoneId = SubscriptionManager.getPhoneId(sir.getSubscriptionId());
                if (SubscriptionManager.isValidPhoneId(phoneId)) {
                    mPhone = PhoneFactory.getPhone(phoneId);
                }
            }
            if (mPhone == null) {
                // Do the best we can
                mPhone = PhoneGlobals.getPhone();
            }
            log("updatePhone:- slotId=" + slotId + " sir=" + sir);

            mImsMgr = ImsManager.getInstance(mPhone.getContext(), mPhone.getPhoneId());
            mTelephonyManager = new TelephonyManager(mPhone.getContext(), mPhone.getSubId());
            if (mImsMgr == null) {
                log("updatePhone :: Could not get ImsManager instance!");
            } else if (DBG) {
                log("updatePhone :: mImsMgr=" + mImsMgr);
            }

            //mPhoneStateListener.updatePhone();
        }
```

`private SubscriptionManager mSubscriptionManager;`

`mSubscriptionManager = SubscriptionManager.from(activity);`



`.framework/base/telephony/java/android/telephony/SubscriptionManager.java` from 方法 传入的是一个 Context，所以可以在 CellularTile 中传入 mContext



```java
private TabHost.OnTabChangeListener mTabListener = new TabHost.OnTabChangeListener() {
    @Override
    public void onTabChanged(String tabId) {
        if (DBG) log("onTabChanged...:");
        // The User has changed tab; update the body.
        updatePhone(convertTabToSlot(Integer.parseInt(tabId)));
        mCurrentTab = Integer.parseInt(tabId);
        updateBody();
        /* updateBody() method updateScreenStatus
                   just when updateBodyAdvancedFields == true
                   so need updateScreenStatus again.
                */
        updateScreenStatus();
    }
};
```

这个 tabId 是什么



## 指示灯状态修改

`frameworks/base$ vim services/core/java/com/android/server/BatteryService.java`

```java
/**
* Synchronize on BatteryService.
*/
public void updateLightsLocked() {
    final int level = mHealthInfo.batteryLevel;
    final int status = mHealthInfo.batteryStatus;
    if (level < mLowBatteryWarningLevel) {
        if (status == BatteryManager.BATTERY_STATUS_CHARGING) {
            // Solid red when battery is charging
            mBatteryLight.setColor(mBatteryLowARGB);
        } else {
            // Flash red when battery is low and not charging
            mBatteryLight.setFlashing(mBatteryLowARGB, Light.LIGHT_FLASH_TIMED,
                                      mBatteryLedOn, mBatteryLedOff);
        }
    } else if (status == BatteryManager.BATTERY_STATUS_CHARGING
               || status == BatteryManager.BATTERY_STATUS_FULL) {
        if (status == BatteryManager.BATTERY_STATUS_FULL || level >= 90) {
            // Solid green when full or charging and nearly full
            mBatteryLight.setColor(mBatteryFullARGB);
        } else {
            // Solid orange when charging and halfway full
            mBatteryLight.setColor(mBatteryMediumARGB);
        }
    } else {
        // No lights if not charging and not low
        mBatteryLight.turnOff();
    }
}
```



DataUsageControler.java

229

` return true;`



##### 文件修改错了，正确文件

`vendor/mediatek/proprietary/packages/apps/SystemUI$ cd src/com/android/systemui/statusbar/policy/BatteryControllerImpl.java`



`writeSysFile("sys/kernel/led_board/led_inod","2");` 蓝色

`writeSysFile("sys/kernel/led_board/led_inod","0");` 橙色

`writeSysFile("sys/kernel/led_board/led_inod","1");` 绿色



## 代码修改记录

CellularTile.java：

```java
// 62 行
import android.telephony.SubscriptionManager;	
import com.android.internal.telephony.Phone;

// 88 行
private Phone mPhone;
private SubscriptionManager mSubscriptionManager;

// 96 行
mSubscriptionManager = SubscriptionManager.from(mContext);
```

TelephonyManager.java:

```java
public boolean isDataEnabled() {
    //return getDataEnabled(getSubId(SubscriptionManager.getDefaultDataSubscriptionId()));
    return getDataEnabled(SubscriptionManager.getDefaultSubscriptionId());
}
```

6429 行：

```java
@SuppressAutoDoc // Blocked by b/72967236 - no support for carrier privileges
@RequiresPermission(android.Manifest.permission.MODIFY_PHONE_STATE)
public void setDataEnabled(boolean enable) {
    //setDataEnabled(getSubId(SubscriptionManager.getDefaultDataSubscriptionId()), enable);
    setDataEnabled(SubscriptionManager.getDefaultSubscriptionId(), enable);
}
```



# 1127

## 现在移动网络已经正常了，但是默认值的设置还是无效，要解决这个问题

启动开机开启 wifi、移动网络的类：

`frameworks/base/services/java/com/android/server/SystemServer.java`

```java
// 901 行
wm = WindowManagerService.main(context, inputManager, mFactoryTestMode != FactoryTest.FACTORY_TEST_LOW_LEVEL, !mFirstBoot, mOnlyCore, new PhoneWindowManager());
```

`./services/core/java/com/android/server/wm/WindowManagerService.java`

1153

`./services/core/java/com/android/server/SystemServiceManager.java`



`.vendor/mediatek/proprietary/packages/apps/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java`



`.frameworks/base/services/core/java/com/android/server/ConnectivityService.java`

```java
// line 989
private void handleMobileDataAlwaysOn() {
    final boolean enable = toBool(Settings.Global.getInt(
        mContext.getContentResolver(), Settings.Global.MOBILE_DATA_ALWAYS_ON, 1));
    final boolean isEnabled = (mNetworkRequests.get(mDefaultMobileDataRequest) != null);
    if (enable == isEnabled) {
        return;  // Nothing to do.
    }

    if (enable) {
        handleRegisterNetworkRequest(new NetworkRequestInfo(
            null, mDefaultMobileDataRequest, new Binder()));
    } else {
        handleReleaseNetworkRequest(mDefaultMobileDataRequest, Process.SYSTEM_UID);
    }
}
```



##### 修改默认值，发现好像是要改 cellular，在 DatabaseHelper 文件中保存一个默认值

`.frameworks/base/core/java/android/provider/Settings.java`

```java
//line 12722
/**
* Whether to enable cellular on boot.
* The value 1 - enable, 0 - disable
* @hide
*/
public static final String ENABLE_CELLULAR_ON_BOOT = "enable_cellular_on_boot";
```
`./vendor/mediatek/proprietary/packages/apps/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java`

模仿其中的 `Global.WIFI_ON`

```java
MOVED_TO_SECURE.add(Secure.WIFI_ON);

/**
* @deprecated Use {@link android.provider.Settings.Global#WIFI_ON} instead
*/
@Deprecated
public static final String WIFI_ON = Global.WIFI_ON;

MOVED_TO_GLOBAL.add(Settings.Global.WIFI_ON);

/**
* @deprecated Use {@link android.provider.Settings.Global#WIFI_ON}
* instead.
*/
@Deprecated
public static final String WIFI_ON = Global.WIFI_ON;

public static final String WIFI_ON = "wifi_on";
```

添加代码：

要把 @hide 去掉

`.frameworks/base/core/java/android/provider/Settings.java`

```java
// 4801
MOVED_TO_GLOBAL.add(Settings.Global.ENABLE_CELLULAR_ON_BOOT);

// 2280
MOVED_TO_SECURE.add(Secure.ENABLE_CELLULAR_ON_BOOT);

// 4602
/**
* @deprecated Use {@link android.provider.Settings.Global#ENABLE_CELLULAR_ON_BOOT} instead
*/
@Deprecated
public static final String ENABLE_CELLULAR_ON_BOOT = Global.ENABLE_CELLULAR_ON_BOOT;

// 6581
/**
* @deprecated Use {@link android.provider.Settings.Global#ENABLE_CELLULAR_ON_BOOT}
* instead.
*/
@Deprecated
public static final String ENABLE_CELLULAR_ON_BOOT = ENABLE_CELLULAR_ON_BOOT;
```

`./vendor/mediatek/proprietary/packages/apps/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java`

```java
// 2552
loadSetting(stmt, Settings.Global.ENABLE_CELLULAR_ON_BOOT, 0);
```

# 1128



## 横竖屏壁纸

##### 关键类和方法：

`frameworks/base/core/java/android/app/WallpaperManager.java`

```java
public Bitmap peekWallpaperBitmap(Context context, boolean returnDefault,
                                  @SetWallpaperFlags int which, int userId, boolean hardware) {
    if (mService != null) {
        try {
            if (!mService.isWallpaperSupported(context.getOpPackageName())) {
                return null;
            }
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
    /*synchronized (this) {
        if (mCachedWallpaper != null && mCachedWallpaperUserId == userId
            && !mCachedWallpaper.isRecycled()) {
            return mCachedWallpaper;
        }
        mCachedWallpaper = null;
        mCachedWallpaperUserId = 0;
        try {
            mCachedWallpaper = getCurrentWallpaperLocked(context, userId, hardware);
            mCachedWallpaperUserId = userId;
        } catch (OutOfMemoryError e) {
            Log.w(TAG, "Out of memory loading the current wallpaper: " + e);
        } catch (SecurityException e) {
            if (context.getApplicationInfo().targetSdkVersion < Build.VERSION_CODES.O_MR1) {
                Log.w(TAG, "No permission to access wallpaper, suppressing"
                      + " exception to avoid crashing legacy app.");
            } else {
                // Post-O apps really most sincerely need the permission.
                throw e;
            }
        }
        if (mCachedWallpaper != null) {
            return mCachedWallpaper;
        }
    }*/
    if (returnDefault) {
        Bitmap defaultWallpaper = mDefaultWallpaper;
        /*if (defaultWallpaper == null) {
            defaultWallpaper = getDefaultWallpaper(context, which);
            synchronized (this) {
                mDefaultWallpaper = defaultWallpaper;
            }
        }*/
        if (true) {
            defaultWallpaper = getDefaultWallpaper(context, which);
            synchronized (this) {
                mDefaultWallpaper = defaultWallpaper;
            }
        }
        return defaultWallpaper;
    }
    return null;
}
```

最后发现只要改一个延迟时间的 final 值就行了，配合上面的代码就能防止旋转屏幕图片被放大：

ImageWallpaper.java：

`private static final long DELAY_FORGET_WALLPAPER = 0;`

`mSurfaceRedrawNeeded = true;`