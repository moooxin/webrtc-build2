# 开发者文档

## 本地构建

使用 `run.py` 可以本地构建 WebRTC。

使用以下命令进行构建：

```
python3 run.py build <target>
```

`<target>` 部分可以填入 `windows` 或 `ubuntu-20.04_x86_64` 等目标名称。

详细信息请参考 `python3 run.py --help` 或 `python3 run.py build --help`。

这样会在 `_build` 目录下生成 `libwebrtc.a` 等库文件。

首次执行 build 命令时，会自动下载 WebRTC 的源代码和工具，并应用补丁后进行构建。

第二次执行 build 命令时，只会进行构建。不会更新 WebRTC 源代码或重新执行 gn gen。

更详细地说，不带选项参数执行 build 命令时，会执行以下操作：

- 如果所需的 WebRTC 源代码和工具不存在，则下载并应用时雨堂补丁
- 如果还没有 ninja 文件，则使用 gn gen 命令生成 ninja 文件
- 使用 ninja 命令进行构建

第二次执行时，因为 WebRTC 的源代码、工具和 ninja 文件已经存在，所以不会重新获取或生成，只会进行构建。

如果手动修改了 WebRTC 的源代码，只需再次执行 build 命令即可。

### --webrtc-gen

同样，如果需要重新执行 gn gen 命令，可以使用 `--webrtc-gen` 参数。

```
python3 run.py build <target> --webrtc-gen
```

这样会重新执行 gn gen，然后进行构建。

另外，还有一个 `--webrtc-gen-force` 参数，可以强制删除所有现有的构建目录并重新生成。

### iOS, Android 的构建

iOS 的 `WebRTC.xcframework` 和 Android 的 `webrtc.aar` 也可以像其他情况一样通过 build 命令生成。

但是 `--webrtc-gen` 命令不生效，总是会执行 gn gen。

另外，如果只需要 iOS 或 Android 的 `libwebrtc.a`，而不需要生成 `WebRTC.xcframework` 或 `webrtc.aar`，那么使用 `--webrtc-nobuild-ios-framework` 或 `--webrtc-nobuild-android-aar` 参数即可。

### 目录结构

- 源代码放在 `_source` 目录下，构建产物放在 `_build` 目录下。
- 像 `_source/<target>/` 和 `_build/<target>/` 这样，`_source` 和 `_build` 都会根据目标分别放在不同的目录中。
- 像 `_build/<target>/<configuration>` 这样，`_build` 会根据是调试构建还是发布构建分别放在不同的目录中。

也就是说，默认情况下会有如下的布局。

```
webrtc-build/
|-- _source/
|   |-- ubuntu-20.04_x86_64/
|   |   |-- depot_tools/...
|   |   `-- webrtc/...
|   `-- android/
|       |-- depot_tools/...
|       `-- webrtc/...
`-- _build/
    |-- ubuntu-20.04_x86_64/
    |   |-- debug/
    |   |   `-- webrtc/...
    |   `-- release/
    |       `-- webrtc/...
    `-- android/
        |-- debug/
        |   `-- webrtc/...
        `-- release/
            `-- webrtc/...
```

另外，可以通过以下选项指定源代码目录和构建目录的位置。

- `--source-dir`: 源代码目录
  - 默认是 `<run.py所在的目录>/_source/<目标名称>`
- `--webrtc-source-dir`: 存放 WebRTC 源代码的目录。优先级高于 `--source-dir`。
  - 默认是 `<source-dir>/webrtc`
- `--build-dir`: 构建目录
  - 默认是 `<run.py所在的目录>/_build/<目标名称>/<配置>`
- `--webrtc-build-dir`: 存放 WebRTC 构建产物的目录。优先级高于 `--build-dir`。
  - 默认是 `<build-dir>/webrtc`

这些目录可以指定为相对于当前目录的相对路径。

### 限制

本地构建有以下限制：

- Windows 只能构建 `windows` 目标。
- macOS 只能构建 `macos_x86_64`, `macos_arm64`, `ios` 目标。
- Ubuntu 的 x86_64 环境只能构建上述以外的目标。
  - `android`, `raspberry-pi-os_armv*`, `ubuntu-*_armv8` 等 ARM 环境无论 Ubuntu 版本如何都可以构建。
  - `ubuntu-18.04_x86_64` 需要 Ubuntu 18.04
  - `ubuntu-20.04_x86_64` 需要 Ubuntu 20.04
- 非 x86_64 的 Ubuntu 环境无法构建。
- 非 Ubuntu 的 Linux 系统无法构建。

## 获取源代码

如果需要重新从仓库获取 WebRTC 的源代码，可以使用 `fetch` 命令。

```
python3 run.py fetch <target>
```

这样 WebRTC 的源代码会根据 `VERSION` 文件中的 `WEBRTC_COMMIT` 更新，并应用补丁。不会进行构建。

请注意，这会将所有手动修改的部分和添加的文件全部还原。

## 还原编辑过的源代码

如果需要还原 WebRTC 的源代码或重新应用补丁，可以使用 `revert` 命令。

```
python3 run.py revert <target>
```

这会对所有相关的仓库执行 `git clean -df` 和 `git reset --hard`。

另外，编辑补丁时可以使用 `--patch` 命令。

```
python3 run.py revert <target> --patch <patch>
```

这样会将此补丁之前应该应用的补丁应用/提交后，将此补丁应用，但不提交。

如果希望提交指定 `--patch` 选项的补丁的提交，可以指定 `--commit` 选项。

```
python3 run.py revert <target> --patch <patch> --commit
```

这样会将此补丁及之前的所有补丁应用/提交。

`--commit` 选项可以在想要改变补丁应用顺序时使用。

## 输出 libwebrtc 的源代码差异

如果需要确认 WebRTC 源代码的差异，可以使用以下命令：

```
python3 run.py diff <target>
```

## 创建补丁

创建新补丁时，按照以下步骤操作：

1. 使用 `python3 run.py revert <target>` 命令清理源代码
2. 编辑 libwebrtc 的源代码
3. 使用 `python3 run.py diff <target>` 命令确认差异后，如果没有问题则使用 `python3 run.py diff <target> > <patch>` 将补丁输出到文件
4. 在 run.py 的 PATCHES 中添加新补丁
5. 使用 `python3 run.py revert <target>` 命令确认补丁是否正确应用

上述方法是在添加的补丁最后应用的情况下进行的。

如果想要在补丁中间应用新的补丁，可以使用 `--patch` 和 `--commit` 选项。

1. 使用 `python3 run.py revert <target> --patch <patch> --commit` 命令，先应用想要新创建补丁之前想要应用的补丁
2. 编辑 libwebrtc 的源代码
3. 使用 `python3 run.py diff <target>` 命令确认差异后，如果没有问题则使用 `python3 run.py diff <target> > <patch>` 将补丁输出到文件
4. 在 run.py 的 PATCHES 中，在最初添加的补丁之后的位置添加新补丁
5. 使用 `python3 run.py revert <target>` 命令确认补丁是否正确应用
  - 由于应用顺序变化，后续补丁应用可能会出现错误

## 编辑补丁

编辑现有补丁时，按照以下步骤操作：

1. 使用 `python3 run.py revert <target> --patch <patch>` 命令，使此补丁应用但未提交
2. 编辑 libwebrtc 的源代码使其正确
3. 使用 `python3 run.py diff <target>` 命令确认差异后，如果没有问题则使用 `python3 run.py diff <target> > <patch>` 覆盖补丁
4. 使用 `python3 run.py revert <target>` 命令确认补丁是否正确应用

## 修正错误的补丁

基本上和编辑补丁的步骤相同。

```bash
# 检出出错的分支
git checkout feature/<libwebrtc-version>
# 获取出错的版本
python3 run.py fetch <target>
```

这个 `fetch` 的前提是假设在某个补丁应用时出错。

1. 确认出错的补丁文件，执行 `python3 run.py revert <target> --patch <patch>` 命令
  - 这时可能会出现错误，但不用理会，继续下一步
2. 编辑 libwebrtc 的源代码使其正确
  - 查看原始补丁文件的差异，考虑如何修改
  - 有时可能需要查看旧版本的源文件，虽然本地下载很麻烦，但可以从 https://source.chromium.org/chromium 等网站查找
3. 使用 `python3 run.py diff <target>` 命令确认差异后，如果没有问题则使用 `python3 run.py diff <target> >

