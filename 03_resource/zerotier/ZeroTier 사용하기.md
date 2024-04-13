ZeroTier는 TCP/UDP 터널링 프로그램으로 ngrok 의 오픈소스 대안중 하나이다. C++로 개발되었으며, 모바일기기와 다수의 OS를 지원한다.

# 1. 설치하기

[ZeroTier 다운로드페이지](https://www.zerotier.com/download/)에서 받은 설치파일을 실행하면 설치할 수 있다 

# 2. 네트워크 생성하기
설치를 끝냈다면 가장 먼저 [ZeroTier 홈페이지](https://www.zerotier.com/)에서 가입을 한다. 가입을 하면 아래와 같은 대시보드를 쓸 수 있다. 해당 화면에서 `Create A Network` 버튼을 누르면 네트워크를 생성할 수 있다.

![[zerotier_dashboard.png]]

# 3.  네트워크 가입
네트워크를 생성했다면 16자리 `NETWORK ID`를 발급받게된다. 이제 해당 ID로 Join할 수 있다. Mac의 경우 상단 메뉴바에서 아이콘을 누른 다음 `Join New Network`를 누르면 해당 컴퓨터는 지정한 네트워크에 가입되게 된다.
![[zerotier_dashboard_member.png]]
![[zerotier_join_newnetwork.png]]
# 4. CLI 에서 확인하기
어플리케이션이 아닌 CLI에서도 확인할 수 있다.
커맨드라인에서 다음과 같이 명령을 내린다.

```
$ zerotier-cli status
200 info aabb112233 1.12.2 ONLINE
```
현재 zerotier 의 상태를 알 수 있다. zerotier는 기기별로 고유한 주소값을 부여한다. GUI에서 볼 수 있는 `My Address` 가 그것이다. 해당 정보는 CLI의 `status`명령으로도 확인이 가능하다.

원하는 네트워크에 가입하기위해서는 `join` 명령을 쓴다.

```
$ zerotier-cli join [network id]
```

지정한 네트워크 아이디를 넣으면 해당 네트워크에 현재 기기를 등록요청을 한다.

# 5. 등록 기기의 관리
등록이 요청된 기기는 [대시보드](https://my.zerotier.com/network)에서 해당 네트워크를 눌러 들어가면 확인할 수 있다.
하단의 Member 탭에서 확인할 수 있다.

![[zerotier_dashboard_member.png]]
등록된 리스트에서 맨 좌측의 `Auth?` 열 체크박스를 클릭하면, 해당 기기는 이제 이 네트워크를 통한 기기간 통신을 할 수 있다. `Managed IPs` 부분에 할당된 주소로 상호 통신이 가능하다.