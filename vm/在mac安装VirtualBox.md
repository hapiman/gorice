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

