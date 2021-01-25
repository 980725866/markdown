


// 授权和撤销
grantRuntimePermission
revokeRuntimePermissio
// 安装时权限
grantInstallPermission
	grantPermission
		userState.mGranted = true;
		final int[] newGids = computeGids(userId);

// 运行时权限
grantRuntimePermission
	grantPermission
		userState.mGranted = true;


ActivityCompat

Normal Permissions与Dangerous Permission
PackageManagerService 提供了 addPermission/ removePermission 接口用来动态添加和删除一些权限。但是这些权限必须是所谓的动态权限（ BasePermission.TYPE_DYNAMIC ）

public void grantUriPermission(IApplicationThread caller, String targetPkg, Uri uri, int mode) throws RemoteException;

// 清除所有通过 grantUriPermission 对某个 Uri 授予的权限。

public void revokeUriPermission(IApplicationThread caller, Uri uri, int mode) throws RemoteException;

grantRuntimePermission

runtime-permissions.xml
./system/users/0/runtime-permissions.xml

/data/system/packages.xml
android_filesystem_config.h


checkCallingOrSelfPermission

if (mContext.checkCallingOrSelfPermission(android.Manifest.permission.STATUS_BAR_SERVICE)
		!= PackageManager.PERMISSION_GRANTED) {
	throw new SecurityException("Caller does not hold permission "
			+ android.Manifest.permission.STATUS_BAR_SERVICE);
}

mContext.checkCallingOrSelfPermission(android.Manifest.permission.WRITE_EXTERNAL_STORAGE)

updatePermissions
    grantPermissions(pkg, replace, changingPkgName, callback);
	permissionsState.setGlobalGids(mGlobalGids);
	bp.isNormal()
		grant = GRANT_INSTALL;
		permissionsState.grantInstallPermission(bp)
	bp.isRuntime()
		grant = GRANT_RUNTIME;
		// Grant previously granted runtime permissions. 授予以前授予的运行时权限
		permissionsState.grantRuntimePermission(bp, userId)
		permissionsState.updatePermissionFlags(bp, userId, flags, flags);


#########################################################################################################

PackageManagerService为Android系统的中的核心功能，主要有APK安装卸载，为APK分配userId, 权限管理，dex优化，四大组件查询等等功能，下面主要分析权限功能
大致涉及到的类如下：

```java
PackageManagerService
Settings
DefaultPermissionGrantPolicy
PermissionManagerService
PermissionState
PermissionData
BasePermission
PermissionSettings
PermissionsState
PackageSetting
PackageParser.Package
PermissionInfo
```

Android系统中权限主要有以下四种：
```java
PROTECTION_NORMAL 普通权限（网络，蓝牙使用等）
PROTECTION_DANGEROUS 危险权限（位置信息获取，联系人获取等）
PROTECTION_SIGNATURE 系统签名权限
PROTECTION_SIGNATURE_OR_SYSTEM system/priv-app下且系统签名权限
```

```java
public class PermissionInfo extends PackageItemInfo implements Parcelable {

    public static final int PROTECTION_NORMAL = 0;

    public static final int PROTECTION_DANGEROUS = 1;

    public static final int PROTECTION_SIGNATURE = 2;

    public static final int PROTECTION_SIGNATURE_OR_SYSTEM = 3;
}
```
AndroidManifest.xml


PermissionManagerInternal
先来看PackageManagerService的创建过程，首先创建了mPermissionManager对象，然后给mDefaultPermissionPolicy赋值，DefaultPermissionPolicy对象是在PermissionManagerService mPermissionManager，mDefaultPermissionPolicy,mSettings赋值，权限相关的功能主要在PermissionManagerService，
```java
public PackageManagerService(Context context, Installer installer,
            boolean factoryTest, boolean onlyCore) {
	synchronized (mPackages) {
		// Expose private service for system components to use.
		LocalServices.addService(
				PackageManagerInternal.class, new PackageManagerInternalImpl());
    /* 1 mPermissionManager对应的实现为PermissionManagerService的内部类PermissionManagerInternalImpl，
		方便systemServer进程自己内部调用PermissionManagerService的接口
		*/
		mPermissionManager = PermissionManagerService.create(context,
				new DefaultPermissionGrantedCallback() {
					@Override
					public void onDefaultRuntimePermissionsGranted(int userId) {
						synchronized(mPackages) {
							mSettings.onDefaultRuntimePermissionsGrantedLPr(userId);
						}
					}
				}, mPackages /*externalLock*/);
		/* 2 获取默认的权限策略，和运行时权限相关 */
		mDefaultPermissionPolicy = mPermissionManager.getDefaultPermissionGrantPolicy();

		/* 3 创建Settings对象，Setting对象用来记录安装信息，下次开机重启恢复应用的安装信息，授权记录的登记等 */
		mSettings = new Settings(mPermissionManager.getPermissionSettings(), mPackages);
	}

  /* 4 读取上一次安装信息 */
	mFirstBoot = !mSettings.readLPw(sUserManager.getUsers(false));

  /* 5 扫描system,vendor, data目录下的apk,读取apk中的 */
	scanDirTracedLI();

  /* 6 更新权限，给某些apk授权 */
	mPermissionManager.updateAllPermissions(
		StorageManager.UUID_PRIVATE_INTERNAL, sdkUpdated, mPackages.values(),
		mPermissionCallback);

  /* 7 将相关权限信息写入文件 */
	/* */
	mSettings.writeLPr();
}
```

```java
public static PermissionManagerInternal PermissionManagerService::create(Context context,
		@Nullable DefaultPermissionGrantedCallback defaultGrantCallback,
		@NonNull Object externalLock) {
	final PermissionManagerInternal permMgrInt =
			LocalServices.getService(PermissionManagerInternal.class);
	if (permMgrInt != null) {
		return permMgrInt;
	}
	/* 1 创建PermissionManagerService对象，PermissionManagerService对象负责主要的权限管理工作 */
	new PermissionManagerService(context, defaultGrantCallback, externalLock);
	return LocalServices.getService(PermissionManagerInternal.class);
}
```

```java
PermissionManagerService(Context context,
            @Nullable DefaultPermissionGrantedCallback defaultGrantCallback,
            @NonNull Object externalLock) {
  /* 1 创建PermissionSettings对象 */
	mSettings = new PermissionSettings(context, mLock);
	/* 2 创建DefaultPermissionGrantPolicy对象 */
	mDefaultPermissionGrantPolicy = new DefaultPermissionGrantPolicy(
			context, mHandlerThread.getLooper(), defaultGrantCallback, this);
  /* 3 获取系统配置信息 system/etc/platform.xml等等文件中获取 */
	SystemConfig systemConfig = SystemConfig.getInstance();
	mSystemPermissions = systemConfig.getSystemPermissions();
	mGlobalGids = systemConfig.getGlobalGids();
	// propagate permission configuration
	/* 4 PermissionEntry对象对应 system/etc/platform.xml的一个permission项 */
	final ArrayMap<String, SystemConfig.PermissionEntry> permConfig =
			SystemConfig.getInstance().getPermissions();
	synchronized (mLock) {
		for (int i=0; i<permConfig.size(); i++) {
			final SystemConfig.PermissionEntry perm = permConfig.valueAt(i);
			BasePermission bp = mSettings.getPermissionLocked(perm.name);
			if (bp == null) {
				// add to mSettings
				bp = new BasePermission(perm.name, "android", BasePermission.TYPE_BUILTIN);
				/* 将权限信息存入PermissionSettings中的mPermissions属性中*/
				mSettings.putPermissionLocked(perm.name, bp);
			}
			if (perm.gids != null) {
				bp.setGids(perm.gids, perm.perUser);
			}
		}
	}
  /* 创建PermissionManagerInternalImpl对象，并添加到LocalServices中 */
	LocalServices.addService(PermissionManagerInternal.class, new PermissionManagerInternalImpl());
}
```

```java
SystemConfig() {
	// Read configuration from system
	/* 读取配置文件，system/etc/platform.xml */
	readPermissions(Environment.buildPath(
			Environment.getRootDirectory(), "etc", "sysconfig"), ALLOW_ALL);

	// Read configuration from the old permissions dir
	readPermissions(Environment.buildPath(
			Environment.getRootDirectory(), "etc", "permissions"), ALLOW_ALL);
}
```

### system/etc/platform.xml
```java
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <permission name="android.permission.BLUETOOTH_ADMIN" >
        <group gid="net_bt_admin" />
    </permission>

    <permission name="android.permission.BLUETOOTH" >
        <group gid="net_bt" />
    </permission>

    <permission name="android.permission.READ_EXTERNAL_STORAGE" />
    <permission name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <assign-permission name="android.permission.MODIFY_AUDIO_SETTINGS" uid="media" />

</permissions>
```

### gid数字对应定义在android_filesystem_config.h
```c
#define AID_SYSTEM 1000 /* system server */
#define AID_NET_BT_ADMIN 3001 /* bluetooth: create any socket */
#define AID_NET_BT 3002       /* bluetooth: create sco, rfcomm or l2cap sockets */
#define AID_INET 3003         /* can create AF_INET and AF_INET6 sockets */
#define AID_NET_RAW 3004      /* can create raw INET sockets */
#define AID_NET_ADMIN 3005    /* can configure interfaces and routing tables. */
#define AID_NET_BW_STATS 3006 /* read bandwidth statistics */
#define AID_NET_BW_ACCT 3007  /* change bandwidth statistics accounting */
#define AID_READPROC 3009     /* Allow /proc read access */
#define AID_WAKELOCK 3010     /* Allow system wakelock read/write access */
#define AID_UHID 3011         /* Allow read/write to /dev/uhid node */
```

```java
private void SystemConfig::readPermissionsFromXml(File permFile, int permissionFlag) {
	if ("permission".equals(name) && allowPermissions) {
		String perm = parser.getAttributeValue(null, "name");
		if (perm == null) {
			Slog.w(TAG, "<permission> without name in " + permFile + " at "
				+ parser.getPositionDescription());
			XmlUtils.skipCurrentTag(parser);
			continue;
		}
		perm = perm.intern();
		readPermission(parser, perm);
	}
}
```

```java
void SystemConfig::readPermission(XmlPullParser parser, String name)
		throws IOException, XmlPullParserException {
	final boolean perUser = XmlUtils.readBooleanAttribute(parser, "perUser", false);
	final PermissionEntry perm = new PermissionEntry(name, perUser);
	mPermissions.put(name, perm);

	int outerDepth = parser.getDepth();
	int type;
	while ((type=parser.next()) != XmlPullParser.END_DOCUMENT
		   && (type != XmlPullParser.END_TAG
				   || parser.getDepth() > outerDepth)) {
		String tagName = parser.getName();
		if ("group".equals(tagName)) {
			String gidStr = parser.getAttributeValue(null, "gid");
			if (gidStr != null) {
				int gid = Process.getGidForName(gidStr);
				/* 给对应的权限设置相应的gids */
				perm.gids = appendInt(perm.gids, gid);
			} else {
				Slog.w(TAG, "<group> without gid at "
						+ parser.getPositionDescription());
			}
		}
		XmlUtils.skipCurrentTag(parser);
	}
}
```


```java
Settings(File dataDir, PermissionSettings permission, Object lock) {
	mLock = lock;
	mPermissions = permission;
	/* mSettingsFilename存放apk安装信息，权限相关为主 */
	mSettingsFilename = new File(mSystemDir, "packages.xml");
	/* mPackageListFilename存放apk安装信息 uid,gids */
	mPackageListFilename = new File(mSystemDir, "packages.list");

}
```

### /data/system/packages.xml
```c
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<packages>

    <!-- 保存的所有权限，以及所在的包名，系统权限包名为android -->
    <permissions>
      <item name="android.permission.REAL_GET_TASKS" package="android" protection="18" />
      <item name="android.permission.ACCESS_CACHE_FILESYSTEM" package="android" protection="18" />
    </permissions>

    <!-- com.mediatek.mtklogger apk信息 -->
    <package name="com.mediatek.mtklogger" codePath="/system/app/MTKLogger" nativeLibraryPath="/system/app/MTKLogger/lib" publicFlags="805879365" privateFlags="0" ft="11e8dc5d800" it="11e8dc5d800" ut="11e8dc5d800" version="40000" userId="10087" isOrphaned="true">
        <sigs count="1" schemeVersion="3">
            <cert index="0" />
        </sigs>
        <perms>
            <item name="android.permission.SYSTEM_ALERT_WINDOW" granted="true" flags="0" />
            <item name="android.permission.FOREGROUND_SERVICE" granted="true" flags="0" />
            <item name="android.permission.RECEIVE_BOOT_COMPLETED" granted="true" flags="0" />
            <item name="android.permission.BLUETOOTH" granted="true" flags="0" />
            <item name="android.permission.WRITE_MEDIA_STORAGE" granted="true" flags="0" />
            <item name="android.permission.GET_TASKS" granted="true" flags="0" />
            <item name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" granted="true" flags="0" />
            <item name="android.permission.STATUS_BAR_SERVICE" granted="true" flags="0" />
            <item name="android.permission.MANAGE_USERS" granted="true" flags="0" />
            <item name="android.permission.READ_LOGS" granted="true" flags="0" />
            <item name="com.mediatek.engineermode.permission.WIFI_LOG" granted="true" flags="0" />
            <item name="android.permission.VIBRATE" granted="true" flags="0" />
            <item name="android.permission.READ_FRAME_BUFFER" granted="true" flags="0" />
            <item name="android.permission.DUMP" granted="true" flags="0" />
        </perms>
        <proper-signing-keyset identifier="1" />
    </package>
</packages>
```

### /data/system/packages.list
apk安装信息uid,gids
```java
com.android.fmradio 10018 0 /data/user/0/com.android.fmradio platform:privapp:targetSdkVersion=28 3002,1013
com.mediatek.gba 1001 0 /data/user/0/com.mediatek.gba platform:privapp:targetSdkVersion=28 1065,3002,1023,3003,3001,3007,1002,3010,3011,3006
com.mediatek.ims 1001 0 /data/user/0/com.mediatek.ims platform:privapp:targetSdkVersion=28 1065,3002,1023,3003,3001,3007,1002,3010,3011,3006
```

```java
boolean Settings::readLPw(@NonNull List<UserInfo> users) {
	str = new FileInputStream(mSettingsFilename);
	if (tagName.equals("permissions")) {
		// all mPermissions
		/* 读取package.xml中的permissions tag下的所有权限信息 */
		mPermissions.readPermissions(parser);
	}

	for (UserInfo user : users) {
		/* 读取runtime-permissions.xml中的runtime-permissions tag下的所有权限信息 */
		mRuntimePermissionsPersistence.readStateForUserSyncLPr(user.id);
	}
}
```

```java
public void PermissionSettings::readPermissions(XmlPullParser parser) throws IOException, XmlPullParserException {
	synchronized (mLock) {
		readPermissions(mPermissions, parser);
	}
}
```

```java
public static void PermissionSettings::readPermissions(ArrayMap<String, BasePermission> out, XmlPullParser parser)
            throws IOException, XmlPullParserException {
	int outerDepth = parser.getDepth();
	int type;
	while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
			&& (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) {
		if (type == XmlPullParser.END_TAG || type == XmlPullParser.TEXT) {
			continue;
		}

		if (!BasePermission.readLPw(out, parser)) {
			PackageManagerService.reportSettingsProblem(Log.WARN,
					"Unknown element reading permissions: " + parser.getName() + " at "
							+ parser.getPositionDescription());
		}
		XmlUtils.skipCurrentTag(parser);
	}
}
```

```java
public static boolean BasePermission::readLPw(@NonNull Map<String, BasePermission> out,
            @NonNull XmlPullParser parser) {
	final String tagName = parser.getName();
	/* <item name="android.permission.READ_EXTERNAL_STORAGE" granted="true" flags="0" /> */
	if (!tagName.equals(TAG_ITEM)) {
		return false;
	}
	final String name = parser.getAttributeValue(null, ATTR_NAME);
	final String sourcePackage = parser.getAttributeValue(null, ATTR_PACKAGE);
	final String ptype = parser.getAttributeValue(null, "type");
	if (name == null || sourcePackage == null) {
		PackageManagerService.reportSettingsProblem(Log.WARN,
				"Error in package manager settings: permissions has" + " no name at "
						+ parser.getPositionDescription());
		return false;
	}
	final boolean dynamic = "dynamic".equals(ptype);
	/* mPermissions中是否存在对应的bp对象 */
	BasePermission bp = out.get(name);
	// If the permission is builtin, do not clobber it.
	/* 系统中还没有这个权限，则创建一个对应的BasePermission */
	if (bp == null || bp.type != TYPE_BUILTIN) {
		bp = new BasePermission(name.intern(), sourcePackage,
				dynamic ? TYPE_DYNAMIC : TYPE_NORMAL);
	}
	bp.protectionLevel = readInt(parser, null, "protection",
			PermissionInfo.PROTECTION_NORMAL);
	bp.protectionLevel = PermissionInfo.fixProtectionLevel(bp.protectionLevel);
	if (dynamic) {
		final PermissionInfo pi = new PermissionInfo();
		pi.packageName = sourcePackage.intern();
		pi.name = name.intern();
		pi.icon = readInt(parser, null, "icon", 0);
		pi.nonLocalizedLabel = parser.getAttributeValue(null, "label");
		pi.protectionLevel = bp.protectionLevel;
		bp.pendingPermissionInfo = pi;
	}
	/* 将bp对象添加到PermissionSettings的mPermissions中 */
	out.put(bp.name, bp);
	return true;
}
```


```java
public void RuntimePermissionPersistence::readStateForUserSyncLPr(int userId) {
	/* getUserRuntimePermissionsFile 文件对应runtime-permissions.xml */
	File permissionsFile = getUserRuntimePermissionsFile(userId);
	if (!permissionsFile.exists()) {
		return;
	}

	FileInputStream in;
	try {
		in = new AtomicFile(permissionsFile).openRead();
	} catch (FileNotFoundException fnfe) {
		Slog.i(PackageManagerService.TAG, "No permissions state");
		return;
	}

	try {
		XmlPullParser parser = Xml.newPullParser();
		parser.setInput(in, null);
		/* 解析runtime-permissions.xml */
		parseRuntimePermissionsLPr(parser, userId);

	} catch (XmlPullParserException | IOException e) {
		throw new IllegalStateException("Failed parsing permissions file: "
				+ permissionsFile , e);
	} finally {
		IoUtils.closeQuietly(in);
	}
}
```


```java
private void RuntimePermissionPersistence::parseRuntimePermissionsLPr(XmlPullParser parser, int userId)
                throws IOException, XmlPullParserException {
	switch (parser.getName()) {
		/* TAG_PACKAGE = "pkg"; */
		case TAG_PACKAGE: {
			String name = parser.getAttributeValue(null, ATTR_NAME);
			PackageSetting ps = mPackages.get(name);
			if (ps == null) {
				Slog.w(PackageManagerService.TAG, "Unknown package:" + name);
				XmlUtils.skipCurrentTag(parser);
				continue;
			}

			/* 解析tag为pkg下的item */
			parsePermissionsLPr(parser, ps.getPermissionsState(), userId);
		} break;
	}
}
```

```java
 private void RuntimePermissionPersistence::parsePermissionsLPr(XmlPullParser parser, PermissionsState permissionsState,
                int userId) throws IOException, XmlPullParserException {
	final int outerDepth = parser.getDepth();
	int type;
	while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
			&& (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) {
		if (type == XmlPullParser.END_TAG || type == XmlPullParser.TEXT) {
			continue;
		}

		switch (parser.getName()) {
			case TAG_ITEM: {
				String name = parser.getAttributeValue(null, ATTR_NAME);
				BasePermission bp = mPermissions.getPermission(name);
				if (bp == null) {
					Slog.w(PackageManagerService.TAG, "Unknown permission:" + name);
					XmlUtils.skipCurrentTag(parser);
					continue;
				}

				String grantedStr = parser.getAttributeValue(null, ATTR_GRANTED);
				/* <item name="android.permission.READ_EXTERNAL_STORAGE" granted="true" flags="0" /> */
				/* 读取granted的值 */
				final boolean granted = grantedStr == null
						|| Boolean.parseBoolean(grantedStr);

				String flagsStr = parser.getAttributeValue(null, ATTR_FLAGS);
				final int flags = (flagsStr != null)
						? Integer.parseInt(flagsStr, 16) : 0;

				if (granted) {
				  /* 如果之前授权过，则直接授权 */
					permissionsState.grantRuntimePermission(bp, userId);
					/* 更新mUserStates相关权限信息 */
					permissionsState.updatePermissionFlags(bp, userId,
								PackageManager.MASK_PERMISSION_FLAGS, flags);
				} else {
					permissionsState.updatePermissionFlags(bp, userId,
							PackageManager.MASK_PERMISSION_FLAGS, flags);
				}

			} break;
		}
	}
}
```

// apk中的权限合并到mPermissions中位置在哪, apk中的requestedPermissions权限在什么时候授权
// apk的扫描只分析关键部分
```java
private Package PackageParser::parseBaseApkCommon(Package pkg, Set<String> acceptedTags, Resources res,
	XmlResourceParser parser, int flags, String[] outError) throws XmlPullParserException,
	IOException {
	 /* 一个apk对应一个pkg对象 */
	 if (tagName.equals(TAG_PERMISSION)) {
		/* 解析pkg中定义的权限 */
		if (!parsePermission(pkg, res, parser, outError)) {
			return null;
		}
	} else if (tagName.equals(TAG_PERMISSION_TREE)) {
		if (!parsePermissionTree(pkg, res, parser, outError)) {
			return null;
		}
	} else if (tagName.equals(TAG_USES_PERMISSION)) {
		/* <uses-permission android:name="android.permission.INTERNET" /> */
		/* 解析pkg中使用的权限 */
		if (!parseUsesPermission(pkg, res, parser)) {
			return null;
		}
	}
}
```


```java
private boolean PackageParser::parsePermission(Package owner, Resources res,
            XmlResourceParser parser, String[] outError)
        throws XmlPullParserException, IOException {
	/* 创建Permission对象 */
	Permission perm = new Permission(owner);
  /* 将定义的权限添加到pkg对象中 */
	owner.permissions.add(perm);
    return true;
}
```

```java
private void PackageManagerService::commitScanResultsLocked(@NonNull ScanRequest request, @NonNull ScanResult result)
            throws PackageManagerException {
	 commitPackageSettings(pkg, oldPkg, pkgSetting, user, scanFlags,
        (parseFlags & PackageParser.PARSE_CHATTY) != 0 /*chatty*/);
}
```


```java
private void PackageManagerService::commitPackageSettings(PackageParser.Package pkg,
            @Nullable PackageParser.Package oldPkg, PackageSetting pkgSetting, UserHandle user,
            final @ScanFlags int scanFlags, boolean chatty) {
  /* 把pkg，即apk中的permissions合并到PermissionSettings的mPermissions中 */
	/* addAllPermissions调用的PermissionManagerService中的addAllPermissions方法 */
	mPermissionManager.addAllPermissions(pkg, chatty);
}
```

```java
private void PermissionManagerService::addAllPermissions(PackageParser.Package pkg, boolean chatty) {
	final int N = pkg.permissions.size();
	for (int i=0; i<N; i++) {
		PackageParser.Permission p = pkg.permissions.get(i);

		// Assume by default that we did not install this permission into the system.
		p.info.flags &= ~PermissionInfo.FLAG_INSTALLED;

		synchronized (PermissionManagerService.this.mLock) {
			if (p.tree) {
				final BasePermission bp = BasePermission.createOrUpdate(
						mSettings.getPermissionTreeLocked(p.info.name), p, pkg,
						mSettings.getAllPermissionTreesLocked(), chatty);
				/* 把bp对象添加到PermissionSettings的mPermissionGroups中 */
				mSettings.putPermissionTreeLocked(p.info.name, bp);
			} else {
				/* 去重 */
				final BasePermission bp = BasePermission.createOrUpdate(
						mSettings.getPermissionLocked(p.info.name),
						p, pkg, mSettings.getAllPermissionTreesLocked(), chatty);
				/* 把bp对象添加到PermissionSettings的mPermissions中 */
				mSettings.putPermissionLocked(p.info.name, bp);
			}
		}
	}
}
```


```java
private void PermissionManagerService::updateAllPermissions(String volumeUuid, boolean sdkUpdated,
		Collection<PackageParser.Package> allPackages, PermissionCallback callback) {
	final int flags = UPDATE_PERMISSIONS_ALL |
			(sdkUpdated
					? UPDATE_PERMISSIONS_REPLACE_PKG | UPDATE_PERMISSIONS_REPLACE_ALL
					: 0);
	updatePermissions(null, null, volumeUuid, flags, allPackages, callback);
}
```

```java
private void PermissionManagerService::updatePermissions(String changingPkgName, PackageParser.Package changingPkg,
		String replaceVolumeUuid, int flags, Collection<PackageParser.Package> allPackages,
		PermissionCallback callback) {
  /* changingPkgName = null, changingPkg = null, flags = UPDATE_PERMISSIONS_ALL */
	flags = updatePermissionTrees(changingPkgName, changingPkg, flags);
	/* 更新权限 */
	flags = updatePermissions(changingPkgName, changingPkg, flags);

	Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "grantPermissions");
	if ((flags & UPDATE_PERMISSIONS_ALL) != 0) {
		for (PackageParser.Package pkg : allPackages) {
			if (pkg != changingPkg) {
				// Only replace for packages on requested volume
				final String volumeUuid = getVolumeUuidForPackage(pkg);
				final boolean replace = ((flags & UPDATE_PERMISSIONS_REPLACE_ALL) != 0)
						&& Objects.equals(replaceVolumeUuid, volumeUuid);
				/* 给apk授权 */
				grantPermissions(pkg, replace, changingPkgName, callback);
			}
		}
	}

	if (changingPkg != null) {
		// Only replace for packages on requested volume
		final String volumeUuid = getVolumeUuidForPackage(changingPkg);
		final boolean replace = ((flags & UPDATE_PERMISSIONS_REPLACE_PKG) != 0)
				&& Objects.equals(replaceVolumeUuid, volumeUuid);

		grantPermissions(changingPkg, replace, changingPkgName, callback);
	}
	Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
}
```

```java
private void PermissionManagerService::grantPermissions(PackageParser.Package pkg, boolean replace,
            String packageOfInterest, PermissionCallback callback) {

    final PackageSetting ps = (PackageSetting) pkg.mExtras;
    if (ps == null) {
        return;
    }
		/* 一个pkg对应一个PermissionsState对象 */
    final PermissionsState permissionsState = ps.getPermissionsState();
    PermissionsState origPermissions = permissionsState;

    final int[] currentUserIds = UserManagerService.getInstance().getUserIds();

    boolean runtimePermissionsRevoked = false;
    int[] updatedUserIds = EMPTY_INT_ARRAY;

    boolean changedInstallPermission = false;
    permissionsState.setGlobalGids(mGlobalGids);

    synchronized (mLock) {
        final int N = pkg.requestedPermissions.size();
        /// M: CTA requirement - permission control
        boolean pkgReviewRequired = isPackageNeedsReview(pkg,  ps.getSharedUser());

        for (int i = 0; i < N; i++) {
            final String permName = pkg.requestedPermissions.get(i);
            final BasePermission bp = mSettings.getPermissionLocked(permName);
            final boolean appSupportsRuntimePermissions =
                    pkg.applicationInfo.targetSdkVersion >= Build.VERSION_CODES.M;
            final String perm = bp.getName();
            boolean allowedSig = false;
            int grant = GRANT_DENIED;

            if (bp.isNormal()) {
                // For all apps normal permissions are install time ones.
								/* 安装权限 */
                grant = GRANT_INSTALL;
            } else if (bp.isRuntime()) {
                // If a permission review is required for legacy apps we represent
                // their permissions as always granted runtime ones since we need
                // to keep the review required permission flag per user while an
                // install permission's state is shared across all users.
                if (!appSupportsRuntimePermissions && !mSettings.mPermissionReviewRequired) {
                    // For legacy apps dangerous permissions are install time ones.
                    grant = GRANT_INSTALL;
                } else if (origPermissions.hasInstallPermission(bp.getName())) {
                    // For legacy apps that became modern, install becomes runtime.
                    grant = GRANT_UPGRADE;
                } else if (isLegacySystemApp) {
                    // For legacy system apps, install becomes runtime.
                    // We cannot check hasInstallPermission() for system apps since those
                    // permissions were granted implicitly and not persisted pre-M.
                    grant = GRANT_UPGRADE;
                } else {
                    // For modern apps keep runtime permissions unchanged.
										/* 运行时权限 */
                    grant = GRANT_RUNTIME;
                }
            } else if (bp.isSignature()) {
                // For all apps signature permissions are install time ones.
                allowedSig = grantSignaturePermission(perm, pkg, bp, origPermissions);
                if (allowedSig) {
                    grant = GRANT_INSTALL;
                }
            }

            switch (grant) {
                case GRANT_INSTALL: {
                    for (int userId : UserManagerService.getInstance().getUserIds()) {
                        if (origPermissions.getRuntimePermissionState(
                                perm, userId) != null) {
                            // Revoke the runtime permission and clear the flags.
                            origPermissions.revokeRuntimePermission(bp, userId);
                            origPermissions.updatePermissionFlags(bp, userId,
                                  PackageManager.MASK_PERMISSION_FLAGS, 0);
                            // If we revoked a permission permission, we have to write.
                            updatedUserIds = ArrayUtils.appendInt(
                                    updatedUserIds, userId);
                        }
                    }

                    // Grant an install permission.
										/* 给安装权限 */
                    if (permissionsState.grantInstallPermission(bp) !=
                            PermissionsState.PERMISSION_OPERATION_FAILURE) {
                        changedInstallPermission = true;
                    }
                } break;

                case GRANT_RUNTIME: {
                    // Grant previously granted runtime permissions.
                    for (int userId : UserManagerService.getInstance().getUserIds()) {
												/* 使用之前授权的状态 ，状态从runtime-permissions.xml中读取 */
												final PermissionState permissionState = origPermissions
                                .getRuntimePermissionState(perm, userId);
                        int flags = permissionState != null
                                ? permissionState.getFlags() : 0;
                        if (origPermissions.hasRuntimePermission(perm, userId)) {
                        }

                        // Propagate the permission flags.
                        permissionsState.updatePermissionFlags(bp, userId, flags, flags);
                    }
                } break;
            }
        }
    }

    if (callback != null) {
			  /* 回调PackageManagerService中的onPermissionUpdated函数 */
        callback.onPermissionUpdated(updatedUserIds, runtimePermissionsRevoked);
    }
}
```


```java
public void PackageManagerService::onPermissionUpdated(int[] updatedUserIds, boolean sync) {
    synchronized (mPackages) {
        for (int userId : updatedUserIds) {
					  /* 将运行时相关信息写入runtime-permissions.xml中 */
            mSettings.writeRuntimePermissionsForUserLPr(userId, sync);
        }
    }
}
```


```c
void Settings::writeLPr() {
	  /* packages.xml */
    FileOutputStream fstr = new FileOutputStream(mSettingsFilename);
    BufferedOutputStream str = new BufferedOutputStream(fstr);

		serializer.startTag(null, "permissions");
		/* 权限信息写入 */
    mPermissions.writePermissions(serializer);
    serializer.endTag(null, "permissions");

		for (final PackageSetting pkg : mPackages.values()) {
			  /* 写入pkg信息 */
    		writePackageLPr(serializer, pkg);
    }

		/* 写入运行时权限 */
		writeAllRuntimePermissionsLPr();
}
```

```java
public void PermissionSettings::writePermissions(XmlSerializer serializer) throws IOException {
    synchronized (mLock) {
        for (BasePermission bp : mPermissions.values()) {
            bp.writeLPr(serializer);
        }
    }
}
```

```java
public void BasePermission::writeLPr(@NonNull XmlSerializer serializer) throws IOException {
    if (sourcePackageName == null) {
        return;
    }
    serializer.startTag(null, TAG_ITEM);
    serializer.attribute(null, ATTR_NAME, name);
    serializer.attribute(null, ATTR_PACKAGE, sourcePackageName);
    if (protectionLevel != PermissionInfo.PROTECTION_NORMAL) {
        serializer.attribute(null, "protection", Integer.toString(protectionLevel));
    }
    if (type == BasePermission.TYPE_DYNAMIC) {
        final PermissionInfo pi = perm != null ? perm.info : pendingPermissionInfo;
        if (pi != null) {
            serializer.attribute(null, "type", "dynamic");
            if (pi.icon != 0) {
                serializer.attribute(null, "icon", Integer.toString(pi.icon));
            }
            if (pi.nonLocalizedLabel != null) {
                serializer.attribute(null, "label", pi.nonLocalizedLabel.toString());
            }
        }
    }
    serializer.endTag(null, TAG_ITEM);
}
```

```java
void Settings::writeAllRuntimePermissionsLPr() {
    for (int userId : UserManagerService.getInstance().getUserIds()) {
        mRuntimePermissionsPersistence.writePermissionsForUserAsyncLPr(userId);
    }
}
```


```java
public void RuntimePermissionPersistence::writePermissionsForUserAsyncLPr(int userId) {
    final long currentTimeMillis = SystemClock.uptimeMillis();
    if (mWriteScheduled.get(userId)) {
        mHandler.removeMessages(userId);
        mHandler.sendMessageDelayed(message, writeDelayMillis);
    } else {
        mLastNotWrittenMutationTimesMillis.put(userId, currentTimeMillis);
        Message message = mHandler.obtainMessage(userId);
        mHandler.sendMessageDelayed(message, WRITE_PERMISSIONS_DELAY_MILLIS);
        mWriteScheduled.put(userId, true);
    }
}
```

```java
private void RuntimePermissionPersistence::writePermissionsSync(int userId) {
	 /* runtime-permissions.xml */
   AtomicFile destination = new AtomicFile(getUserRuntimePermissionsFile(userId),
           "package-perms-" + userId);
   ArrayMap<String, List<PermissionState>> permissionsForPackage = new ArrayMap<>();
   ArrayMap<String, List<PermissionState>> permissionsForSharedUser = new ArrayMap<>();
   synchronized (mPersistenceLock) {
       mWriteScheduled.delete(userId);
       final int packageCount = mPackages.size();
       for (int i = 0; i < packageCount; i++) {
           String packageName = mPackages.keyAt(i);
           PackageSetting packageSetting = mPackages.valueAt(i);
           if (packageSetting.sharedUser == null) {
               PermissionsState permissionsState = packageSetting.getPermissionsState();
               List<PermissionState> permissionsStates = permissionsState
                       .getRuntimePermissionStates(userId);
               if (!permissionsStates.isEmpty()) {
								   /* 将permissionsState状态信息添加到 permissionsForPackage中 */
                   permissionsForPackage.put(packageName, permissionsStates);
               }
           }
       }
   }

   FileOutputStream out = null;
   try {
       out = destination.startWrite();

       XmlSerializer serializer = Xml.newSerializer();
       serializer.setOutput(out, StandardCharsets.UTF_8.name());
       serializer.setFeature("http://xmlpull.org/v1/doc/features.html#indent-output", true);
       serializer.startDocument(null, true);

       serializer.startTag(null, TAG_RUNTIME_PERMISSIONS);

       String fingerprint = mFingerprints.get(userId);
       if (fingerprint != null) {
           serializer.attribute(null, ATTR_FINGERPRINT, fingerprint);
       }

       final int packageCount = permissionsForPackage.size();
       for (int i = 0; i < packageCount; i++) {
           String packageName = permissionsForPackage.keyAt(i);
           List<PermissionState> permissionStates = permissionsForPackage.valueAt(i);
           serializer.startTag(null, TAG_PACKAGE);
           serializer.attribute(null, ATTR_NAME, packageName);
					 /* 写入xml文件 */
           writePermissions(serializer, permissionStates);
           serializer.endTag(null, TAG_PACKAGE);
       }
       serializer.endTag(null, TAG_RUNTIME_PERMISSIONS);
   // Any error while writing is fatal.
   } catch (Throwable t) {
       Slog.wtf(PackageManagerService.TAG,
               "Failed to write settings, restoring backup", t);
       destination.failWrite(out);
   } finally {
       IoUtils.closeQuietly(out);
   }
}
```

```java
private void RuntimePermissionPersistence::writePermissions(XmlSerializer serializer,
        List<PermissionState> permissionStates) throws IOException {
    for (PermissionState permissionState : permissionStates) {
        serializer.startTag(null, TAG_ITEM);
        serializer.attribute(null, ATTR_NAME,permissionState.getName());
        serializer.attribute(null, ATTR_GRANTED,
                String.valueOf(permissionState.isGranted()));
        serializer.attribute(null, ATTR_FLAGS,
                Integer.toHexString(permissionState.getFlags()));
        serializer.endTag(null, TAG_ITEM);
    }
}
```

```java
PackageManagerService::systemReady() {
		// If we upgraded grant all default permissions before kicking off.
	  for (int userId : grantPermissionsUserIds) {
			  /* 给默认系统某些apk授予特定的运行权限 */
	    	mDefaultPermissionPolicy.grantDefaultPermissions(userId);
	  }

		synchronized (mPackages) {
			/* 再次进行对所有的apk授权更新 */
			mPermissionManager.updateAllPermissions(
					StorageManager.UUID_PRIVATE_INTERNAL, false, mPackages.values(),
					mPermissionCallback);
		}
}
```


```java
public void DefaultPermissionGrantPolicy::grantDefaultPermissions(int userId) {
    /* 给系统apk授运行时权限，以及给一些特定的apk授权 */
		grantPermissionsToSysComponentsAndPrivApps(userId);
    grantDefaultSystemHandlerPermissions(userId);
    grantDefaultPermissionExceptions(userId);
}
```

```java
private void DefaultPermissionGrantPolicy::grantPermissionsToSysComponentsAndPrivApps(int userId) {
    Log.i(TAG, "Granting permissions to platform components for user " + userId);
    final PackageList packageList = mServiceInternal.getPackageList();
    for (String packageName : packageList.getPackageNames()) {
        final PackageParser.Package pkg = mServiceInternal.getPackage(packageName);
        if (pkg == null) {
            continue;
        }
        if (!isSysComponentOrPersistentPlatformSignedPrivApp(pkg)
                || !doesPackageSupportRuntimePermissions(pkg)
                || pkg.requestedPermissions.isEmpty()) {
            continue;
        }
				/* 给系统apk授运行时权限 */
        grantRuntimePermissionsForPackage(userId, pkg);
    }
}
```

```java
private void DefaultPermissionGrantPolicy::grantRuntimePermissionsForPackage(int userId, PackageParser.Package pkg) {
    Set<String> permissions = new ArraySet<>();
    for (String permission :  pkg.requestedPermissions) {
        final BasePermission bp = mPermissionManager.getPermission(permission);
        if (bp == null) {
            continue;
        }
        if (bp.isRuntime()) {
            permissions.add(permission);
        }
    }
    if (!permissions.isEmpty()) {
			  /* 授权为true */
        grantRuntimePermissions(pkg, permissions, true, userId);
    }
}
```

```java
private void grantRuntimePermissions(PackageParser.Package pkg, Set<String> permissions,
				boolean systemFixed, boolean ignoreSystemPackage, int userId) {
		if (pkg.requestedPermissions.isEmpty()) {
				return;
		}

		List<String> requestedPermissions = pkg.requestedPermissions;
    Set<String> grantablePermissions = null;

		final int grantablePermissionCount = requestedPermissions.size();
    for (int i = 0; i < grantablePermissionCount; i++) {
       String permission = requestedPermissions.get(i);
			 /* mServiceInternal为PackageManagerInternal对象，实际调用为PackageManagerService的grantRuntimePermission函数 */
			 mServiceInternal.grantRuntimePermission(pkg.packageName, permission, userId, false);

			 int newFlags = PackageManager.FLAG_PERMISSION_GRANTED_BY_DEFAULT;
			 if (systemFixed) {
			   newFlags |= PackageManager.FLAG_PERMISSION_SYSTEM_FIXED;
			 }

			 mServiceInternal.updatePermissionFlagsTEMP(permission, pkg.packageName,
			       newFlags, newFlags, userId);
    }
}
```

```java
public void PackageManagerInternalImpl::updatePermissionFlagsTEMP(String permName, String packageName, int flagMask,
        int flagValues, int userId) {
    PackageManagerService.this.updatePermissionFlags(
            permName, packageName, flagMask, flagValues, userId);
}
```
