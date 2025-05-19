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

```
yum install -y epel-release yum-utils
yum install -y dnf dnf-plugins-core
dnf update
```

## Ansible 2.10+ on RHEL 7.6
