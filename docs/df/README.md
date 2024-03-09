# df

- 디스크 영유 공간 및 inode 공간 확인
- 반드시 모니터링해야하는 요소
- Filesystem이 100%일 때 최악의 상황의 경우 SSH 접속도 안됨

```shell
$ df -h # human readable
$ du -sh ./* # directory usage
```

![](/docs/df/images/df.png)

![](/docs/df/images/du.png)

## 파일을 삭제했음에도 용량이 늘어나지 않는 경우

2GB 파일을 만들고 Docker Desktop에서 vi로 열어둔 후 생성한 파일을 다시 삭제한 뒤에 용량을 확인해보면 그대로 인것을 알 수 있다.

```shell
$ mkdir test
$ cd test
$ dd if=/dev/zero of=2GBfile bs=1M count=2048
```

![](/docs/df/images/dd.png)

![](/docs/df/images/docker.png)

![](/docs/df/images/remove.png)

Docker Desktop에서 해당 파일의 참조를 제거할 경우, 정상적으로 삭제되는 것을 확인할 수 있다.

![](/docs/df/images/remove-reference.png)

### 파일 핸들

만약 다른 프로세스에서 삭제한 파일에 대한 참조를 하고있는 상황에서 `lsof` 명령어를 이용하면 파일 핸들을 확인할 수 있다. 즉, vi가 참조하고 있는 프로세스가 있다는 것을 확인할 수 있다.

```shell
$ lsof | grep 2GBfile
```

![](/docs/df/images/file-handle.png)

만약 삭제가 된 상황이라면, deleted 문구가 출력된다.

![](/docs/df/images/file-handle-deleted.png)

## inode

- 파일 또는 디렉토리에 대한 메타데이터를 저장하는 구조체
- 파일과 디렉토리의 수 (ex. 파일 10개 = inode 10개)

```shell
$ df -i
```

## 정리

- `df` 명령어를 이용하여 디스크 여유 공간 및 inode 공간을 확인할 수 있다.
- 파일 핸들이 남아있어서 파일을 지웠지만 지워지지 않는 경우가 있다. 이 경우에는 `lsof` 명령어를 통해 파일 핸들을 확인하여 용량 확보를 해야한다.
- inode는 파일과 디렉토리의 개수로 생각하면 된다. inode에도 최대값이 있으며, 최대값 이상의 파일을 만들 수는 없다.
