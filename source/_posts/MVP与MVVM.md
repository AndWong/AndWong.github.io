---
title: mvp与mvvm
date: 2016-12-17 21:10:41
tags: 架构
---
一.MVP
首先来看MVP各自负责什么：

Model，负责定义数据（解决什么是数据）
Presenter, 负责在Model和View之间，从model里取出数据，格式化后在View上展示（解决如何把数据和用户界面放在一起）
View，用于展示数据（解决如何展示数据）

很显然Presenter作为中间者，它是同时拥有View和Model的引用的.
而Model和View必须是完全隔离的，不允许两者之间互相通信。
<!--more-->
Model在三者中是独立性最高的，Model不应该拥有对View的引用，而且Model也不需要保存对Presenter的引用，对于Presenter而已，Model只需要提供接口，等着Presenter来调用时返回相应数据即可，这和经典MVC模式中是非常不同的，在MVC中Model在数据发送变化后，是需要发送广播来告之View去更新用户界面的，而在MVP中，Model是不应该去通知View，而是通知Presenter来间接的更新View的。

而Presenter和Model的关系也应该是基于接口来通信，这样才能把Model和Presenter的耦合度也降到最低，那么在需要改变Model内部实现，甚至彻底替换Model的时候，Presenter则是无需随之改变的。这样做带来的另一个好处就是你可以通过Mock一个Model来对Presenter以及View做模拟测试了，从而提高了可测试性。

那么View和Presenter的关系呢？View是需要拥有对Presenter的引用，但仅仅是为了将用户的操作和事件立即传递给Presenter，为了让View和Presenter耦合较低，View也只应该通过接口与Presenter通信，从而保证View是完全被动的，一方面它由用户的操作触发来和Presenter通信，另一方面它完全受Presenter控制，唯一需要做的事情就是如何展示数据。

简要总结三者之间的关系是：View和Model之间没有联系，View通过接口向Presenter来传递用户操作，Model不主动和Presenter联系，被动的等着Presenter来调用其接口，Presenter通过接口和View/Model来联系。

谷歌官方给的![MVP例子]https://github.com/googlesamples/android-architecture

为了便于理解，这里提供一些伪代码加注释演示一下如何实现MVP模式：

View
```
interface IUserView {

  void setPresenter(presenter);
  void showUsers(users);
  void showDeleteUserComplete();
  void showDeleteUserError();

}

class UserView implements IUserView {

  UserPresenter presenter;

  // 保持对Presenter的引用，用于路由用户操作
  void setPresenter(presenter) {
      this.presenter = presenter;
  }

  // 将Presenter传递来的数据展示出来
  void showUsers(users) {
      draw(users);
  }

  // Model操作数据成功后，通过Presenter来告之View需要更新用户界面
  void showDeleteUserComplete() {
      alert("Delete User Complete");
  }

  // Model操作数据失败后，也是通过Presenter来告之View需要更新用户界面
  void showDeleteUserError() {
      alert("Delete User Fail");
  }

  // 当用户点击某个按钮时，将用户操作路由给presenter，由presenter去处理
  void onDeleteButtonClicked(event) {
      presenter.deleteUser(event);
  }

}
```
Model
```
interface IUserModel {

  List<User> getUsers();
  boolean deleteUserById();

}

class UserModel implements IUserModel {

  // 在数据库里查找数据，并将数据返回给presenter
  List<User> getUsers() {
       return getUsersInDatabase(id);
  }

  // 在数据库里删除数据，并将结果返回给presenter
  User deleteUserById(id) {
      return deleteUserByIdInDatabase(id);
  }

}
```
Presenter
```
interface IUserUserPresenter {

  void deleteUser(event);

}

class UserUserPresenter implements IUserPresenter {

  // 保持对View的引用
  IUserView view;
  // 保持对Model的引用
  IUserModel model;

  UserUserPresenter(IUserView view, IUserModel model) {
    this.view = view;    
    this.model = model;

    this.view.setPresenter(this);   
  }

  void start() {
    // 从Model中取出数据
    List<User> users = model.getUsers();
    // 将数据发送给View，让其展示到用户界面
    view.showUsers(users);
  }

  void deleteUser(event) {
    // View将用户操作路由过来，由Presenter来处理
    long uid = whichUserNeedToDeleteBy(event);
    // 将用户操作翻译成命令或消息传递给model，以改变数据
    boolean success = model.deleteUserById(uid);
    // 将Model操作数据后的结果通知View来改变用户界面
    if (success) {
          view.onDeleteUserSuccess();
    } else {
        view.onDeleteUserFail();  
    }
  }
}
```

二.MVVP
MVVM是由三个核心部分，每个都有它自己的不同而独立的作用：

Model - 包含业务和验证逻辑的数据模型
View -- 定义结构,布局和View 在屏幕上的显示
ViewModel - 充当 View 和 ViewModel 之间的连接, 处理所有视图逻辑.
