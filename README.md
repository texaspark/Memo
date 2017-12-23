1. phpmyadmin for cf source
https://github.com/cloudfoundry-community/phpmyadmin-cf

2. Home 에서 링크 - home.jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!  
</h1>

<P>  The time on the server is ${serverTime}. </P>

<a href="<%=request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + "/3dGlassLamiCtq.do"%>">3dGlassLamiCtq</a>
<br/>
<br/>
<a href="<%=request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + "/cncMachineToolManagement.do"%>">cncMachineToolManagement</a>
</body>
</html>

3. Redis를 Session 저장소로 구성하여 Session 유지 
- Session에 setAttribute하고 getAttribute하는 간단한 Sample app 작성

3.1 Set.jsp 작성
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>Set Session Value</title>
</head>
<body>
CF_INSTANCE_INDEX: <%=System.getenv("CF_INSTANCE_INDEX") %> <br>

<%
	session.setAttribute("gtc", "gmes3");
	out.println("Set gtc value as gmes3...");
%>
</body>
</html>

3.2 get.jsp 작성 
<%@ page language="java" contentType="text/html; charset=EUC-KR"
    pageEncoding="EUC-KR"%>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=EUC-KR">
<title>GTC Session Test</title>
</head>
<body>
CF_INSTANCE_INDEX: <%=System.getenv("CF_INSTANCE_INDEX") %> <br>


<%
	String sessionValue = session.getAttribute("gtc") + "";
	out.println("Session gtc value=" + sessionValue);
%>
</body>
</html>

4. 운영중인 사이트를 중단하지 않고 JDK, WAS 버전 업그레이드
4.1)step1: 무중단 배포와 동일한 기법을 사용하여 Tomcat Version을 8.5.15로 upgrade한다. 현재 tomcat version 확인
# cf ssh fmb-server -c “/home/vcap/app/.java-buildpack/oracle_jre/bin/java -cp /home/vcap/app/.java-buildpack/tomcat/lib/catalina.jar org.apache.catalina.util.ServerInfo” 명령으로 tomcat version을 확인한다.

4.2)step2: Tomcat upgrade를 위하여 manifest.yml에 tomcat 상위버전(8.1.15) 명시하고 신규버전(Green) 배포

manifest.yml 파일 
applications:
- name: fmb-server-tc85
  instances: 1
  memory: 3072M
  disk_quota: 4096M
  routes:
  - route: fmb-server-tc85.apps.cfpush.net
  path: fmb_20170623.war
  buildpack: oracle-jdk78-tomcat78-offline
  stack: cflinuxfs2
  timeout: 300
  env:
    JBP_CONFIG_ORACLE_JRE: '{ jre: { version: 1.7.0_+ } }'
    JBP_CONFIG_TOMCAT: '{ tomcat: { version: 8.5.15 } }'
    JBP_CONFIG_SPRING_AUTO_RECONFIGURATION: '{ enabled: false }'

4.3)Step3: fmb-server-tc85로 업그레이드 버전(green) 배포
#cf push fmb-server-tc85 -n fmb-server-tc85

4.4)Step4: fmb-server-tc85 container 접속하여 tomcat version 확인
# cf ssh fmb-server-tc85 -c “/home/vcap/app/.java-buildpack/oracle_jre/bin/java -cp /home/vcap/app/.java-buildpack/tomcat/lib/catalina.jar org.apache.catalina.util.ServerInfo” 명령으로 tomcat version을 확인한다.

4.5)Step5: Blue/Green 배포 기법으로 routing 정보를 변경하여 upgrade를 마무리 한다 
#cf map-route fmb-server-tc85 apps.cfpush.net --hostname fmb-server
#cf unmap-route fmb-server apps.cfpush.net --hostmane fmb-server
#cf stop fmb-server

5. Auto-Scaling: Apache bench 사용 부하 발생 시키는 명령어
# ab -c 10 -n 1000000 http://spring-music-gtc2.apps.cfpush.net/albums

6. 기타 지식
#!/bin/bash

cf api --skip-ssl-validation https://api.system.cfpush.net
cf login -u 'parker' -p 'park7143' 

sudo su chmod 775 cf.parker.sh
소유자: 읽기, 쓰기, 실행
그룹: 읽기, 쓰기, 실행
모두: 읽기, -, 실행

특정 문자열만 추출: grep
파이프라인 | : 명령줄을 이어줌

7. Scripts

7.1) bosh_login.sh

export BUNDLE_GEMFILE=/home/tempest-web/tempest/web/vendor/bosh/Gemfile
bundle exec bosh --ca-cert /var/tempest/workspaces/default/root_ca_certificate target 172.16.30.11  << EOF
director
o0OFizsrq5sY_muBSNW5ZeCElD3nQeXO
EOF

7.2) bosh_vm.sh

#!/bin/sh
ssh vcap@172.16.30.11

7.3) cf-admin.sh

#!/bin/bash

cf api --skip-ssl-validation https://api.system.cfpush.net
cf login -u 'admin' -p 'vIantAm6K4Wpj5-qCb1J65y_FTYom6vG' -o system -s system

7.4) cf-gtc1.sh

#!/bin/bash

cf api --skip-ssl-validation https://api.system.cfpush.net
cf login -u 'gtc1' -p 'gtc1' -o gtc-dev -s gtc1

7.5) cf-log-search-mon.sh

#!/bin/bash

export BUNDLE_GEMFILE=/home/tempest-web/tempest/web/vendor/bosh/Gemfile

echo " " | awk '{ printf("%-30s %-6s %-6s %-6s %6s %8s %8s\n", "", "CPU-U", "CPU-S", "CPU-W", "MEM", "(MB)", "P-DISK"); }'
echo "+--------------------------------------------------------------------------------+"

bundle exec bosh vms --vital p-logsearch-0be12d40c721075f9352 2> /dev/null | egrep -e "elasticsearch_data|elasticsearch_master|firehose-to-syslog|ingestor|kibana|maintenance|parser|queue|router" | awk -F'|' '{ print $2, $8, $9, $10, $11, $15; }' | awk '{ printf("%-30s %-6s %-6s %-6s %6s %8s %8s\n",$1, $3, $4, $5, $6, $7, $8); }' | sort 

7.6) cf-mon.sh

#!/bin/bash

export BUNDLE_GEMFILE=/home/tempest-web/tempest/web/vendor/bosh/Gemfile

echo " " | awk '{ printf("%-35s %-6s %-6s %-6s %6s %8s %8s\n", "", "CPU-U", "CPU-S", "CPU-W", "MEM", "(MB)", "P-DISK"); }'
echo "+-------------------------------------------------------------------------------+"

bundle exec bosh vms --vital cf-4a61444a4a10c7bcdbef 2> /dev/null | egrep -e "clock_global|cloud_controller|consul_server|diego_brain|diego_cell|doppler|etcd_server|ha_proxy|loggregator|mysql|nats|router|uaa|nfs_server" | awk -F'|' '{ print $2, $8, $9, $10, $11, $15; }' | awk '{ printf("%-35s %-6s %-6s %-6s %6s %8s %8s\n",$1, $3, $4, $5, $6, $7, $8); }' | sort 

7.7) cf-parker.sh

#!/bin/bash
cf api --skip-ssl-validation https://api.system.cfpush.net
cf login -u 'parker' -p 'park7143' 

7.8) cf-semp.sh

#!/bin/bash

cf api --skip-ssl-validation https://api.system.cfpush.net
cf login -u 'gtc1' -p 'gtc1' -o gtc-dev -s semp

7.9) load1.sh

#!/bin/bash

TARGET=http://spring-music-gtc2.apps.cfpush.net/albums

ab -c 10 -n 100000 "${TARGET}"

#ab -c 50 -n 1000000 "${TARGET}/v1/grids/monthly?IF=IF-XPG-MNGR-007&ver=1.0&response_format=json&id_package=20&ui_name=NXUHDSTB2Q&resltn_cd=180x258&rst_type=4k&hash_id=&yn_per=N&sw_ver=10.2.41&stb_model=BHX-UH400&sub_pack=25&iptv_pack=1&co_pack=0&stb_id=%7B7968597F-E1DA-11E5-A490-FDCFBFF8EC17%7D&menu_id=A000001663" &

#ab -c 50 -n 1000000 "${TARGET}/v1/grids/monthly?IF=IF-XPG-MNGR-007&ver=1.0&response_format=json&id_package=20&ui_name=NXUHDSTB2Q&resltn_cd=180x258&rst_type=4k&hash_id=&yn_per=N&sw_ver=10.2.41&stb_model=BHX-UH400&sub_pack=25&iptv_pack=1&co_pack=0&stb_id=%7B7968597F-E1DA-11E5-A490-FDCFBFF8EC17%7D&menu_id=A000001715" &

#ab -c 50 -n 1000000 "${TARGET}/v1/grids/monthly?IF=IF-XPG-MNGR-007&ver=1.0&response_format=json&id_package=20&ui_name=NXUHDSTB2Q&resltn_cd=180x258&rst_type=4k&hash_id=&yn_per=N&sw_ver=10.2.41&stb_model=BHX-UH400&sub_pack=25&iptv_pack=1&co_pack=0&stb_id=%7B7968597F-E1DA-11E5-A490-FDCFBFF8EC17%7D&menu_id=A000003146" &

#ab -c 50 -n 1000000 "${TARGET}/v1/grids/monthly?IF=IF-XPG-MNGR-007&ver=1.0&response_format=json&id_package=20&ui_name=NXUHDSTB2Q&resltn_cd=180x258&rst_type=4k&hash_id=&yn_per=N&sw_ver=10.2.41&stb_model=BHX-UH400&sub_pack=25&iptv_pack=1&co_pack=0&stb_id=%7B7968597F-E1DA-11E5-A490-FDCFBFF8EC17%7D&menu_id=A000003151" &

#ab -c 50 -n 1000000 "${TARGET}/v1/grids/monthly?IF=IF-XPG-MNGR-007&ver=1.0&response_format=json&id_package=20&ui_name=NXUHDSTB2Q&resltn_cd=180x258&rst_type=4k&hash_id=&yn_per=N&sw_ver=10.2.41&stb_model=BHX-UH400&sub_pack=25&iptv_pack=1&co_pack=0&stb_id=%7B7968597F-E1DA-11E5-A490-FDCFBFF8EC17%7D&menu_id=A000003153" &

7.10) t1.sh

#!/bin/bash

export CF_TRACE=true
cf api --skip-ssl-validation https://api.system.cfpush.net

echo "====================[ auth ]======================="

cf auth 'system_services' 'rw02PM-qUqCg8GGVFhwd5VwFucFsvRDS'

echo "==================================================="


