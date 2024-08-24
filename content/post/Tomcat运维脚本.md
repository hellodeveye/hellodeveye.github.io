+++
title = "Tomcat运维脚本"
date = 2021-03-16
tags = ["- Tomcat"]
categories = ["Tomcat"]
+++

```
#!/bin/sh

BASE_HOME=/deveye/dsp
SERVER_PORT=8960

# set customer varibales
ulimit -n 65535
export JAVA_HOME=$BASE_HOME/opt/jdk
export JAVA_OPTS="-server -Xms16000m -Xmx16000m -XX:PermSize=128m -XX:MaxPermSize=256m -Djava.security.egd=file:/dev/./urandom"

log () {
    echo "========>> $1 <<========"
}

get_pid_by_port () {
    PID=$(netstat -anp | grep $1 | grep LISTEN | awk '{printf $7}' | cut -d/ -f1)

    if [[ "$PID" != "" && "${PID:0:1}" != "-" ]] ; then
        echo $PID
    else
        echo -1
    fi
}

kill_by_port () {
    PID=$(get_pid_by_port $1)

    echo "[$1] pid: [$PID]"

    if [ "$PID" != "-1" ] ; then
        kill -9 $PID
    fi
}

case "$1" in
    'start' )
        log "cdi start ..."
        /deveye/opt/rsync-client/bin/rsync-dsp
        $BASE_HOME/opt/tomcat/bin/startup.sh
        log "cdi start ok."
        ;;
    'stop' )
        log "cdi stop ..."
        kill_by_port $SERVER_PORT
        sleep 1s
        log "cdi stop ok."
        ;;
    'restart' )
        $0 stop
        sleep 1s
        $0 start
        ;;
    'cc' )
        > $BASE_HOME/opt/tomcat/logs/catalina.out
        exit 1
        ;;
    'tc' )
        tail -f $BASE_HOME/opt/tomcat/logs/catalina.out
        exit 1
        ;;
    'vc' )
        vim $BASE_HOME/opt/tomcat/logs/catalina.out
        exit 1
        ;;
    * )
        echo "Usage: $0 [ start | stop | restart | cc | tc | vc ]"
        exit 1
        ;;
esac
```
