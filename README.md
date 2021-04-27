# docker로 gitlab 설치하기

--hostname 옵션의 url은 적당히 바꿔준다. --publish 옵션의 22는 22번 포트를 쓴다는거니 ssh가 올라와 있다면 적당히 설정해준다. ‘:’ 을 기준으로 앞의 숫자가 실물 서버(이하 host)의 포트고, 뒤의 숫자가 host와 연결된 docker의 포트다. docker는 하나의 서버처럼 인식되어 host와 통신하기에 이런 설정을 해줘야 한다. --volume 옵션은 다음 절에서 설명한다. --restart 옵션을 줬기 때문에 OS를 재기동해도 docker가 gitlab을 알아서 띄워준다. 더이상의 자세한 설명은 가이드를 참고하면 된다.

```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```

잘 실행되는지 웹브라우저를 열고 `http://127.0.0.1` 에 접속해본다. 네트워크 상의 다른 컴퓨터에서는 브라우저에서 host의 ip로 접속할 수 있다. gitlab 로그인 페이지가 뜨면 잘 된 것이다.

# docker-compose

[_docker-compose.yaml_](docker-compose.yaml)
```
# docker-compose.yml
web:
  image: 'gitlab/gitlab-ce:latest'
  restart: always
  hostname: 'gitlab.example.com'
  container_name: gitlab
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'http://gitlab.example.com'
      # Add any other gitlab.rb configuration here, each on its own line
  ports:
    - '80:80'
    - '443:443'
    - '22:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
    - '내 백업폴더:/var/opt/gitlab/backups' # 백업 폴더 추가
```

docker-compose.yml 파일을 다 구성한 후, 아래 명령어를 실행하면 된다. 마찬가지로 url과 port는 상황에 맞게 적당히 적는다.

```
docker-compose up -d
```

이제 gitlab 컨테이너를 다시 만들어 띄웠다. 지금 당장 쓸 필요는 없지만 앞으로 gitlab 컨테이너를 띄울 때는 이 명령어로 한다.

```
sudo docker start gitlab
```

# update

```
sudo docker stop gitlab                   # 도커 컨테이너 중지
sudo docker rm gitlab                     # 도커 이미지 삭제
sudo docker pull gitlab/gitlab-ce:latest  # gitlab 최신버전 이미지 받아오기
docker-compose up -d                      # gitlab 실행
```
or

```
docker-compose.yml의 위치로 이동
docker-compose pull   # gitlab update
docker-compose up -d  # gitlab 재실행
```

# 메일 발송 설정

사용자가 비밀번호를 분실했을 경우, 메일로 비밀번호를 재설정할 수 있으면 편리하다. 도커 컨테이너에 직접 접속하여 설정 파일을 수정하자.

```
sudo docker exec -it gitlab /bin/bash # 컨테이너에 접속한다
vi etc/gitlab/gitlab.rb # 설정 파일을 수정하자
```

내가 사용하려는 stmp서버의 예제를 보고 똑같이 수정해주면 된다
`https://docs.gitlab.com/omnibus/settings/smtp.html` 

```
# /etc/gitlab/gitlab.rb 를 변경한 후에는 항상 이 명령어로 변경사항을 적용한다(Omnibus GitLab 한정)
gitlab-ctl reconfigure
```

# fork
`https://fork.dev/home`