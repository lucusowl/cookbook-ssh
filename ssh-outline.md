# SSH 개요

Secure SHell(SSH)는 insecure 네트워크에서 다른 컴퓨터에 로그인하거나 secure 서비스를 실행할 수 있도록 해주는 protocol입니다

SSH 버전은 SSH-1과 SSH-2로 나뉩니다.
SSH-2는 2006년 이후로 표준으로 채택된 버전으로, SSH-1과는 호환되지 않습니다

OpenSSH는 기존의 SSH software를 대채하기 위해 [Open BSD](https://www.openbsd.org/) 프로젝트에서 나온 **secure networking utility**로써 1999년 OpenBSD 2.6 릴리즈에 첫 공개되었습니다
2019년 Windows 10에 공식적으로 포함되면서 OpenSSH는 거의 대부분의 운영체제에서 사용되는 SSH implementation이 되었습니다

## SSH Architecture

SSH는 client-server model에 사용되는 protocol입니다. 
SSH protocol은 서버인증을 제공하는 Transport Layer Protocol, 클라이언트 인증하는 User Authentication Protocol, 암호화된 채널을 정의하는 Connection Layer Protocol 이렇게 3가지 주요 protocol로 구성되어 작동합니다.

이를 통해 서버와 사용자 인증이 이루어지고 나서 데이터 암호화를 통해 교환하게 됩니다

SSH는 공개키 암호화(비대칭 키 암호화)를 이용해 서로 인증과정을 거치고, 이후 대칭키를 통해 네트워크에 오가는 내용을 암호화합니다

SSH를 통해 secure한 작업이 가능하기에 주로 아래와 같은 용도로 사용됩니다.

- 원격 컴퓨터에 접속/로그인/명령수행
- 원격 컴퓨터와의 파일 전송(scp, sftp, ftps, fasp)
- 패킷 포워딩, 터널링

# SSH-client Set-up

기본적으로 거의 대부분의 운영체제(`Windows`, `linux`, `macOS`)에 OpenSSH가 탑재되어 있습니다

아래의 명령어로 탑재된 ssh utility 버전을 확인해 줍니다

```
ssh -V
```

<details>
<summary>존재하지 않는 명령어라고 뜨거나 제대로 동작하지 않는 경우</summary>

- `linux`에서는 아래의 명령어를 입력하여 설치해주세요

	```bash
	sudo apt install openssh-client
	```

- `Windows`에서는 버전이 `Windows 10`(빌드 1809)미만일 경우, openssh가 설치되어 있지 않을 수 있습니다. 아래의 과정을 따라주세요

	1. `설정`에서 `앱` > `앱 및 기능` > `선택적 기능`을 선택합니다
	2. `설치된 기능` 아래에 `OpenSSH 클라이언트`이 있는 지 확인합니다
	3. 없다면 위의 `기능 추가`를 선택한 후, `OpenSSH 클라이언트`를 찾아 설치합니다

	자세한 내용은 [Microsoft 문서, OpenSSH 설치](https://docs.microsoft.com/ko-kr/windows-server/administration/openssh/openssh_install_firstuse)을 통해 확인해주세요.
</details>

# Reference

- [RFC 4251, SSH Protocol Architecture](https://datatracker.ietf.org/doc/html/rfc4251#section-1)
- [SSH homepage, Academy](https://www.ssh.com/academy/ssh)
- [OpenSSH Manual Pages](https://www.openssh.com/manual.html)
- [OpenSSH Protocol overview](http://cvsweb.openbsd.org/cgi-bin/cvsweb/~checkout~/src/usr.bin/ssh/PROTOCOL?content-type=text/plain)

## Standard documentation

인터넷 표준으로 발행한 RFC문서들은 IETF의 "secsh"에서 SSH-2입니다