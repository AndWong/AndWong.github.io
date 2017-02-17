---
title: 如何在moudle中引入aar
date: 2017-01-16 20:32:45
tags: Android Studio
---
app已经引入aar-A,如何在moudle中引入aar-B而不会出现问题?

1.在moudle目录下的build.gradle
```
android{
  repositories {
    flatDir {
        dirs 'libs'
    }
  }
}

dependencies {
  compile(name: 'aar-B', ext: 'aar')
}
```
 2.在app目录下的build.gradle
```
repositories {
    flatDir {
        dirs('libs','../moudle/libs')
    }
}
```
