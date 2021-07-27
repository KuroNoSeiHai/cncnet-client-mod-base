# CnCNet Client 快速使用指南 #

以下是本客户端的使用说明。

1. 编辑 `Resources\GameCollectionConfig.ini`:
   1. 将节 `CustomGame`下的 `InternalName`  改成你MOD的简称(不要超过4个字符)。
   2. 将节 `CustomGame` 下的 `UIName` 改成你MOD的全称。
   3. 将节 `CustomGame` 下的 `ChatChannel` 和 `GameBroadcastChannel` 改成有效的 IRC 频道名称. 例如, `ChatChannel`改成`#cncnet-X`  和 `GameBroadcastChannel`改成`#cncnet-X-games` , 这里请将 `X` 换成你对应的 `InternalName`。
      - **注意:** 如果你想注册一个自定义的频道名称, 你不能在你的频道名称里包含 `cncnet` 。 在这种情况下, 推荐使用您mod的全称来代替, 例如 `#customgame` / `#customgame-games`。
   4. 将节 `CustomGame` 下的 `IconFilename`改成有效的 16x16像素的图标文件名用于在 CnCNet 大厅 & CnCNet 设置里显示。文件 (最好是 PNG 格式) 需要放在 `Resources` 文件夹。
2. 编辑 `Resources\ClientDefinitions.ini`:
   1. 将节 `Settings` 下的 `LocalGame` 与你刚才的 `InternalGame`相匹配。
   2. 将节 `Settings` 下的 `LongGameName` 改成你MOD全称。
   3. 将节 `Settings` 下的 `WindowTitle` 改成你想显示的客户端名称。
   4. *可选:* 如果你准备配置更新器相关的功能, 请将节`Settings` 下的 `ModMode` 设置成 `false`.。更多有关使用更新器功能的信息, 请查阅 [更新器文档](Updater_zh.md).
3. 编辑 `Resources\GameOptions.ini`:
   1. 将节 `General` 下的 修改 `Sides` , 使其匹配你的 `rulesmd.ini` 中的 `Countries` 列表
      - 如果你想让你客户端的国家列表与你 `rulesmd.ini`中的顺序不同, 请使用 `InternalSideIndices` 来重写你的 `Sides` 中的阵营(国家) 索引 来匹配你 `rulesmd.ini`中的顺序。
   2. 在节 `RandomSelectors` 配置好匹配 `Sides` 列表的随机国家选择器, 如果你不想使用随机国家选择器, 请移除这个键。
   3. 如果你想变更游戏内颜色, 或者想通过 Ares 添加新的玩家颜色, 请按找你的颜色设置来修改节 `MPColors` 下的内容。
4. 编辑 `Resources\FHCConfig.ini`:
   1. 调整节 `FilenameList` 下的文件名列表, 此文件名列表用于比对玩家之间的文件以此来检测作弊。 这个列表也可以检测比如放在 `expandmd99.mix` 里的所有 INI files 和 `shroud.shp`等。即使是你从 `FilenameList`设置了排除, 客户端本身的文件例如 `FHCConfig.ini` 仍然会被检测。
   2. *可选:* 节 `Settings` 下的`CalculateGameExeHash` 可以被设置成 `true` 以此来保证游戏主程序 (`gamemd.exe`) 在玩家之间是相同的。 大多数尤里的复仇mod, 游戏文件是不会包含在mod文件的, 不会出现这个问题。
