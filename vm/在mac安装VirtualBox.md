### 相关问题
1. 本地和虚拟机共享文件遇到的问题
```sh
Unable to insert the virtual optical disk /Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso into the machine ubuntu_s.

Could not mount the media/drive '/Applications/VirtualBox.app/Contents/MacOS/VBoxGuestAdditions.iso' (VERR_PDM_MEDIA_LOCKED).

Result Code: NS_ERROR_FAILURE (0x80004005)
Component: ConsoleWrap
Interface: IConsole {872da645-4a9b-1727-bee2-5585105b9eed}
Callee: IMachine {85cd948e-a71f-4289-281e-0ca7ad48cd89}
```
针对这个问题，网上大多数文章都是直接使用`Devices` => `Insert Guest Addtions CD images ...`，但是肯定会出现上面的错误，解决方法是直接在虚拟机中执行`sudo apt-get install virtualbox-guest-dkms`

挂载命令为`sudo mount -t vboxsf 共享文件夹名称 挂载目录`，示例`sudo mount -t vboxsf bill /mnt/shared/`, 每一次启动`linux`都要执行该命令重新挂载文件目录

2. 使用`ssh`连接虚拟机里面的系统，使用`sudo service ssh status`能够发现`Could not load host key: /etc/ssh/ssh_host_rsa_key`问题，导致ssh连接的时候报错`Connection closed by 10.254.111.16 port 22`

该问题的原因在于服务器上生成`ssh key`的方式不正确，在终端`ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key`重新建立`ssh_host_dsa_key`文件即可

同理对于`error: Could not load host key: /etc/ssh/ssh_host_dsa_key`问题，可以直接执行`sudo ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key`解决

3. 配置`ssh`访问时，一定要注意`.ssh`文件的权限，一般权限是`755`, 如果有写权限就要输入密码

4. `ubuntu`上使用`ssh`服务，需要自行安装`sudo apt-get install openssh-server`， 同理客户端也需要安装相应的配置



