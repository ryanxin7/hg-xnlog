# Windows 11 Terminal 安装PowerLine教程





### 一、安装 Oh My Posh for PowerShell

```shell
winget install JanDeDobbeleer.OhMyPosh -s winget
```



### 二、选择主题

主题列表：https://ohmyposh.dev/docs/themes

![](https://cdn1.ryanxin.live/image-20230517105339263.png)

选择主题，并使用此命令更新 PowerShell 配置文件。 （可以将 `notepad` 替换为你选择的文本编辑器。）

```bash
notepad $PROFILE
```



将以下项添加到 PowerShell 配置文件的末尾，以设置 `cloud-native-azure` 主题。 （如果想替换将 `cloud-native-azure` 替换为你选择的主题。）



```bash
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\cloud-native-azure.omp.json" | Invoke-Expression
```





### 三、安装 Nerd Font

 如果你的字体不包含相应字形，则在整个提示符中，你可能会看到若干 Unicode 替换字符“▯”。 若要在终端中查看所有字形，建议安装 [Nerd Font](https://www.nerdfonts.com/font-downloads)



![](https://cdn1.ryanxin.live/image-20230517110124776.png)

官方推荐字体：**MesloLGM NF**



```bash
#安装字体
oh-my-posh font install
```

![](https://cdn1.ryanxin.live/image-20230517104759748.png)

**结果老是timeout**



在字体库直接下载

https://www.nerdfonts.com/font-downloads

![](https://cdn1.ryanxin.live/image-20230517104937861.png)

```bash
#直接导入字体
oh-my-posh font install .\Meslo.zip 
```

![](https://cdn1.ryanxin.live/image-20230517101113664.png)



**修改配置文件使用**

```yaml
    "profiles": 
    {
        "defaults": {
            "font":
            {
                "face": "MesloLGM NF"   
    },
		},
```

找不到所选字体 **MesloLGM NF** 🤮

![](https://cdn1.ryanxin.live/image-20230517103815943.png)



找不到？  字体名称是 **MesloLGM Nerd Font** 替换一下 ？

![](https://cdn1.ryanxin.live/image-20230517104001794.png)

```
    "profiles": 
    {
        "defaults": {
            "font":
            {
                "face": "MesloLGM Nerd Font"   
    },
		},
```



可以看到已经正常显示了

![image-20230517104548750](https://cdn1.ryanxin.live/image-20230517104548750.png)



### 四、Oh My Posh 与 Clink 集成

Windows CMD 没有开箱即用的支持。然而，有一种方法可以使用[Clink](https://chrisant996.github.io/clink/)来实现，同时可以增强您的 cmd 体验。按照安装说明进行操作，并确保选择自动启动。

将 Oh My Posh 与 Clink 集成很容易：在 Clink 脚本目录中创建一个名为 `oh-my-posh.lua` 的新文件（在`clink info`cmd 中运行以找到该文件的位置）。



```lua
load(io.popen('oh-my-posh init cmd'):read("*a"))()
```

添加后，重新启动 cmd 以使更改生效。



![](https://cdn1.ryanxin.live/image-20230517125959310.png)



---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/notes/powerline/  

