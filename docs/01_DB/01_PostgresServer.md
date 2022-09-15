# 목표
- 도커를 이용해 postgres server 띄우기

# 요구사항
1. Postgresql User Requirements
    - Username: postgres
    - Role: Superuser
    - Password: mypassword
2. Docker Run Option
    - port forwarding: 5432:5432
3. psql cli tool 을 통해 생성한 데이터 베이스에 접속합니다.
    - 다음 option을 사용해 접속합니다.
      - `-h, --host=HOSTNAME database server host or socket directory (default: "local socket")`
      - `-U, --username=USERNAME database user name (default: "...")`
4. psql 내부에서 다음 내용을 확인합니다.
    - 생성한 Role name 과 role

# Solution
- M1 Mac 기준

0. docker & docker-compose 설치

1. PostgresSQL Docker-compose 파일 작성

    ```yaml
    # ./docker-compose.yaml
    version: "3"
    services:
      database:
        image: postgres:14.3-alpine
        container_name: pg_db
        restart: always
        ports:
          - 5432:5432
        environment:
          - POSTGRES_USER=postgres
          - POSTGRES_PASSWORD=mypassword
          # - POSTGRES_HOST_AUTH_METHOD=trust # Login without password
        volumes:
          - {path to mount}:/var/lib/postgresql/data
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

4. Postgresql 접속 및 생성한 Role name과 role 확인

    - 아래 명령어 실행 후 설정한 비밀번호를 입력해 접속
        ```bash
        psql -h localhost -U Postgres -p 5432
        ```
    - 접속 후 패스워드 입력
        ```bash
        Password for user Postgres:
        ```

    - 그다음 SQL query를 실행하여 rolname과 superuser 여부 확인

        ```
        postgres=# SELECT rolname, rolsuper FROM pg_roles;
        ```
        실행 결과, `postgres` user가 rolsuper t(true) 인 것을 확인
        ```
                  rolname          | rolsuper 
        ---------------------------+----------
        pg_database_owner         | f
        pg_read_all_data          | f
        pg_write_all_data         | f
        pg_monitor                | f
        pg_read_all_settings      | f
        pg_read_all_stats         | f
        pg_stat_scan_tables       | f
        pg_read_server_files      | f
        pg_write_server_files     | f
        pg_execute_server_program | f
        pg_signal_backend         | f
        postgres                  | t
        (12 rows)
        ```

# 자주하는 실수

아래 명령어 실행시
```bash
docker-compose up -d
psql -h localhost -U Postgres -p 5432
```

다음 에러가 날때
```
psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  role "Postgres" does not exist
```

가능한 경우별 해결방안은 다음과 같다.

1. container로 띄운 postgresql server와 host pc(M1 mac)에서 띄운 postgresql server기 동시에 존재하는 상황에서 host pc의 postgresql에 우선 접근하게 되면서 위 에러가 뜨게 되는 경우 -> host pc의 postgresql 삭제 또는 yaml 파일에서 container의 port 번호를 수정함([참고](https://stackoverflow.com/questions/72863274/cant-connect-to-my-docker-postgres-from-python-suddenly))  

2. host pc에서 postgresql을 설치하지 않고 성공적으로 container를 띄운 후, 삭제 후 다시 시도하는 상황에서 기존 docker-compose.yaml로 만든 이미지가 캐싱되어있어 수정사항이 반영 안된 경우 -> 다음 명령어를 사용하여 기존 컨테이너, 이미지, 캐시 삭제 후 재시도

    ```bash
    docker rm -f {container name}
    docker rmi -f {image name}
    docker system prune --volumes
    rm -rf {host pc mount path}
    mkdir {host pc mount path}
    ```



# Reference
- [[Docker] PostgreSQL Docker Compose 파일 작성, BeomBeomJoJo](https://afsdzvcx123.tistory.com/entry/Docker-PostgreSQL-Docker-Compose-%ED%8C%8C%EC%9D%BC-%EC%9E%91%EC%84%B1)
- [Setting up Docker Compose with Postgres, 
Lester Fernandez](https://youtu.be/z6kHhTW1oV8)
- [Postgresql Documentation - Chapter 18. Database Roles and Privileges](https://www.postgresql.org/docs/8.1/user-manag.html)
- [How to use PostgreSQL on Docker on Mac M1?
](https://www.reddit.com/r/PostgreSQL/comments/v5llxg/how_to_use_postgresql_on_docker_on_mac_m1/)