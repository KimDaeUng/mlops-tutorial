# 목표
- 도커를 이용해 postgres server 띄우기

# 요구사항
1. Username: `Postgres`
2. Role: `Superuser`
3. 비밀번호: `mypassword`
4. port forwarding: 5432:5432

# Solution
- M1 Mac 기준

0. docker & docker-compose & psql 설치

1. PostgresSQL Docker-compose 파일 작성

```yaml
# ./docker-compose.yaml
version: "3"
services:
  database:
    image: postgres
    container_name: postgres-db
    restart: always
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=Postgres
      - POSTGRES_PASSWORD=mypassword
    volumes:
      - {host_path_to_mount}:/var/lib/postgresql/data
      - ./scripts/:/docker-entrypoint-initdb.d/
```

2. Superuser role 지정
`./scripts/01_users.sql` 파일을 다음과 같이 작성.
```sql
ALTER ROLE project_admin SUPERUSER;
```

3. Docker-compose 실행
`docker-compose.yaml` 파일이 있는 경로에서 다음을 실행
```bash
docker-compose up -d
```

4. Postgresql 접속

아래 명령어 실행 후 설정한 비밀번호를 입력해 접속

```bash
psql -h localhost -U Postgres -p 5432
```

접속하면 다음과 같은 화면이 나타남
```bash
Password for user Postgres:
psql (14.5 (Homebrew))
Type "help" for help.

Postgres=#
```


# Reference
- [[Docker] PostgreSQL Docker Compose 파일 작성, BeomBeomJoJo](https://afsdzvcx123.tistory.com/entry/Docker-PostgreSQL-Docker-Compose-%ED%8C%8C%EC%9D%BC-%EC%9E%91%EC%84%B1)
- [Setting up Docker Compose with Postgres, 
Lester Fernandez](https://youtu.be/z6kHhTW1oV8)