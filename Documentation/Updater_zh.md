# 拓展型 CnCNet Client 更新器 #
本文档将说明如何使用包含在本存储库的客户端拓展型更新器。许多功能细节与原版更新器相同, 但是本更新器有更多独特的功能。


更新器相关文件
-------------------

**警告: 所有的示例文件都是不能拿来直接使用的，必须按本说明经过修改后才能正常使用!**

### 开发者文件（MOD开发者需要用的文件）
**这些文件仅对MOD开发者有用, 这意味着不需要分发给普通的玩家!** 
- **版本文件生成器(Version File Writer)**: 可以自动写入 `VersionConfig.ini` 的软件, 并能够根据需要自动的复制更新服务器所需的文件到当前工作文件的 `VersionWriter_CopiedFiles` 子目录 。
- **更新服务器脚本(Update Server Scripts)** (`preupdateexec` 和 `updateexec`): 脚本文件被用来 重命名, 移动或删除文件或目录。他们将在更新器开始更新和结束更新时分别被下载和运行。他们可以被放在与下载镜像定义相同的文件夹里。注意由 `preupdateexec` 脚本造成的文件改变**将无法** 被回复即使更新进程在后面因为某些原因失败。此外，无论当前本地版本或服务器版本状态和信息如何，都会执行这两个脚本。

### 分发文件（给玩家用的文件）
- **更新器配置文件(Updater Configuration File)** (`Resources/UpdaterConfig.ini`): 客户端 [更新器配置文件](#updater-configuration) 将设置更新器的下载镜像地址与组件的信息。需要 [支持拓展更新功能的客户端](#client-support). 如果找不到该文件, 客户端将继续使用原版客户端默认定义的 `updateconfig.ini` 。
- **二级更新器(Second-Stage Updater)** (`clientupdt.dat`): 二级更新器将在下载完所有文件后将其复制到正确的位置, 然后在复制完后自动重启客户端。通常, 他会寻找一个硬编码的启动器执行名称列表, 如果无法寻找到他将不会自动重启客户端。本程序将从文件 `Resources/ClientDefinitions.ini` 的 `LauncherExe` 键读取未被硬编码的启动器执行名称。

基本用途
-----------

## 快速指南
1. 配置一个网络服务器并创建好一个可以被公开访问的目录（用于存放更新游戏所需的文件）。
2. 在支持拓展个更新功能的客户端上配置好文件, 添加上面可用的下载镜像URL到 `Resources/UpdaterConfig.ini`。
3. 制作好要变更的文件和和 `VersionConfig.ini`。
4. 运行`VersionWriter.exe`。
5. 将 `VersionWriter_CopiedFiles` 里的内容和[更新服务器脚本](../Miscellaneous/UpdateServerScripts)上传到上述的网络服务器目录。

## 详细说明
To have automatic updates via XNA CnCNet client, an update server needs to be set up. The update server needs to be a web server with the files accessible through HTTP (**不是HTTPS**, 除非你想排除对Windows XP系统的支持), which would then allow them to be downloaded by client during the update process. The URL path to the file (sans update location part) has to replicate the local path to the file relative to mod folder in order to be succesfully downloaded (for example, with update location `http://your.test/location/of/updates/` the file `Resources/clientdx.exe` would need to be accessible at `http://your.test/location/of/updates/Resources/clientdx.exe` URL). Besides the update server scripts, the updater does not explicitly require any other files or specific software to exist or run on the update web server.

To set up an update information needed to produce the files to upload on a server edit `VersionConfig.ini` file to include all of the redistributed files (or updated files only if you're saving on bandwidth and don't want to allow full downloads). Each time you need to push an update to your players (also if you change something in `VersionConfig.ini`) you have to change the version key under `[Version]` section in aforementioned configuration file so the CnCNet client prompts for an update. In case you need to force users to download an update manually you can change a key under `[UpdaterVersion]` section. After that run `VersionWriter.exe` and upload the contents of the `VersionWriter_CopiedFiles` to your update server along with updater scripts.

Refer below for a more comprehensive explanation of both version writer's and updater's features & configuration files.


功能
-------

### 版本文件生成器
The example `VersionConfig.ini` included with the version writer contains comments explaining most of the functionality and features.

`VersionWriter.exe` accepts a single command-line argument that can be used to set its working directory - this allows running VersionWriter from outside the mod directory itself.

#### Options
These are set under `[Options]` in `VersionConfig.ini`.
- `EnableExtendedUpdaterFeatures`: If set, enables the extended updater features such as compressed archives, updater version and manual download URL.
- `RecursiveDirectorySearch`: If set, will go through every subdirectory recursively for directories given in `[Include]`.
; If set, will always create two version files - one with everything included (version_base) and the proper, actual version file with only changed files (version). 
; version_base should be kept around as it is used to compare which files have been changed next time VersionWriter is ran.
- `IncludeOnlyChangedFiles`: If set, version file writer will always create two version files - one with everything included (`version_base`) and the proper, actual version file with only changed files (`version`). Note that `version_base` should be kept around as it is used to compare which files have been changed next time version file writer is ran.
- `CopyArchivedOriginalFiles`: If set, original versions of archived files will also be copied to copied files directory.

#### 更新器版本 & 手动下载地址
Setting `[UpdaterVersion]` in `VersionConfig.ini` writes this information to the `version` file and allows developers to control which versions are allowed to download files from the version info through the client. Mismatching updater versions between local and server version files will suggest users to download update manually through updater status message. Absent or malformed updater version (both local & server) is equivalent to `N/A` and updater will bypass the mismatch check entirely if server  updater version is set to this or absent.

Additionally setting `[ManualDownloadURL]` will, in addition to displaying the updater status message, also bring up a notification dialog with the provided URL as a download link in case a updater version mismatch occurs.

#### 压缩文件
The extended updater supports downloading and uncompressing LZMA-compressed data archives. Files that are to be compressed should be included under `[ArchiveFiles]` in `VersionConfig.ini`. Note that they still need to be included through `[Include]` in the first place. As a result there would be information in the `version` file which allows the client, assuming the modified updater is in use, to figure out it is supposed to download the archive instead, and instead of the original files the compressed files with `.lzma` extension are placed to the `VersionWriter_CopiedFiles` folder.

#### 自定义组件
Custom components are available even with the original XNA CnCnet Client, but since the IDs and filenames are hardcoded in the updater, their usage is limited. Custom component info for the updater can be set in `Resources/UpdaterConfig.ini`, see below for more info. For version file writer, any custom components should be included under `[AddOns]`, using syntax `ID=filename` as shown in the example `VersionConfig.ini`. Custom component filenames **should not** be listed under `[Include]`. The filenames can be listed under `[ArchiveFiles]` to enable use of compressed archives.

- Custom component download file path (in `Resources/UpdaterConfig.ini`) accepts absolute URLs and uses them properly, so it's possible to define custom components which have to be downloaded from elsewhere.

### 更新器配置
The example `Resources/UpdaterConfig.ini` included with client files contains comments explaining most of the functionality and features.

Only currently supported global updater setting under `[Settings]` is `IgnoreMasks` that allows customizing the list filenames that are exempted from file integrity checks even if they are included in `version` file.

#### Download Mirrors

List of available download mirrors from which to download version info and files from. Listed as comma-separated values under `[DownloadMirrors]`, containing URL, UI display name and location. Location is optional and can be omitted.

Omitting the download mirrors list will default to the hardcoded list for currently set mod / game if available. Updater and Updater & Component options in client options will be unavailable if no download mirrors are found.

#### 自定义组件
List of custom components available for the updater. Listed as comma-separated values under `[CustomComponents]`, containing custom component ID used in the `version` file, download path / URL, local filename, flag that disables archive file extensions for download path / URL.

Download path / URL supports absolute URLs, allowing custom components to be downloaded from location outside the current update server but also restricts it to one download location instead of one per each download mirror.

Download path archive file extension disable flag is a boolean value (yes/no, true/false), is optional and defaults to false.

Omitting the custom components list will default to the hardcoded list for currently set mod / game if available. Custom components and the Components tab in client options will be unavailable if no custom component info is found.


支持的客户端
--------------

推荐使用以下几种客户端来使用此拓展更新功能:
- 包含此功能的客户端存储库 - 源代码可以查看 [这里](https://github.com/Starkku/xna-cncnet-client/tree/modified-updater)
- [Kerbiter的自定义客户端](https://github.com/Metadorius/xna-cncnet-client)

The extended updater library (`DTAUpdater.dll`) should theoretically be backwards compatible with all modern XNA CnCNet client variants, though not all of the extended updater features will be available. Not using it with a corresponding client version is generally speaking advised against and not really supported in any real capacity.
