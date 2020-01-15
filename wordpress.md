## LAMP 웹서버 설치(AMI) 및 WordPress 블로그 호스팅

#### 1. LAMP 웹서버 설치 및 테스트

1.  Apache 웹 서버, MySQL, PHP 소프트웨어 패키지를 설치

   ``` bash
   $ sudo yum install -y httpd24 php70 mysql56-server php70-mysqlnd
   ```

2.  Apache 웹 서버 실행 

   ``` bash
   $ sudo service httpd start
   ```

   * Amazon Linux 2 

     ``` bash
     $ sudo systemctl start httpd
     ```

3.  기존 인스턴스 보안 그룹에서는 인바운드 규칙으로 SSH 연결을 허용하는 규칙만 들어있어 HTTP 연결을 허용하는 규칙을 추가해준다.

4.  파일 권한 설정

   * ec2-user를 apache 그룹에 추가

   ``` bash
   $ sudo usermod -a -G apache ec2-user
   ```

   * /var/www 및 그 콘텐츠의 그룹 소유권을 apache 그룹으로 변경

   ``` bash
   $ sudo chown -R ec2-user:apache /var/www
   ```

   *  `/var/www`와 그 하위 디렉터리의 디렉터리 권한을 변경(그룹 쓰기 권한 추가)

   ``` bash
   $ sudo chmod 2775 /var/www
   $ find /var/www -type d -exec sudo chmod 2775 {} \;
   $ find /var/www -type f -exec sudo chmod 0664 {} \;
   ```

5.  Apache 문서 루트에 PHP 파일 생성

   ``` bash
   $ echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
   ```

6.  웹 브라우저에서 접속

   ``` bash
   http://ec2-13-124-32-215.ap-northeast-2.compute.amazonaws.com/phpinfo.php
   ```

7. `phpinfo.php` 파일을 삭제(보안상의 이유)

   ``` bash
   $ rm /var/www/html/phpinfo.php
   ```

   

#### 2. WordPress 블로그 호스팅

1.  WordPress 설치

   ``` bash
   $ wget https://wordpress.org/latest.tar.gz
   ```

   ``` bash
   $ tar -xzf latest.tar.gz
   ```

2. 데이터베이스 서버 시작

   ``` bash
   $ sudo service mysqld start
   ```

   * Amazon Linux 2

     ``` bash
     $ sudo systemctl start mariadb
     ```

3. 데이터베이스 로그인

   ``` bash
   $ mysql -u root -p
   ```

4. MySQL 사용자, 암호 및 데이터베이스 생성

   ``` bash
   CREATE USER 'cjy'@'localhost' IDENTIFIED BY 'abc123';
   CREATE DATABASE `cjy-db`;
   ```

5. 사용자에게 데이터베이스 전체 권한을 부여

   ``` bash
   GRANT ALL PRIVILEGES ON `cjy-db`.* TO "cjy"@"localhost";
   FLUSH PRIVILEGES;
   ```

6. exit 하고 나온 뒤 wp-config.php 파일을 vi 편집기로 열어 내 정보로 수정한다.

   * /DB_NAME으로 해당 Line을 찾고 수정

     ``` bash
     define('DB_NAME', 'cjy-db');
     define('DB_USER', 'cjy');
     define('DB_PASSWORD', 'abc123');
     ```

7. httpd.conf 파일을 vi 편집기로 열어 html 허용범위 변경

   * /<Directory "/var/www/ht로 해당 Line을 찾고 수정

     ``` bash
     <Directory "/var/www/html">
         #
         # Possible values for the Options directive are "None", "All",
         # or any combination of:
         #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
         #
         # Note that "MultiViews" must be named *explicitly* --- "Options All"
         # doesn't give it to you.
         #
         # The Options directive is both complicated and important.  Please see
         # http://httpd.apache.org/docs/2.4/mod/core.html#options
         # for more information.
         #
         Options Indexes FollowSymLinks
     
         #
         # AllowOverride controls what directives may be placed in .htaccess files.
         # It can be "All", "None", or any combination of the keywords:
         #   Options FileInfo AuthConfig Limit
         #
         AllowOverride All # None -> All
     ```

8. Apache 웹 서버에 대한 파일 권한 수정

   ``` bash
   $ sudo chown -R apache /var/www # 파일 소유권 허용
   $ sudo chgrp -R apache /var/www # 그룹 소유권 허용
   $ sudo chmod 2775 /var/www # 그룹 쓰기 권한 추가
   $ find /var/www -type d -exec sudo chmod 2775 {} \; 
   $ find /var/www -type f -exec sudo chmod 0664 {} \; 
   ```

9. Apache 웹 서버 재시작해서 새 그룹 및 권한 적용

   ``` bash
   $ sudo service httpd restart
   ```

   * Amazon Linux 2

     ``` bash
     $ sudo systemctl restart httpd
     ```

10.  WordPress 설치 스크립트 실행

    ``` bash
    $ sudo chkconfig httpd on && sudo chkconfig mysqld on # 시스템이 부팅될 때마다 httpd, 데이터베이스 실행
    $ sudo service mysqld status # 데이터베이스 서버 실행 확인
    $ sudo service http status # Apache 웹 서버 실행 확인
    $ sudo service mysqld(httpd) start # 실행 중이지 않을 경우, 시작
    ```

11. 웹 브라우저에서 WordPress 블로그의 URL을 입력 -> WordPress 설치 스크립트

    * URL : http://ec2-13-124-32-215.ap-northeast-2.compute.amazonaws.com
    * URL : http://13.124.32.215

​        둘 다 가능

![캡처](images/캡처-1579073057165.PNG)