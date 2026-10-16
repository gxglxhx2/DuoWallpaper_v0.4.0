# Wallpaper Worksheet · 0.3.1

修正 CloudSync 的两个编译错误：通知名称使用 Notification.Name.NSMetadataQueryDidFinishGathering 和 Notification.Name.NSMetadataQueryDidUpdate。

原生 iPhone 照片与视频壁纸编辑器。黑色主界面、银灰文字、蓝紫流光和细边面板。编辑器分为构图、调色、细节、导出四个面板。Emoji 放在细节工具中。

## 先在 iPhone 17 模拟器运行

1. 用 Xcode 打开 `DuoWallpaper.xcodeproj`。
2. 顶部 Scheme 选择 `DuoWallpaper`。
3. 运行目标选择具体的 `iPhone 17` 模拟器。不要选 `Any iOS Simulator Device`，它是通用编译目标，不能启动一个模拟手机。
4. 如果列表里没有具体手机，在运行目标列表的设备管理入口中添加模拟器，按提示下载 iOS runtime。
5. 点击 Run。要安装到真实 iPhone，在 Signing & Capabilities 中选择你的 Team，设置唯一 Bundle Identifier，并选择连接的手机运行。

最低系统为 iOS 16。iPhone 17 是默认壁纸画布，另有 iPhone 8 和 Duo 内外屏配置。iPhone 8 仅作为兼容配置加入，尚无该设备性能测试。

## 已写入源码的功能

| 模块 | 行为 |
| --- | --- |
| Photos | 用系统照片选择器选图片或视频，支持 iCloud Photos 中由系统下载的素材 |
| 文件导入 | 从 Files 导入静态图片、MP4 / MOV 视频 |
| 编辑 | 照片拖动与双指缩放、独立画布裁切、六种滤镜及强度、emoji 的位置 / 大小 / 旋转 |
| 视频 | 时间轴播放、精细选帧、取帧进入壁纸编辑、截取 0.5 至 15 秒原视频片段 |
| Photos 导出 | 当前壁纸、Duo 三张套装、视频片段，保存到系统照片库 |
| 项目 | 保存原图与参数为本地快照，重新打开编辑、删除 |
| iCloud 项目 | 可启用的 iCloud Drive 双向快照同步，下载云端项目、同步应用内删除标记 |
| Cosmos | NASA Image and Video Library 实时搜索 Hubble / 星云 / 星系、加载更多、下载图像进入编辑，保留来源与链接 |
| Liquid | 三个原创程序化主题，流动预览；选定时生成静态画面进入编辑 |
| 付费包 | StoreKit 2 商品查询、购买、验证、权益恢复、交易监听与退款后的权益刷新 |

图片处理和滤镜留在本机。NASA 搜索需要网络。iCloud 项目只在用户点击连接后启用；本地项目可以独立使用。

## 两种 iCloud 同步

### Photos 成品同步

导出图片与视频保存进 Photos。手机已经开启 iCloud Photos 时，系统负责将这些素材同步到同 Apple Account 的设备。App 不会替用户开启 iCloud Photos，也不能承诺即时完成上传。

https://support.apple.com/en-us/108782

### 可编辑项目同步

`CloudSync.swift` 使用 iCloud Documents、NSMetadataQuery 和 NSFileCoordinator。保存项目为不可变快照；应用内删除写入删除标记，防止另一设备再次导入已经删除的项目。首次连接、应用回到前台、保存或删除项目、云端文件变化及手动同步时会触发同步。连接偏好会保存在本机，同一 iCloud 账户下重启 App 会尝试重连；账户变化时断开并要求重新连接，避免自动向另一个账户提交项目。

实际启用步骤：

1. 在 Apple Developer 账号中启用 App ID 的 iCloud / iCloud Documents 能力，并建立相应 container。
2. 更新 Bundle Identifier 与 Team。
3. 在 Signing & Capabilities 配置 iCloud Documents，选择你的 container。`Configuration/Worksheet.entitlements` 是模板，确认其 container 标识与账号中的配置相同。
4. 使用 `Worksheet-iCloud` Scheme，它会应用 entitlements。普通 `DuoWallpaper` Scheme 不应用 iCloud 权限，方便先测试本地功能。
5. 在已登录 iCloud、开启 iCloud Drive 的两台测试设备上，进入 App 设置，点击连接 iCloud 项目。
6. 分别检查保存、下载、重新编辑、应用内删除、离线恢复以及账户切换。

源码无法替你的开发者账号创建 container 或签名权限。状态显示“已提交同步”表示已交给系统处理，不代表服务器上传已经完成。不要在 Files 中手动拆开删除一个项目的图片和 JSON；跨设备删除请使用 App 内的删除入口。

## 测试付费主题

免费 Carbon Current 可以直接使用。Liquid Collection 为一次购买的非消耗型主题包，包含 Blue Orbit 和 Violet Flux。购买后可生成并导出对应主题，恢复购买会重新验证权益。

Product ID：`studio.worksheet.liquidcollection`

本地测试：

1. Xcode 选择 Product → Scheme → Edit Scheme。
2. 在 Run → Options 中选择 `Configuration/Worksheet.storekit` 作为 StoreKit Configuration。
3. 运行 App，查看主题、模拟购买、恢复、退款和未验证交易。

配置中的 2.99 只是本地测试价格，不是已经创建的正式售价。正式发布需要在 App Store Connect 创建相同 Product ID 的商品、配置价格与可用地区，并做 Sandbox / TestFlight 验证。源码只接受经过 StoreKit 验证的权益，不使用本地按钮直接解锁。

https://developer.apple.com/documentation/storekit/transaction/currententitlements

## Cosmos 素材与付费内容

NASA 部分使用官方公开搜索与 asset manifest 接口，选择大尺寸 JPEG / PNG；不将科学照片虚构成实时星空。展示标题、来源、原始说明与链接。素材可用于壁纸构图，但它不是跟随用户位置变化的星图，也不是实时哈勃直播。

Cosmos 当前保持免费浏览与编辑。收费对象是原创 Liquid Collection。要另行出售 NASA / ESA / STScI 的图片主题包，需要逐张检查原图署名与使用条件，不能把整个影像库视作统一商业授权。

- 接口：https://images.nasa.gov/docs/images.nasa.gov_api_docs.pdf
- 素材说明：https://www.nasa.gov/nasa-brand-center/images-and-media/

## 动态功能的准确范围

视频可播放、选时间点取静态帧，并导出原视频的小片段。导出的片段保留原视频内容，不含编辑器中的 emoji 或图片滤镜。

Liquid 的流动是 App 内的原创主题预览，进入编辑器后得到静态照片。当前不生成 Live Photo，不自动设置系统壁纸，也不能让普通 MP4 在锁屏上持续播放。若后续要做 Live Photo 或系统空间墙纸，需要另外实现相应资源生成与真机验证。

所谓流光、空间感是视觉设计，不是四维科学数据。照片最长边按 4096 像素处理；PNG 壁纸为 SDR。低分辨率原图不会因为导出大尺寸而自动增加细节。

## 画布尺寸

| 画布 | 像素 |
| --- | --- |
| iPhone 17 | 1206 × 2622 |
| iPhone 8 | 750 × 1334 |
| Duo 外屏 | 1398 × 2034 |
| Duo 内屏竖向 | 1878 × 2670 |
| Duo 内屏横向 | 2670 × 1878 |

- https://www.apple.com/iphone-17/specs/
- https://www.apple.com/iphone-duo/specs/

适配是导出尺寸、重新构图和可变窗口布局。系统最终裁切、折叠屏切换、色彩和动态效果仍需要真机验证。

## 验证与启动排查

此版本已检查工程文件引用、scheme、plist / entitlements / StoreKit JSON、源码结构与压缩包。当前生成环境没有 macOS、Xcode 或 iOS SDK，没有执行 Swift 编译或真机测试；NASA 网络调用、iCloud 两设备同步和真实 StoreKit 购买也尚未实测。

- 按 Run 后没有模拟器：先确认目标是具体 iPhone，不是 Any iOS Simulator Device。
- 编译红色错误：在 Xcode Issue Navigator 复制第一条错误文本。
- iCloud 签名错误：先切换普通 `DuoWallpaper` Scheme；配置账号能力后再用 iCloud Scheme。
- 商品为空：先选择本地 StoreKit Configuration，或完成 App Store Connect 商品配置。

## 源码文件

- `DuoWallpaperApp.swift`：入口、编辑器、导入、手势和本地项目列表。
- `WallpaperModel.swift`：数据结构、兼容旧项目、滤镜与本地保存。
- `WallpaperCanvas.swift` / `WallpaperExport.swift`：画布与高清图片导出。
- `WorksheetDesign.swift`：界面风格、按钮、原创流光主题与主题详情。
- `ExploreView.swift`：精选、Cosmos 图库与设置。
- `VideoTools.swift`：视频选择、取帧与片段导出。
- `CloudSync.swift`：iCloud Documents 同步。
- `Purchases.swift`：StoreKit 2。
- `NASALibrary.swift`：NASA 接口与响应解析。
