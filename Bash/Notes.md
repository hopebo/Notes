# Commands
## 把home目录软链接到其他目录
```shell
sudo mkdir /flash1/libo
sudo cp -rf /home/libo/. /flash1/libo
sudo rm -rf /home/libo
sudo ln -sf /flash1/libo /home/libo
```
