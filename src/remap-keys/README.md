<!-- vim-markdown-toc GFM -->

* [Remap Keys for Windows](#remap-keys-for-windows)
    * [通过修改注册表重映射键位](#通过修改注册表重映射键位)
        * [重映射说明](#重映射说明)
        * [重映射步骤](#重映射步骤)
        * [取消重映射](#取消重映射)
        * [注册表导出与导入](#注册表导出与导入)
            * [导出](#导出)
            * [导入](#导入)
        * [键位与键码对应表](#键位与键码对应表)
        * [参考](#参考)

<!-- vim-markdown-toc -->

# Remap Keys for Windows

## 通过修改注册表重映射键位

### 重映射说明

这里做的重映射如下:
- `左 Ctrl` 映射为 `左 Alt`
- `左 Alt` 映射为 `左 Ctrl`
- `右 Ctrl` 映射为 `右 Alt`
- `右 Alt` 映射为 `右 Ctrl`

即交换键盘上 `Ctrl` 和 `Alt` 的键位, 因为 `Ctrl` 使用的频率比 `Alt` 要高。

键位和键码对应关系:

| 键位      | 键码    |
|-----------|---------|
| `左 Ctrl` | `00 1D` |
| `左 Alt`  | `00 38` |
| `右 Ctrl` | `E0 1D` |
| `右 Alt`  | `E0 38` |

用改注册表的方式重映射键位较麻烦，需查询键位对应的键码, 但好处是不用额外下载软件，性能更好一些。

### 重映射步骤

1. `Win` + `r`, 输入 regedit, 回车运行并授权。

2. 定位注册表关于修改键位的路径, 可在顶部地址栏输入下面内容并回车:
```
计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout
```

3. 右击注册表项 `Keyboard Layout` -> 新建 -> 二进制值 -> 命名为 `Scancode Map` -> 双击进入编辑数据。

![创建ScanCode Map](./img/create-registry-entry.png)

4. 数据说明和编辑. 数值数据的内容是十六进制的数字，`A` 到 `F` 分别对应十进制的 `11` 到 `15`. 这里把 2 个数字称为一组(字节)，把 4 组数字称为一个映射.

![编辑注册表项数据](./img/edit-registry-value.png)

- 前 2 个映射对应第 1 行，内容都是 `0`，这个用来标记开始，是固定的。
- 第 2 行第 1 个映射的第 1 组 `05` 表示后面键位映射的个数加上 1, 后面有 4 个映射, 所以这里是 `4+1=5`。
- 第 2 行第 1 个映射剩下的 3 组值都为 `0`, 是固定的。
- 这里的数据是小端序, 也即从低位开始排列, 比如 `12 34` 填入其中应该是 `34 12`, 即先填充低位再填充高位。
- 第 2 行第 2 个映射是真正的键位映射, 4 组数字组成一个键位映射, 前 2 组数字对应新键的键码, 后 2 组数字对应旧键的键码, 即把后面的键位映射成前面的。`00 38` 是 `左 Alt` 键的键码, `00 1D` 是 `左 Ctrl`, 就是把原来 `左 Ctrl` 键映射成 `左 Alt`。由上面说的小端序，故这里是低位优先，反向填充。
- 剩下的 3 个映射可类比分析，不再赘述。
- 键位映射完成后，可在末尾再添加 2 个映射，内容都是用 `0` 填充，来表示映射结束。上图没有添加。

5. 重启生效

### 取消重映射

删除上面新建的 `Scancode Map` 后重启即可。

### 注册表导出与导入

如果之前已在其它电脑通过修改注册表做了键位映射，则只需要从旧电脑导出对应的注册表项然后导入。

#### 导出

1. 执行导出键位映射的注册表项命令:

```cmd
reg export "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout" remap-keys.reg
```

2. 使用记事本或其它文本编辑器打开 `remap-keys.reg` 文件，只保留 `Scancode Map` 相关的数据，其它删除。

![编辑 remap-keys.reg 注册表文件](./img/edit-reg-file.png)

内容如下:

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,05,00,00,00,38,00,1d,00,1d,00,38,00,\
  38,e0,1d,e0,1d,e0,38,e0,00,00,00,00,00,00,00,00

```

注意注册表文件的内容各行是以 CRLF 隔开的, 而非 LF。

#### 导入

1. 将 `remap-keys.reg` 传输到想要进行映射的电脑，cmd 进入该文件所在目录，执行:

```cmd
reg import .\remap-keys.reg
```

2. 重启。

### 键位与键码对应表

| 键位              | 键码    |
|-------------------|---------|
| `Backspace`       | `00 0E` |
| `Caps Lock`       | `00 3A` |
| `Delete`          | `E0 53` |
| `End`             | `E0 4F` |
| `Enter`           | `00 1C` |
| `Escape`          | `00 01` |
| `HOME`            | `E0 47` |
| `Insert`          | `E0 52` |
| `Left Alt`        | `00 38` |
| `Left Ctrl`       | `00 1D` |
| `Left Shift`      | `00 2A` |
| `Left Windows`    | `E0 5B` |
| `Num Lock`        | `00 45` |
| `Page Down`       | `E0 51` |
| `Page Up`         | `E0 49` |
| `Power`           | `E0 5E` |
| `PrtSc`           | `E0 37` |
| `Right Alt`       | `E0 38` |
| `Right Ctrl`      | `E0 1D` |
| `Right Shift`     | `00 36` |
| `Right Windows`   | `E0 5C` |
| `Scroll Lock`     | `00 46` |
| `Sleep`           | `E0 5F` |
| `Space`           | `00 39` |
| `Tab`             | `00 0F` |
| `Wake`            | `E0 63` |
| `0`               | `00 52` |
| `1`               | `00 4F` |
| `2`               | `00 50` |
| `3`               | `00 51` |
| `4`               | `00 4B` |
| `5`               | `00 4C` |
| `6`               | `00 4D` |
| `7`               | `00 47` |
| `8`               | `00 48` |
| `9`               | `00 49` |
| `-`               | `00 4A` |
| `/*`              | `00 37` |
| `.`               | `00 53` |
| `/`               | `00 35` |
| `/+`              | `00 4E` |
| `Enter`           | `E0 1C` |
| `F1`              | `00 3B` |
| `F2`              | `00 3C` |
| `F3`              | `00 3D` |
| `F4`              | `00 3E` |
| `F5`              | `00 3F` |
| `F6`              | `00 40` |
| `F7`              | `00 41` |
| `F8`              | `00 42` |
| `F9`              | `00 43` |
| `F10`             | `00 44` |
| `F11`             | `00 57` |
| `F12`             | `00 58` |
| `F13`             | `00 64` |
| `F14`             | `00 65` |
| `F15`             | `00 66` |
| `Down`            | `E0 50` |
| `Left`            | `E0 4B` |
| `Right`           | `E0 4D` |
| `Up`              | `E0 48` |
| `Calculator`      | `E0 21` |
| `E-Mail`          | `E0 6C` |
| `Media Select`    | `E0 6D` |
| `Messenger`       | `E0 11` |
| `My Computer`     | `E0 6B` |
| `’ ”`             | `00 28` |
| `- _`             | `00 0C` |
| `, <`             | `00 33` |
| `. >`             | `00 34` |
| `/ ?`             | `00 35` |
| `; :`             | `00 27` |
| `[ {`             | `00 1A` |
| `\ `              | `00 2B` |
| `] }`             | `00 1B` |
| `<backtick> ~`    | `00 29` |
| `= +`             | `00 0D` |
| `0 )`             | `00 0B` |
| `1 !`             | `00 02` |
| `2 @`             | `00 03` |
| `3 #`             | `00 04` |
| `4 $`             | `00 05` |
| `5 %`             | `00 06` |
| `6 ^`             | `00 07` |
| `7 &`             | `00 08` |
| `8 *`             | `00 09` |
| `9 (`             | `00 0A` |
| `A`               | `00 1E` |
| `B`               | `00 30` |
| `C`               | `00 2E` |
| `D`               | `00 20` |
| `E`               | `00 12` |
| `F`               | `00 21` |
| `G`               | `00 22` |
| `H`               | `00 23` |
| `I`               | `00 17` |
| `J`               | `00 24` |
| `K`               | `00 25` |
| `L`               | `00 26` |
| `M`               | `00 32` |
| `N`               | `00 31` |
| `O`               | `00 18` |
| `P`               | `00 19` |
| `Q`               | `00 10` |
| `R`               | `00 13` |
| `S`               | `00 1F` |
| `T`               | `00 14` |
| `U`               | `00 16` |
| `V`               | `00 2F` |
| `W`               | `00 11` |
| `X`               | `00 2D` |
| `Y`               | `00 15` |
| `Z`               | `00 2C` |
| `Close`           | `E0 40` |
| `Fwd`             | `E0 42` |
| `Help`            | `E0 3B` |
| `New`             | `E0 3E` |
| `Office Home`     | `E0 3C` |
| `Open`            | `E0 3F` |
| `Print`           | `E0 58` |
| `Redo`            | `E0 07` |
| `Reply`           | `E0 41` |
| `Save`            | `E0 57` |
| `Send`            | `E0 43` |
| `Spell`           | `E0 23` |
| `Task Pane`       | `E0 3D` |
| `Undo`            | `E0 08` |
| `Mute`            | `E0 20` |
| `Next Track`      | `E0 19` |
| `Play/Pause`      | `E0 22` |
| `Prev Track`      | `E0 10` |
| `Stop`            | `E0 24` |
| `Volume Down`     | `E0 2E` |
| `Volume Up`       | `E0 30` |
| `? -`             | `00 7D` |
| `Next to Enter`   | `E0 2B` |
| `Next to L-Shift` | `E0 56` |
| `Next to R-Shift` | `E0 73` |
| `DBE_KATAKANA`    | `E0 70` |
| `DBE_SBCSCHAR`    | `E0 77` |
| `CONVERT`         | `E0 79` |
| `NONCONVERT`      | `E0 7B` |
| `Internet`        | `E0 01` |
| `iTouch`          | `E0 13` |
| `Shopping`        | `E0 04` |
| `Webcam`          | `E0 12` |
| `Back`            | `E0 6A` |
| `Favorites`       | `E0 66` |
| `Forward`         | `E0 69` |
| `HOME`            | `E0 32` |
| `Refresh`         | `E0 67` |
| `Search`          | `E0 65` |
| `Stop`            | `E0 68` |
| `My Pictures`     | `E0 64` |
| `My Music`        | `E0 3C` |
| `Mute`            | `E0 20` |
| `Play/Pause`      | `E0 22` |
| `Stop`            | `E0 24` |
| `+ (Volume up)`   | `E0 30` |
| `- (Volume down)` | `E0 2E` |
| `Media`           | `E0 6D` |
| `Mail`            | `E0 6C` |
| `Web/Home`        | `E0 32` |
| `Messenger`       | `E0 05` |
| `Calculator`      | `E0 21` |
| `Log Off`         | `E0 16` |
| `Sleep`           | `E0 5F` |
| `Help(on F1 key)` | `E0 3B` |
| `Undo(on F2 key)` | `E0 08` |
| `Redo(on F3 key)` | `E0 07` |
| `Fwd (on F8 key)` | `E0 42` |
| `Send(on F9 key)` | `E0 43` |

### 参考

- [https://blog.csdn.net/lhdalhd1996/article/details/90741092](https://blog.csdn.net/lhdalhd1996/article/details/90741092)
- [https://learn.microsoft.com/zh-cn/windows/win32/inputdev/about-keyboard-input](https://learn.microsoft.com/zh-cn/windows/win32/inputdev/about-keyboard-input)
- [https://isenselabs.com/posts/keyboard-key-kills-and-remaps-for-windows-users](https://isenselabs.com/posts/keyboard-key-kills-and-remaps-for-windows-users)
