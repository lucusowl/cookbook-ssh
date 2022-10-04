Introduction
===

OpenSSH에서 제공하는 `ssh-keygen`과 `ssh-agent`를 통해 KEY 생성과 관리하는 방법에 대한 설명서입니다

# 1. ssh-key 생성

## IF) KEY가 이미 생성되어 있는 지 확인

KEY와 설정에 대한 정보는 `~/.ssh`디렉토리 아래에 존재합니다

디렉토리가 없을 경우, `.ssh` 디렉토리를 생성해줍니다. (ssh를 이용한 적이 없을 경우 디렉토리가 없을 수 있습니다)

```bash
cd ~
mkdir .ssh
```

디렉토리 아래에
`.pub`와 같은 확장자를 가진 파일(`공개키public-key`에 해당)과
같은 이름을 가진 확장자가 없는 파일(`개인키private-key`에 해당)이 있다면
이미 KEY pair가 생성되어 있는 것입니다

이 경우, KEY 등록부터 진행해주세요

일반적으로 암호화 알고리즘 이름으로 파일이름이 되어 있습니다; _예시) id_ed25519, id_rsa_

## KEY 생성

```
ssh-keygen

ssh-keygen -t ed25519 -C <계정이메일주소> # 권장됌

ssh-keygen -t rsa -b 4096 -C <계정이메일주소> # RSA의 경우 2048-bit이상
```

위의 세 명령어 중 하나 입력합니다

`ssh-keygen` 인자 설명
- `-t` 인자 뒤에는 암호화 방법(알고리즘)을 지칭, 충돌과 속도적인 면에서 우수한 성능을 가진 `ed25519`을 권장
- `-b` 인자 뒤의 숫자는 암호화 비트수(키 길이), ed25519 사용시 지정하진 않음
- `-C` 인자 뒤에는 주석 내용, 다른 key와 구분하기 위한 용도로 많이 설정

이후 아래의 3가지의 절차를 거칩니다

1. 파일명(`키 이름`) 입력

	키 이름을 입력하라는 멘트가 뜨는데 원하는 걸로 아무거나 지정

	그냥 엔터를 입력하면 기본 파일명(`id_ed25519`)으로 생성

2. `passphrase` 입력

	해당 KEY로 ssh 이용시 물어보는 비밀번호를 설정

	아무것도 입력하지 않고 엔터하면 설정되지 않음

	이후, 패스워드 암호 변경은 `ssh-keygen -p <키 이름>` 으로 가능

3. `passphrase` 재확인

	입력한 비밀번호를 다시 입력합니다

`.ssh` 디렉토리 아래에 KEY pair(`<키 이름>.pub`와 `<키 이름>`)가 있으면 성공적으로 생성되었습니다

# 2. ssh-key 등록

## 1) local config로 설정

ssh를 편리하게 사용하기 위해, 사전에 host에 대한 설정을 해둡시다

`~/.ssh/config` 파일 안에 상황에 맞게 아래의 내용을 입력해주세요

- git remote host와 연결

	```conf
	# Default Gitlab
	Host gitlab.com
		PreferredAuthentications publickey
		IdentityFile <개인키 경로>

	# Default GitHub
	Host github.com
		HostName github.com
		PreferredAuthentications publickey
		IdentityFile <개인키 경로>
	```

## 2) local에 등록

KEY생성시, passphrase를 설정하지 않았다면 local에 등록은 생략 가능합니다

OpenSSH 에는 `ssh-agent` 라는 authentication agent가 있어서 등록한 개인키를 passphrase와 함께 메모리에 캐싱해 줍니다.
덕분에 `ssh-agent`가 켜져있는 세션이 유지되는 동안 개인키를 한 번만 입력하면 다시 입력할 필요없이 사용할 수 있습니다.

1. `ssh-agent` 실행

	- linux 터미널이나 bash에서는 아래의 명령어를 입력해 `ssh-agent`를 background에서 엽니다

		`ssh-agent`를 그대로 호출하면 하위의 쉘에서 실행되므로 실행 구성이 현재 쉘 환경에 반영되게 eval로 불러옵니다

		```bash
		eval $(ssh-agent)
		```

		ssh-agent의 pid가 나온다면 정상적으로 실행된 것입니다

		`ssh-agent`뒤에 `-s`나 `-c` 인자들은 추가할 수 있는데, 이는 호출하는 shell(`sh`, `ksh`, `csh`)을 지정해주는 인자입니다

	- Powershell에서는 관리자권한으로 열어 아래의 명령어를 입력해 `ssh-agent`를 실행합니다

		```powershell
		Set-Service -Name ssh-agent -StartupType Automatic
		Start-Service ssh-agent
		```

2. `ssh-add`로 개인키 등록

	```bash
	ssh-add ~/.ssh/<개인키 이름>
	```

	- `-t` 인자를 주어 유효기간(m,h,d,w)을 줄 수도 있습니다; _예시) `ssh-add -t 4w <path>`(4주 유효기간)_

	생성할 때, 해당 개인키의 passphrase를 입력해줍니다

	```
	Identity added: <개인키 경로>
	```

	위와 같은 메세지가 뜨면, 성공적으로 등록된 것입니다

	`ssh-add -l`로 등록된 개인키를 확인할 수 있습니다

## 3) remote에 등록

host에 해당하는 repository에는 `<공개키 이름>.pub`파일의 내용을 전달해야 합니다.

- git의 ssh key 등록의 경우, 아래의 링크를 참고

	1. github: https://docs.github.com/en/authentication/connecting-to-github-with-ssh

	2. gitlab: https://docs.gitlab.com/ee/user/ssh.html

# 3. ssh 연결 테스트

옵션 `-T`를 추가하여 해당 url와의 연결을 테스트할 수 있습니다

연결 내역까지 보고자 한다면 `-v` 옵션을 추가해서 내용을 확인할 수 있습니다

```bash
ssh -T user@domain
ssh -T git@github.com # github의 경우, user영역은 항상 git
ssh -T git@gitlab.com # gitlab의 경우
```

첫 연결 시도일 경우, 아래와 같은 확인 메세지가 뜹니다

```
The authenticity of host 'hostname (IP ADDRESS)' can't be established.
RSA key fingerprint is <FINGERPRINT 내용>.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

`<FINGERPRINT 내용>`이 host의 public host keys fingerprint과 동일할 경우, `yes`를 입력합니다
(참고; [GitHub FINGERPRINTs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints), [GitLab FINGERPRINTs](https://docs.gitlab.com/ee/user/gitlab_com/index.html#ssh-host-keys-fingerprints))

이후 또는 인증이 이미 완료된 경우, 
`Hi ~!`(GitHub)이나 `Welcome ~`(GitLab)와 같은 성공적으로 인증되었다는 메시지가 출력되면 성공적으로 연결된 것입니다

Appendix
===

# ssh-key 해제

KEY를 변경하거나 삭제할 경우, 확인해야할 사항

1. ssh-agent에서 지우기
	- `ssh-add -l`를 통해 현재 등록된 모든 개인키를 확인할 수 있습니다
	- 특정 ssh 연결 지우기: `ssh-add -d <개인키 경로>`
	- 모든 ssh 연결 지우기: `ssh-add -D`

2. config 설정 지우기
	- 해당 KEY 를 적용한 host 내용 지우기
	- `~/.ssh/config`
	- `~/.ssh/known_hosts`
	- 또는 `ssh-keygen -R <IP or host-domain-name>`를 터미널에 입력 => `~/.ssh/known_hosts`에 해당하는 host 삭제

3. 직접적으로 KEY 파일 삭제
	- `~/.ssh/<개인키>`
	- `~/.ssh/<공개키>.pub`

# ssh-agent 종료

`-k`의 인자를 통해 가장 최근에 실행된 ssh-agent를 종료합니다

linux 터미널이나 bash에서는 아래의 명령어를 입력해 종료합니다

```bash
eval $(ssh-agent -k)
```

# References

- [SSH Home Page, Published 220615, Accessed 220718](https://www.ssh.com/academy/ssh)
- [linux manual page - ssh-keygen, Published 220624, Accessed 220718](https://man7.org/linux/man-pages/man1/ssh-keygen.1.html)
- [linux manual page - ssh, die.net, Accessed 220722](https://linux.die.net/man/1/ssh)
- [GitHub Docs, Guide Connecting with ssh, Accessed 220722](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitLab Docs, Use SSH keys to communicate, Accessed 220722](https://docs.gitlab.com/ee/user/ssh.html)
- [Microsoft Docs, Key-based authentication in OpenSSH for Windows, Published 220806, Accessed 220806](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)