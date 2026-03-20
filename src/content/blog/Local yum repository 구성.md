---
title: 'First post'
description: 'Lorem ipsum dolor sit amet'
pubDate: 'Jul 08 2022'
heroImage: '/blog-placeholder-3.jpg'
---
## 3. Local yum repository 구성

### A. Redhat 8 / CentOS 8

- dark site를 전제로 한 것으로 인터넷 접속이 허용되는 서버는 불필요

(1) OS 미디어 마운트

```
# mount /dev/cdrom /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
# cd /etc/yum.repos.d/
# mkdir backup
# mv *.repo backup
```

(2) tar 설치 (이 과정 없이 cp 명령 이용해도 무방함)

```
# cd /mnt/BaseOS/Packages/
# yum -y install tar-1.30-4.el8.x86_64.rpm
```

(3) Repository 디렉토리 생성

```
# mkdir -p /rhel82media
```

(4) OS 미디어의 파일들을 repository 디렉토리로 복사 (cp 명령 사용해도 됨)

```
# cd /mnt
# tar cvf - . | (cd /rhel82media/; tar xvf - )
```

(5) yum repository 파일 생성 : Redhat8 부터 OS 패키지들이 BaseOS와 AppStream으로 나눠서 관리 됨

```
# vi /etc/yum.repos.d/RedhatOS-Media.repo
[InstallMedia-BaseOS]
name=Redhat82 - BaseOS
metadata_expire=1
gpgcheck=1
enabled=1
baseurl=file:///rhel82media/BaseOS/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[InstallMedia-AppStream]
name=Redhat82 - AppStream
metadata_expire=1
gpgcheck=1
enabled=1
baseurl=file:///rhel82media/AppStream/
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release  $→$ for Redhat
```

(6) yum repository 생성 : Redhat8은 dnf 명령 이용

```
# dnf clean all
# subscription-manager clean
# dnf repolist
```

(7) 확인

```
# yum list
# yum grouplist
```

### B. Redhat 7 / CentOS 7

- dark site를 전제로 한 것으로 인터넷 접속이 허용되는 서버는 불필요

(1) OS 미디어 마운트

```
# mount /dev/cdrom /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
```

(2) createrepo 패키지 설치 : “Minimal” 옵션으로 설치 되어 수동설치해야 함 (OS 버전에 따라 각 rpm 버전은 다를 수 있음)

```
# cd /mnt/Packages
# rpm -ivh deltarpm-3.6-3.el7.x86_64.rpm
# rpm -ivh python-deltarpm-3.6-3.el7.x86_64.rpm
# rpm -ivh libxml2-python-2.9.1-6.el7_2.3.x86_64.rpm
# rpm -ivh createrepo-0.9.9-28.el7.noarch.rpm
```

(3) Repository 디렉토리 생성

```
# mkdir -p /redhat7media
```

(4) OS 미디어의 파일들을 repository 디렉토리로 복사 (tar 명령 사용해도 됨)

```
# cd /mnt
# cp -R * /redhat7media/
```

(5) Repository 파일 구성 – 사전에 `/etc/yum.repos.d/` 폴더 내의 파일들을 다른 곳으로 옮겨서 백업 후 아래 내용 신규 생성

```
# vi /etc/yum.repos.d/RedhatOS-Media.repo
[RedhatOS7Media]
name=RedhatOS-$releasever - Media
baseurl=file:///redhat7media/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release  $→$ for Redhat 7
```

(6) yum repository 생성

```
# createrepo /redhat7media
```

(7) 확인

```
# yum list
```

(8) Package group 구성 및 확인

```
# cd /redhat7media/repodata/
# ls *.xml
fe4a0ee2b4c38d9578c38ab83cebe4df87e3a409b48e8fee57-comps-Server.x86_64.xml
# createrepo -g / redhat7media /repodata/fe4a0ee2b4c38d9578c38ab83cebe4df87e3a409b48e8fee57-comps-Server.x86_64.xml /redhat7media/
# yum clean all
# yum makecache
# yum grouplist
```