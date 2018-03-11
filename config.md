# Configure Stack (DB + APP + WEB)

## 1. DB Configuration
#### Installation
```
# yum install mariadb-server -y
# systemctl enable mariadb
# systemctl start mariadb
```

#### Configuration
```
# mysql

> create database studentapp;
> use studentapp;
> CREATE TABLE Students(student_id INT NOT NULL AUTO_INCREMENT,
	student_name VARCHAR(100) NOT NULL,
  student_addr VARCHAR(100) NOT NULL,
	student_age VARCHAR(3) NOT NULL,
	student_qual VARCHAR(20) NOT NULL,
	student_percent VARCHAR(10) NOT NULL,
	student_year_passed VARCHAR(10) NOT NULL,
	PRIMARY KEY (student_id)
);
> grant all privileges on studentapp.* to 'student'@'%' identified by 'student@1';
> flush privileges;
```

## 2. Tomcat Configuration
#### Installation
```
# cd /root
# yum install java -y
# wget -qO- https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.28/bin/apache-tomcat-8.5.28.tar.gz | tar -xz
```
#### Configuration
```
# cd /root/apache-tomcat-8.5.28
# rm -rf webapps/*
# wget https://github.com/cit-ager/APP-STACK/raw/master/student.war -O webapps/student.war
# wget https://github.com/cit-ager/APP-STACK/raw/master/mysql-connector-java-5.1.40.jar -O lib/mysql-connector-java-5.1.40.jar
```
#### Update `context.xml` for DB Connections
```
# vim conf/context.xml
```

##### Add the following config just before the last line
```
<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
               maxActive="50" maxIdle="30" maxWait="10000"
               username="student" password="student@1"
               driverClassName="com.mysql.jdbc.Driver"
               url="jdbc:mysql://<IP ADDRESS OF DB SERVER>:3306/studentapp"/>
```

#### Start the tomcat service
```
# sh bin/startup.sh
```

## 3. Web Server Configuration
#### Installation
```
# yum install httpd httpd-devel gcc -y
```

#### Mod-JK Install
```
# cd /root
# wget -qO- http://redrockdigimark.com/apachemirror/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.42-src.tar.gz | tar -xz
# cd tomcat-connectors-1.2.42-src/native
# ./configure --with-apxs=/bin/apxs
# make 
# make install
```

#### MOd-JK Configuration
```
# cd /etc/httpd/conf.d
# vim workers.properties
worker.list=worker1
worker.worker1.type=ajp13
worker.worker1.host=<IP-ADDRESS-OF-TOMCAT-SERVER>
worker.worker1.port=8009

# vim mod-jk.conf
LoadModule    jk_module  modules/mod_jk.so
JkWorkersFile conf.d/workers.properties
JkLogFile     logs/mod_jk.log
JkMount /student* worker1
```

#### Start Service
```
# systemctl enable httpd
# systemctl start httpd
```

### Reference DOCS.
http://www.diegoacuna.me/installing-mod_jk-on-apache-httpd-in-centos-6-x7-x/

http://www.ramkitech.com/2012/03/virtual-host-apache-httpd-server-tomcat.html

https://tomcat.apache.org/tomcat-3.3-doc/mod_jk-howto.html#s83

