---
title: 记一次使用Realm的体会
date: 2017-02-16 20:42:40
tags: Realm
---
>数据库Realm，是用来替代sqlite的一种解决方案，它有一套自己的数据库存储引擎，比sqlite更轻量级，拥有更快的速度，并且具有很多现代数据库的特性，比如支持JSON，流式api，数据变更通知，自动数据同步,简单身份验证，访问控制，事件处理，最重要的是跨平台，目前已有Java，Objective C，Swift，React-Native，Xamarin这五种实现。
官网:https://realm.io/docs/java/latest

一.基于Version-0.87.5
由于 v0.87.5之后的版本使用的是plugin形式添加引用,
使用时会发现包增大好多(可能是自己使用时缺少了某些配置),
所以这里介绍一个该版本的使用.
1.引入
app下的build.gradle
```
compile 'io.realm:realm-android:0.87.5'
```
2.初始化
Application中
```
  private void initRealm() {
        RealmConfiguration config = new RealmConfiguration.Builder(this).name("guess.realm").schemaVersion(0).build();
        Realm.setDefaultConfiguration(config);
  }
```
3.使用
新建对象
```
public class PskBean extends RealmObject {
    @PrimaryKey
    private String ssid;
    private String bssid;
    private String psk;

    public PskBean() {
    }

    public PskBean(String ssid, String bssid, String psk) {
        this.ssid = ssid;
        this.bssid = bssid;
        this.psk = psk;
    }

    public String getSsid() {
        return ssid;
    }

    public void setSsid(String ssid) {
        this.ssid = ssid;
    }

    public String getBssid() {
        return bssid;
    }

    public void setBssid(String bssid) {
        this.bssid = bssid;
    }

    public String getPsk() {
        return psk;
    }

    public void setPsk(String psk) {
        this.psk = psk;
    }
}
```
数据库帮助类(只示例异步操作)
```
public class RealmDBHelper {
  private Realm mRealm;

  public RealmDBHelper(Context context) {
      mRealm = Realm.getInstance(context);
  }

  public Realm getRealm() {
      return mRealm;
  }

  /**
   * add （增）
   */
  public RealmAsyncTask addBean(final PskBean bean, final RealmCallback callback) {
      RealmAsyncTask addTask = mRealm.executeTransaction(new Realm.Transaction() {
          @Override
          public void execute(Realm realm) {
              realm.copyToRealmOrUpdate(bean);
          }
      }, new Realm.Transaction.Callback() {
          @Override
          public void onSuccess() {
              super.onSuccess();
              callback.success(null);
          }

          @Override
          public void onError(Exception e) {
              super.onError(e);
              callback.failure(e.getMessage());
          }
      });
      return addTask;
  }

  /**
   * delete （删）
   */
  public RealmAsyncTask deleteBean(final String ssid, final String bssid, final RealmCallback callback) {
      RealmAsyncTask deleteTask = mRealm.executeTransaction(new Realm.Transaction() {
          @Override
          public void execute(Realm realm) {
              PskBean bean = realm.where(PskBean.class).equalTo("ssid", ssid).equalTo("bssid", bssid).findFirst();
              bean.removeFromRealm();
          }
      }, new Realm.Transaction.Callback() {
          @Override
          public void onSuccess() {
              super.onSuccess();
              callback.success(null);
          }

          @Override
          public void onError(Exception e) {
              super.onError(e);
              callback.failure(e.getMessage());
          }
      });
      return deleteTask;
  }

  /**
   * update （改）
   */
  public RealmAsyncTask updateBean(final String ssid, final String bssid, final String psk, final RealmCallback callback) {
      RealmAsyncTask updateTask = mRealm.executeTransaction(new Realm.Transaction() {
          @Override
          public void execute(Realm realm) {
              PskBean bean = realm.where(PskBean.class).equalTo("ssid", ssid).equalTo("bssid", bssid).findFirst();
              bean.setPsk(psk);
          }
      },new Realm.Transaction.Callback(){
          @Override
          public void onSuccess() {
              super.onSuccess();
              callback.success(null);
          }

          @Override
          public void onError(Exception e) {
              super.onError(e);
              callback.failure(e.getMessage());
          }
      });
      return updateTask;
  }

  /**
   * query （查询所有）
   */
  private void queryAllBean(RealmCallback callback) {
      RealmResults<PskBean> beans = mRealm.where(PskBean.class).findAllAsync();
      callback.success(beans.load() ? beans : null);
  }

  /**
   * query (单条)
   */
  private void querySingleBean(String ssid, String bssid, RealmCallback callback) {
      RealmResults<PskBean> beans = mRealm.where(PskBean.class).equalTo("ssid", ssid).equalTo("bssid", bssid).findAllAsync();
      callback.success(beans.load() ? beans : null);
  }

  public void close() {
      if (mRealm != null) {
          mRealm.close();
      }
  }

  public interface RealmCallback {
      void success(RealmResults<PskBean> element);

      void failure(String msg);
  }
}
```
4.混淆
```
# Realm #
-keep class io.realm.annotations.RealmModule
-keep @io.realm.annotations.RealmModule class *
-keep class io.realm.internal.Keep
-keep @io.realm.internal.Keep class * { *; }
-dontwarn javax.**
-dontwarn io.realm.**
```
此时打包会发现 包增大好多,大约4M左右.
原因是默认情况下,Realm会引人所有规格的.so库
如何处理?
app下build.gradle,导出你需要的规格.so库
```
splits {
      abi {
          enable true
          reset()
          include 'armeabi', 'armeabi-v7a', 'arm64-v8a', 'mips', 'x86', 'x86_64'
          universalApk true
      }
  }
```
5.最后,数据并不是保存在 data/data/pagekageName/databases/目录下
  而是在data/data/files/name.realm中
