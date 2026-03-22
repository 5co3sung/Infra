# MYSQL

<설치매뉴얼-mysql>

클라우드운영팀 신상원 작성

1. **cmake 도구 설치**

yum -y install cmake

---

▶인터넷이 안될 경우 인터넷이 되는 서버에서 yum install —downloadonly 옵션으로 설치 후 서버에 옮기기

▶설치되는 패키지

cmake-3.20.2-5.el8.x86_64

cmake-data-3.20.2-5.el8.noarch

cmake-filesystem-3.20.2-5.el8.x86_64 cmake-rpm-macros-3.20.2-5.el8.noarch

libuv-1:1.41.1-1.el8_4.x86_64

1. **연관 패키지 설치**

#yum -y install ncurses ncurses-devel bison

---

▶인터넷이 안될 경우 인터넷이 되는 서버에서 yum install —downloadonly 옵션으로 설치 후 서버에 옮기기

▶설치되는 패키지

ncurses-c++-libs-6.1-10.20180224.el8.x86_64

ncurses-devel-6.1-10.20180224.el8.x86_64

1. **연관 패키지 설치{gcc-tollset관련}**

yum -y install gcc-toolset-12-gcc*

---

▶인터넷이 안될 경우 인터넷이 되는 서버에서 yum install —downloadonly 옵션으로 설치 후 서버에 옮기기

▶설치되는 패키지

environment-modules-4.5.2-4.el8.x86_64 gcc-toolset-12-binutils-2.38-17.el8.x86_64 gcc-toolset-12-binutils-gold-2.38-17.el8.x86_64

gcc-toolset-12-gcc-12.2.1-7.4.el8.x86_64 gcc-toolset-12-gcc-c++-12.2.1-7.4.el8.x86_64 gcc-toolset-12-gcc-gfortran-12.2.1-7.4.el8.x86_64

gcc-toolset-12-gcc-plugin-annobin-12.2.1-7.4.el8.x86_64 gcc-toolset-12-gcc-plugin-devel-12.2.1-7.4.el8.x86_64 gcc-toolset-12-libquadmath-devel-12.2.1-7.4.el8.x86_64

gcc-toolset-12-libstdc++-devel-12.2.1-7.4.el8.x86_64 gcc-toolset-12-runtime-12.0-6.el8.x86_64 gmp-c++-1:6.1.2-10.el8.x86_64

gmp-devel-1:6.1.2-10.el8.x86_64 libgfortran-8.5.0-20.el8.x86_64 libmpc-devel-1.1.0-9.1.el8.x86_64

libquadmath-8.5.0-20.el8.x86_64 mpfr-devel-3.1.6-1.el8.x86_64 scl-utils-1:2.0.2-16.el8.x86_64

1. **연관 패키지 설치{RPC 관련 라이브러리}**

#yum -y install libtirpc-devel

---

#yum --enablerepo=powertools install rpcgen

---

▶인터넷이 안될 경우 인터넷이 되는 서버에서 yum install —downloadonly 옵션으로 설치 후 서버에 옮기기

▶설치되는 패키지

libtirpc-devel-1.1.4-8.el8.x86_64

rpcgen-1.3.1-4.el8.x86_64

1. **mysql파일 설치**

https://downloads.mysql.com/archives/community/접속해서버전 선택 operating system : source code로 설정 OS version : ALL operating systems로 설정

boost-버전-tar.gz 파일 다운받기

1. **mysql 경로에 들어가서 cmake로 빌드**

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql-8.0.23 -DDOWNLOAD_BOOST=1 –DWITH_BOOST=/usr/local/src/mysql-8.0.23/boost –DFORCE_INSOURCE_BUILD=1

**비고**

◆◆◆디렉토리를 하나 생성하고 그안에서 빌드를 진행해야함

◆◆◆CMake Error: The source directory "/usr/local/src/build" does not appear to contain CMakeLists.txt.

파일이 깨져서 발생하는 오류로 재다운로드

◆◆◆ yum으로 설치 진행 <최종진행방법> 위에방법은 안됨..

1. https://dev.mysql.com/get/mysql80-community-release-el8-8.noarch.rpm 해당 사이트 주소에서 알맞은 OS와 버전 선택하여 다운로드
2. yum install mysql mysql-server mysql-devel 패키지 설치 진행
3. #systemctl restart mysqld로 서비스 재기동

◆◆◆재기동 시 오류가 발생하면 systemctl status mysqld로 메시지 확인

The datadir located at /var/lib/mysql needs to be upgraded using 'mysql_upgrade' tool.

라고 적혀 있으면 mysql 업그레이드가 필요한 내용이므로

#systemctl start mysqld로 시작하면 이상없이 잘됨

◆설치 후 mysql_secure_installatio으로 설치

[root@localhost ~]# mysql_secure_installation

---

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB

SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current

password for the root user. If you've just installed MariaDB, and

haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): <-----루트 계정의 패스워드 입력

OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody

can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] n  <--OS에 생성한 사용자와 연결

... skipping.

You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n  <--root패스워드를 변경할 것인지 확인 Y를 누르면 변경 패스워드 입력

... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone

to log into MariaDB without having to have a user account created for

them. This is intended only for testing, and to make the installation

go a bit smoother. You should remove them before moving into a

production environment.

Remove anonymous users? [Y/n] n  <---익명의 사용자를 제거할 것인가 (외부에서 접근이 제한됨)

... skipping.

Normally, root should only be allowed to connect from 'localhost'. This

ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y   <---원격에서 root계정의 로그인을 허용할지 말지

... Success!

By default, MariaDB comes with a database named 'test' that anyone can

access. This is also intended only for testing, and should be removed

before moving into a production environment.

Remove test database and access to it? [Y/n] n  <---test데이터베이스의 접근및 제거를 할것인지

... skipping.

Reloading the privilege tables will ensure that all changes made so far

will take effect immediately.

Reload privilege tables now? [Y/n] y   <---변경된 즉시 권한설정을 적용하는지

... Success!

Cleaning up...

All done! If you've completed all of the above steps, your MariaDB

installation should now be secure.

Thanks for using MariaDB!

**#. mysql 데이터 경로 변경하는 방법**

#mysql –u root –p로 접속

---

mysql> select @@datadir;

---

▶현재 데이터 경로 확인

+-----------------+

| @@datadir |

+-----------------+

| /var/lib/mysql/ |

+-----------------+

1 row in set (0.02 sec)

#systemctl stop mysql

---

▶mysql 서비스 중지

#rsync -av /var/lib/mysql /home/test

---

▶해당 명령어를 수행하면 /home/test 경로안에 mysql 폴더가 생긴다.

#chown –R mysql:mysql /home/test/mysql

---

▶mysql 권한을 부여한다.

#vi /etc/my.cnf.d/server.cnf 파일안에 아래 내용추가

[mysqld]

datadir=/home/test/mysql

socket=/home/test/mysql/mysql.sock

---

저장 후 #vi /etc/my.cnf.d/client.cnf 파일안에 아래 내용추가

[client]

socket=/data/mysql/mysql.sock

---

저장 후 아래 명령어로 mariadb재기동

#systemctl start mysql

---

접속 후 select @@datadir로 조회 했을떄 변경 된 경로 확인

~~#mysql –u root –p 으로 접속시도 시~~

~~ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)~~

~~기존 소켓 경로를 찾는 에러가 발생 하므로~~

#ln –s /home/test/mysql/mysql.sock /var/lib/mysq/mysql.sock

---

~~명령어로 링크 설정 후 다시 접속하면 이상없이 로그인 성공~~

**#. mysql utf-8설정 방법**

#mysql –u root –p로 로그인하여 현재 인코딩 타입 조회

mysql> show variables like 'c%';

---

+--------------------------+----------------------------+

| Variable_name | Value |

+--------------------------+----------------------------+

| character_set_client | latin1 |

| character_set_connection | latin1 |

| character_set_database | latin1 |

| character_set_filesystem | binary |

| character_set_results | latin1 |

| character_set_server | latin1 |

| character_set_system | utf8 |

| character_sets_dir | /usr/share/mysql/charsets/ |

| collation_connection | latin1_swedish_ci |

| collation_database | latin1_swedish_ci |

| collation_server | latin1_swedish_ci |

| completion_type | 0 |

| concurrent_insert | 1 |

| connect_timeout | 10 |

+--------------------------+----------------------------+

vi /etc/my.cnf 파일을 열어서 아래 내용 추가

[client]

#추가

default-character-set = utf8

[mysqld]

#추가

init_connect="SET collation_connection = utf8_general_ci"

init_connect="SET NAMES utf8"

character-set-server = utf8

collation-server = utf8_general_ci

[mysqldump]

#추가

default-character-set = utf8

[mysql]

#추가

default-character-set = utf8

---

저장 후 mysql 서비스 재기동 또는 시작

#systemctl start mysqld 또는 systemctl restart mysqld

---

#mysql –u root –p

---

mysql> show variables like 'c%';

---

+--------------------------+----------------------------+

| Variable_name | Value |

+--------------------------+----------------------------+

| character_set_client | utf8 |

| character_set_connection | utf8 |

| character_set_database | utf8 |

| character_set_filesystem | binary |

| character_set_results | utf8 |

| character_set_server | utf8 |

| character_set_system | utf8 |

| character_sets_dir | /usr/share/mysql/charsets/ |

| collation_connection | utf8_general_ci |

| collation_database | utf8_general_ci |

| collation_server | utf8_general_ci |

| completion_type | 0 |

| concurrent_insert | 1 |

| connect_timeout | 10 |

+--------------------------+----------------------------+

릴리즈노트

0.1v 최초작성