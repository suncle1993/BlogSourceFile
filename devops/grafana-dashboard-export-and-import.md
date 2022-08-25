---
categories:
  - devops
date: '2019-11-15T16:57:54'
description: ''
tags:
  - Grafana
  - Prometheus
  - 监控
title: Grafana Dashboard 配置导入导出
---




我司最近的一件大事是AWS迁移阿里云，因此之前部署在AWS上的Prometheus监控系统需要迁移到Aliyun机器上，组件之一展示面板Grafana有很多自定义好的配置，而这些配置是存在于grafana.db中的，因此需要导出导入配置到新的db中。Grafana提供了丰富的api供用户使用，我们调用api导出

## 导出配置

首先获取Grafana的api_key：menu--configuration--api keys -- add API key

然后安装jq：命令行下处理JSON 数据的工具，可以对json数据进行分片、过滤、映射和转换

<!--more-->

```shell
sudo apt install jq
```

导出脚本grafana-dashboard-exporter.sh参考：https://gist.github.com/crisidev/bd52bdcc7f029be2f295

```bash
#/bin/bash
#
# add the "-x" option to the shebang line if you want a more verbose output
#
# set some colors for status OK, FAIL and titles
SETCOLOR_SUCCESS="echo -en \\033[0;32m"
SETCOLOR_FAILURE="echo -en \\033[1;31m"
SETCOLOR_NORMAL="echo -en \\033[0;39m"
SETCOLOR_TITLE_PURPLE="echo -en \\033[0;35m" # purple

# usage log "string to log" "color option"
function log_success() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       echo "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   timestamp=$(date "+%Y-%m-%d %H:%M:%S %Z")

   ${SETCOLOR_SUCCESS}
   printf "[${timestamp}] $1\n"
   ${SETCOLOR_NORMAL}
}

function log_failure() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       echo "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   timestamp=$(date "+%Y-%m-%d %H:%M:%S %Z")

   ${SETCOLOR_FAILURE}
   printf "[${timestamp}] $1\n"
   ${SETCOLOR_NORMAL}
}

function log_title() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       log_failure "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   ${SETCOLOR_TITLE_PURPLE}
   printf "|-------------------------------------------------------------------------|\n"
   printf "|$1|\n";
   printf "|-------------------------------------------------------------------------|\n"
   ${SETCOLOR_NORMAL}
}

function init() {
   # Check if hostname and key are provided
   if [ $1 -lt 2 ]; then
       ${SETCOLOR_FAILURE}
       log_failure "Not enough command line arguments! Expecting two: \$HOSTNAME and \$KEY. Recieved only $1."
       exit 1
   fi

   DASH_DIR=$(echo $HOST | awk -F[/:] '{print $4}')

   if [ ! -d "${DASH_DIR}" ]; then
   	 mkdir "${DASH_DIR}"
   else
   	log_title "----------------- A $DASH_DIR directory already exists! -----------------"
   fi
}


HOST=$1
KEY=$2
init $# $HOST $KEY

counter=0

for dashboard_uid in $(curl -sS -H "Authorization: Bearer $KEY" $HOST/api/search\?query\=\& | jq -r '.[] | select( .type | contains("dash-db")) | .uid'); do

   counter=$((counter + 1))
   url=`echo $HOST/api/dashboards/uid/$dashboard_uid | tr -d '\r'`
   dashboard_json=$(curl -sS -H "Authorization: Bearer $KEY" $url)
   dashboard_title=$(echo $dashboard_json | jq -r '.dashboard | .title' | sed -r 's/[ \/]+/_/g' )
   dashboard_version=$(echo $dashboard_json | jq -r '.dashboard | .version')
   folder_title=$(echo $dashboard_json | jq -r '.meta | .folderTitle')

   mkdir -p "$DASH_DIR/$folder_title"
   echo $dashboard_json > "$DASH_DIR/$folder_title/${dashboard_title}_v${dashboard_version}.json"

   log_success "Dashboard has been saved\t\t title=\"${dashboard_title}\", uid=\"${dashboard_uid}\", path=\"${DASH_DIR}/$folder_title/${dashboard_title}_v${dashboard_version}.json\"."
done

log_title "${counter} dashboards were saved";

log_title "------------------------------ FINISHED ---------------------------------";

```

使用方式（已脱敏）：

```bash
bash grafana-dashboard-exporter.sh http://localhost:3000 eyJrIjoiZkhiQUVDczNFT0QyUVE4YkJwb1RBbHasfaefaefdDVXT1dkVlkiLCJuIjoiemFpaHVpX2FXkiLCJpZCI6MX0=
```

## 导入配置

- 老版本：使用api_key导入
- 新版本（v5.0之后）：使用dockprom的setup.sh启动时导入

### 使用api导入

如果在目标机器上的Grafana已经启动并且可以访问Dashboard拿到api_key，则可以按照以下方式导入配置

导入脚本grafana-dashboard-importer.sh参考：https://gist.github.com/thedoc31/628beeee934f9c84648c108d4ad89f05

```bash
#!/bin/bash
#
# add the "-x" option to the shebang line if you want a more verbose output
#
#
OPTSPEC=":hp:t:k:"

show_help() {
cat << EOF
Usage: $0 [-p PATH] [-t TARGET_HOST] [-k API_KEY]
Script to import dashboards into Grafana
    -p      Required. Root path containing JSON exports of the dashboards you want imported.
    -t      Required. The full URL of the target host
    -k      Required. The API key to use on the target host

    -h      Display this help and exit.
EOF
}

###### Check script invocation options ######
while getopts "$OPTSPEC" optchar; do
    case "$optchar" in
        h)
            show_help
            exit
            ;;
        p)
            DASH_DIR="$OPTARG";;
        t)
            HOST="$OPTARG";;
        k)
            KEY="$OPTARG";;
        \?)
          echo "Invalid option: -$OPTARG" >&2
          exit 1
          ;;
        :)
          echo "Option -$OPTARG requires an argument." >&2
          exit 1
          ;;
    esac
done

if [ -z "$DASH_DIR" ] || [ -z "$HOST" ] || [ -z "$KEY" ]; then
    show_help
    exit 1
fi

# set some colors for status OK, FAIL and titles
SETCOLOR_SUCCESS="echo -en \\033[0;32m"
SETCOLOR_FAILURE="echo -en \\033[1;31m"
SETCOLOR_NORMAL="echo -en \\033[0;39m"
SETCOLOR_TITLE_PURPLE="echo -en \\033[0;35m" # purple

# usage log "string to log" "color option"
function log_success() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       echo "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   timestamp=$(date "+%Y-%m-%d %H:%M:%S %Z")

   ${SETCOLOR_SUCCESS}
   printf "[%s] $1\n" "$timestamp"
   ${SETCOLOR_NORMAL}
}

function log_failure() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       echo "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   timestamp=$(date "+%Y-%m-%d %H:%M:%S %Z")

   ${SETCOLOR_FAILURE}
   printf "[%s] $1\n" "$timestamp"
   ${SETCOLOR_NORMAL}
}

function log_title() {
   if [ $# -lt 1 ]; then
       ${SETCOLOR_FAILURE}
       log_failure "Not enough arguments for log function! Expecting 1 argument got $#"
       exit 1
   fi

   ${SETCOLOR_TITLE_PURPLE}
   printf "|-------------------------------------------------------------------------|\n"
   printf "|%s|\n" "$1";
   printf "|-------------------------------------------------------------------------|\n"
   ${SETCOLOR_NORMAL}
}

if [ -d "$DASH_DIR" ]; then
    DASH_LIST=$(find "$DASH_DIR" -mindepth 1 -name \*.json)
    if [ -z "$DASH_LIST" ]; then
        log_title "----------------- $DASH_DIR contains no JSON files! -----------------"
        log_failure "Directory $DASH_DIR does not appear to contain any JSON files for import. Check your path and try again."
        exit 1
    else
        FILESTOTAL=$(echo "$DASH_LIST" | wc -l)
        log_title "----------------- Starting import of $FILESTOTAL dashboards -----------------"
    fi
else
    log_title "----------------- $DASH_DIR directory not found! -----------------"
    log_failure "Directory $DASH_DIR does not exist. Check your path and try again."
    exit 1
fi

NUMSUCCESS=0
NUMFAILURE=0
COUNTER=0

for DASH_FILE in $DASH_LIST; do
    COUNTER=$((COUNTER + 1))
    echo "Import $COUNTER/$FILESTOTAL: $DASH_FILE..."
    RESULT=$(cat "$DASH_FILE" | jq '. * {overwrite: true, dashboard: {id: null}}' | curl -s -X POST -H "Content-Type: application/json" -H "Authorization: Bearer $KEY" "$HOST"/api/dashboards/db -d @-)
    if [[ "$RESULT" == *"success"* ]]; then
        log_success "$RESULT"
        NUMSUCCESS=$((NUMSUCCESS + 1))
    else
        log_failure "$RESULT"
        NUMFAILURE=$((NUMFAILURE + 1))
    fi
done

log_title "Import complete. $NUMSUCCESS dashboards were successfully imported. $NUMFAILURE dashboard imports failed.";
log_title "------------------------------ FINISHED ---------------------------------";

```

使用方式:

```bash
bash grafana-dashboard-importer.sh -t http:localhost:3000 -k eyJrIjoiZkhiQUVDczNFT0QyUaouefohaoeihoaefhaoiefiomFpaHVpX2FwaV9rZXkiLCJpZCI6MX0= -p ~/grafana-dashboard
```

### 使用grafana provision

新版Grafana支持使用provision的方式通过yaml配置导入 `dashboards | datasources | notifiers` 这三种资源到Grafana db中。

`/etc/grafana/grafana.ini`文件中需要配置provisioning的目录

```ini
[paths]
data = /var/lib/grafana/data
logs = /var/log/grafana
plugins = /var/lib/grafana/plugins
provisioning = /etc/grafana/provisioning
[smtp]
enabled = true
host = smtp.exmail.qq.com:25
...
```

`/etc/grafana/provisioning`目录中新建`dashboards | datasources | notifiers` 这三个文件夹。

在dashboards目录下`provisioning/dashboards/dashboard-provider.yaml`需要配置好需要导入的dashboard的配置文件的path：

```yaml
apiVersion: 1
providers:
- name: 'default'
  orgId: 1
  folder: ''
  type: file
  disableDeletion: false
  options:
    path: /var/lib/grafana/dashboards/General
- name: 'k8s'
  orgId: 1
  folder: 'k8s'
  type: file
  disableDeletion: false
  options:
    path: /var/lib/grafana/dashboards/k8s
- name: 'celery'
  orgId: 1
  folder: 'celery'
  type: file
  disableDeletion: false
  options:
    path: /var/lib/grafana/dashboards/celery
- name: 'pulsar'
  orgId: 1
  folder: 'pulsar'
  type: file
  disableDeletion: false
  options:
    path: /var/lib/grafana/dashboards/pulsar
```

在datasources目录`provisioning/datasources/datasource.yaml`下需要配置好需要导入的datasources的配置文件的path：

```yaml
apiVersion: 1
datasources:
- name: Prometheus
  type: prometheus
  url: http://prometheus:9090/
  access: proxy
  isDefault: true
```

notifiers配置暂时没有用到，保留一个空目录即可

注意事项：

1. `/var/lib/grafana/dashboards/General`目录grafana默认的目录，因此export之后的General需要导入到default中
2. 通过export导出的json配置文件需要做处理：
   1. 只需要json中的dashboard信息，不需要meta信息
   2. json配置需要保持字典序

grafana-dashboard-exporter.sh导出的代码可以通过一下python代码进行转换得到provision的配置：

```python
# -*- coding: utf-8 -*-
import os
import json

dirs = [
    "/home/suncle/dockprom/grafana/dashboards/celery",
    "/home/suncle/dockprom/grafana/dashboards/General",
    "/home/suncle/dockprom/grafana/dashboards/k8s",
    "/home/suncle/dockprom/grafana/dashboards/pulsar",
]


def main():
    for d in dirs:
        for f in os.listdir(d):
            filename = "{}/{}".format(d, f)
            if not f.endswith(".json") or f.startswith("."):
                continue
            try:
                with open(filename, "r+") as fp:
                    json_data = fp.read()
                    data = json.loads(json_data)
                    fp.seek(0)
                    fp.write(json.dumps(data["dashboard"], sort_keys=True))  # 字母序
                    print("{}: success".format(filename))
            except Exception as e:
                print("{}: {}".format(filename, e))


if __name__ == "__main__":
    main()

```



---

参考：

- Dockprom: https://github.com/stefanprodan/dockprom
- Grafana-dashboard-exporter: https://gist.github.com/crisidev/bd52bdcc7f029be2f295
- Grafana-dashboard-import: https://gist.github.com/thedoc31/628beeee934f9c84648c108d4ad89f05

- Provisioning Grafana: https://grafana.com/docs/administration/provisioning/