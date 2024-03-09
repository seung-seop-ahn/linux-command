# netstat

- 네트워크 연결 정보 확인

```shell
$ netstat -napo
```

## 소켓의 상태 테스트

사전에 알아야 하는 개념은 연결을 맺을 때 사용하는 TCP 3-way handshake와 연결을 끊을 때 사용하는 TCP 4-way handshake이 필요하다.

![](/docs/netstat/images/tcp-4-way-handshake.png)

### LISTEN

통신이 이뤄지기 위해 누군가 소켓을 듣고 있는 상태를 의미한다. Nginx를 설치해서 손쉽게 LISTEN 상태를 확인할 수 있다.

```shell
$ apt-get install nginx
$ service nginx start # systemctl start nginx
$ netstat -napo | grep -i nginx
```

![](/docs/netstat/images/tcp-listen.png)

### ESTABLISHED

LISTEN 상태의 nginx에 telnet으로 접속해보면 ESTABLISHED 상태가 추가되는 것을 확인할 수 있다.

```shell
$ telnet <ip>:80

GET / HTTP/1.1
Host: <ip>
```

### TIME_WAIT

요청 후 응답을 받고 기다리면 중간에 연결이 끊어져 TIME_WAIT 상태를 확인할 수 있다. 곧바로 끊어지지않고 서버가 연결을 끊는 이유는, HTTP/1.1의 스펙으로 연결을 유지하는 설정인 `keepalive_timeout` 때문이다.

### keepalive_timeout

HTTP 요청에 대해 커넥션을 새로 맺지 않아도 되도록 지정된 시간동안 새로운 요청을 기다린다. 즉, 매번 연결을 맺고 통신하고 끊는 과정없이 단 한번의 연결로 처리할 수 있도록 하기 위해서다. 예를 들어, 2개의 GET 요청을 전달했을 때, 각 GET 요청별로 커넥션을 만들고 해지하는 과정없이 단 한번의 커넥션에 2개의 GET 요청을 처리할 수 있는 개념이다.

참고로 HTTP/1.0에서는 단 한번의 커넥션에 단 한번 요청을 할 수 있도록 설계되었지만, 비효율적이라는 점에서 개선을 통해 HTTP/1.1이 등장하게 되었다.

```shell
$ cat /etc/nginx/nginx.conf | grep -i keepalive
```

결국, Nginx에서 더이상 연결을 유지할 필요가 없다 생각이되면 서버가 연결을 끊고 TIME_WAIT 소켓이 생성되는 것이다.

### CLOSE_WAIT

여러 상태들 중에서도 CLOSE_WAIT 상태는 가장 주의깊게 살펴봐야한다. CLOSE_WAIT를 지닌 애플리케이션은 이상 동작을 한다고 보면 된다. 이는 정상적으로 소켓을 정리하는 등 연결을 끊기 위한 동작을 하지 못한다는 의미이기 때문이다.

CLOSE_WAIT에서 LAST_WAIT으로 넘어가려면 `close()` 시스템 콜을 수행해야하는데 넘어가지 못하면서 문제가 발생한다.

만약 CLOSE_WAIT 상태가 서버에서 보인다면, 자연적으로 정리되지 않고 계속 쌓이기 때문에 서버 행업, 포트 고갈 등 서비스에 영향을 끼치는 문제를 일으킬 수 있어 반드시 해결해야한다.

## 정리

- `netstat` 명령을 이용하여 네트워크 연결 정보를 확인할 수 있다.
- 커넥션의 상태와 중단간 IP 정보 등 서버의 네트워크 연결 정보를 확인할 수 있다.
- LISTEN, ESTABLISHED, TIME_WAIT는 흔히 만나게 되는 소켓 상태이며, 많다하여 문제가 있는 것은 아니다. 단, TIME_WAIT는 왜 많이 발생하는지 고민해볼 필요가 있다.
- CLOSE_WAIT가 발생한다면 반드시 원인을 확인하고 조치해야한다.
