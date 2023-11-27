# zerotier基本使用

## 在线安装zerotier

```
curl -s https://install.zerotier.com | sudo bash
```

## 查看安装zerotier版本

```
sudo zerotier-cli status
```

## 加入一个netWork

```
sudo zerotier-cli join ################（networkid）
```

## 查看加入的网络的信息，比如network

```
sudo zerotier-cli listnetworks
```

## 退出加入的network网段

```
sudo zerotier-cli leave ################（networkid）
```

## 启动、停止服务

**(mac下，linux尝试发现没有launchctl command)**

```bash
# Stop and start the service with launchctl
# 停止
sudo launchctl unload /Library/LaunchDaemons/com.zerotier.one.plist

# 启动
sudo launchctl load /Library/LaunchDaemons/com.zerotier.one.plist
```

## 卸载服务（未尝试）

```bash
# Cleanly uninstall ZeroTier One, preserving only your secret identity

sudo "/Library/Application Support/ZeroTier/One/uninstall.sh"
```