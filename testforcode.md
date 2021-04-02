```bash
#/bin/bash

# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
	# We have color support; assume it's compliant with Ecma-48
	# (ISO/IEC-6429). (Lack of such support is extremely rare, and such
	# a case would tend to support setf rather than setaf.)
	color_prompt=yes
    else
	color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi

export PATH=$PATH:/usr/local/go/bin:/usr/local/python3/bin
export GOPROXY="https://goproxy.cn"

```

```bash
#! /bin/bash
set -e

#full_path=$UIOT_EDGE_PATH
#if [ "$full_path" = "" ]; then
current_path=$(cd "$(dirname "$0")";pwd)
full_path=$current_path/ucloud-edge
#fi
# echo "full path is ${full_path}"

master_work_path_prefix=${full_path}/run/native
master_work_path=${full_path}/run/native/var/db/uiotedge

BrokerAddr=tcp://{{BROKER_ADDRESS}}:1883
FrpsAddr={{FRPS_ADDRESS}}
BUCKET_ADDRESS={{BUCKET_ADDRESS}}

os="linux"
REQUIRED_PYTHON_VERSION_V1=3
REQUIRED_PYTHON_VERSION_V2=5
REQUIRED_PYTHON_VERSION_V3=3

HARDWARE_VALIDATE_SERVER=""

preflag=sudo
if [ $UID -eq 0 ];then
	preflag=''
fi

pscmd="ps"
# state_line='4'
pid_line='1'
if [ -f "/etc/os-release" ]; then  # for linux release version
    release_linux=`cat /etc/os-release | egrep '^ID=.*' | sed 's/^ID=//g' | sed 's/"//g'`
    if [ "$release_linux" = "ubuntu" ] || [ "$release_linux" = "raspbian" ] || [ "$release_linux" = "debian" ] || [ "$release_linux" = "centos" ];then
        pscmd="ps -elf"
        pid_line='4'
    fi
fi

unset GREP_OPTIONS

usageFunc() {
    echo "Invalid parameter!"
    echo "----------------------------------USAGE----------------------------------------"
    echo "# Install ucloud edge with parameters"
    echo "USAGE1: $0 --install Arch Version Region"
    echo "        Arch   : ARMv7,ARMv8_64,X86_64" 
    echo "        Version: 1.0 "
    echo "        Region : sh,gd "
    echo "--------------------------------------------------------------------------------"
    echo "# Config the edge with startup parameters"
    echo "USAGE2: $0 --config ProductSN DeviceSN DeviceSecret "
    echo "--------------------------------------------------------------------------------"
    echo "# Start the edge"
    echo "USAGE3: $0 --start"
    echo "--------------------------------------------------------------------------------"
    echo "# Get the edge status"
    echo "USAGE4: $0 --status"
    echo "--------------------------------------------------------------------------------"
    echo "# Stop the edge"
    echo "USAGE5.1: $0 --stop"
    echo "USAGE5.2: $0 --stop all"
    echo "--------------------------------------------------------------------------------"
    echo "# upgrade the edge"
    echo "USAGE6.1: $0 --upgrade webportal"
    echo "USAGE6.2: $0 --upgrade uagent"
    echo "USAGE6.3: $0 --upgrade logservice"
    echo "USAGE6.4: $0 --upgrade frpc"
    echo "--------------------------------------------------------------------------------"
}

print_warn() {
    echo -e "\033[1;33m$1\033[0m"
}

check_python_version(){
    U_V1=`python3 -V 2>&1|awk '{print $2}'|awk -F '.' '{print $1}'`
    U_V2=`python3 -V 2>&1|awk '{print $2}'|awk -F '.' '{print $2}'`
    U_V3=`python3 -V 2>&1|awk '{print $2}'|awk -F '.' '{print $3}'`
    
    if [ $U_V1 -lt $REQUIRED_PYTHON_VERSION_V1 ];then
        echo 1
    elif [ $U_V1 -eq $REQUIRED_PYTHON_VERSION_V1 ];then     
        if [ $U_V2 -lt $REQUIRED_PYTHON_VERSION_V2 ];then 
            echo 1
        elif [ $U_V2 -eq $REQUIRED_PYTHON_VERSION_V2 ];then
            if [ $U_V3 -lt $REQUIRED_PYTHON_VERSION_V3 ];then 
                echo 1
            else 
                echo 0
            fi
        else
            echo 0
        fi    
    else 
        echo 0
    fi
}

set_variable_of_region() {
    HARDWARE_VALIDATE_SERVER="sh@https://manage-cn-sh2.iot.ucloud.cn:443,gd@https://manage-cn-gd.iot.ucloud.cn:443"
    if [ "$1" == "sh" ]; then
        BrokerAddr=tls://mqtt-cn-sh2.iot.ucloud.cn:8883
        FrpsAddr=113.31.106.43
        BUCKET_ADDRESS=http://uiotedge.cn-sh2.ufileos.com
    elif [ "$1" == "gd" ]; then
        BrokerAddr=tls://mqtt-cn-gd.iot.ucloud.cn:8883
        FrpsAddr=106.75.174.31
        BUCKET_ADDRESS=http://uiotedge-gd.cn-gd.ufileos.com
    elif [ "$1" == "pre" ]; then
        BrokerAddr=tls://pre-mqtt.iot.ucloud.cn:8883
        FrpsAddr=120.132.11.95
        BUCKET_ADDRESS=http://uiotedge-staging.cn-sh2.ufileos.com
        HARDWARE_VALIDATE_SERVER="pre@http://113.31.103.155:443"
    else
        print_warn "$1 Region does not support!"
        exit 1;
    fi
}

install() {
    set_variable_of_region $3

    information_pre="UCloud Edge will exit for:"
    cpu_arch=`uname -m`
    # echo $cpu_arch
    if [ "$1" = "X86_64" ] || [ "$1" = "ARMv8_64" ]; then
        if [ "$1" = "X86_64" ];then
            if [ "$cpu_arch" != "x86_64" ]; then
                print_warn "${information_pre} cpu arch is $cpu_arch and the aim is $1, mismatched!"
                exit 1;
            fi
            arch="amd64"
        else
            if [ "$cpu_arch" != "aarch64" ]; then
                print_warn "${information_pre} cpu arch is $cpu_arch and the aim is $1, mismatched!"
                exit 1;
            fi
            arch="arm64"
        fi
    elif [ "$1" = "ARMv7" ]; then
        if [ "$cpu_arch" == "armv7l" ] || [ "$cpu_arch" == "ARMv7" ]; then
            arch="arm7"
        else
            print_warn "${information_pre} cpu arch is $cpu_arch and the aim is $1, mismatched!"
            exit 1;
        fi
    else
        print_warn "${information_pre} Arch not support!"
        exit 1;
    fi

    python3_needed=false
    if [ ! -x "$(command -v python3)" ]; then
        information="${information_pre} Could not find the binary python3, while Ucloud Edge requires a python >= $REQUIRED_PYTHON_VERSION_V1.$REQUIRED_PYTHON_VERSION_V2.$REQUIRED_PYTHON_VERSION_V3\n"
        python3_needed=true
    else
        res=$(check_python_version)
        if [ $res = 1 ];then
            information="${information_pre} Ucloud Edge requires a python >= $REQUIRED_PYTHON_VERSION_V1.$REQUIRED_PYTHON_VERSION_V2.$REQUIRED_PYTHON_VERSION_V3\n"
            python3_needed=true
        fi
    fi

    if [ $python3_needed = true ];then
        print_warn "$information"
        exit 1;
    fi

    eval $($pscmd | grep -w 'sshd' | grep -v grep | awk '{printf("Pid=%s",$'$pid_line')}')
    if [ "$Pid" = "" ];then
        eval $($pscmd | grep -w 'dropbear' | grep -v grep | awk '{printf("Pid=%s",$'$pid_line')}')
        if [ "$Pid" = "" ];then
            print_warn "${information_pre} Could not find running sshd/dropbear"
            Pid=""
            exit 1;
        fi
    fi

    python3_version=`python3 -c 'import sys; print(sys.version_info[1])'`
    local version=$2
    ${preflag} rm -rf ${full_path}
    mkdir -p ${full_path} ${full_path}/bin ${full_path}/lib ${full_path}/etc ${full_path}/tmp ${full_path}/python3.${python3_version}  ${full_path}/.bak
    echo -e "export TMPDIR=${full_path}/tmp" >> ${full_path}/etc/profile
    echo -e "export PATH=$PATH:${full_path}/bin:${full_path}/python3.${python3_version}/bin" >> ${full_path}/etc/profile
    echo -e "export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${full_path}/lib" >> ${full_path}/etc/profile
    echo -e "export PYTHONPATH=$PYTHONPATH:${full_path}/python3.${python3_version}/" >> ${full_path}/etc/profile
    echo -e "export UIOT_EDGE_PATH=${full_path}" >> ${full_path}/etc/profile
    echo -e "export HARDWARE_VALIDATE_SERVER=${HARDWARE_VALIDATE_SERVER}" >> ${full_path}/etc/profile
    echo -e "export UIOT_EDGE_REGION=$3" >> ${full_path}/etc/profile
    echo -e "export UIOT_EDGE_ARCH=$arch" >> ${full_path}/etc/profile
    echo -e "export UIOT_EDGE_VERSION=$version" >> ${full_path}/etc/profile

    if [ ! -x "$(command -v unzip)" ]; then 
        wget -O unzip.tar $BUCKET_ADDRESS/$version/unzip-$os-$arch.tar
        tar -xf unzip.tar
        mv unzip ${full_path}/bin/
        rm unzip.tar
        export PATH=$PATH:${full_path}/bin
    fi

    wget -O packages.zip $BUCKET_ADDRESS/$version/python3.${python3_version}-package-$os-$arch.zip
    unzip -q packages.zip -d ${full_path}
    chmod +x ${full_path}/python3.${python3_version}/bin/*
    rm -rf packages.zip
  
    wget -O certs.zip $BUCKET_ADDRESS/$version/certs.zip
    unzip -q certs.zip -d ${full_path}/etc
    rm -rf certs.zip
    echo -e "export SSL_CERT_FILE=${full_path}/etc/certs/ca-certificates.crt" >> ${full_path}/etc/profile
    echo -e "export SSL_CERT_DIR=${full_path}/etc/certs/" >> ${full_path}/etc/profile
  
    if [ ! -x "$(command -v sqlite3)" ]; then
        wget -O sqlite3.zip $BUCKET_ADDRESS/$version/sqlite3-$os-$arch.zip
        mkdir -p ${full_path}/sqlite3
        unzip sqlite3.zip -d ${full_path}/sqlite3
        mv ${full_path}/sqlite3/bin/* ${full_path}/bin
        mv ${full_path}/sqlite3/lib/* ${full_path}/lib
        rm -rf sqlite3.zip ${full_path}/sqlite3
    fi

    wget -O supervisor_conf.zip $BUCKET_ADDRESS/$version/supervisor_conf.zip
    unzip supervisor_conf.zip -d ${full_path}/etc
    rm -rf supervisor_conf.zip
   
    wget -O nats-server.zip $BUCKET_ADDRESS/$version/nats-server-v2.1.2-linux-$arch.zip
    unzip nats-server.zip -d ${full_path}/bin 
    rm -fr nats-server.zip

    wget -O redis-server.zip $BUCKET_ADDRESS/$version/redis-server-v2.6.14-linux-$arch.zip
    unzip redis-server.zip -d ${full_path}/bin
    mv ${full_path}/bin/redis.conf ${full_path}/etc/redis.conf
    rm -fr redis-server.zip

    wget -O logservice.zip $BUCKET_ADDRESS/$version/logservice-linux-$arch.zip
    unzip logservice.zip -d ${full_path}/bin
    rm -fr logservice.zip

    wget -O master.zip $BUCKET_ADDRESS/$version/edge-linux-$arch.zip
    unzip master.zip -d ${full_path}/bin 
    rm -rf master.zip

    wget -O run.zip $BUCKET_ADDRESS/$version/run-native.zip
    unzip run.zip -d ${full_path}
    rm -rf run.zip

    wget -O hwvalidate.zip $BUCKET_ADDRESS/$version/hwvalidate-linux-$arch.zip
    unzip hwvalidate.zip -d ${full_path}/bin
    rm -fr hwvalidate.zip

    wget -O serial.zip $BUCKET_ADDRESS/$version/serial-linux-$arch.zip
    unzip serial.zip -d ${full_path}/bin
    rm -fr serial.zip

    mkdir -p ${full_path}/run/native/var/log/uiotedge/nats
    mkdir -p ${full_path}/run/native/var/log/uiotedge/redis
    mkdir -p ${full_path}/run/native/var/log/uiotedge/frpc
    
    mkdir ${full_path}/download
    wget -P ${full_path}/download $BUCKET_ADDRESS/$version/function-python3.zip
    wget -P ${full_path}/download $BUCKET_ADDRESS/$version/function-manager-linux-$arch.zip
    wget -P ${full_path}/download $BUCKET_ADDRESS/$version/uagent-linux-$arch.zip
    wget -P ${full_path}/download $BUCKET_ADDRESS/$version/deploy-agent-linux-$arch.zip
    wget -P ${full_path}/download $BUCKET_ADDRESS/$version/webportal-linux-$arch.zip
    
    # mkdir -p $master_work_path
    mkdir -p $master_work_path/webportal
    unzip ${full_path}/download/webportal-linux-$arch.zip -d $master_work_path/webportal
    mkdir -p $master_work_path/deploy-agent
    unzip ${full_path}/download/deploy-agent-linux-$arch.zip -d $master_work_path/deploy-agent
    mkdir -p $master_work_path/function-manager
    unzip ${full_path}/download/function-manager-linux-$arch.zip -d $master_work_path/function-manager
    mkdir -p $master_work_path/uagent
    unzip ${full_path}/download/uagent-linux-$arch.zip -d $master_work_path/uagent
    mkdir -p $master_work_path/function-python3
    unzip ${full_path}/download/function-python3.zip -d $master_work_path/function-python3
    rm -rf ${full_path}/download

    wget -O ${full_path}/bin/initPassword $BUCKET_ADDRESS/$version/initPassword_$arch
    chmod +x ${full_path}/bin/initPassword

    wget -O ${full_path}/.bak/default-config.zip $BUCKET_ADDRESS/$version/uiotedge-default-config-native.zip
    unzip -q ${full_path}/.bak/default-config.zip -d ${full_path}/.bak/
    rm -fr ${full_path}/.bak/default-config.zip

    wget -O ${full_path}/.bak/frpc.zip $BUCKET_ADDRESS/$version/frpc-linux-$arch.zip
    unzip -q ${full_path}/.bak/frpc.zip -d ${full_path}/.bak
    mv ${full_path}/.bak/frpc/frpc ${full_path}/bin
    rm -fr ${full_path}/.bak/frpc.zip

    chmod +x ${full_path}/bin/*

    echo "-------install completed-------"
}

config_device_info() {
    source ${full_path}/etc/profile
    set_variable_of_region $UIOT_EDGE_REGION

    cp -fr ${full_path}/.bak/uagent-default $master_work_path
    cp -fr ${full_path}/.bak/webportal-default $master_work_path
    cp -fr ${full_path}/.bak/frpc/frpc.ini ${full_path}/etc

    sed -i "s|{{HardwareValidatorServer}}|${HARDWARE_VALIDATE_SERVER}|" $master_work_path/uagent-default/conf.json
    sed -i "s|{{EdgeFullPath}}|${full_path}|" $master_work_path/uagent-default/conf.json
    sed -i "s|{{BrokerAddr}}|${BrokerAddr}|" $master_work_path/uagent-default/conf.json
    sed -i "s/{{EdgeProductSN}}/$1/" $master_work_path/uagent-default/conf.json
    sed -i "s/{{EdgeDeviceSN}}/$2/" $master_work_path/uagent-default/conf.json
    sed -i "s/{{EdgeDeviceSecret}}/$3/" $master_work_path/uagent-default/conf.json
    sed -i "s/{{APPVersion}}/$UIOT_EDGE_VERSION/" $master_work_path/uagent-default/conf.json #APPVersion
    sed -i "s/{{EdgeCPUArch}}/$UIOT_EDGE_ARCH/" $master_work_path/uagent-default/conf.json
    sed -i "s/{{EdgeOS}}/$os/" $master_work_path/uagent-default/conf.json
    sed -i "s/{{EdgeProductSN}}/$1/" $master_work_path/webportal-default/conf.json
    sed -i "s/{{EdgeDeviceSN}}/$2/" $master_work_path/webportal-default/conf.json
    sed -i "s/{{APPVersion}}/$UIOT_EDGE_VERSION/" $master_work_path/webportal-default/conf.json #APPVersion
    if [ -f ${full_path}/bin/serial ]; then
        sed -i "s/{{EdgeHardwareSerial}}/`${full_path}/bin/serial`/" $master_work_path/webportal-default/conf.json #APPVersion
    else
        sed -i "s/{{EdgeHardwareSerial}}//" $master_work_path/webportal-default/conf.json #APPVersion
    fi
    sed -i "s/{{EdgeCPUArch}}/$UIOT_EDGE_ARCH/" $master_work_path/webportal-default/conf.json
    sed -i "s/{{EdgeOS}}/$os/" $master_work_path/webportal-default/conf.json
    sed -i "s/{{ProductSN}}/$1/" ${full_path}/etc/frpc.ini
    sed -i "s/{{DeviceSN}}/$2/" ${full_path}/etc/frpc.ini
    sed -i "s/{{DeviceSecret}}/$3/" ${full_path}/etc/frpc.ini
    sed -i "s/{{FRPSADDR}}/${FrpsAddr}/" ${full_path}/etc/frpc.ini
}

config() {
    webportalPassword=""
    if [ $# -eq 1 ]; then
        webportalPassword=$1
    elif [ $# -eq 4 ]; then
        webportalPassword=$4
    fi

    if [ "$webportalPassword" != "" ] ;then
        ${full_path}/bin/initPassword $master_work_path/sqlite/edge_web.db $webportalPassword
    fi

    login_account=root
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/conf.d/edge.ini
    sed -i "s|{{user}}|${login_account}|g" ${full_path}/etc/supervisor/conf.d/edge.ini
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/conf.d/frpc.ini
    sed -i "s|{{user}}|${login_account}|g" ${full_path}/etc/supervisor/conf.d/frpc.ini
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/conf.d/nats.ini
    sed -i "s|{{user}}|${login_account}|g" ${full_path}/etc/supervisor/conf.d/nats.ini
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/conf.d/redis.ini
    sed -i "s|{{user}}|${login_account}|g" ${full_path}/etc/supervisor/conf.d/redis.ini
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/conf.d/logservice.ini
    sed -i "s|{{user}}|${login_account}|g" ${full_path}/etc/supervisor/conf.d/logservice.ini
    sed -i "s|{{edge_path}}|${full_path}|g" ${full_path}/etc/supervisor/supervisord.conf

    if [ $# -eq 0 ] || [ $# -eq 1 ]; then
        echo -e "UIoT_Edge_OneClick_Mode=true" > ${full_path}/etc/deviceinfo
    else
        echo -e "UIoT_Edge_OneClick_Mode=false" > ${full_path}/etc/deviceinfo
        config_device_info $1 $2 $3
    fi

    echo "-------config completed-------"
}

validate_hw_serial() {
    if [ "$UIoT_Edge_Reset" = "" ]; then
        result=""
        retcode="1"
        retmessage=""
        while true
        do
            echo "try to validate hardware serial..."
            result=`${full_path}/bin/hwvalidate -addr=${HARDWARE_VALIDATE_SERVER}`
            retcode=`echo $result | awk -F '$' '{print $1}'`
            retmessage=`echo $result | awk -F '$' '{print $2}'`
            if [ "$retcode" = "0" ]; then
                break
            else
                echo "validate failed: $retmessage, retry..."
            fi
            sleep 5
        done
        productsn=`echo $result | awk -F '$' '{print $3}'`
        devicesn=`echo $result | awk -F '$' '{print $4}'`
        password=`echo $result | awk -F '$' '{print $5}'`
        region=`echo $result | awk -F '$' '{print $6}'`

        if [ "$retcode" = "0" ]; then
            sed -i "s/^.*UIOT_EDGE_REGION.*$/export UIOT_EDGE_REGION=$region/" ${full_path}/etc/profile
            config_device_info $productsn $devicesn $password
            echo -e "UIoT_Edge_ProductSN=$productsn" >> ${full_path}/etc/deviceinfo
            echo -e "UIoT_Edge_DeviceSN=$devicesn" >> ${full_path}/etc/deviceinfo
            echo -e "UIoT_Edge_Password=$password" >> ${full_path}/etc/deviceinfo
            echo -e "UIoT_Edge_Region=$region" >> ${full_path}/etc/deviceinfo
            echo -e "UIoT_Edge_Reset=false" >> ${full_path}/etc/deviceinfo
            return 0
        fi
    elif [ "$UIoT_Edge_Reset" = "true" ]; then
        sed -i "s/^.*UIOT_EDGE_REGION.*$/export UIOT_EDGE_REGION=$UIoT_Edge_Region/" ${full_path}/etc/profile
        config_device_info $UIoT_Edge_ProductSN $UIoT_Edge_DeviceSN $UIoT_Edge_Password
        sed -i "s/UIoT_Edge_Reset=true/UIoT_Edge_Reset=false/" ${full_path}/etc/deviceinfo
        return 0
    fi
    return 0
}

restart() {
    source ${full_path}/etc/profile
    if [ "$preflag" = "sudo" ]; then
        preflag="sudo env PATH=$PATH PYTHONPATH=$PYTHONPATH TMPDIR=$TMPDIR LD_LIBRARY_PATH=$LD_LIBRARY_PATH SSL_CERT_FILE=$SSL_CERT_FILE SSL_CERT_DIR=$SSL_CERT_DIR"
    fi

    if [ -f ${full_path}/etc/deviceinfo ]; then
        source ${full_path}/etc/deviceinfo
        if [ "$UIoT_Edge_OneClick_Mode" = "true" ]; then
            validate_hw_serial
        fi
    fi

    ${preflag} supervisorctl -s unix://${full_path}/tmp/ucloud-edge-supervisor.sock restart all
}

start() {
    source ${full_path}/etc/profile
    if [ "$preflag" = "sudo" ]; then
        preflag="sudo env PATH=$PATH PYTHONPATH=$PYTHONPATH TMPDIR=$TMPDIR LD_LIBRARY_PATH=$LD_LIBRARY_PATH SSL_CERT_FILE=$SSL_CERT_FILE SSL_CERT_DIR=$SSL_CERT_DIR"
    fi

    if [ -f ${full_path}/etc/deviceinfo ]; then
        source ${full_path}/etc/deviceinfo
        if [ "$UIoT_Edge_OneClick_Mode" = "true" ]; then
            validate_hw_serial
        fi
    fi

    eval $($pscmd | grep -w "supervisord -c ${full_path}/etc/supervisor/supervisord.conf" | grep -v grep | awk '{printf("Pid=%s",$'$pid_line')}')
    if [ "$Pid" = "" ];then
        ${preflag} supervisord -c ${full_path}/etc/supervisor/supervisord.conf
        sleep 5s

        for i in $(seq 1 10) 
        do 
            if [ ! -e "${full_path}/tmp/ucloud-edge-supervisor.sock" ];then
                if [ $i == 10 ] ; then
                    echo "supervisor start failed"
                    exit 1;
                else
                    sleep 0.5
                fi
            else
                break
            fi
        done
    else
        echo "supervisor started"
        ${preflag} supervisorctl -s unix://${full_path}/tmp/ucloud-edge-supervisor.sock start all
    fi

    for service in "nats-server" "frpc" "edge" "redis-server"
    do
        eval $(${preflag} supervisorctl -s unix://${full_path}/tmp/ucloud-edge-supervisor.sock status $service | grep -v grep | awk '{printf("Stat=%s",$2)}')
        echo $Stat
        if [ "$Stat" == "RUNNING" ] ; then
            echo "$service started"
        elif [ "$Stat" == "STARTING" ]; then
            echo "$service starting"
        else
            echo "$service failed"
            exit 1;
        fi
    done
    echo "-------start completed-------"
    
}

status() {
    echo "----------------------------------UCLOUD EDGE STATUS----------------------------------------"
    for service in "nats-server" "uagent" "frpc" "redis-server" "webportal" "logservice" "deploy-agent" "function-manager"
    do
        eval $($pscmd| grep -w $service | grep -v grep| grep -v ${service}-log | grep -v uiotedge.log| awk  '{printf("Pid=%s",$'$pid_line')}')
        if [ "$Pid" = "" ]; then
            printf "[--\033[31m+\033[0m--]  Service %-1s is \033[31minactive\033[0m\n" $service; 
        else
            printf "[--\033[32m+\033[0m--]  Service %-1s is \033[32mactive\033[0m,PID is %-1s\n" $service $Pid;   
        fi
        Pid="" 
    done
    echo "--------------------------------------------------------------------------------------------"
}

stop() {
    if [ $1 ] && [ $1 != "all" ]; then        
        print_warn "param mismatched!"
        exit 1;
    fi
    source ${full_path}/etc/profile
    if [ "$preflag" = "sudo" ]; then
        preflag="sudo env PATH=$PATH PYTHONPATH=$PYTHONPATH TMPDIR=$TMPDIR LD_LIBRARY_PATH=$LD_LIBRARY_PATH SSL_CERT_FILE=$SSL_CERT_FILE SSL_CERT_DIR=$SSL_CERT_DIR"
    fi
    
 
    if [ ! $1 ]; then
        for service in "nats-server" "logservice" "edge" "redis-server"
            do
                eval $($pscmd| grep -w $service | grep -v grep| grep -v ${service}-log | awk  '{printf("Pid=%s",$'$pid_line')}')
                ${preflag} supervisorctl -s unix://${full_path}/tmp/ucloud-edge-supervisor.sock stop $service
            done
    elif [ $1 == "all" ]; then
        eval $($pscmd | grep -w "supervisord -c ${full_path}/etc/supervisor/supervisord.conf" | grep -v grep | awk '{printf("Pid=%s",$'$pid_line')}')
        ${preflag} supervisorctl -s unix://${full_path}/tmp/ucloud-edge-supervisor.sock stop all
        if [ $Pid != "" ]; then
            ${preflag} kill -15 $Pid
            Pid=""
        fi
    else
        print_warn "param mismatched!"
        exit 1;
    fi

    while true
    do 
        eval $($pscmd | grep -w "edge start" | grep -v grep | awk '{printf("Pid=%s",$'$pid_line')}')
        if [ "$Pid" = "" ];then
            Pid=""
            break
        else
            Pid=""
        fi
    done

    echo "stop all servers completed"
}

upgrade() {
    source ${full_path}/etc/profile
    if [ "$preflag" = "sudo" ]; then
        preflag="sudo env PATH=$PATH PYTHONPATH=$PYTHONPATH TMPDIR=$TMPDIR LD_LIBRARY_PATH=$LD_LIBRARY_PATH SSL_CERT_FILE=$SSL_CERT_FILE SSL_CERT_DIR=$SSL_CERT_DIR"
    fi
    set_variable_of_region $UIOT_EDGE_REGION

    if [ "$1" = "webportal" ]; then
        if [ ! -d "$master_work_path/.bak" ]; then
            mkdir -p $master_work_path/.bak
        fi
        if [ ! -d "$master_work_path/.bak/.webportal" ]; then
            mkdir -p $master_work_path/.bak/.webportal
        fi
        mv $master_work_path/webportal $master_work_path/.bak/.webportal
        mkdir -p $master_work_path/webportal
        wget -P ${full_path}/download $BUCKET_ADDRESS/$UIOT_EDGE_VERSION/webportal-linux-$UIOT_EDGE_ARCH.zip
        if [ -f "${full_path}/download/webportal-linux-$UIOT_EDGE_ARCH.zip" ]; then
            rm -rf $master_work_path/.bak/.webportal
            unzip ${full_path}/download/webportal-linux-$UIOT_EDGE_ARCH.zip -d $master_work_path/webportal
            rm -rf ${full_path}/download/webportal-linux-$UIOT_EDGE_ARCH.zip
        else
            echo "wget failed! Cause the upgrade to fail"
            mv $master_work_path/.bak/.webportal/webportal $master_work_path/
        fi

        eval $($pscmd| grep -w $1 | grep -v grep| grep -v ${1}-log | grep -v upgrade | grep -v uiotedge.log | awk  '{printf("Pid=%s",$'$pid_line')}')
        ${preflag} kill -15 $Pid
        Pid=""
    elif [ "$1" = "uagent" ]; then
        if [ ! -d "$master_work_path/.bak" ]; then
            mkdir -p $master_work_path/.bak
        fi
        if [ ! -d "$master_work_path/.bak/.uagent" ]; then
            mkdir -p $master_work_path/.bak/.uagent
        fi
        mv $master_work_path/uagent $master_work_path/.bak/.uagent
        mkdir -p $master_work_path/uagent
        wget -P ${full_path}/download $BUCKET_ADDRESS/$UIOT_EDGE_VERSION/uagent-linux-$UIOT_EDGE_ARCH.zip
        if [ -f "${full_path}/download/uagent-linux-$UIOT_EDGE_ARCH.zip" ]; then
            rm -rf $master_work_path/.bak/.uagent
            unzip ${full_path}/download/uagent-linux-$UIOT_EDGE_ARCH.zip -d $master_work_path/uagent
            rm -rf ${full_path}/download/uagent-linux-$UIOT_EDGE_ARCH.zip
        else
            echo "wget failed! Cause the upgrade to fail"
            mv $master_work_path/.bak/.uagent/uagent $master_work_path/
        fi

        eval $($pscmd| grep -w $1 | grep -v grep| grep -v ${1}-log | grep -v upgrade | grep -v uiotedge.log | awk  '{printf("Pid=%s",$'$pid_line')}')
        ${preflag} kill -15 $Pid
        Pid=""
    elif [ "$1" = "logservice" ]; then
        if [ ! -d "${full_path}/bin/.bak" ]; then
            mkdir -p ${full_path}/bin/.bak
        fi
        if [ ! -d "${full_path}/bin/.bak/.logservice" ]; then
            mkdir -p ${full_path}/bin/.bak/.logservice
        fi
        mv ${full_path}/bin/logservice ${full_path}/bin/.bak/.logservice
        wget -O logservice.zip $BUCKET_ADDRESS/$UIOT_EDGE_VERSION/logservice-linux-$UIOT_EDGE_ARCH.zip
        if [ -f "logservice.zip" ]; then
            unzip logservice.zip -d ${full_path}/bin
            rm -fr logservice.zip
            rm -rf ${full_path}/bin/.bak/.logservice
        else
            echo "wget failed! Cause the upgrade to fail"
            mv ${full_path}/bin/.bak/.logservice/logservice  ${full_path}/bin
        fi

        eval $($pscmd| grep -w $1 | grep -v grep| grep -v ${1}-log | grep -v upgrade | awk  '{printf("Pid=%s",$'$pid_line')}')
        ${preflag} kill -15 $Pid
        Pid=""
    elif [ "$1" = "frpc" ]; then
        if [ ! -d "${full_path}/bin/.bak" ]; then
            mkdir -p ${full_path}/bin/.bak
        fi
        if [ ! -d "${full_path}/bin/.bak/.frpc" ]; then
            mkdir -p ${full_path}/bin/.bak/.frpc
        fi
        mv ${full_path}/bin/frpc ${full_path}/bin/.bak/.frpc
        
        if [ -f "${full_path}/.bak/.frpc.zip" ]; then
            rm -rf ${full_path}/.bak/.frpc.zip
        fi
        wget -O ${full_path}/.bak/.frpc.zip $BUCKET_ADDRESS/$UIOT_EDGE_VERSION/frpc-linux-$UIOT_EDGE_ARCH.zip
        if [ -f "${full_path}/.bak/.frpc.zip" ]; then
            unzip -q ${full_path}/.bak/.frpc.zip -d ${full_path}
            mv ${full_path}/frpc/frpc ${full_path}/bin
            rm -rf ${full_path}/frpc
            rm -rf ${full_path}/bin/.bak/.frpc
        else
            echo "wget failed! Cause the upgrade to fail"
            mv ${full_path}/bin/.bak/.frpc/frpc ${full_path}/bin
        fi
        eval $($pscmd| grep -w $1 | grep -v grep| grep -v ${1}-log | grep -v upgrade | grep -v uiotedge.log | awk  '{printf("Pid=%s",$'$pid_line')}')
        ${preflag} kill -15 $Pid
        Pid=""
    else
        print_warn "param mismatched!"
        exit 1;
    fi
}

main() {
    input_param=$1

    if [ "$input_param" = "--install" ]; then
        if [ $# != 4 ]; then
            usageFunc
        else
            install $2 $3 $4
        fi
    elif [ "$input_param" = "--config" ]; then
        if [ $# != 4 ] && [ $# != 5 ] && [ $# != 1 ] && [ $# != 2 ]; then
            usageFunc
        else
            config $2 $3 $4 $5
        fi
    elif [ "$input_param" = "--start" ]; then
        if [ $# != 1 ]; then
            usageFunc 
        else
            start
        fi
    elif [ "$input_param" = "--status" ]; then
        if [ $# != 1 ]; then
            usageFunc
        else
            status
        fi
    elif [ "$input_param" = "--stop" ]; then
        if [ $# == 1 ]; then
            stop
        elif [ $# == 2 ]; then
            stop $2
        else
            usageFunc
        fi
    elif [ "$input_param" = "--restart" ]; then
        if [ $# != 1 ]; then
            usageFunc
        else
            restart
        fi
    elif [ "$input_param" = "--upgrade" ]; then
        if [ $# == 2 ]; then
            upgrade $2
        else
            usageFunc
        fi
    else
        usageFunc
    fi
}

main $@
```
