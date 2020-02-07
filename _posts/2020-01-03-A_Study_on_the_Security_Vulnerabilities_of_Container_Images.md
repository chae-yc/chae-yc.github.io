---
title: "논문읽기 - 컨테이너 이미지의 보안 취약성 조사 연구"
categories:
  - scientist
  - paper_reading
tags:
  - container
  - image
  - security
last_modified_at: 2020-01-05T19:21:00+09:00
toc: true
published: true
share: false
---

본 글은
**탁병철. (2018). 컨테이너 이미지의 보안 취약성 조사 연구. 한국차세대컴퓨팅학회 논문지, 14(3), 7-15.**
를 읽고 개인적으로 정리한 글입니다.
문제가 발생될 시 포스트를 삭제하도록 하겠습니다.

## 1장 서론

컨테이너 기술의 활용이 사용의 편의성, 손쉬운 패키징(packaging), 배포(deploy)의 신속성(Time-to-market)의 장점으로 확산되고 있지만,
다수의 사용자들이 이미지를 변경하여 배포할 수 있다는 특징 때문에 그 과정에서 취약점들이 생성된다면 그 또한 빠르게 확산될 수도 있다는 문제가 있다.
이러한 위험성은 아래와 같은 다른 연구들에서도 보고된 적이 있다. [[1](#1), [2](#2), [3](#3)]
문제가 인식되어서 여러 해결책들 또한 개발되고 있다.[[9](#9), [10](#10), [11](#11), [12](#12)]
하지만 아직도 컨테이너의 취약성에 대해서는 연구해야 할 측면이 많다. [[2](#2), [4](#4), [7](#7), [13](#13), [14](#14)]

다음과 같은 **의문**들에 대한 **해답**을 구해야한다.

- 컨테이너 이미지 당 평균 취약점은 얼마나 되는 가?
- 어떠한 취약점들이 가장 많이 발견되는가?
- 취약점이 보완되기 까지는 얼마나 걸리는가?
- 컨테이너를 사용하면 나의 시스템에 보안 측면에서 어떠 한 직간접적인 영향이 있는가?

연구의 **방향**은

- 약 만여 개의 도커 컨테이너 이미지를 분석하여서 컨테이너 이미지 스캐닝 도구인 ASC(Agentless-system-crawler)를 사용하여
컨테이너 이미지의 OS정보, 파일들의 메타데이터, 설치된 패키지들의 목록과 버전 그리고 지정된 설정파일들의 내용 등을 추출한다.
- 추출한 데이터들은 데이터베이스에 저장되어 분석 모듈에서 필요한 정보를 질의할 때에 사용된다.
분석 모듈은 각 이미지 별 취약성 결과를 다시 데이터베이스에 저장한다.

연구에서 **분석한 정보**는

- 컨테이너 이미지들의 기본 OS 정보 통계 분류
- 권장 보안 규칙(Compliance Rule)들의 부합 정도
- 설치된 패키디들의 취약점(Package Vulnerability)

**결론**은

- 99%의 컨테이너 이미지가 평균적으로 다섯 개 이상의 권장 보안 규칙을 어기고 있다.
- 92%의 이미지에서는 평균 10개 이상의 패키지 취약점들이 발견되었다.

이 분석에 대한 결과와 해석을 **본문**에서 설명한다.

- 2장: 컨테이너 이미지 분석을 수행하기 위하여 구성한 분석 플랫폼의 구조를 설명
- 3장: OS 배포판 분포 분석, 권장 보안 규칙 부합성 분석, 패키지 취약점 분석 결과를 제시
- 4장: 분석 결과에 대한 의미와 해석
- 5장: 결론과 향후 계획


## 2장 컨테이너 이미지 분석 플랫폼 구조

컨테이너 이미지 보안 취약성 분석을 위한 플랫폼은
총 5개의 컴포넌트(Imnage Name Extractor, Image Scanner, Compliance Rule Analyzer, Package Vulnerability Analyzer, Package Vulnerability Info Collector)와
플랫폼의 구심점 역할을 하는 하나의 데이터베이스(Scanned Image Database - Elasticsearch[[15](#15)] 사용)로 구성되어 있다.

컴포넌트들 사이의 직접적인 통신이 없고 데이터베이스를 통한 간접적인 참조를 사용하기 때문에 **새로운 컴포넌트를 추가, 확장 하는 것이 용이**하다.

각 컴포넌트들의 **역할** 은

- Image Name Extractor
  - Docker hub에 HTTP query를 하여 인기도 순으로 정렬된 컨테이너 이미지 이름들이 있는 HTML page 들을 받아 차례로 파싱한다.
- Image Scanner
  - 데이터베이스에 새로운 이미지 이름이 저장되는 것을 감지하면
해당 이미지를 Docker hub로 부터 pull 한 다음 ASC [[5](#5), [6](#6)] 를 사용하여 스캔하고 결과를 데이터베이스에 저장한다.
  - 파일 목록, 파일 메타데이터, 패키지 리스트, 설정 파일 내용들, OS 정보 등이다
- Compliance Rule Analyzer(권장 보안 규칙 분석기)
  - 대상 컨테이너 이미지가 보안 규칙들을 만족하는지를 검사하는 일을 담당한다.
  - 현재 정의된 권장 규칙들은 총 40이다. [Compliance Rules](#compliance-rules-table-권장-보안-규칙들) 크게 File/Directory 권한, SSH 서버 설정, 암호 설정 등의 사항들로 분류된다.
  - 다른 컴포넌트 들과 같이 새로운 데이터가 데이터베이스에 저장되었는지를 폴링 (Polling)을 통해 감지하여 분석을 진행하고 결과를 데이터베이스에 저장한다.
- Package Vulnerability Analyzer(패키지 취약점 분석기)
  - 패키지의 취약점을 분석하는 일을 담당한다. 현재 대상 이미지에 설치되어 있는 패키지들의 버전들을 현재 알려진 최신 버전의 패키지들과 비교하고, 만약 최신 버전이 아닐 경우 어떤 취약점이 있는지를 파악한다.
    - Package Vulnerability Info Collector가 데이터베이스에 저장한 값들을 참조한다.
  - 다른 컴포넌트와 동일한 방법으로 동작한다.
- Package Vulnerability Info Collector
  - 인터넷에서 현재 알려진 패키지들의 취약점들의 정보를 주기적으로 모아서 데이터베이스에 저장한다.
  - 패키지 취약점 정보는 리눅스 배포판(Distro) 마다 별도로 공시를 하며 현재 참조하는 배포판은 Debian, Ubuntu, 그리고 CentOS 이다.
  - 각 배포판별 취약점 목록의 웹 주소는 [Package Vunerability Source](#package-vulnerability-source) 에서 볼 수 있다.
  - 각 사이트 별로 별도의 텍스트 파싱 로직을 구현하여 게시판 형식으로 공시된 취약점 정보를 추출한다.
  - 추출하는 정보는 패키지명, CVE(Common Vulnerability Exposure ID), 취약점 설명, Fixed version 번호 이다.

## 3장 컨테이너 이미지 분석 결과

### 3.1 OS 분석

Docker Hub에서 배포되고 있는 이미지들의 base image로 사용되고 있는 리눅스 배포판들의 종류별 분포를 파악한다.
전체적으로는 21개의 리눅스 배포판들이 컨테이너 이미지로 사용되고 있으며,
아래의 **<표 3>**는 연구 과정에서 발견된 리눅스 배포판들의 목록과 각각의 분포를 보여준다.

**<표 3>** Linux Distro Distribution

|Rank|Distro|Count|Percent|
|:-:|:-:|:-:|:-:|
|1|Debian|3189|31.7%|
|2|Alpine|2896|28.8%|
|3|Ubuntu|2634|26.1%|
|4|Centos|766|7.6%|
|5|Scratch|207|2.1%|
|6|buildroot|202|2.0%|
|7|Fedora|67|0.7%|
|8|Opensuse|32|0.3%|
|9|Oracle Linux|20|0.2%|
|10|arch|15|0.1%|
|11|photon|11|0.1%|
|12|rhel|11|0.1%|
|13|amzn|7|0.1%|
|14|sles|6|0.1%|
|15|Gentoo|3|0.0%|
|16|clear-linux-os|2|0.0%|
|17|slackware, linuxmint, mageia, euleros, kali appeared only once each|||

상위 세 개의 배포판인 Debian, Alpine, Ubuntu 비율의 총합이 87%이다.
그 다음인 CentOS는 7% 이며 나머지 배포판 들은 사용이 미미하다.
> Scrtach는 리눅스 배포판의 종류가 아닌, 파일 시스템이 존재하지 않는 빈 컨테이너 이미지를 의미한다.
> 컨테이너 이미지의 내부에는 리눅스 OS 관련 파일들이 반드시 있을 필요는 없으며 단지 실행하고자 하는 어플리케이션 바이너리만 들어있는 이미지들도 존재한다.

Alpine 이미지에 대해서,
> Alpine 이미지는 비교적 새 로 만들어진 배포판이지만 컨테이너에서는 그 사용 빈도가 급격하게 증가하여 주요 배포판 중 하나가 되었으며 사용빈도는 계속 증가할 것으로 예상된다. Alpine 이미지는 컨테이너 사용을 목표로 개발되었 으며, Busybox와 musl 라이브러리와 같이 크기를 최소화 한 라이브러리들을 장착하여 다른 이미지들 (100MB-500MB)에 비하여 그 크기(5-20MB) 가 매우 작은 장점을 가진다.

각 배포판별로 이미지 내부에 포함되어 있는 파일들의 평균 개수와 설치되어 있는 패키지들의 수를 조사한 결과가 <표 4>에 정리되어 있다.

**<표 4>** 평균 파일과 패키지 개수

|Distro|Average File Count|Average Package Count|
|:-:|:-:|:-:|
|Debian|26K|275|
|Alpine|8.5K|36|
|Ubuntu|31.3K|317|
|Centos|29.9K|287|
|Scratch|6.4K|0|
|buildroot|2.2K|0|
|Fedora|26.2K|261|
|Opensuse|31.7K|225|
|Oracle Linux|21.1K|196|
|arch|54K|241|
|photon|7.2K|62|
|rhel|10.8K|186|

Debian, Ubuntu와 같이 잘 알려진 이미지의 경우 수만 개의 파일이 있지만
Alpine 이미지의 경우 파일의 수와 평균 패키지의 수가 적다. 그래서 Alpine 이미지의 크기는 다른 것들에 비해 1/10 정도의 크기를 가진다.

### 3.2 권장 보안 규칙 부합성 분석

> 각 세 개의 모드를 추가로 분석해서 배포판의 구성을 살펴보니 (그림4)와 같이 첫 번째 모드(x축의 값 2)에서는 Alpine 이미지가 94.7%였으며 두 번째의 모드(x축의 값 6)에서는 Debian과 Ubuntu가 각각 58%, 42%, 그 리고 세 번째 모드(x축의 값 9)에서는 CentOS가 54%, Debian과 Ubuntu가 15%, 22%를 차지하였 다. 따라서 배포판별로 서로 다른 위배 패턴을 확실 히 가지고 있으며 이들이 합쳐져서 (그림 3)과 같 은 멀티모드 분포를 보여준다. 이를 통해 또한 Alpine 이미지가 보편적으로 가장 위배율이 낮은 것으로 나타났다. CentOS는 평균적으로 가장 많은 보안 규칙 위배를 하는 것으로 나타났다.

![그림3](/assets/images/2020-01-03-A_Study_on_the_Security_Vulnerabilities_of_Container_Images/Compliance_Analyze_total.png)
![그림4](/assets/images/2020-01-03-A_Study_on_the_Security_Vulnerabilities_of_Container_Images/Compliance_Analyze_each_mode.png)

모든 배포판을 종합적으로 보았을 때 위배된 규칙들.
> 자동으로 로그아웃되도록 하는 규칙은 거의 모든 이미지에서 위배하고 있다.
상위에는 그 외에도 /etc/shadow와 /var/log/wtmp파일의 권한 설정에 문제가 많은 이미지들이 다수로 있는 것으로 밝혀 졌다.
또한, 대체로 password관련된 규칙들을 많 이 위배하는 것으로 파악되었다. 이와 반대로 어떠 한 이미지에서도 한 번도 위배되지 않은 규칙들도 8개 존재하였다.

분석 결과
> 전체적인 평균 규칙 위배 수는 5.2이지만 배포판 별로 조사에서는 편차가 매우 큰 것으로 나타났다.
Alpine 이미지가 평균 위배수가 2.2로 가장 적은 것으로 나타났다.
Debian과 Ubuntu는 모두 6개 이상의 평균 위배수 를 보인다.

### 3.3 패키지 취약점 분석

> 조사에 사용한 전체 이미지의 수는6589개로 보안 규칙 분석에 사용한 10073개의 이미지보다 적은 수이다.

Alpine 이미지의 패키지 취약점 분석이 빠져있다.
> 패키지 취약점을 조사하기 위해서는 배포판 관리기관이 주기적으로 제공하는 Security Notice정보가 필요하지 만 Alpine 이미지는 아직까지는 이를 제공하지 않고 있다. 따라서 Alpine 이미지는 제외하고 Debian, Ubuntu, CentOS 이미지들만을 분석하였다.

결론
> 가장 흔하게 발 견된 취약점은 perl 패키지의 취약점인 것으로 나 타났다. Perl 패키지의 취약점을 없애기 위해서는 버전이 “5.24.1-3+deb9u3”이어야 하지만 16 종류의 다른 이전 버전 perl 패키지들이 발견되었 다. 추가적으로 배포판별로 어떠한 패키지 취약점 들이 가장 많은지 조사를 해보았다. Ubuntu의 경우 libssl이 가장 취약하였으며, Debian의 경우는 openssl 패키지였다. 두 배포판 모두 SSL 관련 패 키지에서 가장 많은 취약점이 드러나고 있다. Perl 패키지의 경우는 Debian에서 더 많이 취약점이 발 견되었다.

## 4장 토의

Alpine의 작은 이미지 크기와 적은 수의 파일, 패키지 수로 인해서 Alpine 이미지의 사용 비율이 높게 나타난다. 천만번 이상 다운로드 된 이미지들 사이에서 Alpine 이미지의 비중이 38% 에 달한다.
Alpine 이미자가 대체로 보안 규칙 위배는 적은 것으로 나타나고 있지만,
현재 사용한 보안 위배 규칙들은 기존의 리눅스 보안 상태 점검을 위해서 일반적으로 사용되는 규칙들이며 Alpine 이미지와 같이 컨테이너를 위해 만들어진 비교적 새로운 배포판의 경우에도 적절한 규칙인지에 대한 추가적인 조사가 필요하다.

종합적으로 분석하면 공개된 컨테이너 이미지들의 보안 취약성은 보편적으로 매우 심각하다.
약 92%이상의 이미지가 적어도 한 종류의 취약점을 가지고 있으며, Alpine 이미지는 다른 배포판에 비해 보안 규칙 위배도 적은 것으로 나타난다.

패키지 취약점은 100개가 넘을 수도 있다는 것이 발견되었고 SSL 관련 취약점이 많다.
따라서 컨테이너 이미지를 다운로드 받아서 검사 없이 바로 사용하는 것은 위험하므로 스캔을 통해 취약점을 발견하고 제거하는 자동화된 시스템을 사용해야한다.

## 5장 결론

99%의 이미지에서 보안 권장 규칙 위배가 발생하였다.
92%의 이미지에서는 한 개 이상의 보안 취약점이 존재하였다.
배포판 별로 서로 종류나 수가 매우 다른 분포를 보이고 있다.

향후 연구 방향은, nginx, mysql, mongodb 등과 같은 어프리케이션에 적용할 수 있는 보안 권장 규칙을 추가하여 어플리케이션의 보안성 조사를 할 예정이다.
Alpine 이미지의 패키지 취약점 분석을 확인하지 못했으므로 그 기능을 추가한다.

## References

### Reference list of this paper

1. <a name="1"></a> B. Tak, C. Isci, S. Duri, N. Bila, S. Nadgowda, and J. Doran. Understanding security implications of using containers in the cloud.
In 2017 USENIX Annual Technical Conference (USENIX ATC 17), pages 313–319, Santa Clara, CA, 2017. USENIX Association.
2. <a name="2"></a>Gummaraju, Jayanth and Desikan,Tarun and Turner, Yoshio. Over 30% of Official Images in Docker Hub Contain High Priority Security Vulnerabilities. https://www.banyanops.com/blog/analyzing-docker-hub/
3. <a name="3"></a>Vulnerability QuickView.
https://pages.riskbasedsecurity.com/hubfs/R eports/2017/2017\%20Year\%20End\%20Vul nerability%20QuickView\%20Report.pdf
4. <a name="4"></a>R. Shu, X. Gu, and W. Enck. A study of security vulnerabilities on docker hub. In Proceedings of the Seventh ACM on Conference on Data and Application Security and Privacy, CODASPY ’17, pages 269–280, New York, NY, USA, 2017. ACM.
5. <a name="5"></a>Agentless system crawler. https://github.com/cloudviz/agentless-syste m-crawler
6. <a name="6"></a>R. Koller, C. Isci, S. Suneja, and E. de Lara. Unified monitoring and analytics in the cloud. In 7th USENIX Workshop on Hot Topics in Cloud Computing (HotCloud 15), Santa Clara, CA, 2015. USENIX Association
7. <a name="7"></a>T. Bui. Analysis of docker security. arXiv preprint arXiv:1501.02967, 2015.
8. <a name="8"></a>Dokerhub. https://hub.docker.com/.
9. <a name="9"></a>Anchore. Open source tools for container security and compliance. http://anchore.com.
10. <a name="10"></a>Aqua. https://www.aquasec.com/.
11. <a name="11"></a>Clair. Automatic container vulnerability and security scanning for appc and docker. http://coreos.com/clair/
12. <a name="12"></a>F. Oliveira, S. Suneja, S. Nadgowda, P.
Nagpurkar, and C. Isci. Opvis: extensible, cross-platform operational visibility and analytics for cloud. In Proceedings of the 18th ACM/IFIP/USENIX Middleware Conference: Industrial Track, pages 43–49. ACM, 2017.
13. <a name="13"></a>T. Combe, A. Martin, and R. Di Pietro. To docker or not to docker: A security perspective. IEEE Cloud Computing, 3(5):54–62, 2016.
14. <a name="14"></a>E. Reshetova, J. Karhunen, T. Nyman, and N. Asokan. Security of os-level virtualization technologies. In Nordic Conference on Secure IT Systems, pages 77–93. Springer, 2014.
15. <a name="15"></a>Elasticsearch. https://www.elastic.co/

### Compliance Rules table (권장 보안 규칙들)

|Category|Rules|
|:------:|--------------|
|SSH Server|SSH server 는 설치되어 있으면 안된다 <br>암호 기반의 SSH 인증은 disable 되어 있어야한다.<br>root로 로그인 하는 것은 disable 되어 있어야한다.<br>SSH protocol version 2 가 사용되어야 한다.|
|Password|암호는 최대 90일 까지만 사용할 수 있어야한다.<br>암호는 최소 8자 이상이어야 한다.<br>암호는 최소 1일은 유지해야한다.|
|Permission 644|/var/log/wtmp, /var/run/utmp, /etc/group, /etc/passwd, /etc/fstab, /etc/sudoers, /etc/crontab, /etc/ld.so.conf|
|Permission 755|/bin, /boot, /dev, /etc, /etc/cron.daily, /etc/cron.hourly, /etc/cron.monthly, /etc/cron.weekly, /lib, /mnt, /sbin|
|Permission 744|/etc/profile, /etc/hosts.allow, /etc/sysctl.conf|
|Permission 700|/root, /etc/mtab|
|Permission 400|/etc/shadow|
|Remote Access|telnet server, rssh server, FTP server는 존재하면 안된다.<br>|
|Others|Umask 는 022 이거나 더 제한적이어야 한다.<br>1시간 이후에는 자동 로그아웃이 되어야한다.<br>/root/.rhosts에 대한 READ/WRITE access는 root만 가능해야한다.<br>/root/.netrc에 대한 READ/WRITE access는 root만 가능해야한다.<br>각 UID는 한번만 사용되어야 한다.|

![compiance rules tables](/assets/images/2020-01-03-A_Study_on_the_Security_Vulnerabilities_of_Container_Images/Compliance-Rules.png){: width="30%" height="30%"}

### Package Vulnerability Source

|Linux Distro Name|URL|
|:-:|-|
|USN(Ubuntu Security Notices)|<https://usn.ubuntu.com/atom.xml><br><https://lists.ubuntu.com/archives/ubuntu-security-announce>|
|DSA(Debian Security Announce)|<https://lists.debian.org/debian-security-announce>|
|CeSA(CentOS Security Alerts)|<https://lists.centos.org/pipermail/centos-announce>|

## 의문점

1. Alpine 이미지의 크기가 작기 때문에 보안 규칙 위배도 적은 것이 아닌가? 그럼 이미지 크기 별 비율 등으로 따져야하지 않을까?
