# SR-Elastic-Cluster-Framework

Elastic Horovod를 이용하여 분산 학습을 진행할 때 GPU를 dynamic하게 scaling in/out을 하기 위해 현재 동작 중인 Elastic Horovod의 상태를 파악해야 됩니다.

Elastic-Cluster-Framework는 이를 동적으로 파악해서 보여주는 interface와 그 정보를 가지고 Elastic Horovod에서 여러 개의 node에서 학습하고 있는 worker의 수를 버튼 하나로 scaling in/out 할 수 있는 API를 제공합니다.

## Setting

해당 framework를 이용하기 위해 Elastic Horovod를 동작할 수 있는 환경 세팅과 여러 Python 라이브러리의 설치가 필요합니다.

1. [Elastic Horovod](https://horovod.readthedocs.io/en/stable/elastic_include.html)  
Framework를 개발하며 Elastic Horovod가 설치된 docker image를 이용하여 docker container 환경에서 작업하였습니다. 기존에 Elastic Horovod를 이용하여 연구를 진행하셨던 CSL 김경록님께서 정리해놓으신 notion 링크를 첨부하겠습니다. 해당 링크의 내용에는 Elastic Horovod를 설치 및 세팅하는 방법에 대해 자세히 설명되어 있습니다.
>+ [notion link](https://discreet-file-a73.notion.site/Elastic-Horovod-6ae5f2c3dac04b62b0f4605cf65b0d36)

2. Python 라이브러리 설치  
REST-API를 이용하기 위해 몇 가지 파이썬 라이브러리가 필요합니다. 필요한 라이브러리의 목록은 다음과 같습니다.  
+ Flask  
+ uwsgi  
+ nginx  

## How to Run

Elastic Horovod가 학습하는 동안 REST-API Server와 지속적으로 상호작용을 해야 합니다.
따라서, 해당 framework를 사용하기 위해서는 우선 REST-API Server가 실행되어 web page를 띄워준 뒤, horovodrun 명령어를 통해 Elastic Horovod가 학습을 시작한 뒤 REST-API를 통해 서로 상호작용 합니다.
이와 같은 실행 과정을 순서대로 설명하면 다음과 같습니다.

우선, REST-API Server를 실행하는 방법에 대해 설명하겠습니다. uwsgi.ini 파일은 /EC-MaS/REST-API-Server-pkg에 위치하고, nginx의 환경 설정을 위한 nginx.conf 파일은 해당 레포지토리 내부에 있습니다.

1. uwsgi

```sh
uwsgi --ini <path-to-uwsgi.ini>
```

2. nginx

```sh
nginx
```

3. horovodrun

```sh
horovodrun -p <SSH port number> --network-interface <nic> -np <num_proc> --min-np <min_num> --max-np <max_num> --host-discovery-script <path-to-job-script> sh <path-to-run-script>
```

## About EC-MaS(Elastic Cluster - Monitoring and Scaling)

앞서 framework의 동작 방법에 대해 설명했는데, 해당 파트에서는 소스 코드에 대해 자세히 설명하겠습니다.

1. GPU-monitoring-pkg
해당 패키지는 GPU status를 확인하기 위해 생성되는 데몬을 생성하고 관리하는 역할을 합니다.
패키지 내부의 gpustat_daemon.py를 통해 데몬을 생성합니다.
해당 코드에서 gpustat 명령어를 통해 GPU status에 대한 정보를 얻는데. 해당 부분을 수정함으로써 원하는 경로로 설정할 수 있습니다.
```sh
...
gpustat_open_string = "SR-Elastic-Cluster-Framework/EC-MaS/Job-control-pkg/" + local_ip + "_gpustat.json"
gpustat_file_string = "gpustat --json > " + gpustat_open_string
...
```






