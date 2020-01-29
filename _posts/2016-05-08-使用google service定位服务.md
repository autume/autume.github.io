---
layout: post
title: 使用google service定位服务
categories: Android
keywords: Android, GPS, google service
---
使用google service的API实现定位功能，封装成一个小模块。需要安装google service相关的sdk包，手机需要装有google服务框架等。主要是国外产品使用，国内需要翻墙，这里做下记录。

导入：
```java
    compile 'com.google.android.gms:play-services-location:9.4.0'
```
## 使用
初始化模块，启动定时上报位置信息
```java
 public void initLocationModule() {
        locationModule = new GoogleLocationModule(mActivity);
        locationModule.LocationInit(context);
        locationModule.googleClientConnect();
        locationModule.setLocationChangedListener(this);
        locationModule.startLocationUpdates();
    }
```
退出时释放模块资源
```java
 @Override
    public void releaseLocationModule() {
        locationModule.stopLocationUpdates();
        locationModule.googleClientDisconnect();
    }
```
监听模块上报数据
```java
 @Override
    public void onLocationChanged(Location location) {
        T.showShort(context, "mCurrentLocation: " + location.getLatitude());
    }
```
## 实现
### 模块接口
```java
public interface LocationModule {
    void LocationInit(Context context);
    void startLocationUpdates();
    void stopLocationUpdates();
    void googleClientConnect();
    void googleClientDisconnect();
    void setLocationChangedListener(LocationChangedListener locationChangedListener);
}
```
### 模块实现
```java
public class GoogleLocationModule implements
        LocationModule,
        GoogleApiClient.ConnectionCallbacks,
        GoogleApiClient.OnConnectionFailedListener,
        LocationListener,
        ResultCallback<LocationSettingsResult> {
    private MyLog myLog = new MyLog("[GoogleLocationModule] ");
    private Context context;
    private Activity mActivity;
    protected static final int REQUEST_CHECK_SETTINGS = 0x1;
    public static final long UPDATE_INTERVAL_IN_MILLISECONDS = 5000;
    public static final long FASTEST_UPDATE_INTERVAL_IN_MILLISECONDS = UPDATE_INTERVAL_IN_MILLISECONDS / 2;

    public static GoogleApiClient mGoogleApiClient;
    protected LocationRequest mLocationRequest;
    public static LocationSettingsRequest mLocationSettingsRequest;
    protected Location mCurrentLocation;
    protected String mLastUpdateTime;
    protected Boolean mRequestingLocationUpdates = false;
    protected Boolean mStartLocationUpdates = false;
    private LocationChangedListener locationChangedListener;

    public GoogleLocationModule(Activity activity) {
        this.mActivity = activity;
    }

    @Override
    public void LocationInit(Context context) {
        this.context = context;
        buildGoogleApiClient();
        createLocationRequest();
        buildLocationSettingsRequest();
        checkLocationSettings();  //检查并请求打开位置权限
    }

    @Override
    public void startLocationUpdates() {
        if (!mGoogleApiClient.isConnected()){//未连接，设置标志位，连接时启动
            mRequestingLocationUpdates = true;
        }else{                              //已连接，判断是否已启动
            if (mStartLocationUpdates)
                return;
            if (ActivityCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED
                    && ActivityCompat.checkSelfPermission(context, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                return;
            }
            LocationServices.FusedLocationApi.requestLocationUpdates(
                    mGoogleApiClient,
                    mLocationRequest,
                    this
            );
            mStartLocationUpdates = true;
        }
    }

    @Override
    public void stopLocationUpdates() {
        LocationServices.FusedLocationApi.removeLocationUpdates(
                mGoogleApiClient,
                this
        );
        mRequestingLocationUpdates = false;
        mStartLocationUpdates = false;
    }

    @Override
    public void googleClientConnect() {
        if (!mGoogleApiClient.isConnected())
            mGoogleApiClient.connect();
    }

    @Override
    public void googleClientDisconnect() {
        mGoogleApiClient.disconnect();
    }

    @Override
    public void setLocationChangedListener(LocationChangedListener locationChangedListener) {
        this.locationChangedListener = locationChangedListener;
    }

    protected void checkLocationSettings() {
        PendingResult<LocationSettingsResult> result =
                LocationServices.SettingsApi.checkLocationSettings(
                        mGoogleApiClient,
                        mLocationSettingsRequest
                );
        result.setResultCallback(this);
    }

    protected synchronized void buildGoogleApiClient() {
        myLog.d("Building GoogleApiClient");
        mGoogleApiClient = new GoogleApiClient.Builder(context)
                .addConnectionCallbacks(this)
                .addOnConnectionFailedListener(this)
                .addApi(LocationServices.API)
                .build();
    }

    protected void createLocationRequest() {
        mLocationRequest = new LocationRequest();
        mLocationRequest.setInterval(UPDATE_INTERVAL_IN_MILLISECONDS);
        mLocationRequest.setFastestInterval(FASTEST_UPDATE_INTERVAL_IN_MILLISECONDS);
        mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
    }

    protected void buildLocationSettingsRequest() {
        LocationSettingsRequest.Builder builder = new LocationSettingsRequest.Builder();
        builder.addLocationRequest(mLocationRequest);
        mLocationSettingsRequest = builder.build();
    }

    @Override
    public void onLocationChanged(Location location) {
        mCurrentLocation = location;
        mLastUpdateTime = DateFormat.getTimeInstance().format(new Date());
        locationChangedListener.onLocationChanged(location);
    }


    @Override
    public void onConnected(Bundle bundle) {
        myLog.d("GoogleApiClient onConnected");
        if (mRequestingLocationUpdates)
            startLocationUpdates();
        if (mCurrentLocation == null) {
            if (ActivityCompat.checkSelfPermission(context, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED
                    && ActivityCompat.checkSelfPermission(context, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                return;
            }
            mCurrentLocation = LocationServices.FusedLocationApi.getLastLocation(mGoogleApiClient);
            mLastUpdateTime = DateFormat.getTimeInstance().format(new Date());
            if (mCurrentLocation != null) {
                T.showShort(context, "getLastLocation: " + mCurrentLocation.getLatitude());
                myLog.d("mCurrentLocation: " + mCurrentLocation.getAltitude());
            } else {
                myLog.d("mCurrentLocation null");
            }
        }
    }

    @Override
    public void onConnectionSuspended(int i) {
        myLog.d("GoogleApiClient onConnectionSuspended");
    }

    @Override
    public void onConnectionFailed(ConnectionResult connectionResult) {
        myLog.e("GoogleApiClient onConnectionFailed: " + connectionResult.getErrorMessage());
    }
    
    //位置权限请求返回结果
    @Override
    public void onResult(LocationSettingsResult locationSettingsResult) {
        final Status status = locationSettingsResult.getStatus();
        switch (status.getStatusCode()) {
            case LocationSettingsStatusCodes.SUCCESS:
                myLog.d("All location settings are satisfied.");
                break;
            case LocationSettingsStatusCodes.RESOLUTION_REQUIRED:
                myLog.d("Location settings are not satisfied. Show the user a dialog to" +
                        "upgrade location settings ");
                try {
                    status.startResolutionForResult(mActivity, REQUEST_CHECK_SETTINGS);
                } catch (IntentSender.SendIntentException e) {
                    myLog.w("PendingIntent unable to execute request.");
                }
                break;
            case LocationSettingsStatusCodes.SETTINGS_CHANGE_UNAVAILABLE:
                T.showShort(context, context.getString(R.string.map_open_gps_remind));
                myLog.e("Location settings are inadequate, and cannot be fixed here. Dialog not created.");
                break;
        }
    }
}
```
