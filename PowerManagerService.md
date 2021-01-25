```java
userActivityNoUpdateLocked(
                        now, PowerManager.USER_ACTIVITY_EVENT_OTHER, 0, Process.SYSTEM_UID);
updatePowerStateLocked();
```
这个方法中调用到了updatePowerStateLocked()方法，这是整个PMS中最重要的方法

PowerManagerService.java
PowerManager.java

goToSleep

wakeUp

http://blog.chinaunix.net/uid-30510400-id-5569393.html

updateSuspendBlockerLocked
