# Centos7.4mini离线安装ansible



```bash
# 需要搭建本地本地镜像源(非mini镜像)
yum -y install python-devel openssl-devel libffi-devel gcc gcc-c++  cmake

#百度网盘下载所需要的包
链接：https://pan.baidu.com/s/1RJp3d6o3DTLVjLCq5ZraNg 
提取码：xhz2

# 安装 setuptools
tar -zxf setuptools-41.0.1.tar.gz
cd setuptools-41.0.1
python setup.py install
cd ..


# 安装pycrypto
tar -xzf pycrypto-2.6.1.tar.gz 
cd pycrypto-2.6.1
python setup.py install
cd ..



# 安装 PyYAML
tar -xzf PyYAML-5.1.tar.gz 
cd PyYAML-5.1
python setup.py install
cd ..

# 安装MarkupSafe
tar -xzf MarkupSafe-1.1.1.tar.gz 
cd MarkupSafe-1.1.1
python setup.py  install
cd ..

# 安装Jinja2
tar -xzf Jinja2-2.10.1.tar.gz 
cd Jinja2-2.10.1
python setup.py  install
cd ..

# 安装ecdsa
tar -xzf ecdsa-0.13.2.tar.gz 
cd ecdsa-0.13.2
python setup.py install
cd ..


# 安装simplejson
tar -xzf simplejson-3.16.0.tar.gz 
cd simplejson-3.16.0
python setup.py install
cd ..

# 安装pycparser
tar -xzf pycparser-2.19.tar.gz
cd pycparser-2.19
python setup.py install
cd ..


# 安装cffi
tar -xzf cffi-1.12.3.tar.gz 
cd cffi-1.12.3
python setup.py install
cd ..

# 安装ipaddress
tar -xzf ipaddress-1.0.22.tar.gz 
cd ipaddress-1.0.22
python setup.py install
cd ..


# 安装six
tar -xzf six-1.12.0.tar.gz 
cd six-1.12.0
python setup.py install
cd ..

# 安装asn1crypto
tar -xzf asn1crypto-0.24.0.tar.gz 
cd asn1crypto-0.24.0
python setup.py install
cd ..


# 安装idna
tar -xzf idna-2.8.tar.gz 
cd idna-2.8
python setup.py install
cd ..

# 安装pyasn1
tar -xzf pyasn1-0.4.5.tar.gz 
cd pyasn1-0.4.5
python setup.py install
cd ..

# 安装PyNaCl
tar -xzf PyNaCl-1.3.0.tar.gz 
cd PyNaCl-1.3.0
python setup.py install
cd ..


# 安装cryptography
tar -xzf cryptography-2.6.1.tar.gz 
cd cryptography-2.6.1
python setup.py install
cd ..


# 安装bcrypt
tar -xzf bcrypt-3.1.6.tar.gz 
cd bcrypt-3.1.6
python setup.py install
cd ..

# 安装paramiko
tar -xzf paramiko-2.4.2.tar.gz 
cd paramiko-2.4.2
python setup.py install
cd ..

# 安装ansible
tar -zxvf ansible-2.9.7.tar.gz
cd ansible-2.9.7
python setup.py install
cd ..



# 修改Ansible配置文件位置
mkdir -p /etc/ansible/
cp -a /$(pwd)/ansible-2.9.7/examples/ansible.cfg /etc/ansible/
cp -a /$(pwd)/ansible-2.9.7/examples/hosts /etc/ansible/

ansible --version
```

