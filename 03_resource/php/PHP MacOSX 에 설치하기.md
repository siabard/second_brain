
# PHP 설치
```bash
brew install php
```

# PHP 버전 검사
PHP의 정상 설치 여부를 확인하기위해서는 버전을 확인하는 것이 좋다. 다음 명령으로 버전을 확인한다.
```bash
php -v
```


# Apache 설치
Apache 웹서버에 mod_php로 사용할 것이므로 Apache 웹서버를 설치한다.
```bash
brew install httpd
```

## httpd.conf 수정
- Listen port 설정 (원하는 포트)
```
Listen 8080
```

- `mod_rewrite` 주석제거
```
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

- `php_module` 추가
```
LoadModule php_module /opt/homebrew/opt/php@8.3/lib/httpd/modules/libphp.so


<IfModule php_module>
	  AddType application/x-httpd-php .html .php
	  AddType application/x-httpd-php-source .phps
	  PHPIniDir /opt/homebrew/etc/php/8.3
</IfModule>
```

- 사용자 그룹 설정 (일반 사용자 대상으로 권한을 설정한다.
```
User siabard
Group staff
```

- `ServerAdmin` (Admin 정보를 등록한다.)
```
ServerAdmin siabard@gmail.com
```

- Virtual Host 설정 주석을 푼다.
```
# Virtual hosts
Include /opt/homebrew/etc/httpd/extra/httpd-vhosts.conf

```

- Vitual Host 를 해당 파일에 설정한다.
```
<VirtualHost *:8080>
    ServerAdmin siabard@gmail.com
    DocumentRoot "/Users/siabard/project/www"
    ServerName web.myhost.com
    ErrorLog "/opt/homebrew/var/log/httpd/web.myhost.com-error_log"
    CustomLog "/opt/homebrew/var/log/httpd/web.myhost.com-access_log" common
    <Directory "/Users/siabard/project/www">
    	       Options Indexes FollowSymLinks
	       AllowOverride All
	       Require all granted
    </Directory>
</VirtualHost>

```

- 지정한 ServerName 에 맞게 `/etc/hosts` 에 원하는 내역을 추가한다.
```

127.0.0.1	web.myhost.com
```

마지막으로 httpd 테스트 및 데몬을 시작한다.


# MariaDB 설치
데이터베이스로는 Maria DB를 설치한다.

```bash
brew install mariadb
```

설치를 했다면 MariaDB 서버를 시작한다.

```
brew services start mariadb
```

이후에는 `mysql` 명령으로 작업한다.

# MCrypt 라이브러리 설치

구버전의 (php 7 이하와 별도의 mcrypt) brew 를 사용해야한다.

```bash
brew tap shivammathur/php
brew install shivammathur/php/php@7.4
brew tap shivammathur/extensions

brew install shivammathur/extensions/mcrypt@7.4
brew install mcrypt@7.4
```

`.zshrc`에 다음을 추가한다.
```bash
export PATH="/opt/homebrew/opt/php@7.4/bin:$PATH"
export PATH="/opt/homebrew/opt/php@7.4/sbin:$PATH"


export LDFLAGS="-L/opt/homebrew/opt/php@7.4/lib"
export CPPFLAGS="-I/opt/homebrew/opt/php@7.4/include"
```

## httpd.conf 변경

```
LoadModule php7_module /opt/homebrew/opt/php@7.4/lib/httpd/modules/libphp7.so

<IfModule php7_module>
	  AddType application/x-httpd-php .html .php
	  AddType application/x-httpd-php-source .phps
	  PHPIniDir /opt/homebrew/etc/php/7.4
	  <FilesMatch \.php$>
	  	      SetHandler application/x-httpd-php
	  </FilesMatch>
</IfModule>
```