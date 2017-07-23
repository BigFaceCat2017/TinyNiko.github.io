# 虚拟定位
## 前言
近年来，越来越多的手机应用通过定位系统来实现强大功能，比如钉钉的打卡，微信的实时位置共享，咕咚的路径记录。这些应用给大众带来了便利，但也让一部分人看到了商机，通过虚拟定位技术，使用者可以出现在世界的任何一个角落，并且不会被应用检查出问题。这样，使用者就可以进行远程打卡，秀定位了。

## 技术原理
### 手机定位的几种方式

1. 卫星定位(GPS)
2. 基站定位
3. wifi/移动网络/蓝牙定位
4. AGPS 定位
5. 混合式定位(综合以上几种方式)

手机上的应用一般可以直接使用系统提供的接口获取地理位置，从而不需要了解具体使用了哪种定位方式。手机系统则会使用当前最佳的定位方式进行定位。对于一些精确度要求高的应用,比如xx地图,微信等，会采用混合式定位来获取地理位置。针对这些定位方式,我们可以从软件和硬件层面对地理位置进行伪造。本文将从软件层面对虚拟定位进行分析。

### Android修改定位的几种方式
#### 1.使用系统的模拟位置
打开手机的设置，进入开发者选项,在允许模拟位置上打勾就可以。Android 6.0以后选择模拟位置信息应用。然后开发者使用下面一段就可以修改定位了
```java
lm = (LocationManager) getSystemService(Context.LOCATION_SERVICE) ;
lm.addTestProvider("gps",false,false,false,false,true,true,true,0,5);
lm.setTestProviderEnabled("gps",true);
Location lc = new Location("gps");
// 设置经纬度
lc.setLatitude(111.111);
lc.setAltitude(19.111);
lm.setTestProviderLocation("gps",lc);
```

这种方式比较简单，不需要root权限，但是需要打开开发者选项里的模拟定位,也容易被检测到。典型app: 神出鬼没

#### 2.hook系统函数
应用通常会调用系统提供的LocationManager类去查询地理位置，这种方式最直接。如果我们将这个类hook住,然后返回我们伪造的地理位置,就可以实现虚拟定位。但是地图类应用通常会采用混合式获取地址，也就是说，如果要让地图类应用也获取到假的的位置，我们还需要同时对其他几种获取位置的方式进行hook。
举个例子：
```java
HookUtils.hookMethod(v1, "()Landroid/location/Location;", a.a(), a.class);
HookUtils.hookMethod(v2.invoke(v5.getSystemService("phone"), v3).getClass().getDeclaredMethod("getCellLocation", v3), "()Landroid/os/Bundle;", a.a(), a.class);
HookUtils.hookMethod(Class.forName("android.telephony.gsm.GsmCellLocation").getDeclaredMethod("getLac", v3), "()I", a.a(), a.class);	

```
这几个函数都和获取地理位置有一定的关系，如果这几个函数返回的信息都是伪造的，那么地图获取到的信息也会是假的。这里只列出了一部分函数，实际还有很多函数需要进行hook。

这种方式通常需要用到hook技术或者是hook框架(Xposed),需要root,功能强大并且很难被发现,典型的app如神行者和anywhere.

#### 3.修改APP

通过逆向找到处理位置信息的代码，手动进行修改。这种方式一般没人用，耗时耗力，但是效果好。

### Android虚拟定位检测

1. 第一种的检测相对简单,通过看模拟定位开关是否开启来判断
```java
Settings.Secure.getString(context.getContentResolver(), Settings.Secure.ALLOW_MOCK_LOCATION).equals("0")
//6.0 以上可以检测addTestProvider返回是否成功

```
2. root之后可以去检测hook框架是否存在，插件是否存在，存在就有一定的风险

3. 对app进行完整性校验，加强保护措施。

### IOS修改定位的几种方式
#### 1.针对模拟器
可以通过使用Xcode来模拟定位。在Xcode中有一个选项可以设置是否使用模拟定位，但是这个方法只针对模拟器，对真机没有用
#### 2.hook系统函数
和Android一样，IOS也提供了一个用于获取地理位置的函数，我们可以通过hook这些函数来伪造地理位置。通过对Cydia上的一些虚拟定位应用进行分析，发现大致分为两类。第一类应用和Android一样，直接hook了系统中的多个跟获取位置相关的函数，比如
```c
    //位置随心行
  v0 = objc_getClass("CLLocation");
  MSHookMessageEx(v0, "coordinate", sub_5C3E, &dword_828C);
  v1 = objc_getClass("CLLocationManager");
  MSHookMessageEx(v1, "startUpdatingLocation", sub_5CAC, &dword_8290);
  v2 = objc_getClass("CLPlacemark");
  MSHookMessageEx(v2, "addressDictionary", sub_5DDA, &dword_8294);
  MSHookMessageEx(v2, "name", sub_66B0, &dword_8298);
  MSHookMessageEx(v2, "thoroughfare", sub_67C6, &dword_829C);
  MSHookMessageEx(v2, "subThoroughfare", sub_68DC, &dword_82A0);
  MSHookMessageEx(v2, "locality", sub_69D2, &dword_82A4);
  MSHookMessageEx(v2, "subLocality", sub_6AC8, &dword_82A8);
  MSHookMessageEx(v2, "administrativeArea", sub_6BBE, &dword_82AC);
  MSHookMessageEx(v2, "subAdministrativeArea", sub_6CB4, &dword_82B0);
  MSHookMessageEx(v2, "postalCode", sub_6DAA, &dword_82B4);
  MSHookMessageEx(v2, "ISOcountryCode", sub_6E90, &dword_82B8);
  MSHookMessageEx(v2, "country", sub_6F76, &dword_82BC);
  MSHookMessageEx(v2, "inlandWater", sub_706C, &dword_82C0);
  MSHookMessageEx(v2, "ocean", sub_7162, &dword_82C4);
  
  //神行者
  v4 = objc_getClass("CLLocation");
  MSHookMessageEx(v4, "coordinate", sub_14A40, &dword_8547C);
  MSHookMessageEx(v4, "initWithLatitude:longitude:", sub_15630, &dword_85480);
  MSHookMessageEx(v4,"initWithCoordinate:altitude:horizontalAccuracy:verticalAccuracy:timestamp:",sub_16178,&dword_85484);
  MSHookMessageEx(v4,"initWithCoordinate:altitude:horizontalAccuracy:verticalAccuracy:course:speed:timestamp:",sub_16DF0,&dword_85488);
  v5 = objc_getClass("CLLocationManager");
  MSHookMessageEx(v5, "startUpdatingLocation", sub_17AC0, &dword_8548C);
  MSHookMessageEx(v5, "setDelegate:", sub_1B9A8, &dword_85490);MSHookMessageEx(v5, "stopUpdatingLocation", sub_1DB98, &dword_85494);

```
观察两个app的实现，可以发现神行者app会对一些私有函数进行hook，这么做更加靠谱。 使用这一类技术的app有，神行者,anywhere,位置随心行。

第二类应用直接将获取位置信息的代理函数设置到自己的LocationManager中，这样，只要实现自己的代理方法就可以了。在我分析的几个app中，发现它们用了相同的代码，可能是同一个作者。使用这类技术的app有，fakegps,筋斗云，360虚拟定位等。 


#### 3.修改APP
通过直接修改APP的代码来实现虚拟定位，这种方式耗时耗力，一般没有人会做。

### IOS虚拟定位检测
针对hook这种方法，可以检测函数的地址是否在模块内进行判断，如果函数的地址出现在了别的模块，那么很可能这个函数就被hook了。


## 虚拟定位在实际生活中的运用
1. 最早接触虚拟定位是因为PokemonGO，当时大陆锁区，无法进行游戏，所以很多人购买了虚拟定位，这样就可以跑到开放区域抓捕精灵，同时，虚拟定位也提供了扫街(自由走动)功能，可以让玩家在地图中自由行走。这样可以让玩家快速的获取补给品和抓取精灵，但这已经违反了游戏的公平性。 
2. 钉钉打卡。钉钉有了企业打卡功能后，只要人出现在公司就可以进行打卡签到，但是通过虚拟定位就可以在家进行打卡，再也不用担心会迟到了。
3. 发微博和朋友圈的时候，可以现实地理位置，这个时候就可以假装出现在某地，然后秀一把优越。


## 结语

目前实现虚拟定位的方法大都比较统一，需要手机的最高权限。虚拟定位的出现，无疑让使用者有了瞬间移动的技能。

