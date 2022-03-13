# 解决Mac不能写入Windows格式(NTFS)磁盘问题

使用Mac的同学都知道Mac默认不能在NTFS格式的磁盘中写入内容。下面介绍一个简单的方法，简单几行命令解决所有问题。

大概的思路就是：Mac自动挂载的时候没有开放写权限，我们自己手动挂载一遍，把写权限加上。仅此而已！！！

### 第一步：查看磁盘设备文件名

这一步需要看一下，目标磁盘叫什么名字

1. `diskutil list`

![diskutil_list](https://static.oschina.net/uploads/img/201807/16133751_Z7DD.png)

可以看到我移动硬盘被挂载了disk2的位置上，其中Windows那个磁盘**设备文件名**为**disk2s4**

以上信息告诉我们：1. 在`/dev`目录下； 2. 设备名称为`disk2s4`

### 第二步：新建挂载点

其实他的意思也就是要告诉电脑，你这张盘要放在那里，就好像Windows电脑在你点击**我的电脑**之后可以看到所有的盘一样。  
这里选择放在桌面。  
其实是在桌面上新建一个叫`Windows`的文件夹：

1. `mkdir ~/Desktop/Windows`

### 第三步：推出磁盘（重新挂载）

Mac默认挂载的时候不可写磁盘，这里我们需要重新挂载一次，但是在此之前，需要先取消挂载（等同于鼠标右键菜单中的**推出**，但是不要选择推出全部）

1. `sudo umount /dev/disk2s4`

![umount](https://static.oschina.net/uploads/img/201807/16133756_WLn0.png)

### 第四部：重新挂载

手动挂载

1. `sudo mount_ntfs -o rw,nobrowse /dev/disk2s4 ~/Desktop/Windows`

成功，磁盘可以正常读写了！！！

扩充  可以生成一个快捷方式 来一键启动 方便 以后使用

1. 打开终端，查看赢盘的Volume Name

> diskutil list

2. 更新fstab文件，此步骤需要输入密码

> sudo nano /etc/fstab

3. 在fstab文件中写入一下内容(movie替换为你自己的Volume Name，建议用英文命名)

> LABEL=movie none ntfs rw,auto,nobrowse

4.CTRL + X保存，选择 Y，然后按回车键。

5. 在桌面上建立一个NTFS硬盘的快捷方式。

> sudo ln –s /Volumes/movie ~/Desktop/movie

或

> sudo ln –s /Volumes/movie ~/Desktop/Volumes

重新插入硬盘后生效。
