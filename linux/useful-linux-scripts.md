---
abbrlink: 3898548677
alias: 2019/03/30/useful-linux-scripts/index.html
categories:
- linux
date: '2019-03-30T01:13:35'
description: ''
tags:
- Linux
- Bash
title: 常用Linux脚本集锦
---









常用linux脚本：

- Linux系统bash脚本获取系统硬件资源
- Aix系统ksh脚本获取系统硬件资源
- 服务器打开session时显示帮助信息
- curl post中带有变量的脚本
- 金三系统基线包安装部署脚本

<!--more-->

## Linux系统bash脚本获取系统硬件资源

```bash
#!/bin/sh
echo "系统资源使用情况：以下各个数字均为百分比%"
echo "CPU平均使用率："
top -n 3 b|grep 'Cpu(s)'|awk '{print $2}'|cut -d '%' -f 1|awk '{sum+= $1} END {printf "%.2f\n",sum/3}'
echo "内存使用率："
free|grep 'Mem:'|awk '{realused=$3-$6-$7} END {printf "%.2f\n",realused*100/$2}'
echo "磁盘使用率："
df|awk '{if(length($6)==1) print $5}'|cut -d '%' -f 1
echo "Inode使用率："
df -i|awk '{if(length($6)==1) print $5}'|cut -d '%' -f 1
```

## Aix系统ksh脚本获取系统硬件资源

```shell
#!/bin/sh
echo "以下数据均为百分比"
echo "cpu"
vmstat 1 3|sed -n '7,9p'|awk '{free+=$16} END {printf "%.2f\n",100-free/3.0}'
echo "memory"
svmon|grep memory|awk '{printf "%.2f\n",1.0*$3/$2}'
echo "disk"
df /u01|sed -n 2p|awk '{print $4}'|cut -d '%' -f 1
echo "Inode"
df /u01|sed -n 2p|awk '{print $6}'|cut -d '%' -f 1
```

## 服务器打开session时显示帮助信息

将脚本的执行语句写在用户profile中，登录时即可看到帮助信息

脚本名称：help.sh

```bash
tput setaf 2
tput blink
echo "====================测试服务器133======================"
tput sgr0
tput setaf 2
echo "
--------------**支持dt,hx,sj,js---------------------
 log-gs**.sh            查看日志
 re-gs**.sh             重启应用
--------------**支持dt,hx,sj,js---------------------

 *****    help.sh       呼唤帮助信息      *****"
tput blink  
echo "====================测试服务器133======================"
tput sgr0
```

## curl post中带有变量的脚本

```bash
#!/bin/bash
username="suncle"
apiKey="xxxxxx"
domain=$1
echo $domain
date=`date -R -u | awk '{print $1" "$2" "$3" "$4" "$5" GMT"}'`
password=`echo -en "$date" | openssl dgst -sha1 -hmac $apiKey -binary | openssl enc -base64`
curl -i --url "https://open.chinanetcenter.com/api/report/flow/stream/detail" \
-X "POST" \
-u $username:$password \
-H "Date: $date" \
-H "Accept: application/json" \
-d '{
    "dateFrom": "2018-03-01T00:00:00+08:00",
    "dateTo": "2018-03-02T00:00:00+08:00",
    "domainStream": [
        {
            "domain": "'"$domain"'"
        }
    ],
    "dataType": "bandwidth",
    "dataInterval": "5m"
}'
```

> 参见domain变量的处理："domain": "'"$domain"'"

## 金三系统基线包安装部署脚本

**判断weblogic用户和组是否存在，判断/weblogic目录是否存在**

> 脚本名称：install-gs-1-init.sh

```bash
#!/bin/sh

USER_NAME="weblogic"
GROUP_NAME="weblogic"

if [ "$(whoami)" != "root" ]; then
echo -e "\033[31m当前登录用户不是root用户，请使用root用户操作。\033[0m";exit 1
fi

echo -e "\033[31m该脚本需在所有应用服务器执行。\033[0m"

testing1=$(egrep "^${USER_NAME}:" /etc/passwd)
if [ "${testing1}" != "" ]; then
    GROUP_ID=$(egrep "^${USER_NAME}:" /etc/passwd|cut -d ":" -f 4)
    GROUP_NAME=$(egrep ":${GROUP_ID}:" /etc/group|cut -d ":" -f 1)
    echo -e "\033[32m用户${USER_NAME}已存在，用户组为${GROUP_NAME}。\033[0m"
else
    testing2=$(egrep "^${GROUP_NAME}:" /etc/group)
    if [ "$testing2" != "" ]; then
        useradd -g ${GROUP_NAME} ${USER_NAME}
        echo -e "\033[32m用户组${GROUP_NAME}存在，但用户${USER_NAME}不存在，已创建。\033[0m"
    else
        groupadd ${GROUP_NAME};useradd -g ${GROUP_NAME} ${USER_NAME}
        echo -e "\033[32m用户组${GROUP_NAME}和用户${USER_NAME}都不存在，均已创建。\033[0m"
    fi
fi

if test -d /weblogic; then
    CURRENT_TIME=$(date +%Y%m%d%H%M%S)
    mv /weblogic /weblogic_${CURRENT_TIME}
    mkdir -p /weblogic;chown -R ${USER_NAME}:${GROUP_NAME} /weblogic
    echo -e "\033[31m目录/weblogic已存在，自动备份为/weblogic_${CURRENT_TIME}，并重新创建空目录/weblogic。\033[0m"
else
    mkdir -p /weblogic;chown -R ${USER_NAME}:${GROUP_NAME} /weblogic
    echo -e "\033[32m目录/weblogic不存在，已创建。\033[0m"
fi

echo -e "\033[31m请确认已设置weblogic用户的密码。\033[0m"
```

**解压安装weblogic中间件和jdk并且设置环境变量**

> 脚本名称：install-gs-2-base.sh

```bash
#!/bin/sh

INITIAL_PATH="/tmp/weblogic_initial"

if [ "$(whoami)" != "weblogic" ]; then
    echo -e "\033[31m当前登录用户不是weblogic用户，请使用weblogic用户操作。\033[0m";exit 1
fi

echo -e "\033[31m该脚本需在所有应用服务器执行。\033[0m"

if test -d ${INITIAL_PATH}; then
    CURRENT_TIME=$(date +%Y%m%d%H%M%S)
    mv ${INITIAL_PATH} ${INITIAL_PATH}_${CURRENT_TIME}&&mkdir -p ${INITIAL_PATH}&&echo -e "\033[31m目录${INITIAL_PATH}已存在，自动备份为${INITIAL_PATH}_${CURRENT_TIME}，并重新创建空目录${INITIAL_PATH}。\033[0m"
else
    mkdir -p ${INITIAL_PATH}&&echo -e "\033[32m目录${INITIAL_PATH}不存在，已创建。\033[0m"
fi

echo -e "请将基础软件包weblogic_base_*.tar.bz2上传到该服务器${INITIAL_PATH}目录下。\033[31m请使用weblogic用户上传。\033[0m"
echo -e "\033[31m上传完毕后，\033[0m请按任意键继续..."
read -n 1 var

num=$(ls ${INITIAL_PATH}/weblogic_base_*.tar.bz2|wc -l)
while true
do
    echo -e "检测到【${num}】个基础软件包${INITIAL_PATH}/weblogic_base_*.tar.bz2"
    if [ ${num} -eq 0 ]; then
        echo -e "\033[31m请确认是否已正确上传。\033[0m"
    elif [ ${num} -gt 1 ]; then
        echo -e "\033[31m请移除多余的base包。\033[0m"
    else
        base_package=$(ls ${INITIAL_PATH}/weblogic_base_*.tar.bz2)
        echo -e "包名为\033[32m${base_package}\033[0m"
        read -p "请确认包名是否正确？正确输入Y，否则输入N。" yn
        if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
            yn=
            break
        else
            echo -e "\033[32m输入不是Y，包名错误，需要重新上传。\033[0m"
            yn=
        fi
    fi
    echo -e "\033[31m处理完毕后，\033[0m请按任意键继续..."
    read -n 1 var
    num=$(ls ${INITIAL_PATH}/weblogic_base_*.tar.bz2|wc -l)
done

echo -e "开始解压安装包..."
tar -xjvf ${base_package} -C /
echo -e "\033[32m解压base包到/weblogic完毕。\033[0m"

mkdir -p /weblogic/user_projects/domains
mkdir -p /weblogic/user_projects/scripts

if [ "~" != "/weblogic" ]; then
    cp /weblogic/.bash_profile ~/.bash_profile
    echo -e "\033[31m用户环境变量已经更新，请执行命令source ~/.bash_profile使环境变量生效。\033[0m"
fi
```

根据具体服务器情况抽取相应的应用到各台服务器

> 脚本名称：install-gs-3-gsap.sh

```bash
#!/bin/sh

INITIAL_PATH="/tmp/weblogic_initial"

if [ "$(whoami)" != "weblogic" ]; then
    echo -e "\033[31m当前登录用户不是weblogic用户，请使用weblogic用户操作。\033[0m";exit 1
fi

echo -e "\033[31m该脚本只需在【**一台[可任选]**】应用服务器执行。\n\033[0m"
echo -e "\033[31m请确认【**所有**】应用服务器都已经成功执行过脚本1和脚本2？\033[0m"
read -p "已执行过输入Y，否则输入N。" yn
if [ "$yn" != "Y" -a "$yn" != "y" ]; then
    yn=
    echo -e "\033[31m输入不是Y，请先在所有服务器都执行脚本1和脚本2。\033[0m"
    exit 1
fi

if test -d ${INITIAL_PATH}/user_projects; then
    CURRENT_TIME=$(date +%Y%m%d%H%M%S)
    mv ${INITIAL_PATH}/user_projects ${INITIAL_PATH}/user_projects_${CURRENT_TIME}
    echo -e "\033[31m目录${INITIAL_PATH}/user_projects已存在，自动备份为${INITIAL_PATH}/user_projects_${CURRENT_TIME}。\033[0m"
fi

echo -e "请将个税应用包weblogic_gsap_*.tar.bz2上传到该服务器${INITIAL_PATH}目录下"
echo -e "\033[31m上传完毕后，\033[0m请按任意键继续..."
read -n 1 var

num=$(ls ${INITIAL_PATH}/weblogic_gsap_*.tar.bz2|wc -l)
while true
do
    echo -e "检测到【${num}】个个税应用包${INITIAL_PATH}/weblogic_gsap_*.tar.bz2"
    if [ ${num} -eq 0 ]; then
        echo -e "\033[31m请确认是否已正确上传。\033[0m"
    elif [ ${num} -gt 1 ]; then
        echo -e "\033[31m请移除多余的gsap包。\033[0m"
    else
        gsap_package=$(ls ${INITIAL_PATH}/weblogic_gsap_*.tar.bz2)
        echo -e "包名为\033[32m${gsap_package}\033[0m"
        read -p "请确认包名是否正确？正确输入Y，否则输入N。" yn
        if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
            yn=
            break
        else
            echo -e "\033[32m输入不是Y，包名错误，需要重新上传。\033[0m"
            yn=
        fi
    fi
    echo -e "\033[31m处理完毕后，\033[0m请按任意键继续..."
    read -n 1 var
    num=$(ls ${INITIAL_PATH}/weblogic_gsap_*.tar.bz2|wc -l)
done

echo -e "开始解压安装包..."
tar -xjvf ${gsap_package} -C ${INITIAL_PATH}
echo -e "\033[32m解压gsap包到${INITIAL_PATH}完毕。\033[0m"


echo -e "\033[32m开始安装管理节点应用：\n\033[0m"
while true
do
    read -p "请输入【管理节点】所在服务器的IP：" IP_gl
    echo -e "输入的\033[32m【管理节点】\033[0m所在服务器${i}的IP为：\033[32m【${IP_gl}】\033[0m" 
    read -p "请确认是否正确？正确输入Y，否则输入N。" yn
    if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
        yn=
        break
    else
        echo -e "\033[32m输入不是Y，设置有误，需要重新输入。\033[0m"
        yn=
    fi
done
echo -e "【管理节点】所在服务器(IP：${IP_gl})包含应用为："
echo -e "个税管理：gsadmin_domain"
echo -e "开始抽取管理节点domain到${IP_gl}/weblogic/user_projects/domains，请耐心等待。"
scp -r ${INITIAL_PATH}/user_projects/domains/gsadmin_domain ${IP_gl}:/weblogic/user_projects/domains
scp -r ${INITIAL_PATH}/user_projects/scripts/*-gsadmin.sh ${INITIAL_PATH}/user_projects/scripts/upgrade-*.sh ${IP_gl}:/weblogic/user_projects/scripts
echo -e "\033[32m管理节点应用抽取完毕。\n\033[0m"


echo -e "\033[32m开始安装前端应用：\n\033[0m"
while true
do
    read -p "请输入【前端】服务器台数(注意：只能是1/2/3/4四种情况之一)：" number_qd
    if [ "$number_qd" != "1" -a "$number_qd" != "2" -a "$number_qd" != "3" -a "$number_qd" != "4" ]; then
        echo -e "输入服务器台数\033[31m【$number_qd】\033[0m，输入有误，需要重新输入。"
        continue
    fi
    echo -e "设置【前端】服务器\033[32m【$number_qd】\033[0m台。"
    read -p "请确认设置是否正确？正确输入Y，否则输入N。" yn
    if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
        yn=
        break
    else
        echo -e "\033[32m输入不是Y，设置有误，需要重新输入。\033[0m"
        yn=
        continue
    fi
done

for (( i=1; i<=$number_qd; i=i+1 ))
do
	while true
    do
        read -p "请输入【前端】服务器${i}的IP：" IP_qd
        echo -e "输入的【前端】服务器${i}的IP为：\033[32m【${IP_qd}】\033[0m" 
        read -p "请确认是否正确？正确输入Y，否则输入N。" yn
        if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
            yn=
            break
        else
            echo -e "\033[32m输入不是Y，设置有误，需要重新输入。\033[0m"
            yn=
        fi
    done
    echo -e "【前端】服务器${i} (IP：${IP_qd})包含应用为："
    echo -e "个税大厅：gsdt0$[ i ]_domain、gsdt0$[ i + number_qd ]_domain；查询统计：cxtj0$[ i ]_domain；"
    echo -e "开始抽取需要的domain到${IP_qd}:/weblogic/user_projects/domains，请耐心等待。"
	scp -r ${INITIAL_PATH}/user_projects/domains/gsdt0$[ i ]_domain ${INITIAL_PATH}/user_projects/domains/gsdt0$[ i + number_qd ]_domain  ${INITIAL_PATH}/user_projects/domains/cxtj0$[ i ]_domain ${IP_qd}:/weblogic/user_projects/domains
	scp -r ${INITIAL_PATH}/user_projects/scripts/*-gsdt.sh ${INITIAL_PATH}/user_projects/scripts/*-cxtj.sh  ${IP_gl}:/weblogic/user_projects/scripts
    echo -e "\033[32m前端应用抽取完毕。\n\033[0m"
done


echo -e "\033[32m开始安装后端应用：\n\033[0m"
while true
do
    read -p "请输入【后端】服务器台数(注意：只能是1/2/3/4四种情况之一)：" number_hd
    if [ "$number_hd" != "1" -a "$number_hd" != "2" -a "$number_hd" != "3" -a "$number_hd" != "4" ]; then
        echo -e "输入服务器台数\033[31m【$number_hd】\033[0m，输入有误，需要重新输入。"
        continue
    fi   
    echo -e "设置【后端】服务器\033[32m【$number_hd】\033[0m台。"
    read -p "请确认设置是否正确？正确输入Y，否则输入N。" yn
    if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
        yn=
        break
    else
        echo -e "\033[32m输入不是Y，设置有误，需要重新输入。\033[0m"
        yn=
        continue
    fi
done

for (( i=1; i<=$number_hd; i=i+1 ))
do
	while true
    do
        read -p "请输入【后端】服务器${i}的IP：" IP_hd
        echo -e "输入的【后端】服务器${i}的IP为：\033[32m【${IP_hd}】\033[0m" 
        read -p "请确认是否正确？正确输入Y，否则输入N。" yn
        if [ "$yn" == "Y" -o  "$yn" == "y" ]; then
            yn=
            break
        else
            echo -e "\033[32m输入不是Y，设置有误，需要重新输入。\033[0m"
            yn=
        fi
    done
    echo -e "【后端】服务器${i} (IP：${IP_hd})包含应用为："
    echo -e "个税核心：gshx0$[ i ]_domain、gshx0$[ i + number_hd ]_domain；个税工作流：gswf0$[ i ]_domain；个税间接登记：gsjjdj0$[ i ]_domain"
    echo -e "开始抽取需要的domain到${IP_hd}:/weblogic/user_projects/domains，请耐心等待。"
	scp -r ${INITIAL_PATH}/user_projects/domains/gshx0$[ i ]_domain ${INITIAL_PATH}/user_projects/domains/gshx0$[ i + number_hd ]_domain ${INITIAL_PATH}/user_projects/domains/gswf0$[ i ]_domain ${INITIAL_PATH}/user_projects/domains/gsjjdj0$[ i ]_domain ${IP_hd}:/weblogic/user_projects/domains
	scp -r ${INITIAL_PATH}/user_projects/scripts/*-gshx.sh ${INITIAL_PATH}/user_projects/scripts/*-gswf.sh ${INITIAL_PATH}/user_projects/scripts/*-gsjjdj.sh ${IP_gl}:/weblogic/user_projects/scripts
    echo -e "\033[32m后端应用抽取完毕。\n\033[0m"
done

echo -e "\033[32m安装完毕。\n\033[0m"
echo -e "\033[31m请根据说明文档继续手工修改某些配置。\n\033[0m"

```

