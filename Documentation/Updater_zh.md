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
1. 配置一个 Web 服务器并创建好一个可以被公开访问的目录（用于存放更新游戏所需的文件）。
2. 在支持拓展个更新功能的客户端上配置好文件, 添加上面可用的下载镜像URL到 `Resources/UpdaterConfig.ini`。
3. 制作好要变更的文件和和 `VersionConfig.ini`。
4. 运行`VersionWriter.exe`。
5. 将 `VersionWriter_CopiedFiles` 里的内容和[更新服务器脚本](../Miscellaneous/UpdateServerScripts)上传到上述的网络服务器目录。

## 详细说明
为了能通过 XNA CnCNet client 进行更新, 需要搭建一个更新服务器。这个更新服务器需要是一个能创建HTTP访问的公开目录的 Web 服务器(**不是HTTPS**, 除非你想排除对Windows XP系统的支持), 这样就能允许客户端在更新时从这个服务器下载更新所需的文件。文件的 URL 路径（无更新位置部分）必须复制相对于 mod 文件夹的文件的本地路径才能成功下载 (例如, 更新路径是 `http://your.test/location/of/updates/` 其中文件是 `Resources/clientdx.exe` 应该被设置成`http://your.test/location/of/updates/Resources/clientdx.exe` URL)。除了更新服务器脚本之外，更新程序不要求在更新 Web 服务器上存在或运行任何其他文件或特定软件。

要生成要上传到服务器上的文件所需的更新信息，请编辑“VersionConfig.ini”文件以包含所有重新分发的文件（或仅当您节省带宽并且不想允许完整 下载）。 每次您需要向播放器推送更新时（如果您在 `VersionConfig.ini` 中更改某些内容），您必须更改上述配置文件中 `[Version]` 部分下的版本密钥， . 如果您需要强制用户手动下载更新，您可以更改“[UpdaterVersion]”部分下的密钥。 之后运行“VersionWriter.exe”并将“VersionWriter_CopiedFiles”的内容与更新程序脚本一起上传到您的更新服务器。

要生成要上传到服务器上的文件所需的更新信息, 需要编辑 `VersionConfig.ini` 文件以包含所有重新分发的文件 (或仅当您节省带宽并且不想允许完整下载)。每次您需要向您的玩家推送更新时(包括您想在 `VersionConfig.ini` 更改某些内容时) 您必须更改上述配置文件中 `[Version]` 部分下的版本密钥以便 CnCNet 客户端提示更新。如果您需要强制用户手动下载更新, 您可以更改节 `[UpdaterVersion]` 部分下的密钥。然后再运行 `VersionWriter.exe` 并上传 `VersionWriter_CopiedFiles` 里的内容和更新服务器脚本。

有关版本文件生成器和更新器的功能和配置文件的更全面说明，请参阅下文。


功能
-------

### 版本文件生成器
版本文件生成器中包含的示例 `VersionConfig.ini` 包含解释大部分功能和特性的注释。

`VersionWriter.exe` 允许使用单个命令行参数设置其工作目录 - 这允许从 mod 目录本身之外运行版本文件生成器。

#### 选项
这些选项在 `VersionConfig.ini` 文件下的 `[Options]` 进行设置。
- `EnableExtendedUpdaterFeatures`: 如果设置，则启用扩展更新程序功能，例如压缩档案、更新版本和手动下载 URL。
- `RecursiveDirectorySearch`: 如果设置，将递归遍历每个`[Include]` 中给出的子目录。
- `IncludeOnlyChangedFiles`: 如果设置，将始终创建两个版本文件 - 一个包含所有内容(`version_base`)和正确的版本文件，实际的版本文件将仅用来更改文件(`version`)。
注意version_base 应该保留，因为可以用于比较下次运行 VersionWriter 时哪些文件已更改。
- `CopyArchivedOriginalFiles`: 如果设置，档案文件的原始版本也将复制到复制文件目录。

#### 更新器版本 & 手动下载地址
在 `VersionConfig.ini` 中设置 `[UpdaterVersion]` 会将信息写入 `version` 文件并允许开发人员通过客户端版本信息来控制允许从下载文件的版本。本地和服务器版本文件的更新程序版本不匹配时, 将通过更新程序通知对话框建议用户手动下载更新。不存在或格式错误的更新程序版本（本地和服务器）等效于 `N/A` ，如果服务器更新程序版本设置为此或不存在，更新程序将完全绕过不匹配检查。

此外，设置 `[ManualDownloadURL]` , 除了显示更新程序状态消息外，还会显示一个通知对话框，其中提供的 URL 作为下载链接，以防更新程序版本不匹配。

#### 压缩档案
扩展更新程序支持下载和解压 LZMA 压缩的数据档案。要压缩的文件应包含在 `VersionConfig.ini` 中的 `[ArchiveFiles]` 下。注意, 它们仍然需要在最先通过 `[Include]` 包含。因此, `version` 文件中会包含信息（假设你使用了此更新器）, 用于允许客户端确定它应该下载档案文件，而不是位于 `VersionWriter_CopiedFiles` 文件夹的 `.lzma` 拓展名原始文件。

#### 自定义组件
即使使用原始的 XNA CnCnet 客户端也可以使用自定义组件，但由于 ID 和文件名在更新器中是硬编码的，因此它们的使用受到限制。更新器的自定义组件信息可以在 `Resources/UpdaterConfig.ini` 中进行设置, 更多信息见下文。对于版本文件生成器, 任何自定义组件都应被包含在 `[AddOns]` 中, 使用语法为 `ID=filename` 如在 `VersionConfig.ini` 中示例所示。自定义组件的文件名 **不应** be 被列于 `[Include]` 下。但可以被列于 `[ArchiveFiles]` 下以使用压缩档案。

- 自定义组件下载文件路径 (位于 `Resources/UpdaterConfig.ini`) 接受绝对 URL 并正确使用它们，因此可以定义必须从其他地方下载的自定义组件。

### 更新器配置
版本文件生成器中包含的示例 `VersionConfig.ini` 包含解释大部分功能和特性的注释。

当前仅在 `[Settings]` 下支持的全局更新程序设置是 `IgnoreMasks` 的情况下, 它允许自定义从文件完整性检查中免除的列表文件名, 即使它们包含在 `version` 文件中。

#### 下载镜像
版本信息和文件的可用下载镜像列表。 在 `[DownloadMirrors]` 下以逗号分隔值列出，包含 URL、UI 显示名称和位置。 位置是可选的，可以省略。

省略下载镜像列表将默认为当前设置的模组/游戏的硬编码列表（如果可用的话）。如果找不到下载镜像，客户端选项中的更新和组件选项将不可用。

#### 自定义组件
可用于更新程序的自定义组件列表。 在 `[CustomComponents]` 下以逗号分隔值列出，包含在 `version` 文件中使用的自定义组件 ID、下载路径/URL、本地文件名、禁用下载路径/URL 的存档文件扩展名的标志。

下载路径/URL 支持绝对 URL，允许从当前更新服务器之外的位置下载自定义组件，但也将其限制为一个下载位置，而不是每个下载镜像一个。

下载路径存档文件扩展名禁用标志是一个可选的布尔值 (yes/no, true/false), 默认值是 false。

省略自定义组件列表将默认为当前设置的模组/游戏的硬编码列表（如果可用的话）。 如果未找到自定义组件信息，则自定义组件和客户端选项中的组件选项卡将不可用。 


支持的客户端
--------------

推荐使用以下几种客户端来使用此拓展更新功能:
- 包含此功能的客户端存储库 - 源代码可以查看 [这里](https://github.com/Starkku/xna-cncnet-client/tree/modified-updater)
- [Kerbiter的自定义客户端](https://github.com/Metadorius/xna-cncnet-client)
- [译者本地化的中文Starkku客户端（目前未完成请暂时使用上面两个）](https://github.com/KuroNoSeiHai/xna-cncnet-client/tree/modified-updater)

扩展更新程序库（`DTAUpdater.dll`）理论上应该与所有现行的 XNA CnCNet 客户端变体向后兼容，尽管并非所有扩展更新程序功能都可用。不过不建议将它与不相应的客户端版本一起使用，因为没有实际被支持。

