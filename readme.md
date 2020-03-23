## OTUS-Linux-Lesson 8  
# Задание основные (без *)  

### Задание 1. Создать свой RPM пакет  

Убеждаемся, что установлены все необходимые пакеты  

```
yum install -y redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils
``` 
  
Скачиваем SRPM NGINX
```
wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
```
  
Создаём древо каталогов для сборки
```
rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm
```
  
Скачиваем и разархивируем последние исходники OpenSSL и ставим все зависимости
```
wget https://www.openssl.org/source/latest.tar.gz  
tar -xvf latest.tar.gz
yum-builddep rpmbuild/SPECS/nginx.spec
```

Забираем SPEC из методички, заменяем путь к OpenSSL  
```
wget https://gist.githubusercontent.com/lalbrekht/6c4a989758fccf903729fc55531d3a50/raw/8104e513dd9403a4d7b5f1393996b728f8733dd4/gistfile1.txt  
sed 's/1.1.1a/1.1.1e/' gistfile1.txt > rpmbuild/SPECS/nginx.spec
```
  
Приступаем к сборке RPM пакета
```
rpmbuild -bb rpmbuild/SPECS/nginx.spec
```
  
Устанавливаем пакет, запускаем nginx
```
yum localinstall -y rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
systemctl start nginx
```
  

### Задание 2. Создаем своей репозиторий и размещаем там пакет  

Создаем каталог для репозитория, коприруем туда созданный пакет NGINX и пакет для установки репозитория Percona-Server  
```
mkdir /usr/share/nginx/html/repo
cp rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm
```
  
Инициализируем репозиторий
```
createrepo /usr/share/nginx/html/repo/
```
  
Настраиваем доступ к листингу каталога и перезапускаем NGINX
```
sed -i '/index  index.html index.htm;/ a autoindex on;' /etc/nginx/conf.d/default.conf
nginx -s reload
```
  
Добавляем репозиторий в Yum и проверяем 
```
touch /etc/yum.repos.d/otus.repo
echo -e "[otus] \nname=otus-linux \nbaseurl=http://localhost/repo \ngpgcheck=0 \nenabled=1" >> /etc/yum.repos.d/otus.repo
yum repolist enabled | grep otus
yum list | grep otus
yum install percona-release -y
```
