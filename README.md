### Hướng dẫn cài đặt DOMjudge với docker

#### Yêu cầu:

- Đã cài đặt docker, docker-compose

Clone project và mở terminal cd vào thư mục của project

Tạo file .env và config tương tự file example.env

Start mariadb domserver trước để lấy JUDGEDAEMON_PASSWORD:

```bash
sudo docker-compose up -d dj-mariadb domserver
```

Lấy judgehost password:

```bash
sudo docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
```

Thay password của biến JUDGEDAEMON_PASSWORD trong file docker-compose

Chạy các máy chấm judgehost(s):

```bash
sudo docker-compose up -d
```

DOMserver started: http://localhost

Lấy password username admin:

```bash
sudo docker exec -it domserver cat /opt/domjudge/domserver/etc/initial_admin_password.secret
```

Stop DOMjudge:

```bash
sudo docker-compose down
```

### Hướng dẫn cài đặt DOMjudge thủ công

#### Môi trường:

- Ubuntu 20.04.3

- DOMjudge version 7.3.3

Tải source code và giải nén: https://www.domjudge.org/download

#### Cài DOMserver:

Mở terminal trong folder vừa giải nén chứa source code:

```bash
sudo apt install acl zip unzip mariadb-server apache2 \
      php php-fpm php-gd php-cli php-intl php-mbstring php-mysql \
      php-curl php-json php-xml php-zip composer ntp
sudo apt-get install build-essential
sudo apt-get install libjsoncpp-dev libcurl4-gnutls-dev libmagic-dev
sudo apt-get install libcgroup-dev
./configure --prefix=$HOME/domjudge
```

```bash
nano etc/genrestapicredentials
```

Sửa url: xóa bỏ “domjudge/” sau “localhost/”
Bấm tổ hợp phím: Ctrl + O -> Enter -> Ctrl + X

```bash
nano etc/apache.conf.in
```

Sửa chỗ VirtualHost thành như sau:

![image](https://user-images.githubusercontent.com/55653291/133781475-cfb12e44-9ce9-46ee-9596-6ddfaed4d37d.png)

Bấm tổ hợp phím: Ctrl + O -> Enter -> Ctrl + X

```bash
nano etc/domjudge-fpm.conf.in
```

Sửa: ;php_admin_value[date.timezone] = Asia/Bangkok

```bash
make domserver
sudo make install-domserver
```

```bash
./sql/dj_setup_database genpass
sudo ./sql/dj_setup_database -u root -r install
```

Sau đó nhập 2 lần password: lần đầu là password MYSQL, lần thứ hai là password MYSQL_ROOT

##### Note:

- /home/tattrung/Downloads/domjudge-7.3.3: đường dẫn đến folder domjudge
- etc/php/7.4/fpm/ | php7.4-fpm | php7.4-fpm: version php 7.4 có thể được thay đổi nếu cần

```bash
sudo ln -s /home/tattrung/Downloads/domjudge-7.3.3/etc/apache.conf /etc/apache2/conf-available/domjudge.conf
sudo ln -s /home/tattrung/Downloads/domjudge-7.3.3/etc/apache.conf /etc/apache2/sites-available/domjudge.conf
sudo ln -s /home/tattrung/Downloads/domjudge-7.3.3/etc/domjudge-fpm.conf /etc/php/7.4/fpm/pool.d/domjudge.conf
sudo a2enmod proxy_fcgi setenvif rewrite
sudo a2ensite domjudge.conf
sudo a2dissite 000-default.conf
sudo a2enconf php7.4-fpm domjudge
sudo service php7.4-fpm reload
sudo service apache2 reload
```

DOMserver đã được chạy tại url: http://localhost

Xem password admin:

```bash
cat etc/initial_admin_password.secret
```

Tài khoản admin:
username: admin
password: <xem trong file initial_admin_password.secret>

#### Cài judgehost:

```
sudo apt install make sudo debootstrap libcgroup-dev lsof \
      php-cli php-curl php-json php-xml php-zip procps \
      gcc g++ default-jre-headless default-jdk-headless \
      ghc fp-compiler
./configure --prefix=$HOME/domjudge
make judgehost
sudo make install-judgehost
sudo useradd -d /nonexistent -U -M -s /bin/false domjudge-run
sudo ./misc-tools/dj_make_chroot
```

Chạy máy chấm:

```bash
./judge/judgedaemon
```

Truy cập: http://localhost/jury/judgehosts

![image](https://user-images.githubusercontent.com/55653291/133781565-6960e4fd-a38a-461d-a94d-a0aa1f71fc8f.png)

Máy chấm đã được bật
