**1. Small Sample Python Server**



* from here:

https://pythonbasics.org/webserver/



/usr/local/bin/mywebserver.py

```
# Python 3 server example
from http.server import BaseHTTPRequestHandler, HTTPServer
import time

hostName = "localhost"
serverPort = 8080

class MyServer(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header("Content-type", "text/html")
        self.end_headers()
        self.wfile.write(bytes("<html><head><title>https://pythonbasics.org</title></head>", "utf-8"))
        self.wfile.write(bytes("<p>Request: %s</p>" % self.path, "utf-8"))
        self.wfile.write(bytes("<body>", "utf-8"))
        self.wfile.write(bytes("<p>This is an example web server.</p>", "utf-8"))
        self.wfile.write(bytes("</body></html>", "utf-8"))

if __name__ == "__main__":        
    webServer = HTTPServer((hostName, serverPort), MyServer)
    print("Server started http://%s:%s" % (hostName, serverPort))

    try:
        webServer.serve_forever()
    except KeyboardInterrupt:
        pass

    webServer.server_close()
    print("Server stopped.")
```



Test with curl: 

curl http://localhost:8080



**2. Start / Stop Scrip**

* from here:

https://gist.github.com/tinogomes/447191/f751239eb48e3341b18a79b11995cee7978ee77b



```
#!/bin/bash
#
BASE=/tmp
PID=/tmp/app.pid
LOG=/tmp/app.log
ERROR=/tmp/app-error.log


COMMAND='/bin/python3  /usr/local/bin/mywebserver.py'

USR=user

status() {
    echo
    echo "==== Status"

    if [ -f $PID ]
    then
        echo
        echo "Pid file: $( cat $PID ) [$PID]"
        echo
        ps -ef | grep -v grep | grep $( cat $PID )
    else
        echo
        echo "No Pid file"
    fi
}

start() {
    if [ -f $PID ]
    then
        echo
        echo "Already started. PID: [$( cat $PID )]"
    else
        echo "==== Start"
        touch $PID
        if nohup $COMMAND >>$LOG 2>&1 &
        then echo $! >$PID
             echo "Done."
             echo "$(date '+%Y-%m-%d %X'): START" >>$LOG
        else echo "Error... "
             /bin/rm $PID
        fi
    fi
}

kill_cmd() {
    SIGNAL=""; MSG="Killing "
    while true
    do
        LIST=`ps -ef | grep -v grep | grep $CMD | grep -w $USR | awk '{print $2}'`
        if [ "$LIST" ]
        then
            echo; echo "$MSG $LIST" ; echo
            echo $LIST | xargs kill $SIGNAL
            sleep 2
            SIGNAL="-9" ; MSG="Killing $SIGNAL"
            if [ -f $PID ]
            then
                /bin/rm $PID
            fi
        else
           echo; echo "All killed..." ; echo
           break
        fi
    done
}

stop() {
    echo "==== Stop"

    if [ -f $PID ]
    then
        if kill $( cat $PID )
        then echo "Done."
             echo "$(date '+%Y-%m-%d %X'): STOP" >>$LOG
        fi
        /bin/rm $PID
        kill_cmd
    else
        echo "No pid file. Already stopped?"
    fi
}

case "$1" in
    'start')
            start
            ;;
    'stop')
            stop
            ;;
    'restart')
            stop ; echo "Sleeping..."; sleep 1 ;
            start
            ;;
    'status')
            status
            ;;
    *)
            echo
            echo "Usage: $0 { start | stop | restart | status }"
            echo
            exit 1
            ;;
esac

exit 0

```



Test:

```
/usr/local/bin/start_mywebserver.sh start
/usr/local/bin/start_mywebserver.sh stop
etc.
```







**3. Systemd**

/etc/systemd/system/mywebserver.service

```
#
# Boot script mywebserver with systemd
# /etc/systemd/system/mywebserver.service
#

[Unit]
Description=My Webserver
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/start_mywebserver.sh start
ExecStop=/usr/local/bin/start_mywebserver.sh stop


User=root
Restart=always

[Install]
WantedBy=multi-user.target

```



Test:

```
systemctl daemon-reload
systemctl start  mywebserver.service
systemctl status  mywebserver.service

```

