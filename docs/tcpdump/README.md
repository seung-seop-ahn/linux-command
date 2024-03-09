# tcpdump

- 네트워크 패킷 수집 및 분석
- 네트워크 패킷의 흐름을 확인
- 해석하기 쉽지 않음

```shell
# 프로토콜과 포트 번호를 숫자 그대로 표현
$ tcp -nn

# 출력 결과에 더 많은 정보를 출력
$ tcp -vvv

# 패킷의 내용도 함께 출력
$ tcp -A
```

트러블 슈팅할 때는 목적지와 포트가 존재하기 때문에 굳이 모든 패킷을 확인할 필요가 없다. 따라서 필요한 정보만을 출력하도록 명령어를 만들어줘야 한다.

```shell
$ tcp -vvv -nn -A port 80 # and host..
```

## wireshark

- tcpdump로 pcap 파일을 생성 후 wireshark에 분석

```shell
$ tcp -vvv -nn -A host <ip> and port 80 -w http_dump.pcap
```
