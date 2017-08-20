# Android/IOS 设备指纹

>内容非原创，只是引用和总结了一下

##Android 设备指纹
常见的就5种方式
* 网卡mac地址
* IMEI
* AndroidID
* 序列号
* 蓝牙mac地址

检测代码如下
`java
//来自简书上的一篇文章
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.i("ProjectN", "IMEI："+ getLocalIMEI(this));
        Log.i("ProjectN", "Serial："+getSerial());
        Log.i("ProjectN", "Mac："+getMac(this));
        Log.i("ProjectN", "AndroidId："+getAndroidId(this));
        Log.i("ProjectN", "bluetooth："+getBluetooth());
    }
    /**
     *序列号
     *从Android 2.3 (“Gingerbread”)开始可用，可以通过android.os.Build.SERIAL获取，对于没有通话功能的设备，它会
     *返回一个唯一的device ID
     * @return
     */
    public String getSerial()
    {
        try
        {
            String str = android.os.Build.class.getField("SERIAL").get(null).toString();
            return str;
        } catch (IllegalAccessException | IllegalArgumentException
                | NoSuchFieldException e)
        {
            e.printStackTrace();
        }
        return null;
    }

    /**
     *
     * 获取设备的IMEI
     * IMEI
     *方式：TelephonyManager.getDeviceId():
     *问题
     *范围：网上说“只能支持拥有通话功能的设备，对于平板不可以”，但是我测试了型号FDR-A01w平板确实拿到的是null,
     *而 型号S7-601的平板却能拿到。
     *持久性：返厂，数据擦除的时候不彻底，保留了原来的标识。
     *权限：需要权限：android.permission.READ_PHONE_STATE
     *bug: 有些厂家的实现有bug，返回一些不可用的数据
     * @return
     */
    public String getLocalIMEI(Context context)
    {
        TelephonyManager tm = null;
        try
        {
            tm = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
            if (null != tm)
            {
                return tm.getDeviceId();
            }
        } catch (Exception ex)
        {
        } finally
        {
            tm = null;
        }
        return null;
    }

    /**
     * Mac地址
     *ACCESS_WIFI_STATE权限
     *有些设备没有WiFi，或者蓝牙，就不可以，如果WiFi没有打开，硬件也不会返回Mac地址
     *6.0以前可用
     * @return
     */
    public String getMac(Context context)
    {
        WifiManager wifi = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
        WifiInfo info = wifi.getConnectionInfo();
        return info.getMacAddress();
    }

    //兼容性强
    //需要权限
    //<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />  
    //<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" /> 
    public static String getMac() {
        String str = "";
        String macSerial = "";
        try {
            Process pp = Runtime.getRuntime().exec(
                    "cat /sys/class/net/wlan0/address ");
            InputStreamReader ir = new InputStreamReader(pp.getInputStream());
            LineNumberReader input = new LineNumberReader(ir);

            for (; null != str;) {
                str = input.readLine();
                if (str != null) {
                    macSerial = str.trim();// 去空格
                    break;
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        if (macSerial == null || "".equals(macSerial)) {
            try {
                return loadFileAsString("/sys/class/net/eth0/address")
                        .toUpperCase().substring(0, 17);
            } catch (Exception e) {
                e.printStackTrace();

            }

        }
        return macSerial;
    }

    public static String loadFileAsString(String fileName) throws Exception {
        FileReader reader = new FileReader(fileName);
        String text = loadReaderAsString(reader);
        reader.close();
        return text;
    }

    public static String loadReaderAsString(Reader reader) throws Exception {
        StringBuilder builder = new StringBuilder();
        char[] buffer = new char[4096];
        int readLength = reader.read(buffer);
        while (readLength >= 0) {
            builder.append(buffer, 0, readLength);
            readLength = reader.read(buffer);
        }
        return builder.toString();
    }

    // Android Id
    /**
     *    ANDROID_ID
     *2.2（Froyo，8）版本系统会不可信，来自主要生产厂商的主流手机，至少有一个普遍发现的bug，这些有问题的手机相同的ANDROID_ID: 9774d56d682e549c
     *但是如果返厂的手机，或者被root的手机，可能会变
     * @param context
     * @return
     */
    private static String getAndroidId(Context context) {
        String androidId = Settings.Secure.getString(
                context.getContentResolver(), Settings.Secure.ANDROID_ID);
        return androidId;
    }
    // bluetooth mac id 
    // 需要Bluetooth 的权限
    private  static String getBluetooth()
    {
        //BluetoothAdapter m = null ;
        //m = BluetoothAdapter.getDefaultAdapter() ;
        //return m.getAddress() ;
         return Settings.Secure.getString(context.getContentResolver(), "bluetooth_address");
    }

}

`
我自己测试的时候发现bluetooth和网卡mac地址是一样的。本着研究的心态，我去看了源码
`java
// /frameworks/base/services/core/java/com/android/server/BluetoothManagerService.java
//  要自己看的话最好了解一下binder.
public String getAddress() {
1043        mContext.enforceCallingOrSelfPermission(BLUETOOTH_PERM,
1044                "Need BLUETOOTH permission");
1045
1046        if ((Binder.getCallingUid() != Process.SYSTEM_UID) &&
1047                (!checkIfCallerIsForegroundUser())) {
1048            Slog.w(TAG,"getAddress(): not allowed for non-active and non system user");
1049            return null;
1050        }
1051
1052        if (mContext.checkCallingOrSelfPermission(Manifest.permission.LOCAL_MAC_ADDRESS)
1053                != PackageManager.PERMISSION_GRANTED) {
1054            return BluetoothAdapter.DEFAULT_MAC_ADDRESS;
1055        }
1056
1057        try {
1058            mBluetoothLock.readLock().lock();
1059            if (mBluetooth != null) return mBluetooth.getAddress();
1060        } catch (RemoteException e) {
1061            Slog.e(TAG, "getAddress(): Unable to retrieve address remotely. Returning cached address", e);
1062        } finally {
1063            mBluetoothLock.readLock().unlock();
1064        }
1065
1066        // mAddress is accessed from outside.
1067        // It is alright without a lock. Here, bluetooth is off, no other thread is
1068        // changing mAddress
1069        return mAddress;
1070    }
 public static final String DEFAULT_MAC_ADDRESS = "02:00:00:00:00:00";
`

可以看到前一个if检查是不是system用户，或者是不是在前台，然后检查权限LOCAL_MAC_ADDRESS,这个权限貌似app申请不到，所以返回了默认地址。
从stackoverflow上找到一个回答
Access to the mac address has been deliberately removed:
To provide users with greater data protection, starting in this release, Android removes programmatic access to the device’s local hardware identifier for apps using the Wi-Fi and Bluetooth APIs.
也就是Android6.0以后不能直接访问mac地址了。但是可以通过别的方式访问.
另外，在手机root的情况下，这些数据就可能很不可靠了，比如常见的一些hook框架就可以对关键的api进行hook，返回错误结果。

##IOS

##如果有新的姿势会继续更新

