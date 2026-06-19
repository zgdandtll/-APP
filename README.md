# 手机鼠标 (Phone Mouse)

把 Android 手机变成一个**蓝牙鼠标**：手机注册成标准的蓝牙 HID 设备，电脑像连接普通蓝牙鼠标一样配对它即可使用，**电脑端无需安装任何软件或驱动**。

支持功能：
- 触摸板单指拖动 → 移动光标
- 单指点按 → 左键单击
- 双指点按 → 右键单击
- 双指上下滑动 → 滚轮
- 屏幕底部「左键 / 右键」按钮（支持按住，可做拖拽框选）

## 关于连接方式的说明

需求里写的是「手机和电脑用 USB 线连接，鼠标通过蓝牙连接电脑」。
实际上 Android 应用**无法**把手机的 USB 口变成 HID 鼠标设备（那需要内核 gadget/configfs 权限，普通手机不开放给应用层）。
而 Android 9 (API 28) 起提供了 `BluetoothHidDevice` 接口，可以让手机注册成系统级的蓝牙鼠标——这正好满足「鼠标通过蓝牙连接电脑」这一核心要求，且电脑端免驱。
因此本工程采用**蓝牙 HID** 方案。USB 线此时仅用于给手机供电（可选）。

## 运行要求

- Android 9 (API 28) 及以上的真机（**必须真机**，模拟器没有蓝牙 HID 能力）
- 电脑支持蓝牙
- Android Studio (Giraffe / 2023.3 或更新)，JDK 17

## 构建与运行

1. 用 Android Studio 打开本目录（`File > Open`，选 `phone_mouse`）。
   首次打开时 Android Studio 会自动补全 Gradle Wrapper 并下载依赖。
   > 命令行用户：装好 Gradle 后先执行 `gradle wrapper`，再 `./gradlew assembleDebug`。
2. 用 USB 线把手机连到电脑，手机开启「开发者选项 → USB 调试」。
3. 点 Android Studio 的 ▶ Run，把 App 装到手机上。

## 使用步骤

1. 打开手机上的「手机鼠标」App，授予蓝牙权限。
2. 顶部状态显示「已就绪」后，在**电脑**的蓝牙设置里搜索设备，
   会出现一个名为 **Phone Mouse** 的鼠标，点击配对。
3. 配对成功后 App 状态变为「已连接 ✓」，即可用触摸板和按钮操作电脑。

## 代码结构

| 文件 | 职责 |
|------|------|
| `HidConstants.kt` | 鼠标 HID 报告描述符与 4 字节报告的封装 |
| `BluetoothHidManager.kt` | 注册/注销蓝牙 HID Profile、维护连接、发送报告 |
| `TouchpadView.kt` | 触摸板手势识别（移动 / 点按 / 双指滚动） |
| `MainActivity.kt` | 权限申请、UI、把手势和按钮事件转成 HID 报告 |

## 已知限制

- 蓝牙鼠标功能依赖具体手机厂商对 `BluetoothHidDevice` 的实现，绝大多数主流机型可用；
  个别定制 ROM 可能限制该 Profile。
- 同一时间手机只能作为一个 HID 主机的鼠标连接。
