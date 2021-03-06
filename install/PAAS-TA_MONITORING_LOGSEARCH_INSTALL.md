### [Index](https://github.com/PaaS-TA/Guide) > [Monitoring Install](PAAS-TA_MONITORING_INSTALL_GUIDE.md) > [PaaS] Logsearch


# Logsearch Install Guide

1\. [개요](#1)  
2\. [Logsearch 설치](#2)  
　2.1. [Prerequisite](#3)  
　2.2. [설치 파일 다운로드](#4)  
　2.3. [Logsearch 설치 환경설정](#5)   
　　● [common_vars.yml](#6)  
　　● [logsearch-vars.yml](#7)  
　　● [deploy-logsearch.sh](#8)  
　2.4. [Logsearch 설치](#9)  
　2.5. [서비스 설치 확인](#10)


## <div id='1'/>1. 개요

본 문서(Logsearch 설치 가이드)는 PaaS-TA Monitoring을 설치하기 앞서 BOSH와 PaaS-TA의 VM Log 수집을 위하여 BOSH 2.0을 이용하여 Logsearch를 설치하는 방법을 기술하였다.

## <div id='2'/>2. Logsearch 설치
### <div id='3'/>2.1 Prerequisite

1. BOSH 설치가 되어있으며, BOSH Login이 되어 있어야 한다.
2. cloud-config와 runtime-config가 업데이트 되어있는지 확인한다.
3. Stemcell 목록을 확인하여 서비스 설치에 필요한 Stemcell(ubuntu bionic 1.34)이 업로드 되어 있는 것을 확인한다.


> cloud-config 확인  
> $ bosh -e {director-name} cloud-config  

> runtime-config 확인  
> $ bosh -e {director-name} runtime-config  

> stemcell 확인  
> $ bosh -e {director-name} stemcells  


### <div id='4'/>2.2. 설치 파일 다운로드

- Logsearch를 설치하기 위한 deployment가 존재하지 않는다면 다운로드 받는다
```
$ cd ${HOME}/workspace
$ git clone https://github.com/paas-ta/common.git
$ git clone https://github.com/paas-ta/monitoring-deployment.git
```

### <div id='5'/>2.3. Logsearch 설치 환경설정

PaaS-TA VM Log수집을 위해서는 Logsearch가 설치되어야 한다. 

```
$ cd ${HOME}/workspace/monitoring-deployment/logsearch
```

### <div id='6'/>● common_vars.yml
common 폴더에 있는 common_vars.yml PaaS-TA 및 각종 Service 설치시 적용하는 공통 변수 설정 파일이 존재한다.  
Logsearch를 설치할 때는 syslog_address의 값을 변경 하여 설치 할 수 있다.  
syslog_address는 Monitoring 옵션을 포함한 BOSH와 PaaS-TA를 설치할 때의 변수값과 같은 값을 주어 설치를 한다.

```
# BOSH INFO
bosh_ip: "10.0.1.6"				# BOSH IP
bosh_url: "https://10.0.1.6"				# BOSH URL (e.g. "https://00.000.0.0")
bosh_client_admin_id: "admin"			# BOSH Client Admin ID
bosh_client_admin_secret: "giej8ett7mqsho9tx7s3"	# BOSH Client Admin Secret('echo $(bosh int ~/workspace/paasta-5.0/deployment/paasta-deployment/bosh/{iaas}/creds.yml --path /admin_password)' 명령어를 통해 확인 가능)
bosh_director_port: 25555			# BOSH director port
bosh_oauth_port: 8443				# BOSH oauth port
bosh_version: 271.2				# BOSH version('bosh env' 명령어를 통해 확인 가능, on-demand service용, e.g. "271.2")

# PAAS-TA INFO
system_domain: "10.0.1.80.nip.io"		# Domain (nip.io를 사용하는 경우 HAProxy Public IP와 동일)
paasta_admin_username: "admin"			# PaaS-TA Admin Username
paasta_admin_password: "admin"			# PaaS-TA Admin Password
paasta_nats_ip: "10.0.1.121"
paasta_nats_port: 4222
paasta_nats_user: "nats"
paasta_nats_password: "7EZB5ZkMLMqT73h2JtxPv1fvh3UsqO"	# PaaS-TA Nats Password (CredHub 로그인후 'credhub get -n /micro-bosh/paasta/nats_password' 명령어를 통해 확인 가능)
paasta_nats_private_networks_name: "default"	# PaaS-TA Nats 의 Network 이름
paasta_database_ips: "10.0.1.123"		# PaaS-TA Database IP (e.g. "10.0.1.123")
paasta_database_port: 5524			# PaaS-TA Database Port (e.g. 5524(postgresql)/13307(mysql)) -- Do Not Use "3306"&"13306" in mysql
paasta_database_type: "postgresql"                      # PaaS-TA Database Type (e.g. "postgresql" or "mysql")
paasta_database_driver_class: "org.postgresql.Driver"   # PaaS-TA Database driver-class (e.g. "org.postgresql.Driver" or "com.mysql.jdbc.Driver")
paasta_cc_db_id: "cloud_controller"		# CCDB ID (e.g. "cloud_controller")
paasta_cc_db_password: "cc_admin"		# CCDB Password (e.g. "c418e687c4Kx!" 영어/숫자/특수문자 혼용 8자리 이상)
paasta_uaa_db_id: "uaa"				# UAADB ID (e.g. "uaa")
paasta_uaa_db_password: "uaa_admin"		# UAADB Password (e.g. "ifb2497iEA5!" 영어/숫자/특수문자 혼용 8자리 이상)
paasta_api_version: "v3"

# UAAC INFO
uaa_client_admin_id: "admin"			# UAAC Admin Client Admin ID
uaa_client_admin_secret: "admin-secret"		# UAAC Admin Client에 접근하기 위한 Secret 변수
uaa_client_portal_secret: "clientsecret"	# UAAC Portal Client에 접근하기 위한 Secret 변수

# Monitoring INFO
metric_url: "10.0.1.101"			# Monitoring InfluxDB IP
elasticsearch_master_ip: "10.0.1.105"           # Logsearch의 elasticsearch master IP
elasticsearch_master_port: 9200                 # Logsearch의 elasticsearch master Port
index_retention_period: "10"                    # Logsearch의 logstash index 보유 기간(Days)
syslog_address: "10.0.1.100"            	# Logsearch의 ls-router IP
syslog_port: "2514"                          	# Logsearch의 ls-router Port
syslog_transport: "relp"                        # Logsearch Protocol
saas_monitoring_url: "61.252.53.248"	   	# Pinpoint HAProxy WEBUI의 Public IP
monitoring_api_url: "61.252.53.241"        	# Monitoring-WEB의 Public IP

### Portal INFO
portal_web_user_ip: "52.78.88.252"
portal_web_user_url: "http://portal-web-user.52.78.88.252.nip.io" 

### ETC INFO
abacus_url: "http://abacus.61.252.53.248.nip.io"	# abacus url (e.g. "http://abacus.xxx.xxx.xxx.xxx.nip.io")

### Zabbix Agent for monitoring ###
server_ip: "10.37.2.114"               # Zabbix server (or Proxy) IP address
listen_port: "10050"                   # Zabbix server (or Proxy) Port number
server_active: "10.37.2.114:10051"     # Zabbix server (or Proxy) IP:Port string
host_metadata: "paasta"                # Metadata for Zabbix Agent autoregistration
```



### <div id='7'/>● logsearch-vars.yml

```
# SERVICE VARIABLE
inception_os_user_name: "ubuntu"                     # Deployment Name

# STEMCELL
stemcell_os: "ubuntu-bionic"                         # Stemcell OS
stemcell_version: "1.61"                             # Stemcell Version

# ELASTICSEARCH-MASTER
elasticsearch_master_azs: ["z1"]                     # Elasticsearch-Master 가용 존
elasticsearch_master_instances: 1                    # Elasticsearch-Master 인스턴스 수
elasticsearch_master_vm_type: "medium"               # Elasticsearch-Master VM 종류
elasticsearch_master_network: "default"              # Elasticsearch-Master 네트워크
elasticsearch_master_persistent_disk_type: "50GB"    # Elasticsearch-Master 영구 Disk 종류

# ELASTICSEARCH-DATA
elasticsearch_data_azs: ["z1"]                       # Elasticsearch-Data 가용 존
elasticsearch_data_instances: 1                      # Elasticsearch-Data 인스턴스 수
elasticsearch_data_vm_type: "medium"                 # Elasticsearch-Data VM 종류
elasticsearch_data_network: "default"                # Elasticsearch-Data 네트워크
elasticsearch_data_persistent_disk_type: "50GB"      # Elasticsearch-Data 영구 Disk 종류

# CLUSTER-MONITOR
cluster_monitor_azs: ["z1"]                          # Cluster-Monitor 가용 존
cluster_monitor_instances: 1                         # Cluster-Monitor 인스턴스 수
cluster_monitor_vm_type: "medium"                    # Cluster-Monitor VM 종류
cluster_monitor_network: "default"                   # Cluster-Monitor 네트워크
cluster_monitor_persistent_disk_type: "10GB"         # Cluster-Monitor 영구 Disk 종류

# MAINTENANCE
maintenance_azs: ["z1"]                              # Maintenance 가용 존
maintenance_instances: 1                             # Maintenance 인스턴스 수
maintenance_vm_type: "medium"                        # Maintenance VM 종류
maintenance_network: "default"                       # Maintenance 네트워크
maintenance_persistent_disk_type: "5GB"

# KIBANA
kibana_azs: ["z1"]                                   # Kibana 가용 존
kibana_instances: 1                                  # Kibana 인스턴스 수
kibana_vm_type: "medium"                             # Kibana VM 종류
kibana_network: "default"                            # Kibana 네트워크
kibana_persistent_disk_type: "5GB"                   # Kibana 영구 Disk 종류

# INGESTOR
ingestor_azs: ["z1"]                                 # Ingestor 가용 존
ingestor_instances: 1                                # Ingestor 인스턴스 수
ingestor_vm_type: "medium"                           # Ingestor VM 종류
ingestor_network: "default"                          # Ingestor 네트워크
ingestor_persistent_disk_type: "10GB"                # Ingestor 영구 Disk 종류

# LS-ROUTER
ls_router_azs: ["z1"]                                # LS-Router 가용 존
ls_router_instances: 1                               # LS-Router 인스턴스 수
ls_router_vm_type: "small"                           # LS-Router VM 종류
ls_router_network: "default"                         # LS-Router 네트워크
```

### <div id='8'/>● deploy-logsearch.sh
```
bosh -e micro-bosh -n -d logsearch deploy logsearch-deployment.yml \
	-o operations/enable-router.yml \
	-o addons/enable-zabbix-agent.yml \
	-l logsearch-vars.yml \
	-l ../../common/common_vars.yml
```

### <div id='9'/>2.4. Logsearch 설치

- 서버 환경에 맞추어 Deploy 스크립트 파일의 설정을 수정한다.  
  기본적으로 온라인 설치로 되어 있음.

> $ vi ${HOME}/workspace/monitoring-deployment/logsearch/deploy-logsearch.sh

```
# .yml 파일들 설정 변경 이슈 있는지 같이 확인한다.  
bosh -e micro-bosh -n -d logsearch deploy logsearch-deployment.yml \
	-o operations/enable-router.yml \
	-o addons/enable-zabbix-agent.yml \
	-l logsearch-vars.yml \
	-l ../../common/common_vars.yml
```

- Logsearch 설치 Shell Script 파일 실행 (BOSH 로그인 필요)

```
$ cd ~/workspace/monitoring-deployment/logsearch
$ sh deploy-logsearch.sh
```

### <div id='10'/>2.5. 서비스 설치 확인
logsearch가 설치 완료 되었음을 확인한다.
```
$ bosh –e {director_name} vms
```
![PaaSTa_logsearch_vms_5.0]


### [Index](https://github.com/PaaS-TA/Guide) > [Monitoring Install](PAAS-TA_MONITORING_INSTALL_GUIDE.md) > Logsearch


<!-- Images Links -->
[PaaSTa_logsearch_vms_5.0]:./images/logsearch_5.0.png
