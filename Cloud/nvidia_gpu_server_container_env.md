# NVIDIA GPU 서버(Container 환경)

> NVIDIA 파트너사 / 미루웨어 - 이지훈 차장

### NVIDIA GPU 종류

* 새로운 V1000 GPU 아키텍처가 나오면, NVIDIA 는 세 종류를 따로 개발하는 것이 아니라, 각 라인업에 맞는 최신 아키텍처를 가지는 GPU를 만든다.
* 딥러닝, 머신러닝이 급성장하면서 Geforce의 최신 라인업인 TITAN라인을 NVIDIA 가 자체 관리한다. 직접 생산, 판매에 관여한다. Geforce -> NVIDIA TITAN
  * 딥러닝 분야는 반복연산을 통한 학습에 초점이 맞춰져 있기 때문에, 부동 소수점 연산이 뛰어난 Tesla나 Quadro보다는 Geforce의 TITAN 라인을 사용한다.
  * 테스트 및 개발단계에 Geforce GPU를 사용하고, 이 단계를 거쳐서 실제 사용 단계로 넘어갈 경우 Tesla와 Quadro로 넘어간다.
* Cad는 OpenGL, Direct X에서 같은 퍼포먼스를 보이지만, Cad종류에 따라서 어떤 GPU 종류를 사용하느냐에 따라 퍼포먼스의 차이가 있다.
* Tesla 라인이 국내에서 Quadro 보다 가격이 높은 이유는 기술지원이 포함되어 있기 때문이다.
* 기능적으로 봤을때, Quadro는 Tesla의 모든 기능 + OpenGL 을 사용할 수 있다.

| Geforce              | Tesla             | Quadro             |
| -------------------- | ----------------- | ------------------ |
| 게임용                  | GPGPU(연산)         | CAD                |
| direct X             | 화면 출력이랑 관련 X, 연산용 | OpenGL             |
| 최신 - TITAN V(160만원선) | 최신 - V100(880만원선) | 최신 - V6000(980만원선) |
| 벤더사 생산               | NVIDIA 생산         | NVIDIA 생산          |

## GPU 서버

* 구축 환경 - IBM power PPC Minsky + GPU 4장 + Linux(Ubuntu 16.04)

* Infra Stack

  * 물리서버
    * APP + BINS/LABS - 도커 컨테이터(CUDA Toolkit + Applications)
    * Docker Engine - Docker 엔진 + NVIDIA Docker(Docker 엔진의 Plugin 스크립트)
    * Host OS - 운영체제와 CUDA Driver
    * Infrastructure - 서버와 NVIDIA GPUs

* 접속

  * ssh [user ID]@[server IP]

* pci 장치 조회(NVIDIA GPU)

  * $ lspci

  * $ lspci | grep -i nvidia

    ```shell
    0002:01:00.0 3D controller: NVIDIA Corporation Device 15f9 (rev a1)
    0003:01:00.0 3D controller: NVIDIA Corporation Device 15f9 (rev a1)
    000a:01:00.0 3D controller: NVIDIA Corporation Device 15f9 (rev a1)
    000b:01:00.0 3D controller: NVIDIA Corporation Device 15f9 (rev a1)
    ```

  * 장치 상세조회

    * $ lspci -vvs 0002:01:00.0

    ```shell
    0002:01:00.0 3D controller: NVIDIA Corporation Device 15f9 (rev a1)
            Subsystem: NVIDIA Corporation Device 116b
            Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr+ Stepping- SERR- FastB2B- DisINTx-
            Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
            Latency: 0
            Interrupt: pin A routed to IRQ 325
            Region 0: Memory at 3fe100000000 (32-bit, non-prefetchable) [size=16M]
            Region 1: Memory at 220000000000 (64-bit, prefetchable) [size=16G]
            Region 3: Memory at 220400000000 (64-bit, prefetchable) [size=32M]
            Capabilities: <access denied>
            Kernel driver in use: nvidia
            Kernel modules: nvidiafb, nouveau, nvidia_375_drm, nvidia_375

    ```

  * NVIDIA Driver를 설치하려면, 먼저 NVIDIA Driver에 영향을 주는 nouveau 모듈을 커널 부팅때 제외 해야 한다.

  * 그 다음 NVIDIA Driver를 설치하고, nvidia-smi를 통해서 GPU 정보를 조회한다.

* GPU 정보 조회

  * NVIDIA smi를 통해서 GPU정보를 조회하고, 컨트롤한다.
  * document 별도로 제공
  * $ nvidia-smi

  ```shell
  Fri Feb  9 04:30:31 2018
  +-----------------------------------------------------------------------------+
  | NVIDIA-SMI 375.88                 Driver Version: 375.88                    |
  |-------------------------------+----------------------+----------------------+
  | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
  | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
  |===============================+======================+======================|
  |   0  Tesla P100-SXM2...  Off  | 0002:01:00.0     Off |                    0 |
  | N/A   33C    P0    30W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+
  |   1  Tesla P100-SXM2...  Off  | 0003:01:00.0     Off |                    0 |
  | N/A   32C    P0    29W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+
  |   2  Tesla P100-SXM2...  Off  | 000A:01:00.0     Off |                    0 |
  | N/A   34C    P0    31W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+
  |   3  Tesla P100-SXM2...  Off  | 000B:01:00.0     Off |                    0 |
  | N/A   32C    P0    30W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+

  +-----------------------------------------------------------------------------+
  | Processes:                                                       GPU Memory |
  |  GPU       PID  Type  Process name                               Usage      |
  |=============================================================================|
  |  No running processes found                                                 |
  +-----------------------------------------------------------------------------+
  ```

  * $ nvidia-smi -a -q | less
    * GPU의 개별 Clocks, Memory, Temperature, ECC 등 상세정보를 조회한다.

### Docker - NVIDIA Docker

* 컨테이너 위에서 Tensorflow를 구동하려면, CUDA(Computed Unified Device Architecture, NVIDIA GPU 개발 툴)가 필요하고, NVIDIA Driver가 필요하다. VM환경에서 Ubuntu OS를 사용한다면, apt-get으로 CUDA를 설치하면 자동으로 CUDA와 NVIDIA Driver를 설치한다. 그러나 컨테이너환경에서는 Host OS의 커널에 NVIDIA Driver가 설치되어 있고, 그것을 컨테이너가 공유하므로 apt-get으로 한번에 설치하면 안되고, NVIDIA Driver를 제외한 CUDA만 설치하는 프로세스를 거쳐야 한다.


* Docker 데몬은 root에서 동작한다.
  * 따라서 일반 사용자를 추가 했다면,
  * $ sudo usermod -aG docker [사용자이름]으로 도커 그룹에 추가한다.
  * 도커 그룹은 도커 데몬에 접속가능한 그룹이다.


* NVIDIA Docker는 Docker의 플러그인이다.
* 기본적으로 docker 명령어의 앞에 nvidia- 가 붙는다.
* 도커의 동작원리

  * Docker CLI -> REST API -> Docker Daemon -> Docker Image -> Container 생성
* 도커 버전 조회

  * $ docker --version
  * $ docker info
* 도커 이미지 조회

  * $ docker images
  * $ nividia-docker images
* 도커 프로세스 확인

  * $ docker ps
  * $ docker ps -a
* 도커 컨테이너 생성

  * $ docker run [Image Name]:[TAG]

    * docker run의 옵션

      * -ti

        * 터미널 입출력

      * --rm

        * --rm을 붙이지 않고 컨테이너에서 exit할 경우 컨테이너만 종료되고, 삭제되지 않는다.
        * --rm 옵션은 컨테이너에서 exit로 빠져나오면 컨테이너가 삭제되도록 한다.
      * -v A:B
        * 디스크 맵핑 A - 로컬머신 path, B - 컨테이너 path
        * 이렇게 디스크를 맵핑하면  컨테이너가 종료 되더라도,
        * Tensorflow를 통해서 나온 결과 데이터가 로컬머신의 A path에 저장된다.
      * -p A:B
        * 포트 포워딩
        * A - 로컬머신 port, B - 컨테이너 port
      * --name [이름]

        * 컨테이너 이름 설정


* exit한 컨테이너 다시 start

  * docker start [컨테이너이름]

* exit한 컨테이너에 접속하는 방법

  * docker attach [컨테이너이름]
    * 컨테이너에 접근한다.
  * docker exec -ti [컨테이너이름] bash
    * tty로 접근한다. 각자의 터미널이 생긴다.

* 실행중인 컨테이너를 커스텀 이미지로 생성

  * $ docker ps 명령어로 해당 컨테이너 정보 확인
  * $ nvidia-docker commit [컨테이너 name] \[이미지 이름]:[이미지 태그]

* 커스텀 이미지 저장

  * $ nvidia-docker save \[이미지 이름]:[이미지 태그] > [파일이름].tar
  * 저장한 tar 파일만 가지고 있다면, docker 환경에서 그대로 사용할 수 있다.


  * 커스텀 이미지 load
    * $ nvidia-docker load -i [파일경로]/[파일이름].tar

### GPU 할당

* 컨테이너 환경에서 GPU의 할당은 NV_GPU 옵션에 의해 이루어진다. 컨테이너를 생성할 때, NV_GPU=[할당할 GPU] 명령을 통해 GPU를 할당하면 해당 컨테이너에 접근해서, GPU 정보를 조회 했을때, 할당 된 GPU만 보이게 된다.

* $ NV_GPU=0,3 nvidia-docker run -ti --rm -v /src:/src mw/cuda8.0_cudnn7_edu_build:2018-02-25 bash

* $ nvidia-smi

  ```shell
  Fri Feb  9 06:05:23 2018
  +-----------------------------------------------------------------------------+
  | NVIDIA-SMI 375.88                 Driver Version: 375.88                    |
  |-------------------------------+----------------------+----------------------+
  | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
  | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
  |===============================+======================+======================|
  |   0  Tesla P100-SXM2...  Off  | 0002:01:00.0     Off |                    0 |
  | N/A   33C    P0    30W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+
  |   1  Tesla P100-SXM2...  Off  | 000B:01:00.0     Off |                    0 |
  | N/A   32C    P0    30W / 300W |      0MiB / 16276MiB |      0%      Default |
  +-------------------------------+----------------------+----------------------+

  +-----------------------------------------------------------------------------+
  | Processes:                                                       GPU Memory |
  |  GPU       PID  Type  Process name                               Usage      |
  |=============================================================================|
  |  No running processes found                                                 |
  +-----------------------------------------------------------------------------+

  ```

  ​