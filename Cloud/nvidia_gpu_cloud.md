# NVIDIA GPU 클라우드 소개

NIVIDIA GPU Cloud (NGC)는 깊은 학습과 과학적 컴퓨팅을 위해 최적화 된 GPU 가속 클라우드 플랫폼이다. NGC에는 NGC 컨테이너, NGC 컨테이너 레지스트리, NGC 웹 사이트 및 딥러닝 컨테이너를 실행하기 위한 플랫폼 소프트웨어가 포함되어 있다. 이 문서는 NVIDIA GPU 클라우드의 개요와 사용법을 제공한다.

  

### NGC 컨테이너

NGC 컨테이너는 최소한의 OS 요구 사항을 중심으로 하는 소프트웨어 플랫폼, 서버 또는 워크 스테이션의 Docker 및 드라이버 설치를 지원하고 NGC 컨테이너 레지스트리를 통해 NGC 컨테이너의 모든 응용 프로그램 및 SDK를 프로비저닝 할 수 있도록 설계되어있다.

NGC는 단일 GPU 및 다중 GPU 구성에서 NVIDIA GPU를 최대한 활용하는 통합되고 최적화 된 딥러닝프레임 워크 카달로그를 관리한다. NVCaffe, Caffe2, Microsoft Congitive Toolkit(CNTK), MXNet, PyTorch, TensorFlow, Theano 및 Torch 등 CUDA 툴킷, DIGITS 워크 플로우 및 딥러닝 프레임워크가 포함되어 있다. 이러한 프레임 워크 컨테이너는 CUDA 런타임, NIVIDIA 라이브러리 및 운영체제와 같은 모든 필요한 종속성을 포함하여 즉시 실행할 수 있도록 제공된다.

각 프레임워크 컨테이너 이미지에는 전체 소프트웨어 개발 스택과 함께 사용자 정의 수정 및 향상 기능을 사용하기 위한 프레임워크 소스 코드도 포함되어 있다.

NIVIDIA는 이러한 딥러닝 컨테이너를 매월 업데이트하여 지속적으로 성능을 극대화 한다. 또한, NGC는 현재 베타 버전으로 제공되는 HPC 시각화 컨테이너 카탈로그를 제공하며, 대화 형 실시간 및 고품질을 위해 NVIDIA Index 볼륨 랜더러가있는 ParaView, NVIDIA OptiX 레이 트레이싱 라이브러리 및 NVIDIA Holodeck을 포함한 시각화 도구를 제공한다.

  

### NGC 컨테이너 등록기

NGC 컨테이너 레지스트리는 배포 할 컨테이너 이미지를 nvcr.io에 저장한다. NGC API 키를 사용하면 레지스트리에서 NGC 컨테이너를 가져와 실행할 수 있다.

  

### NGC 웹 사이트

NGC 웹 사이트([https://ngc.nvidia.com](https://ngc.nvidia.com))는 NGC를 관리하기 위한 포털이다. NGC 컨테이너 레지스트리의 내용을 쉽게보고 컨테이너를 사용할 수 있는 권한을 부여하는 API 키를 만들고 NGC 컨테이너에 최적화 된 가상 시스템 인스턴스를 제공한다.

  

### 최적화 된 가속 컴퓨팅 환경

모든 NGC 컨테이너는 NVIDIA GPU를 최대한 활용할 수 있는 자격을 갖추고 있으며 NVIDIA DGX 시스템 및 지원되는 클라우드 서비스 공급자와 같은 지원 되는 ACE(Accelerated Conputing Evironment)에서 즉시 실행할 수 있다.

현재 지원되는 클라우드 서비스 공급자에는 Amazon EC2 P3 인스턴스를 서비스하는 Amazon Web Service가 있다.

  

### NGC를 이용한 딥러닝 프레임워크 실행

딥러닝 프레임워크 컨테이너를 실행하는 프로세스는 다음과 같이 요약 할 수 있다.

  

### ACE (Accelerated Conputing Environment) 준비

NGC 컨테이너를 실행하기 위한 가속화 된 컴퓨팅 환경(ACE)을 준비한다.  ACE 설정에 대한 지침은 아래와 같다.
