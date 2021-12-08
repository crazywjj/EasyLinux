







# linux查看服务器信息



# 1 查看服务器硬件信息

## 1.1 查看服务器型号、序列号



dmidecode|grep "System Information" -A9|egrep "Manufacturer|Product|Serial" 





## 1.2 查看主板型号

dmidecode |grep -A16 "System Information$" 







## 1.3 查看BIOS信息

```
dmidecode -t bios
```





## 1.4 查看内存槽及内存条















