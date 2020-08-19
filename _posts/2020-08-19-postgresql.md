sqlite에서 postgresql로 전환하며 이틀간의 고생을 한 내용의 요약이다.
혹시 다시 데이터 이전을 할 일이 있다면 참고한다

### 1. python3 manage.py dumpdata > db.json

해당 코드로 기존에 sqlite에 있던 데이터를 json형태로 저장해둔다. json은 어느 db에서든 접근 가능하기 때문에 이런 형태로 저장해둔다

### 2. setting부분에 database정보를 바꾼다.

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'contag',
        'USER': 'kwon',
        'PASSWORD': '****', (가칭)
        'HOST': '15.223.311.45', (가칭)
        'PORT': '5432',
    }
}

을 설정하면, postgresql db를 사용할 수 있다

### 3. postgresql을 ubuntu에 설치한다. 

### 4. 계정/ db를 만든다

psql을 치면 progresql용 shell로 전환이 될 것이다.
여기서 create database contag; 를 입력하고
CREATE USER kwon WITH PASSWORD '****'; 를 입력한다.
그리고 GRANT ALL PRIVILEGES ON DATABASE contag TO kwon; 을 통해 모든 db에 대한 권한을 준다.

### 5. progresql 설정을 한다

지금 progresql에 대한 접근은 제한적이다. 이를 풀어주어 로컬 (ubuntu server)에서의 모든 접근을 허락해야한다.
우선 locate postgresql.conf 를 하고 locate pg_hba.conf 를 해서 위치를 파악한 후 설정을 변경한다
postgresql.conf는 
listen_address = '*'로 해서 모든 곳에서 접근이 가능하게 설정한다.

pg_hba.conf는 stackoverflow 가이드마다 지시 방식이 다르고, 여차하면 이것 저것 시도하다 꼬일 수 있다.
현재 사용되고 있는 내 로컬 설정은 아래와 같다.

```
local   all           posgres       -                  md5
local   all           all                               md5
host    all           all           127.0.0.1/32       md5
host    all           all           ::1/128            md5
local   replication   all                              peer
host    replication   all           127.0.0.1/32       md5
host    replication   all           ::1/128            md5
host    all           all           15.***.***.***/32   md5
```

### 기타

postgresql을 다시 시작하고싶을 때는 /etc/init.d/postgresql restart를 입력한다
sudo service postgresql status를 입력하면 현재 db서버를 확인할 수 있다

### 이후 작업

sudo python3 manage.py runserver 0:8000 을 실행하면 이제 수행이 될 것이다.
다만 db가 (당연히) 비어져있다. 아래와 같이 입력하면 백업이 된다.

python3 manage.py migrate --run-syncdb

를 입력하면 db가 postgres로 변경되었는지 알 수 있다. 별 문제가 없다면
python3 manage.py shell 를 실행하고

>>> from django.contrib.contenttypes.models import ContentType
>>> ContentType.objects.all().delete()
>>> quit()

를 실행해서 현재 db를 확실하게 비운 후

python3 manage.py loaddata db.json을 실행한다. 데이터가 이전과 같이 채워져있음을 알 수 있다

### Why PostgreSQL?

RDBMS - 관계형 Database Management System으로 여러 유저가 동시에 실시간으로 일어나는 거래를 파악할 수 있다.
마찬가지로, 이를 통해 실시간 analytics가 가능하다

확장성 - db에서 연산 처리를 바로 할 수 있게 해주는 extensions들이 있음.
POSTGIS - cross-network db 연산등이 가능하고, geo-spatial join (다른 지역 db간의 조인)도 가능함

장고 개발자 - posgresql : cost, features, speed and stability.


