# dmesg

- 커널에서 발생하는 다양한 메시지들을 출력
- `T` 옵션을 통해 타임스탬프 출력

```shell
$ dmesg -T
```

![](/docs/dmesg/images/dmesg-T.png)

## OOME (Out-of-Memory Error)

OOME는 가용한 메모리가 부족해서 더이상 프로세스에게 할당해 줄 메모리가 없는 상황을 의미한다. 커널에서는 내부적으로 OOME Killer를 실행시켜 누군가 가지고 있는 메모리를 회수한다. 즉, 메모리를 많이 사용하고 있는 프로세스를 죽이게 된다.

```
1. OOME 상황 발생
2. 프로세스 선택
3. 프로세스 종료
4. 시스템 안정화
```

OOM Killer가 종료시킬 프로세스를 선택하는 기준은 가장 높은 `oom_score` 점수를 가진 프로세스를 먼저 선택한다. `oom_score`는 프로세스들의 메타데이터를 볼 수 있는 `proc` 디렉토리에서 확인할 수 있다.

![](/docs/dmesg/images/dmesg-proc-oom-score.png)

## OOME 테스트

C언어의 `malloc`을 통해 메모리를 지속적으로 요청했을 때를 확인해보면 OOME가 발생하는 것을 확인할 수 있다.

```c
#include <stdlib.h>

int main(void) {
  void* ptr;
  while(1) {
    ptr = malloc(1024*1024);
    if(ptr == NULL) {
      break;
    }
  }
  return 0;
}
```

![](/docs/dmesg/images/c.png)

![](/docs/dmesg/images/dmesg-oom-killer.png)

전체 메세지가 아닌 oom 메세지만을 출력하여 보고 싶을 경우에는 다음과 같은 명령어를 통해 확인할 수 있다. 만약, OOME가 발생한다면, 충분한 메모리 확보를 통해 해결할 수 있다.

```shell
$ dmesg -TL | grep -i oom
```

![](/docs/dmesg/images/dmesg-oom.png)

## SYN Flooding

SYN Flooding은 공격자가 대량의 SYN 패킷만 보내서 소켓을 고갈시키는 공격이다. SYN Flooding은 기본적으로 TCP 3-way handshake에 대한 이해를 하고 있어야 한다.

### TCP 3-way handshake

![](/docs/dmesg/images/tcp-3-way-handshake.png)

```
1. 서버에 SYN 요청
2. listen() 시스템 콜을 통해 소켓이 열리면 SYN 패킷을 SYN Backlog에 쌓아둠
3. 서버가 SYN+ACK로 응답
4. 서버에 ACK 요청
5. ACK 패킷을 Listen Backlog에 쌓아둠
6. accept() 시스템 콜을 통해 실제 애플리케이션에 처리권을 넘김 (HTTP, gRPC...)
```

### TCP 3-way handshake Issue

![](/docs/dmesg/images/tcp-3-way-handshake-issue.png)

마지막에 ACK 요청을 서버에 보내지 않을 경우에는 SYN Backlog에 패킷이 계속 쌓이는 문제가 발생한다. SYN Backlog는 기본적으로 큐 형태로 동작하며, 큐가 가득차면 SYN 패킷을 저장하지 못해 새롭게 들어온 요청을 처리하지 못하는 상황이 발생한다.

이러한 점을 막기 위해 리눅스에서는 SYN Cookie 기능을 제공한다.

### SYN Cookie

SYN Cookie는 SYN 패킷의 정보들을 바탕으로 쿠키를 만들고, 해당 값을 SYN+ACK의 시퀀스 번호로 만들어 응답한다. 원래라면 SYN 패킷에 대한 정보를 SYN Backlog에 담겠지만, SYN Cookie 방식은 Backlog에 담지않고 Cookie 값을 넣어 응답을 보내준다.

즉, Cookie값을 지정하여 SYN+ACK로 응답하고, ACK가 넘어올 때 Cookie에 1을 더해 서버에 전달되는 구조로 동작한다. 서버에서는 해당 쿠키값이 올바른지 계산하여 올바른 값이 넘어왔는지 확인한다.

SYN Cookie가 동작할 경우 SYN Backlog에 쌓이지 않으며 자원 고갈 현상이 발생하지 않는다. 하지만 TCP Option 헤더를 무시하기 때문에 Windows Scaling 등 성능 향상을 위한 옵션이 동작하지 않는 문제가 있다.

참고로 SYN Cookie는 SYN Flooding이 발생할 것 같은 시점에 사용되기에 항상 SYN Cookie 방식으로 동작하지 않는다. 즉, TCP Option 헤더를 매번 무시하지는 않는다는 의미다.

최근에는 ALB, WAF, Shield 등 다양한 솔루션이 있기 때문에 SYN Flooding이 발생하지는 않겠지만, 혹여나 의심되는 경우 해당 명령어를 통해 SYN Flooding 발생 여부를 확인하여 방화벽을 통해 해결해야한다.

```shell
$ dmesg -TL | grep -i "syn flooding"
```
