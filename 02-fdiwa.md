# Fixing Dynamic IP with Avahi

macbook貌似没法通过wifi开启移动热点，所以我用我的手机开了移动热点，然后通过串口将板子连接到macbook，然后板子自动连接手机热点。但问题出现，手机每次开热点，会给连接到热点的设备重新分配逻辑ip地址，就导致我每次重新开热点，不知道licheepi4a的ip是多少，自然也没法ssh到板子。所以我上网查资料，找到了一种方法。

就是在macbook上查看自己的ip地址，比如是192.168.8.60，那就从192.168.8.1到192.168.8.254都ping一遍，除了macbook和网关，只剩下两个可以ping通，那就是板子和手机。于是我把我的需求告诉LLM，它给我写了一个shell脚本：

```shell
#!/bin/bash

# 获取 en0 接口的 IP 地址（如 192.168.8.60）
en0_ip=$(ifconfig en0 | grep 'inet ' | awk '{print $2}')
if [ -z "$en0_ip" ]; then
    echo "Error: Could not find en0 interface IP address."
    exit 1
fi

# 提取网络前缀（如 192.168.8）
network_prefix=$(echo $en0_ip | cut -d'.' -f1-3)
gateway_ip="$network_prefix.1"  # 假设网关是 x.x.x.1

echo "Your IP: $en0_ip"
echo "Gateway IP: $gateway_ip"
echo "Scanning network: $network_prefix.1-254..."

# 扫描网络并找出活跃设备
active_ips=()
for i in {1..254}; do
    ip="$network_prefix.$i"
  
    # 跳过本机 IP 和网关
    if [ "$ip" == "$en0_ip" ] || [ "$ip" == "$gateway_ip" ]; then
        continue
    fi
  
    # Ping 检测（快速 ping，超时 1 秒）
    if ping -c 1 -W 1 "$ip" &> /dev/null; then
        echo "Found active device: $ip"
        active_ips+=("$ip")
    fi
done

# 输出结果
echo ""
echo "Scan completed."
if [ ${#active_ips[@]} -eq 0 ]; then
    echo "No other active devices found in the network."
elif [ ${#active_ips[@]} -eq 1 ]; then
    echo "The active device is likely your LicheePi 4A: ${active_ips[0]}"
    echo "You can try to SSH to it:"
    echo "  ssh username@${active_ips[0]}"
else
    echo "Multiple active devices found:"
    for ip in "${active_ips[@]}"; do
        echo "  $ip"
    done
    echo "One of these is likely your LicheePi 4A. Try to SSH to each."
fi
```

尝试过后发现，的确可行，但是这样有点慢啊，整个过程需要四分多钟，然后我就给LLM说了，它就给了我一个更快的方法，的确可行：

```shell
#!/bin/bash

# 获取本机 IP 和网关
en0_ip=$(ifconfig en0 | grep 'inet ' | awk '{print $2}')
network_prefix=$(echo $en0_ip | cut -d'.' -f1-3)
gateway_ip="$network_prefix.1"

echo "Scanning $network_prefix.0/24 (parallel ping, wait 2-5 seconds)..."

# 并行 ping 所有 IP，超时 0.2 秒
seq 1 254 | xargs -P 50 -I {} bash -c "
  ip='$network_prefix.{}'
  [ \"\$ip\" == \"$en0_ip\" ] || [ \"\$ip\" == \"$gateway_ip\" ] && exit
  ping -c 1 -W 0.2 \"\$ip\" &>/dev/null && echo \"Active: \$ip\"
"
```

但这还有一个问题，就是每次查到新的ip之后，我还要去改～/.ssh/config中的ip，那咋办呢？LLM告诉我了一个很好的方法：

```shell
# 在licheepi4a上操作一下shell命令

# 安装avahi
sudo dnf install avahi

# 找到[server]下的host-name，改为lp4a
vi /etc/avahi/avahi-daemon.conf

# 设置成开机自启
sudo systemctl enable avahi-daemon

# 启动服务
sudo systemctl start avahi-daemon

# 查看服务是否启动
sudo systemctl status avahi-daemon
```

然后把~/.ssh/config中的ip改成lp4a.local就行了
