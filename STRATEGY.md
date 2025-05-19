## FIX YUM AND INSTALL DNF :

```bash
cat > /etc/yum.repos.d/centos76-vault.repo <<'EOF'
[centos76-base]
name=CentOS-7.6.1810 Base
baseurl=http://vault.centos.org/7.6.1810/os/x86_64/
gpgcheck=0
enabled=1

[centos76-extras]
name=CentOS-7.6.1810 Extras
baseurl=http://vault.centos.org/7.6.1810/extras/x86_64/
gpgcheck=0
enabled=1
EOF
```

```bash
yum-config-manager --disable centos76-optional
yum-config-manager --disable libnvidia-container
```

## CLEAN && update :

```bash
yum clean all
yum update
```

## INSTALL DNF / EPEL && UPDATE:

```bash
yum install -y epel-release yum-utils
yum install -y dnf dnf-plugins-core
dnf update
```

## Remove every Mellanox/OFED package

```bash
sudo dnf remove --allowerasing $(rpm -qa | grep -E 'mlnx|OFED')
sudo rpm -e --nodeps kernel-3.10.0-862.el7.x86_64
```
## INSTALL PYTHON 3.6 :

```bash
sudo yum install -y epel-release
sudo yum install -y python36 python36-pip

```

## INSTALL ANSIBLE :

```bash
# 1) Fetch the get-pip for Python 3.6:
curl -fsSL https://bootstrap.pypa.io/pip/3.6/get-pip.py -o get-pip3.6.py

# 2) Install a modern pip into your system Python 3.6:
sudo python3.6 get-pip3.6.py

# 3) Use that pip to install Ansible 2.10+ (ansible-core)  
sudo python3.6 -m pip install 'ansible-core>=2.10' \
    --trusted-host pypi.org \
    --trusted-host files.pythonhosted.org

# 4) Verify:
ansible --version
```
