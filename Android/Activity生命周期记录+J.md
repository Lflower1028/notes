# Activity生命周期记录

## 接听电话
### 电话来了，去查看是否接听。
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@31ffd4ba time:662805479
D/QuizActivity，第一个: onPause() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用


### 接听电话
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@31ffd4ba time:662828152
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onResume() 方法被调用



---------------------

onSaveInstanceState()只是为了防止Activity被意外杀死，和设备配置变更？ 它的目的是？ 
目的：在Activity对象被销毁时，还能够进行一些数据恢复。而当 activity is paused or stopped，其状态是保存的，没有丢失，不需要恢复。按返回键后数据不能通过这种方法恢复。 

而当前Activity没有被销毁，数据是一直都存在的，也就不需要进行恢复。所以恢复操作只在onCreate()中进行? 也要注意恢复。

---------------------

## 打开App
### 打开应用
D/QuizActivity，第一个: onCreate(Bundle) 方法被调用
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
V/RenderScript: 0xa340d400 Launching thread(s), CPUs 4


## 两Activity之间的切换

此时还涉及了回退栈，① 启动② 后，② 被至于回退栈顶，① 被暂停并停止；用户点击返回键返回① 时，② 被弹出栈外并被销毁，此时① 立即重启并回复运行。

### 开启子Activity 2
I/Timeline: Timeline: Activity_launch_request time:664200869
D/QuizActivity，第一个: onPause() 方法被调用
D/CheatActivity，第二个: onCreate() 方法被调用
D/CheatActivity，第二个: onStart() 方法被调用
D/CheatActivity，第二个: onResume() 方法被调用
D/ActivityThreadInjector: clearCachedDrawables.
D/OpenGLRenderer: endAllActiveAnimators on 0xa34b9080 (RippleDrawable) with handle 0xa3343500
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@3faa24cd time:664201051
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用


### 在子Activity 2按返回键，返回到父Activity 1
D/CheatActivity，第二个: onPause() 方法被调用
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@31ffd4ba time:664237243
D/CheatActivity，第二个: onStop() 方法被调用
D/CheatActivity，第二个: onDestroy() 方法被调用


### 数据保存问题
> 在第一个Activity中覆盖了onSaveInstanceState()

第一个Activity，跳转到 The Pacific Ocean 问题
再进入第二个.
还是在，The Pacific Ocean 问题，证明保存了数据
I/Timeline: Timeline: Activity_launch_request time:664585777
D/QuizActivity，第一个: onPause() 方法被调用
D/CheatActivity，第二个: onCreate() 方法被调用
D/CheatActivity，第二个: onStart() 方法被调用
D/CheatActivity，第二个: onResume() 方法被调用
D/ActivityThreadInjector: clearCachedDrawables.
D/OpenGLRenderer: endAllActiveAnimators on 0xa34c1c80 (RippleDrawable) with handle 0xa33430e0
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@359bfd19 time:664585984
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用
从2回到1
D/CheatActivity，第二个: onPause() 方法被调用
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@3f03be5c time:664608508
D/CheatActivity，第二个: onStop() 方法被调用
D/CheatActivity，第二个: onDestroy() 方法被调用




## 点击Home
### 点击Home，打开其他应用
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用
然后通过Menu返回
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@31ffd4ba time:664413084


## 点击Back
注意：当按Back键时，系统会彻底销毁当前的Activity；此时暂存的Activity记录也会被清除。另区别 销毁Activity 和 停止当前应用的进程的区别

### 点击Back按钮：    并不会保存Activity的状态
D/QuizActivity，第一个: onPause() 方法被调用
D/QuizActivity，第一个: onStop() 方法被调用
D/QuizActivity，第一个: onDestroy() 方法被调用

### 载从Menu返回  
D/QuizActivity，第一个: onCreate(Bundle) 方法被调用
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
D/ActivityThreadInjector: clearCachedDrawables.
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@3f03be5c time:664459350




## 在任务管理器中切换
通过Menu跳转到，之前已经打开过的其他app
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用
并且长时间未返回该应用
W/art: Suspending（暂停） all threads took: 14.373ms
W/art: Suspending all threads took: 17.272ms

再通过Menu返回
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用




## 配置变更(屏幕转换)
横屏（配置变更）
需要重建Activity。需要保存状态数据。


切换横屏
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用
D/QuizActivity，第一个: onDestroy() 方法被调用
D/QuizActivity，第一个: onCreate(Bundle) 方法被调用
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@d763ac8 time:698065578



## 接听电话
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@31ffd4ba time:662828152
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onResume() 方法被调用


## 闹钟响起
闹钟，响起，来到前台
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
闹钟结束
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@d763ac8 time:697865555


## 通知栏
从消息栏进入半透明的，薄暮微光设置对话框
没有任何反应


> Dialog并非通过启动一个新的Activity实现；所以当弹出一个Dialog是当前Activity并不会调用onPause()方法。

## 锁屏
D/QuizActivity，第一个: onPause() 方法被调用
I/QuizActivity，第一个: onSaveInstanceState
D/QuizActivity，第一个: onStop() 方法被调用
## 解屏
D/QuizActivity，第一个: onStart() 方法被调用
D/QuizActivity，第一个: onResume() 方法被调用
I/Timeline: Timeline: Activity_idle id: android.os.BinderProxy@d763ac8 time:698034180



