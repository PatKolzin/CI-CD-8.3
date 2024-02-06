### Домашнее задание к занятию 9.3 «Процессы CI/CD»

### Подготовка к выполнению

1. Создайте два VM в Yandex Cloud с параметрами: 2CPU 4RAM Centos7 (остальное по минимальным требованиям).

2. Пропишите в [inventory](./infrastructure/inventory/cicd/hosts.yml) [playbook](./infrastructure/site.yml) созданные хосты.

```
---
all:
  hosts:
    sonar-01:
      ansible_host: 158.160.115.121
    nexus-01:
      ansible_host: 130.193.51.67
  children:
    sonarqube:
      hosts:
        sonar-01:
    nexus:
      hosts:
        nexus-01:
    postgres:
      hosts:
        sonar-01:
  vars:
    ansible_connection_type: parcamiko
    ansible_user: centos
```

3. Добавьте в [files](./infrastructure/files/) файл со своим публичным ключом (id_rsa.pub). Если ключ называется иначе — найдите таску в плейбуке, которая использует id_rsa.pub имя, и исправьте на своё.

4. Запустите playbook, ожидайте успешного завершения.

![monitoring](https://github.com/PatKolzin/CI-CD-8.3/blob/main/IMG/1.png)

5. Проверьте готовность SonarQube через [браузер](http://localhost:9000).

6. Зайдите под admin\admin, поменяйте пароль на свой.

7.  Проверьте готовность Nexus через [бразуер](http://localhost:8081).

8. Подключитесь под admin\admin123, поменяйте пароль, сохраните анонимный доступ.

## Знакомоство с SonarQube

### Основная часть

1. Создайте новый проект, название произвольное.

2. Скачайте пакет sonar-scanner, который вам предлагает скачать SonarQube.

3. Сделайте так, чтобы binary был доступен через вызов в shell (или поменяйте переменную PATH, или любой другой, удобный вам способ).

```
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/example/sonar-scanner-5.0.1.3006-linux/bin# export PATH=$(pwd):$PATH
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/example#
```

4. Проверьте `sonar-scanner --version`.

```
pat@olZion:~/1home/cicd/dz$ sonar-scanner --version
INFO: Scanner configuration file: /home/pat/1home/cicd/dz/mnt-homeworks/09-ci-03-cicd/example/sonar-scanner-5.0.1.3006-linux/conf/sonar-scanner.properties
INFO: Project root configuration file: NONE
INFO: SonarScanner 5.0.1.3006
INFO: Java 17.0.7 Eclipse Adoptium (64-bit)
INFO: Linux 5.15.133.1-microsoft-standard-WSL2 amd64
pat@olZion:~/1home/cicd/dz$
```

5. Запустите анализатор против кода из директории [example](./example) с дополнительным ключом `-Dsonar.coverage.exclusions=fail.py`.

<details><summary>sonar-scanner</summary>

```
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/example# sonar-scanner \
  -Dsonar.projectKey=netology \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://178.154.200.93:9000 \
  -Dsonar.login=24dcd473ea58e71933b60b33a5b633331d5460b7 \
> -Dsonar.coverage.exclusions=fail.py


```

</details>


6. Посмотрите результат в интерфейсе.

![monitoring](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/images/9.3-1_SONAR-SCANER-1.png)

7. Исправьте ошибки, которые он выявил, включая warnings.

8. Запустите анализатор повторно — проверьте, что QG пройдены успешно.

<details><summary>sonar-scanner(повторно)</summary>

```
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/example# sonar-scanner   -Dsonar.projectKey=netology   -Dsonar.sources=.   -Dsonar.host.url=http://178.154.200.93:9000   -Dsonar.login=24dcd473ea58e71933b60b33a5b633331d5460b7 -Dsonar.coverage.exclusions=fail.py

INFO: ------------------------------------------------------------------------
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/example#

```

</details>


9. Сделайте скриншот успешного прохождения анализа, приложите к решению ДЗ.

![monitoring](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/images/9.3-1_SONAR-SCANER-2.png)



### Знакомство с Nexus

### Основная часть


1. В репозиторий `maven-public` загрузите артефакт с GAV-параметрами:

 *    groupId: netology;
 *    artifactId: java;
 *    version: 8_282;
 *    classifier: distrib;
 *    type: tar.gz.
   
2. В него же загрузите такой же артефакт, но с version: 8_102.

![monitoring](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/images/9.3_MAVEN-PUBLIC.png)

3. Проверьте, что все файлы загрузились успешно.

4. В ответе пришлите файл `maven-metadata.xml` для этого артефекта.

[maven-metadata.xml](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/maven-metadata.xml)

```
<metadata modelVersion="1.1.0">
<groupId>netology</groupId>
<artifactId>java</artifactId>
<versioning>
<latest>8_282</latest>
<release>8_282</release>
<versions>
<version>8_102</version>
<version>8_282</version>
</versions>
<lastUpdated>20240123133533</lastUpdated>
</versioning>
</metadata>
```

![monitoring](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/images/9.3_MAVEN-PUBLIC_HTML.png)



### Знакомство с Maven


### Подготовка к выполнению


1. Скачайте дистрибутив с [maven](https://maven.apache.org/download.cgi).

```
pat@olZion:~/1home/cicd/dz/mnt-homeworks/09-ci-03-cicd/mvn$ wget -q https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_linux-x64_bin.tar.gz && 
sudo mkdir -p /opt/jdk && 
sudo tar -zxf openjdk-11+28_linux-x64_bin.tar.gz -C /opt/jdk/ && 
/opt/jdk/jdk-11/bin/java -version

openjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode)
pat@olZion:~/1home/cicd/dz/mnt-homeworks/09-ci-03-cicd/mvn$ 
```

```
pat@olZion:~/1home/cicd/dz/mnt-homeworks/09-ci-03-cicd/mvn$ wget -q https://dlcdn.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.zip && 
sudo unzip -q -d /opt apache-maven-3.9.6-bin.zip && 
export PATH=$PATH:/opt/apache-maven-3.9.6/bin && 
export JAVA_HOME=/opt/jdk/jdk-11 && mvn --version

```


2. Разархивируйте, сделайте так, чтобы binary был доступен через вызов в shell (или поменяйте переменную PATH, или любой другой, удобный вам способ).

3. Удалите из `apache-maven-<version>/conf/settings.xml` упоминание о правиле, отвергающем HTTP- соединение — раздел mirrors —> id: my-repository-http-unblocker.

Удаляем из apache-maven-<version>/conf/settings.xml упоминание о правиле, отвергающем http соединение( раздел mirrors->id: my-repository-http-blocker): закомментирован блок

```
<!--<mirror>
   <id>maven-default-http-blocker</id>
   <mirrorOf>external:http:*</mirrorOf>
   <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
   <url>http://0.0.0.0/</url>
   <blocked>true</blocked>
 </mirror> -->
```

4. Проверьте `mvn --version`.

5. Заберите директорию [mvn](./mvn) с pom.


### Основная часть


1. Поменяйте в `pom.xml` блок с зависимостями под ваш артефакт из первого пункта задания для Nexus (java с версией 8_282).

[pom.xml](https://github.com/12sergey12/9.3_CI-CD-processes/blob/main/mvn/pom.xml)

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.netology.app</groupId>
  <artifactId>simple-app</artifactId>
  <version>1.0-SNAPSHOT</version>
   <repositories>
    <repository>
      <id>my-repo</id>
      <name>maven-public</name>
      <url>http://178.154.205.189:8081/repository/maven-public/</url>
    </repository>
  </repositories>
  <dependencies>
     <dependency>
      <groupId>netology</groupId>
      <artifactId>java</artifactId>
      <version>8_282</version>
      <classifier>distrib</classifier>
      <type>tar.gz</type>
    </dependency>
  </dependencies>
</project>
```

2. Запустите команду `mvn package` в директории с `pom.xml`, ожидайте успешного окончания.

<details><summary>Вывод команды mvn package</summary>

```
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/mvn# mvn package
[INFO] Scanning for projects...


```

</details>

3. Проверьте директорию `~/.m2/repository/`, найдите ваш артефакт.

```
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/mvn# ls ~/.m2/repository/netology/java/8_282/
java-8_282-distrib.tar.gz	
java-8_282.pom.lastUpdated
java-8_282-distrib.tar.gz.sha1	
_remote.repositories
root@baranovsa:/home/baranovsa/9.3_baranov/mnt-homeworks/09-ci-03-cicd/mvn# 
```
4. В ответе пришлите исправленный файл `pom.xml`.

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <groupId>com.netology.app</groupId>
  <artifactId>simple-app</artifactId>
  <version>1.0-SNAPSHOT</version>
   <repositories>
    <repository>
      <id>my-repo</id>
      <name>maven-public</name>
      <url>http://178.154.205.189:8081/repository/maven-public/</url>
    </repository>
  </repositories>
  <dependencies>
     <dependency>
      <groupId>netology</groupId>
      <artifactId>java</artifactId>
      <version>8_282</version>
      <classifier>distrib</classifier>
      <type>tar.gz</type>
    </dependency>
  </dependencies>
</project>
```


---

### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
