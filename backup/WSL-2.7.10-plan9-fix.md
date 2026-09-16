昨天尝试配置WSL使用了开发，遇到了一个神奇的问题

在 WSL2 发行版中，将 `/etc/wsl.conf` 设置为：

```
   [automount]
   enabled = false
   mountFsTab = false
```

运行 `wsl --shutdown` 重启以应用更改。
重新启动发行版，然后尝试从 Windows 访问 `\\wsl.localhost\<distro>` （或 `\\wsl$\<distro>` ）。
然后就直接报访问错误

问了半天 AI 也没说配置错误，AI 说按理说配置这两个不会影响wsl.localhost的访问，我把`mountFsTab = true`则是能够正常访问

于是我让AI查了查， 发现有同样问题的请求

`https://github.com/microsoft/WSL/issues/41056`

我看到已经在 `2.9.8`预览版本修复了这个问题，想着那敢情好啊，直接更新完了呗`wsl --update --pre-release`，然后一启动，直接框框报错

**“无法加载远程桌面服务 ActiveX 控件。请确保 rdclientx.dll 已添加到路径中。”**

我靠，那时候直接给我气得，我开始还以为是有什么依赖没安装，结果修半天不好，找了一下，有相关的issue 
`https://github.com/microsoft/WSL/issues/41001`

还直接给我不修了，看issue里说`2.7.11`以上版本都有这个问题。。。上一个问题又要`2.9.8`预览版本，给我气得，尝试了各种方法，直接替换二进制文件啊什么的，都不管用，有人说直接不用WSL 的GUI不就好了，这怎么能妥协呢！身为一个**强迫症**怎么能够允许是因为bug导致用不了这个功能呢！不然我直接第一个问题的时候设置`mountFsTab = true`不就好了

直接一股作气，于是 [Releases · WslzGmzs/WSL-2.7.10-plan9-fix](https://github.com/WslzGmzs/WSL-2.7.10-plan9-fix/releases)，反正也就一行代码,就是编译等死我了，谁懂10个提交8个是打包工作流的（修复）的心情。。。

---

<img width="841" height="807" alt="Image" src="https://github.com/user-attachments/assets/4a72d501-0466-43c3-973d-6a3df8b7288c" />

---

目前用着没什么问题，有问题再说吧。用法的话可以用安装包直接覆盖安装试试，我没测试过安装包，我是安装官方的`2.7.10`，然后直接把`C:\Program Files\WSL\tools`文件夹下的`init`，`initrd.img`两个二进制文件给覆盖了，就没第一个问题了，至于第二个问题，不更新不管了