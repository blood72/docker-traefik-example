# Traefik을 통한 멀티도메인 개발환경 구축

Traefik은 go 언어 기반의 오픈소스 reverse proxy 라우터 및 로드밸런서이다.
대시보드는 `127.0.0.1:8080`으로 접속할 수 있다.
혹은 `labels` 주석을 풀어 특정 도메인으로 접근하게 할 수 있다(이 경우, 8080 포트 개방이 필요없음).

## 사용 설명

[docker-compose.yml](./docker-compose.yml) 파일을 참조해서 구성하면 끝이다.

모든 요청은 Traefik을 거치며 등록된 라우터 규칙에 따라 각 서비스로 전달한다.
이때, Traefik이 도커 이벤트를 수신받고 있으므로 레이블 선언만으로 별도 설정없이 등록된다.

## 예시

먼저 `/etc/hosts`(윈도우는 `%SYSTEMROOT%\System32\drivers\etc\hosts`)에 `whoami.localhost`를 등록한다.

```yaml
whoami:
  image: "traefik/whoami"
  container_name: "simple-service"
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
```

외부에서 선언된 네트워크일 경우 직접 연결해야 한다.

```yaml
networks:
  traefik:
    name: traefik
    external: true

services:
  whoami:
    image: "traefik/whoami"
    container_name: "simple-service"
    networks:
      traefik:
      default:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.whoami.rule=Host(`whoami.localhost`)"
```

`whoami.localhost` 도메인으로 접근이 가능한 지 확인해본다.

## 로컬에서 HTTPS 사용하기
해당 예제에서는 [mkcert](https://github.com/filosottile/mkcert)를 사용한다.

### mkcert에서 관리하는 인증서를 신뢰할 수 있는 기관으로 등록
```
mkcert -install
```

### 인증서 발급
```
mkcert *.example.com
```

### 관련 설정 활성화
- `/conf/certificates.toml`에서 인증서 경로 지정

- 활성화하고 싶은 컨테이너 label에 다음 명령어 추가
  ```
  "traefik.http.routers.<NAME>.tls=true"
  ```

### HTTPS와 HTTP 둘 다 허용하기
이 경우 HTTPS용 라우트와 HTTP용 라우트를 label에 정의해주면 됨.


## Reference
- [Traefik](https://traefik.io/)
- [Traefik Docs](https://doc.traefik.io/traefik/)
- [kkame/traefik-develop-with-docker-example](https://github.com/kkame/traefik-develop-with-docker-example)
