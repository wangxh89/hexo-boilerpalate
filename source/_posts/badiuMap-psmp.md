title: 百度地图Android sdk 在PSMP中的应用
date: 2014-3-3 15:39:31
tags: [百度地图,Android]
categories: Android
---
*作者： 王小虎*
*日期：2014-3-3*

### 问题

Q：如何获取百度地图上的经纬度

- 在百度地图上点击右键 “添加新地点”
- 在打开的链接中有 lat=32.030257410591055&lng=118.78681159690731

### 在App中添加百度地图

#### 创建应用 获取密钥

  - 进入百度地图开发台 获取密钥
  - 创建应用获取密钥  安全码为：数字签名（SHA1）+;+包名
  - 需要创建两个密钥一个是debug的数字签名  一个是release的数字签名
  - 在相关下载中 下载开发包 及示例参考

#### 在App 中加入百度地图

  -  将下载开发包中的BaiduLBS_Android.jar 及armeabi目录 拷贝到工程libs
  -  在application中添加开发密钥
```xml
		<application>
		    <meta-data
		        android:name="com.baidu.lbsapi.API_KEY"
		        android:value="开发者 key" />
		</application>
```
  -  添加所需权限
```xml
	    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
	    <uses-permission android:name="android.permission.USE_CREDENTIALS" />
	    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS" />
	    <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS" />
	    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	    <uses-permission android:name="android.permission.INTERNET" />
	    <uses-permission android:name="com.android.launcher.permission.READ_SETTINGS" />
	    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
	    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
	    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
	    <uses-permission android:name="android.permission.BROADCAST_STICKY" />
	    <uses-permission android:name="android.permission.WRITE_SETTINGS" />
	    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
```
- 在布局xml文件中添加地图控件
```xml
	        <com.baidu.mapapi.map.MapView
            android:id="@+id/bmapView"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:clickable="true" />
```
- 在应用程序创建时初始化 SDK  创建地图Activity，管理地图生命周期
```java
          //在使用SDK各组件之前初始化context信息，传入ApplicationContext
        //注意该方法要再setContentView方法之前实现
        SDKInitializer.initialize(getApplicationContext());
             //获取地图控件引用
        mMapView = (MapView) findViewById(R.id.bmapView);
	    @Override
	    protected void onDestroy() {
	        super.onDestroy();
	        //在activity执行onDestroy时执行mMapView.onDestroy()，实现地图生命周期管理
	        mMapView.onDestroy();
	    }
	    @Override
	    protected void onResume() {
	        super.onResume();
	        //在activity执行onResume时执行mMapView. onResume ()，实现地图生命周期管理
	        mMapView.onResume();
	        }
	    @Override
	    protected void onPause() {
	        super.onPause();
	        //在activity执行onPause时执行mMapView. onPause ()，实现地图生命周期管理
	        mMapView.onPause();
	        }
	    }

```
### 设置地图放大系数
```java
		//获取地图控件引用
        mMapView = (MapView) view.findViewById(R.id.bmapView);
        //放大
        mBaiduMap = mMapView.getMap();
        MapStatusUpdate msu = MapStatusUpdateFactory.zoomTo(15.0f);
        mBaiduMap.setMapStatus(msu);
```
### 设置地图类型
```java
	mBaiduMap.setMapType(BaiduMap.MAP_TYPE_NORMAL);
	mBaiduMap.setMapType(BaiduMap.MAP_TYPE_SATELLITE);//卫星地图
	mBaiduMap.setTrafficEnabled(true);  //实时交通
```
