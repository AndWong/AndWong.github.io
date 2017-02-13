---
title: Android SQLite的使用
date: 2017-02-13 20:34:42
tags: Android
---
Android自带的数据库SQLite，是一个轻量级便捷的数据库。

1.创建
Android SDK有一个抽象类SQLiteOpenHelper用于创建和升级数据库，
所以继承SQLiteOpenHelper并实现其中的onCreate()和onUpgrade()即可创建和升级数据库。

示例：
public class PskDBHelper extends SQLiteOpenHelper {

    /**
     * 构造方法，通常用这个就可以了
     *
     * @param context
     * @param name    数据库名称 如： psk.db
     * @param factory 数据库游标工厂 通常传入 null
     * @param version 版本号 如：1
     */
    public PskDBHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        //创建表SQL语句
        //integer 表示整型，real 表示浮点型，text 表示文本类型，blob 表示二进制类型。
        // 另外，primary key表示将id列设为主键，并用autoincrement关键字表示id列是自增长的。
        String pskTable = "create table psktable(_id integer primary key autoincrement,ssid text,bssid text,psk text)";
        db.execSQL(pskTable);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}

2.使用
通过调用 pskDBHelper.getReadableDatabase() 和 pskDBHelper.getWritableDatabase()来获取 SQLiteDatabase 对象。
其中如果getReadableDatabase()的数据库不存在则自动创建，存在则返回相应的对象，
如果调用getWritableDatabase()的数据库不存在则会报错。

3、增删改查
这里只简单示例用SQL语句操作的方法。

(1)插入数据
public void insert(SQLiteDatabase db) {
    //插入数据SQL语句
    String insertSQL = "insert into psktable(ssid,bssid,psk) values('wong','20:6a:8a:68:81:ce','12345678')";
    //执行SQL语句
    db.execSQL(insertSQL);
}

(2)删除数据
public void del(SQLiteDatabase db) {
    //删除SQL语句
    String sql = "delete from psktable where _id = 1";
    //执行SQL语句
    db.execSQL(sql);
}

(3)更新数据
public void update(SQLiteDatabase db) {
    //修改SQL语句
    String sql = "update psktable set ssid = 'wong' where _id = 1";
    //执行SQL
    db.execSQL(sql);
}

//在Android中查询数据是通过Cursor类来实现的，
// 当我们使用SQLiteDatabase.query()方法时，
// 会得到一个Cursor对象，Cursor指向的就是每一条数据
(4)查询数据
public void search(SQLiteDatabase db) {
    //查询获得游标
    Cursor cursor = db.query("psktable", null, null, null, null, null, null);
    //判断游标是否为空
    if (cursor.moveToFirst()) {
        //遍历游标
        for (int i = 0; i < cursor.getCount(); i++) {
            cursor.move(i);
            //获得ID
            int id = cursor.getInt(0);
            //获得ssid
            String ssid = cursor.getString(1);
            //获得bssid
            String bssid = cursor.getString(2);
            //获取psk
            String psk = cursor.getString(3);
        }
    }
}
