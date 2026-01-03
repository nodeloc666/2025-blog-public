# 一、前言
通过VMware与宿主机共享文件，提高工作效率。话不多说，直接上操作。


# 二、步骤
## 主机操作
- 选择共享文件夹
依次点击`编辑虚拟机设置`--> `高级`-->`共享文件夹`-->`总是启用`-->`添加共享文件夹`-->`确定`


![image.png](https://img.uutv.dpdns.org/file/blog/1760611832163_image.png)


## 虚拟系统操作（Ubuntu24为例）
打开命令行，执行
```bash
sudo mkdir /mnt/hgfs
``` 
- 该命令是创建该文件夹，要不然可能会显示` fuse: bad mount point '/mnt/hgfs': No such file or directory `


```bash
sudo mount -t fuse.vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```


![image.png](https://img.uutv.dpdns.org/file/blog/1760611646673_image.png)


- 这步是挂载该文件夹，这样就可以看到共享文件夹了，如下图。


![image.png](https://img.uutv.dpdns.org/file/blog/1760611765634_image.png)


# 结束






