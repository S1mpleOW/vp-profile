# Devops practice project

## Prerequisites

- Vagrant
- Virtualbox

## Technologies

- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL

## How to run manually

### Set up vagrant

1. Install vagrant
2. Install plugin vagrant-hostmanager
3. Access manual provisioning folder and run VagrantFile

### Set up database

1. After vagrant is up, access the db01 server
2. Update package: yum update -y
3. Install perl-release package
4. Install mariadb-server package
5. Start mariadb service: systemctl start mariadb
6. Enable mariadb service: systemctl enable mariadb
7. Run mysql_secure_installation to secure mariadb
8. Create database: mysql -u root -p<password> -e "create database accounts"
9. Create user and grant privileges: mysql -u root -p<password> -e "grant all privileges on accounts.\* to 'root'@'%' identified by <password>"
10. Import database: mysql -u root -p<password> accounts < src/main/resources/db_backup.sql
11. Restart mariadb service: systemctl restart mariadb

### Set up memcached

1. After vagrant is up, access the cache01 server
2. Update package: yum update -y
3. Install perl-release package
4. Install memcached package
5. Start memcached service: systemctl start memcached
6. Enable memcached service: systemctl enable memcached
7. Update firewall: firewall-cmd --permanent --add-port=11211/tcp
8. Reload firewall: firewall-cmd --reload
9. Update memcached config: vi /etc/sysconfig/memcached
10. Update memcached config: OPTIONS="0.0.0.0, ::1"
11. Restart memcached service: systemctl restart memcached

### Set up rabbitmq

1. After vagrant is up, access the rmq01 server
2. Update package: yum update -y
3. Install perl-release package
4. Install rabbitmq-server package
5. Start rabbitmq-server service: systemctl start rabbitmq-server
6. Enable rabbitmq-server service: systemctl enable rabbitmq-server
7. Update firewall: firewall-cmd --permanent --add-port=5672/tcp
8. Reload firewall: firewall-cmd --reload
9. Create user: rabbitmqctl add_user test test
10. Set user tag: rabbitmqctl set_user_tags test administrator
11. Restart rabbitmq-server service: systemctl restart rabbitmq-

### Set up tomcat

1. After vagrant is up, access the app01 server
2. Update package: yum update -y
3. Install perl-release package
4. Install JDK 11, maven, git, wget: yum install java-11-openjdk java-11-openjdk-devel -y
5. Access /tmp folder: cd /tmp
6. Install tomcat: wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz
7. Extract tomcat: tar -xzvf apache-tomcat-8.5.37.tar.gz
8. After extract, add tomcat user: useradd --home-dir /usr/local/tomcat8/ --shell /sbin/nologin tomcat
9. Copy all tomcat files to /usr/local/tomcat8/: cp -r apache-tomcat-9.0.75/\* /usr/local/tomcat8/
10. Change owner of /usr/local/tomcat8/: chown -R tomcat.tomcat /usr/local/tomcat8/
11. Create tomcat service: vi /etc/systemd/system/tomcat.service and add the following content:
    [Unit]
    Description=Tomcat
    After=network.target

[Service]

User=tomcat
Group=tomcat

WorkingDirectory=/usr/local/tomcat8

#Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre

Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
Environment=CATALINA_HOME=/usr/local/tomcat8
Environment=CATALINE_BASE=/usr/local/tomcat8

ExecStart=/usr/local/tomcat8/bin/catalina.sh run
ExecStop=/usr/local/tomcat8/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target

12. Reload daemon: systemctl daemon-reload
13. Start tomcat service: systemctl start tomcat
14. Enable tomcat service: systemctl enable tomcat
15. Update firewall: firewall-cmd --permanent --add-port=8080/tcp
16. Clone project: git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git
17. Access project folder: cd vprofile-project
18. Build project: mvn clean install
19. Copy war file to tomcat webapps folder: cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war
20. Copy application.properties file to tomcat conf folder: cp src/main/resources/application.properties /usr/local/tomcat8/webapps/ROOT/WEB-INF/classes/application.properties
21. Restart tomcat service: systemctl restart tomcat

### Set up nginx

1. After vagrant is up, access the web01 server
2. Update package: yum update -y
3. Install perl-release package
4. Install nginx package
5. Create nginx config: vi /etc/nginx/conf.d/vprofile.conf and add the following content:
   upstream vproapp {
   server app01:8080;
   }
   server {
   listen 80;
   location / {
   proxy_pass http://vproapp;
   }
   }
6. Start nginx service: systemctl start nginx
7. Enable nginx service: systemctl enable nginx
8. After all servers are up, find the ip address of web01 server and access it from browser
