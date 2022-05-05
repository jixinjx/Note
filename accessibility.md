# accessibility

创建一个继承系统AccessibilityService

```
package com.example.android.apis.accessibility;

import android.accessibilityservice.AccessibilityService;
import android.view.accessibility.AccessibilityEvent;

public class MyAccessibilityService extends AccessibilityService {
...
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
    }

    @Override
    public void onInterrupt() {
    }

...
}
```

其中有两个必须实现的方法：onAccessibilityEvent和onInterrupt。

在AndroidManifest中添加声明

```
  <application>
    <service android:name=".MyAccessibilityService"
        android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE"
        android:label="@string/accessibility_service_label">
      <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
      </intent-filter>
       <meta-data
    android:name="android.accessibilityservice"
    android:resource="@xml/accessibility_service_config" />
    </service>
  </application>
```

meta可以配置设置

此元数据元素引用您在应用的资源目录中创建的一个 XML 文件 (`<project_dir>/res/xml/accessibility_service_config.xml`)。以下代码展示了该服务配置文件的示例内容：

1. 配置你需要监听的事件类型、要监听哪个程序，最小监听间隔等属性。这里有两种方式可以进行配置，一种是在manifest中通过meta-data配置，一种是在代码中通过setServiceInfo(AccessibilityServiceInfo)设置。
    **方式一：通过meta-data设置**

```xml
        <service
            android:name=".AliAccessibilityService"
            android:label="支付测试"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>

            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessible_service_config_ali" />
        </service>
```

accessible_service_config_ali.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowStateChanged|typeWindowContentChanged|typeNotificationStateChanged"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:accessibilityFlags="flagReportViewIds"
    android:canRetrieveWindowContent="true"
    android:description="@string/pay_test"
    android:notificationTimeout="10"
    android:canPerformGestures="true"
    android:packageNames="com.eg.android.AlipayGphone" />
```

**方式二：在代码中通过setServiceInfo设置**



```kotlin
AccessibilityServiceInfo serviceInfo = new AccessibilityServiceInfo();
        serviceInfo.eventTypes = AccessibilityEvent.TYPES_ALL_MASK;
        serviceInfo.feedbackType = AccessibilityServiceInfo.FEEDBACK_GENERIC;
        serviceInfo.packageNames = new String[]{"com.vivo.weather"};
        serviceInfo.notificationTimeout=100;
        setServiceInfo(serviceInfo);
```

这里我建议使用meta-data的方式进行配置，因为我实践过程中发现有的属性不能通过代码配置。

此时的service应该为

```
package com.example.accessibility;

import android.accessibilityservice.AccessibilityService;
import android.accessibilityservice.AccessibilityServiceInfo;
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.util.Log;
import android.view.accessibility.AccessibilityEvent;

public class MyAccessibility extends AccessibilityService {
    private static final String TAG = "xys";
    public MyAccessibility() {
    }

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {

        AccessibilityServiceInfo serviceInfo = new AccessibilityServiceInfo();
        serviceInfo.eventTypes = AccessibilityEvent.TYPES_ALL_MASK;
        serviceInfo.feedbackType = AccessibilityServiceInfo.FEEDBACK_GENERIC;
        serviceInfo.packageNames = new String[]{"com.vivo.weather"};
        serviceInfo.notificationTimeout=100;
        setServiceInfo(serviceInfo);


        Log.d(TAG, "onAccessibilityEvent: " + event.toString());
        int eventType = event.getEventType();
        switch (eventType) {
            case AccessibilityEvent.TYPE_VIEW_CLICKED:
                //界面点击
                Log.d("TAG", "onAccessibilityEvent: 22");
                break;
            case AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED:
                //界面文字改动
                break;
        }
    }

    @Override
    public void onInterrupt() {

    }


}
```

需要设无障碍里面设置开启状态，然后点击以后就可以看到log