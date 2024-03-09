# uptime

- 서버가 받고 있는 부하 평균
- 단위 시간 1분, 5분, 15분 동안의 **R과 D 상태의 프로세스 개수**

```shell
$ uptime
 15:29:13 up 7 min,  0 users,  load average: 0.01, 0.12, 0.07
```

![](/docs/uptime/images/uptime.png)

## Load Average와 CPU의 관계

> - CPU가 1개일 때 프로세스가 2개일 경우: Load Average = 2 (Context-switching)
> - CPU가 2개일 때 프로세스가 2개일 경우: Load Average = 2
> - CPU가 4개일 때 프로세스가 2개일 경우: Load Average = 2

`Load Average > CPU 개수`는 현재 처리 가능한 수준에 비해 많은 수의 프로세스가 존재한다는 의미다. CPU의 개수는 `lscpu -e` 명령어로 확인할 수 있으며, 해당 명령어를 통해 얻어낸 출력된 결과가 CPU의 개수보다 Load Average가 작으면 괜찮은 수준인셈이다.

![](/docs/uptime/images/lscpu.png)

그렇다고 하여 항상 괜찮다는 의미는 아니다. Load Average가 높은 이유를 생각해보자. 앞에서 언급했듯, **R과 D 상태의 프로세스 개수**라고 했다. 그럼 R과 D는 무엇일까?

## R과 D의 상태

- R 상태는 CPU 위주의 작업
- D 상태는 I/O 위주의 작업

만약 R 상태의 프로세스가 많을 경우 CPU 개수를 늘리거나 쓰레드 개수를 조절해야햔다. 반대로 D 상태의 프로세스가 많을 경우 IOPS가 높은 디바이스로 변경하거나 처리량을 줄여주어야 한다.

R과 D의 상태는 `vmstat` 명령어로 확인할 수 있다. 해당 명령어로 출력되는 내용의 `r`과 `b`는 각각 R 상태와 D 상태의 프로세스 수를 의미한다.

![](/docs/uptime/images/rnd.png)
