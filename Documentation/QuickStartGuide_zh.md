# CnCNet Client 快速使用指南 #

以下是本客户端的使用说明。

1. 编辑 `Resources\GameCollectionConfig.ini`:
   1. 将字段 `CustomGame`下的 `InternalName`  改成你MOD的简称(不要超过4个字符)。
   2. 将字段 `CustomGame` 下的 `UIName` 改成你MOD的全称。
   3. 将字段 `CustomGame` 下的 `ChatChannel` 和 `GameBroadcastChannel` 改成有效的 IRC 频道名称. 例如, `ChatChannel`改成`#cncnet-X`  和 `GameBroadcastChannel`改成`#cncnet-X-games` , 这里请将 `X` 换成你对应的 `InternalName`。
      - **注意:** 如果你想注册一个自定义的频道名称, 你不能在你的频道名称里包含 `cncnet` 。 在这种情况下, 推荐使用您mod的全称来代替, 例如 `#customgame` / `#customgame-games`。
   4. 将字段 `CustomGame` 下的 `IconFilename`改成有效的 16x16像素的图标文件名用于在 CnCNet 大厅 & CnCNet 设置里显示。文件 (最好是 PNG 格式) 需要放在 `Resources` 文件夹。
2. 编辑 `Resources\ClientDefinitions.ini`:
   1. Set `LocalGame` under section `Settings` to match the abbreviation from custom game's `InternalGame`.
   2. Set `LongGameName` under section `Settings` to full name of your mod.
   3. Set `WindowTitle` under section `Settings` to what you wish to display in client window titlebar.
   4. *Optional:* If you are planning on setting up the updater, set `ModMode` under section `Settings` to `false`. For information on how to set up the updater, refer to the [updater documentation](Updater.md).
3. 编辑 `Resources\GameOptions.ini`:
   1. Change `Sides` list under section `General` to match the `Countries` list from your `rulesmd.ini`.
      - If you wish to list the `Countries` in different order between client and `rulesmd.ini`, use `InternalSideIndices` to override the used side (country) indices for `Sides` list to match the actual order in `rulesmd.ini`.
   2. Set up random selectors under section `RandomSelectors` to match the `Sides` list, or remove keys under the section if don't wish to use additional random side selectors.
   3. If you have changed in-game colors, or added more available multiplayer colors through use of Ares, adjust the colors list under section `MPColors` accordingly.
4. 编辑 `Resources\FHCConfig.ini`:
   1. Adjust the filenames list under section `FilenameList` accordingly if need to remove or add additional files that are compared between players to see if anyone might be cheating by modifying files. The list included by default works well if `expandmd99.mix` contains all INI files and a copy of `shroud.shp`. Certain client files like `FHCConfig.ini` itself are always included in the check even if absent from `FilenameList`.
   2. *Optional:* `CalculateGameExeHash` under section `Settings` can be set to `true` if it is guaranteed that the game executable (`gamemd.exe`) is identical between all players. With almost all C&C: Yuri's Revenge mods, as the game files are not distributed together with mods, this is not the case.
