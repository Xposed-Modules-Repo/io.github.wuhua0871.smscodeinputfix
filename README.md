# 验证码输入修复 / SmsCode Input Fix

为旧版 XposedSmsCode 2.5.1 在 Android 15 上恢复被系统权限检查拒绝的验证码模拟输入。
保留原版短信识别、通知、自动输入开关、延迟和屏蔽应用逻辑。

版本：**1.0.0**（versionCode **1**）  
包名：`io.github.wuhua0871.smscodeinputfix`

## 支持范围

- **仅 Android 15 / API 35**。安装最低版本为 35；运行时也检查版本必须等于 35。Android 16 及其他版本不加载修复，不能将最低安装版本理解为支持所有更高系统。
- 需要 Root 和能够注入「系统框架」的 Xposed 兼容框架，使用传统 Xposed API 82。
- 面向原版 **XposedSmsCode 2.5.1 / versionCode 52**，原应用包名 `com.github.tianma8023.xposed.smscode`。
- 既有自用版实测环境：Pixel 8 Pro、Android 15、Vector 2.2（3080）。其他 ROM、框架及新版 SmsCode 未实测，尚无独立 LSPosed 实机测试记录。

## 安装与启用

1. 保留原版短信验证码应用及其数据，安装 `SmsCodeInputFix-1.0.0.apk`。
2. 在框架管理器中启用「验证码输入修复」，作用域**只勾选「系统框架」**。管理器可能显示为 `system`；模块声明使用 `android`。
3. 保持原短信验证码模块启用，并按其原有要求配置作用域、自动输入开关及屏蔽应用。
4. 重启手机，首次解锁。收到验证码前让目标验证码输入框获得焦点。

每次开机由框架加载 Hook；随后在系统输入检查时生效。没有桌面入口、设置页、独立后台服务或定时任务。系统进程会保留少量模块代码，不能理解为绝对零内存或零运行开销。无需为本补丁另行配置自启动或加入后台休眠工具。

## 从自用版迁移

旧包名 `local.smscode.inputfix` 与投稿版是两个独立模块，不能覆盖安装。先在框架中停用旧模块，再启用投稿版及「系统框架」作用域，然后重启。确认工作正常后可卸载旧补丁。**不要同时启用两版；不要卸载或清除原版短信验证码应用。**

已有自用版正常工作的用户无需切换。本次投稿资料准备不替换手机上已安装的模块。

## 实现与授权边界

模块 Hook 系统进程的 `com.android.server.input.InputManagerService.checkCallingPermission(String, String, boolean)`，仅当以下条件同时满足时返回允许：

- Binder 调用者 UID **1001**（主用户 radio / 电话服务身份）；
- 检查权限为 `android.permission.INJECT_EVENTS`；
- 操作名称为 `injectInputEvent()`，且第三个参数为 `true`。

其他 UID、其他权限和其他操作保留原检查。本模块不向系统权限数据库永久授予权限。

**这是按 UID 放行模拟输入，不是按验证码内容授权。**所有共享 UID 1001 的进程都符合身份条件；补丁不校验具体进程名、调用来自哪个模块、输入内容或目标输入框，也不检查原 SmsCode 是否安装或启用。即使原 SmsCode 的自动输入关闭，本补丁的放行规则仍存在；只是不再由原模块发起自动输入。其他普通应用 UID 不因本补丁获得输入权限。仅在信任电话服务及所注入模块的设备上启用。

## 隐私与资源

本补丁不申请 Android 权限，不读取短信、验证码、剪贴板或用户配置，不联网、不收集或上传数据。它不改变原应用自身的短信访问行为。框架日志只记录 Hook 安装情况和首次命中，不记录验证码内容。原应用继续负责识别、记录和通知；补丁不修改原版 APK 或历史数据库。

## 验证与限制

此前自用版已完成重启加载和相同 Binder UID 的真机输入验证：UID 1001 从 `INJECT_EVENTS` 权限拒绝恢复为成功，固定数字码 314159、271828 和追加 161803 在独立测试输入框中核对；普通应用 UID 10307 仍被拒绝。原应用设置修复前后保持一致，用户随后确认问题已修复。

**本次 1.0.0 投稿 APK 仅调整包名、版本文字和作者署名，保留原 Hook 逻辑。**已完成构建、签名、作用域条件及 APK 元数据校验，尚未安装投稿包进行实机复测，也未触发真实短信登录。历史自用版测试不能冒充投稿包的完整实机验证。

补丁不解决短信识别失败、无输入焦点、应用专有键盘或其他输入兼容问题。含字母输入在当前 Fcitx 输入法中可能留在候选栏，需要确认候选后提交。其他系统版本不加载修复，其他 ROM 的同名接口也不保证行为一致。升级系统或框架后应重新确认兼容性。

## 停用与回退

在框架中停用或卸载补丁，然后重启，已加载的 Hook 才会撤销。原版验证码应用、设置和记录保留。回退自用版时先停用投稿版，再启用旧版及原作用域并重启。

## 项目与参考

这是独立的第三方兼容补丁，与 XposedSmsCode 原作者、Google、LSPosed 或 Vector 官方无隶属关系。本模块不替代完整的验证码应用，也不分发原版应用 APK。

- 原版应用：[tianma8023/XposedSmsCode](https://github.com/tianma8023/XposedSmsCode)。
- 对应系统接口：[AOSP Android 15 InputManagerService](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/android15-release/services/core/java/com/android/server/input/InputManagerService.java)。

## 作者与反馈

作者：**wuhua0871**  
GitHub：[wuhua0871](https://github.com/wuhua0871)  
邮箱：[wuhua0871@live.cn](mailto:wuhua0871@live.cn)  
博客：[wuhua0871.com](https://wuhua0871.com)  
资源站：[app.wuhua0871.com](https://app.wuhua0871.com)

反馈请附手机型号、Android 版本、原短信验证码模块版本、框架版本及复现步骤。请删除日志中的验证码、手机号、短信正文等个人信息。
