# VLAN详解
Type:2bytes, PRI:3bits, CFI:1bit, VID:12bits

vlan标签长4个字节  
*TPID：2字节，固定值0x8100,表明是一个802.1Q标签的帧；  
*PRI:帧的优先级，0-7，值越大优先级越高；  
*CFI:标识MAC地址是否为经典格式，CFI=0说明是经过格式，CFI=1表明为非经典格式。  
*VLAN ID: 12bit,1和4095被保留，可用的端口范围2~2094；  

