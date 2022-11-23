# 操作系统/镜像/快照相关

## CentOS中yum update很慢，如何处理？
将/etc/yum.repos.d/CentOS-Base.repo中6.3（或者其他版本号）改为$releasever，然后yum update。

## CentOS上, 我如何制作自定义的内核rpm包?
确认当前内核版本号, 下载对应的SRPM包 :

    #确认当前版本号
    uname -r
    
    # 到valut.centos.org找到SRPM并下载
    # 注意事项
    # (1) 确认是否用了plus版本的内核, 是的话SRPM在/centosplus/Source/SPackages/
    # (2) 非plus版在以下两个目录: /updates/Source/SPackages, /os/Source/SPackages
    wget http://vault.centos.org/6.4/updates/Source/SPackages/kernel-2.6.32-358.14.1.el6.src.rpm

安装SRPM以及相关RPM工具 :

``` 
# 安装SRPM
rpm -ivh kernel-2.6.32-358.14.1.el6.src.rpm

# 安装相关RPM工具
yum install rpm-build redhat-rpm-config patchutils xmlto asciidoc elfutils-libelf-devel zlib-devel binutils-devel newt-devel python-devel perl-ExtUtils-Embed hmaccalc rng-tools kernel-firmware

# 启动rngd服务, 提供足够的熵值 cat /dev/null >/etc/sysconfig/rngd echo 'EXTRAOPTIONS="--rng-device /dev/urandom"' >/etc/sysconfig/rngd service rngd start

```

生成内核源码, 使用diff生成patch文件 :

    # 生成内核源码
    cd ~/rpmbuild/SPECS
    rpmbuild -bp kernel.spec
    
    # 修改并生成diff文件
    cd ~/rpmbuild/BUILD
    cp -r kernel-2.6.32-358.14.1.el6 kernel-2.6.32-358.14.1.el6.mine
    diff -urpN kernel-2.6.32-358.14.1.el6 kernel-2.6.32-358.14.1.el6.mine > this-patch-to-fix-that-bug.patch
    
    # 将patch拷贝到SOURCES下
    cp this-patch-to-fix-that-bug.patch ~/rpmbuild/SOURCES
    
    # 清理
    rm -rf ~/rpmbuild/BUILD/kernel-2.6.32-358.14.1.el6*


修改SPEC文件, 生成新的内核RPM包 :

    # 找到以下几行, 再后面添加一行
    # Source84: config-s390x-generic-rhel
    # Source85: config-powerpc64-debug-rhel
    # Source86: config-s390x-debug-rhel
    
    # 新添加行
    Source87: this-patch-to-fix-that-bug.patch
    Patch001: this-patch-to-fix-that-bug.patch
    
    # (可选) 修改changelog, 找到%changelog这行, 再后插入行:
    * Tue Aug 03 2013 Your Name<yourname@company.com> [2.6.32-358.14.1.el6.centos]
      - [XXX] path to fix that bug

打包生成新的内核RPM包 :

    # 执行SPEC, 生成过程很长请耐心等待.
    # 执行前, 请仔细review上述步骤, 避免出错重来
    rpmbuila -ba kernel.spec
    
    # 会生成若干个RPM包, 其中最关键的如下, 使用rpm -ivh安装即可
    # kernel-2.6.32-358.14.1.el6.x86_64.rpm
    # kernel-devel-2.6.32-358.14.1.el6.x86_64.rpm
    # kernel-headers-2.6.32-358.14.1.el6.x86_64.rpm

## CentOS系统安装软件包出现对kernel-devel依赖的问题，应该如何解决？
为了确保内核的稳定，默认情况下，我们在yum的配置文件/etc/yum.conf中加了exclude=kernel*
centos-release*，这样可以防止在装软件包时无意更新内核相关的东西。
如果确实需要安装此软件包，将/etc/yum.conf中的这行代码注释即可。

## 如何激活UHost上的Windows Server?
UCloud上创建的Windows云主机，默认是自动激活的，用户无需再操作。参考 [KMS激活说明](uhost/windows_op/kms)
若遇特殊原因，需要手动激活一下，则步骤如下：

1.KMS地址
首先定义下各个数据中心的KMS地址（$kms\_name）。步骤4中会用到$kms\_name，请用相应的地址替代。例如华北一可用区E的地址为hb06.kms.ucloud.cn。若需其他可用区地址，请联系技术支持。
- 进入命令目录
用管理员身份打开cmd -> cd C:\Windows\system32
![image](/images/02.png)
- 清除密钥并重启
执行 cscript.exe slmgr.vbs /rearm 来清除统一密钥，完成后重启操作系统。
![image](/images/03.png)
- 配置KMS

用管理员身份打开cmd -> cd C:\Windows\system32。执行cscript.exe slmgr.vbs /skms
$kms\_name(见步骤1)
![image](/images/04.png)
- 激活Windows

执行cscript.exe slmgr.vbs /ato 激活windows
![image](/images/05.png)

