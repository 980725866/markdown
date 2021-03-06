
OomAdjuster
OomAdjProfiler.java

updateOomAdjLocked：更新adj，当目标进程为空或者被杀则返回false；否则返回true;
computeOomAdjLocked：计算adj，返回计算后RawAdj值;
applyOomAdjLocked：应用adj，当需要杀掉目标进程则返回false；否则返回true
在Android中，Process.setProcessGroup(int pid, int group)用来设置进程的调度组。调度组会影响进程的CPU占用时间

，Android系统还定义了两个和进程相关的状态值，一个就是定义在ProcessList.java里的adj值，另一个
是定义在ActivityManager.java里的procState值


static final String OOM_ADJ_REASON_METHOD = "updateOomAdj";
static final String OOM_ADJ_REASON_NONE = OOM_ADJ_REASON_METHOD + "_meh";
static final String OOM_ADJ_REASON_ACTIVITY = OOM_ADJ_REASON_METHOD + "_activityChange";
static final String OOM_ADJ_REASON_FINISH_RECEIVER = OOM_ADJ_REASON_METHOD + "_finishReceiver";
static final String OOM_ADJ_REASON_START_RECEIVER = OOM_ADJ_REASON_METHOD + "_startReceiver";
static final String OOM_ADJ_REASON_BIND_SERVICE = OOM_ADJ_REASON_METHOD + "_bindService";
static final String OOM_ADJ_REASON_UNBIND_SERVICE = OOM_ADJ_REASON_METHOD + "_unbindService";
static final String OOM_ADJ_REASON_START_SERVICE = OOM_ADJ_REASON_METHOD + "_startService";
static final String OOM_ADJ_REASON_GET_PROVIDER = OOM_ADJ_REASON_METHOD + "_getProvider";
static final String OOM_ADJ_REASON_REMOVE_PROVIDER = OOM_ADJ_REASON_METHOD + "_removeProvider";
static final String OOM_ADJ_REASON_UI_VISIBILITY = OOM_ADJ_REASON_METHOD + "_uiVisibility";
static final String OOM_ADJ_REASON_WHITELIST = OOM_ADJ_REASON_METHOD + "_whitelistChange";
static final String OOM_ADJ_REASON_PROCESS_BEGIN = OOM_ADJ_REASON_METHOD + "_processBegin";
static final String OOM_ADJ_REASON_PROCESS_END = OOM_ADJ_REASON_METHOD + "_processEnd";









展讯自己加的
ProcessSchedExecutor.java
PerformanceManagerService
./base/core/java/android/app/IPerformanceManagerInternal.aidl
void windowReallyDrawnDone(in String pkgName);
    TaskThumbnail getTaskThumbnail(in Intent intent);
    void removePendingUpdateThumbTask();
    boolean pkgSupportRecentThumbnail(in String pkgName);
    void removeApplcationSnapShot(in String pkgName);
    void scheduleBoostRRForApp(in boolean boost);
    void scheduleBoostRRForAppName(in boolean boost, String pkgName, int type);
    void scheduleBoostWhenTouch();
    void attachPreforkApplication(in int userId, IBinder app, String procName);
    boolean isUserAMonkeyNoCheck();


SchedulingPolicyService
./base/core/java/android/os/ISchedulingPolicyService.aidl
int requestPriority(int pid, int tid, int prio, boolean isForApp);
int requestCpusetBoost(boolean enable, IBinder client);




int maxAdj;                 // Maximum OOM adjustment for this process
int curRawAdj;              // Current OOM unlimited adjustment for this process
int setRawAdj;              // Last set OOM unlimited adjustment for this process
int curAdj;                 // Current OOM adjustment for this process
int setAdj;                 // Last set OOM adjustment for this process


int curSchedGroup;          // Currently desired scheduling class
int setSchedGroup;          // Last set to background scheduling class
adjType

adjType,schedGroup,procState的对应关系



addAppLocked


oom_score_adj = oom_adj



updateLruProcessLocked

removeLruProcessLocked



handleProcessStartedLocked

updateOomLevels

    /** @hide Not a real process state. */
    public static final int PROCESS_STATE_UNKNOWN = -1;

    /** @hide Process is a persistent system process. */
    public static final int PROCESS_STATE_PERSISTENT = 0;

    /** @hide Process is a persistent system process and is doing UI. */
    public static final int PROCESS_STATE_PERSISTENT_UI = 1;

    /** @hide Process is hosting the current top activities.  Note that this covers
     * all activities that are visible to the user. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_TOP = 2;

    /** @hide Process is hosting a foreground service with location type. */
    public static final int PROCESS_STATE_FOREGROUND_SERVICE_LOCATION = 3;

    /** @hide Process is bound to a TOP app. This is ranked below SERVICE_LOCATION so that
     * it doesn't get the capability of location access while-in-use. */
    public static final int PROCESS_STATE_BOUND_TOP = 4;

    /** @hide Process is hosting a foreground service. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_FOREGROUND_SERVICE = 5;

    /** @hide Process is hosting a foreground service due to a system binding. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_BOUND_FOREGROUND_SERVICE = 6;

    /** @hide Process is important to the user, and something they are aware of. */
    public static final int PROCESS_STATE_IMPORTANT_FOREGROUND = 7;

    /** @hide Process is important to the user, but not something they are aware of. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_IMPORTANT_BACKGROUND = 8;

    /** @hide Process is in the background transient so we will try to keep running. */
    public static final int PROCESS_STATE_TRANSIENT_BACKGROUND = 9;

    /** @hide Process is in the background running a backup/restore operation. */
    public static final int PROCESS_STATE_BACKUP = 10;

    /** @hide Process is in the background running a service.  Unlike oom_adj, this level
     * is used for both the normal running in background state and the executing
     * operations state. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_SERVICE = 11;

    /** @hide Process is in the background running a receiver.   Note that from the
     * perspective of oom_adj, receivers run at a higher foreground level, but for our
     * prioritization here that is not necessary and putting them below services means
     * many fewer changes in some process states as they receive broadcasts. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_RECEIVER = 12;

    /** @hide Same as {@link #PROCESS_STATE_TOP} but while device is sleeping. */
    public static final int PROCESS_STATE_TOP_SLEEPING = 13;

    /** @hide Process is in the background, but it can't restore its state so we want
     * to try to avoid killing it. */
    public static final int PROCESS_STATE_HEAVY_WEIGHT = 14;

    /** @hide Process is in the background but hosts the home activity. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_HOME = 15;

    /** @hide Process is in the background but hosts the last shown activity. */
    public static final int PROCESS_STATE_LAST_ACTIVITY = 16;

    /** @hide Process is being cached for later use and contains activities. */
    @UnsupportedAppUsage
    public static final int PROCESS_STATE_CACHED_ACTIVITY = 17;

    /** @hide Process is being cached for later use and is a client of another cached
     * process that contains activities. */
    public static final int PROCESS_STATE_CACHED_ACTIVITY_CLIENT = 18;

    /** @hide Process is being cached for later use and has an activity that corresponds
     * to an existing recent task. */
    public static final int PROCESS_STATE_CACHED_RECENT = 19;

    /** @hide Process is being cached for later use and is empty. */
    public static final int PROCESS_STATE_CACHED_EMPTY = 20;

    /** @hide Process does not exist. */
    public static final int PROCESS_STATE_NONEXISTENT = 21;




--------------------------------------------------------------------------



##### uid
用户ID，每个不同的应用程序都有一个uid，uid是你安装应用程序时系统赋予的，是不变的,卸载重新安装有可能会变

##### pid
进程ID

##### tid
线程ID

Android中设置进程或线程API，通过使用linux的sched_setscheduler，setpriority函数和操作CGroup文件节点来设置线程优先级和设置调度策略

##### 设置线程优先级
```java
Process.setThreadPriority(int tid, int priority)
Process.setThreadPriority(int priority)
```

##### 设置线程组
```java
Process.setThreadGroup(int tid, int group)
```

##### 设置进程组
```java
Process.setProcessGroup(int pid, int group)
```

##### 设置线程调度器
```java
Process.setThreadScheduler(int tid, int policy, int priority)
```

##### 线程优先级
```java
// 应用的默认优先级
public static final int THREAD_PRIORITY_DEFAULT = 0;
// 线程的最低优先级
public static final int THREAD_PRIORITY_LOWEST = 19;
// 后台线程的默认优先级
public static final int THREAD_PRIORITY_BACKGROUND = 10;
// 前台进程的优先级
public static final int THREAD_PRIORITY_FOREGROUND = -2;
// 显示功能的优先级
public static final int THREAD_PRIORITY_DISPLAY = -4;
// 紧急显示功能的优先级
public static final int THREAD_PRIORITY_URGENT_DISPLAY = -8;
// 视频线程默认优先级
public static final int THREAD_PRIORITY_VIDEO = -10;
// 音频线程默认优先级
public static final int THREAD_PRIORITY_AUDIO = -16;
// 紧急音频线程默认优先级
public static final int THREAD_PRIORITY_URGENT_AUDIO = -19;
```

##### 线程组
对应sched_policy.h中的SchedPolicy
```java
// 默认线程组
public static final int THREAD_GROUP_DEFAULT = -1;
// 后台非交互组
public static final int THREAD_GROUP_BG_NONINTERACTIVE = 0;
// 前台线程组
private static final int THREAD_GROUP_FOREGROUND = 1;
// 系统组
public static final int THREAD_GROUP_SYSTEM = 2;
// 音频应用程序组
public static final int THREAD_GROUP_AUDIO_APP = 3;
// 系统音频应用程序组
public static final int THREAD_GROUP_AUDIO_SYS = 4;
// 头等app组
public static final int THREAD_GROUP_TOP_APP = 5;
// phone等通讯app组
public static final int THREAD_GROUP_RT_APP = 6;
```

##### 调度策略
```java
// 默认调度策略
public static final int SCHED_OTHER = 0;
// FIFO调度策略
public static final int SCHED_FIFO = 1;
// RR调度策略
public static final int SCHED_RR = 2;
// 批调度策略
public static final int SCHED_BATCH = 3;
// idle调度策略
public static final int SCHED_IDLE = 5;
```

##### linux系统api设置调度器
```java
int sched_setscheduler(pid_t pid, int policy, const struct sched_param *param);

policy：
	SCHED_OTHER
	SCHED_BATCH
	SCHED_IDLE
	SCHED_FIFO
	SCHED_RR
```

##### linux系统api设置线程或进程优先级
```java
int setpriority(int which, int who, int prio);

which:
	PRIO_PROCESS
	PRIO_PGRP
	PRIO_USER
```

##### cgroups
cgroups，其名称源自控制组群（control groups）的简写，是Linux内核的一个功能，用来限制、控制与分离一个进程组的资源（如CPU、内存、磁盘输入输出等）<br><br>
Cgroups提供了以下功能:
- 1.限制进程组可以使用的资源数量（Resource limiting ）
- 2.进程组的优先级控制（Prioritization ）
- 3.记录进程组使用的资源数量（Accounting ）
- 4.进程组隔离（Isolation）
- 5.进程组控制（Control）

##### libprocessgroup
set_sched_policy和set_cpuset_policy通过操作Cgroups文件节点以达到设置优先级，限制进程CPU资源的目的
```java
int set_cpuset_policy(int tid, SchedPolicy policy);
int set_sched_policy(int tid, SchedPolicy policy);
```

```c
/* Keep in sync with THREAD_GROUP_* in frameworks/base/core/java/android/os/Process.java */
typedef enum {
    SP_DEFAULT = -1,
    SP_BACKGROUND = 0,
    SP_FOREGROUND = 1,
    SP_SYSTEM = 2,  // can't be used with set_sched_policy()
    SP_AUDIO_APP = 3,
    SP_AUDIO_SYS = 4,
    SP_TOP_APP = 5,
    SP_RT_APP = 6,
    SP_RESTRICTED = 7,
    SP_CNT,
    SP_MAX = SP_CNT - 1,
    SP_SYSTEM_DEFAULT = SP_FOREGROUND,
} SchedPolicy;
```

##### cgroups.json部分节点如下
```java
{
  "Cgroups": [
    {
      "Controller": "cpu",
      "Path": "/dev/cpuctl",
      "Mode": "0755",
      "UID": "system",
      "GID": "system"
    },
    {
      "Controller": "cpuset",
      "Path": "/dev/cpuset",
      "Mode": "0755",
      "UID": "system",
      "GID": "system"
    },
    {
      "Controller": "memory",
      "Path": "/dev/memcg",
      "Mode": "0700",
      "UID": "root",
      "GID": "system"
    },
  ]
}
```

https://www.jianshu.com/p/0501bc2bbe7c
https://www.jianshu.com/p/4ee14aa23f07?from=singlemessage
https://baike.baidu.com/item/Cgroup/4988200?fr=aladdin





 







































