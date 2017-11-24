# NVIDIA GPU CLOUD on AWS

### 준비 사항

* AWS EC2 GPU Instance를 사용할 수 있는 계정
* NGC 웹 사이트에서 기본 절차 준비 
  * [Getting Started Using NVIDIA GPU Cloud 페이지](http://docs.nvidia.com/ngc/ngc-getting-started-guide/index.html#content)
    * 계정 생성
    * NGC 웹 사이트 로그인 후 NGC API 키 생성(생성 후 보관)


### AWS 예비 설정

* AWS Key Pair 설정 [링크](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

  * key pair pem파일 다운로드 한 후 사용자 권한 변경

    ```shell
    chmod 400 my-key-pair*
    ```

  * EC2 인스턴스 ssh접근할 때, pem파일 사용

    ```shell
    ssh -i [pem파일경로] [ec2-user계정명]@[ec2 instance의 public DNS]
    ```

* Security Group 정의 [링크](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/using-network-security.html#creating-security-group)

  * '인바운드' 탭에서 SSH에 대한 규칙 추가
    * 유형 : SSH
    * 프로토콜 :  TCP
    * 포트 : 22
  * 그 외에 사용하는 딥러닝 프레임워크 등에 따라 추가적인 규칙 정의가 필요할 수 있음





### AWS Console에서 VM 인스턴스 시작

* AWS Console에 로그인 후 EC2 Section 선택
* 상단 메뉴의 오른쪽에서 AWS Zone 선택
  * AWS에서 NIVIDIA Volta GPU를 사용하려면 Amazon EC2 P3 인스턴스가 있는 지역을 선택해야 한다.
* 인스턴스 시작버튼을 누르고 인스턴스를 생성을 시작한다.
* AMI 선택 단계(1단계)에서 AWS Marketplace를 선택하고 "NVIDIA Volta Deep Learning AMI"를 검색하여 선택한다.
  * 인스턴스의 유형마다 시간당 요금이 결정된다.
* 인스턴스 유형 선택 단계(2단계)에서, 선택 가능한 GPU Compute를 선택한다.
* 인스턴스 세부 정보 구성 단계(3단계), 스토리지 추가(4단계), 태그 추가(5단계)는 기본 설정으로 선택한다.
* 보안 그룹 구성 단계(6단계)에서 위의 AWS 예비 설정에서 만들어 놓은 Security Group을 선택한다.
* 마지막으로 키 페어를 선택하는 창이 뜨면, 기존에 위에서 생성했던 키페어를 선택하고 인스턴스를 시작한다.


### VM 인스턴스에 연결

* AWS 예비 설정 탭의 AWS Key Pair 설정을 완료 한 상태에서 pem파일의 경로와 EC2의 public DNS를 통해 VM에 연결한다.

  ```shell
  ssh -i "D:/aws/keypair/test.pem" ubuntu@[ec2 instance의 public DNS]
  ```

* 연결하면 아래와 같이 NVIDIA GPU Regisrty에 접근하기 위한 API Key를 입력하라는 문구가 나온다.

  * 맨 처음 준비 단계에서 진행했던 NGC 웹 사이트에서 발급받은 NGC API 키를 입력하면 성공할 경우 Login Succeded 메세지가 return된다.
  * 접속 완료

  ```shell
  Please enter your NGC APIkey to login to the NGC Registry:
  [APIKey 입력]
  Logging into the NGC Registry at nvcr.io....Login Succeeded
  ubuntu@ip-xxx-xx-xx-xxx:~$
  ```

  ​

### TensorFlow 컨테이너를 사용한 MNIST 교육 예제

* TensorFlow 컨테이너를 NVIDIA GPU Registry에서 pull한다.

  * tensorflow:뒤에 있는 \<xx.xx\>는 버전이다.
  * 이미지가 성공적으로 설치 됐을 경우 아래의 메세지를 확인할 수 있다.

  ```shell
  $ docker pull nvcr.io/nvidia/tensorflow:17.10

  17.10: Pulling from nvidia/tensorflow
  f5c64a3438f6: Pull complete
  51899d335aae: Pull complete
  6ff2b7de3c13: Pull complete
  50366c31f7fd: Pull complete
  f441b9a68d13: Pull complete
  8bc3cd6b8363: Pull complete
  167b211710ea: Pull complete
  010f509c1f2c: Pull complete
  0318015c2c5a: Pull complete
  b3dc559bb348: Pull complete
  eda985ffff46: Pull complete
  da7af78b4b5e: Pull complete
  2b2c2af8656d: Pull complete
  b5cb02eb38c5: Pull complete
  eeb4047aa179: Pull complete
  c54f0ea1a01d: Pull complete
  4b13c0c69041: Pull complete
  d9698d2f3f6d: Pull complete
  0e7cee858e53: Pull complete
  0d41ba9c376a: Pull complete
  35f62680bde8: Pull complete
  b6880c123237: Pull complete
  795927611214: Pull complete
  77581fa421a3: Pull complete
  9dab6a58db19: Pull complete
  e6d05f9b5f4f: Pull complete
  157efad440a5: Pull complete
  f36d1ee944ce: Pull complete
  be460dac2e3d: Pull complete
  28bdde79affa: Pull complete
  Digest: sha256:c64c8cbb4b8420da5e7e3ee345b5a454a865e4fcd55e6ab77a18d21396e60c5c
  Status: Downloaded newer image for nvcr.io/nvidia/tensorflow:17.10
  ```

* 내려받은 컨테이너를 실행한다.

  * 컨테이너의 실행이 완료되면 다음과 같이 root 사용자로 권한이 변경된다.

  ```shell
  $ nvidia-docker run --rm -it nvcr.io/nvidia/tensorflow:17.10

  ================
  == TensorFlow ==
  ================

  NVIDIA Release 17.10 (build 192916)

  Container image Copyright (c) 2017, NVIDIA CORPORATION.  All rights reserved.
  Copyright 2017 The TensorFlow Authors.  All rights reserved.

  Various files include modifications (c) NVIDIA CORPORATION.  All rights reserved.
  NVIDIA modifications are covered by the license terms that apply to the underlying project or file.

  root@fa49e8f3c798:/workspace#
  ```

* MNIST 교육예제 실행

  * tensorflow폴더에 내장된 예제는 MNIST 데이터 세트를 웹에서 가져오는 코드를 포함하고 있다.
  * 예제 실행

  ```shell
   root@fa49e8f3c798:/workspace# python /opt/tensorflow/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py


  Extracting /tmp/tensorflow/mnist/input_data/train-images-idx3-ubyte.gz
  Extracting /tmp/tensorflow/mnist/input_data/train-labels-idx1-ubyte.gz
  Extracting /tmp/tensorflow/mnist/input_data/t10k-images-idx3-ubyte.gz
  Extracting /tmp/tensorflow/mnist/input_data/t10k-labels-idx1-ubyte.gz
  2017-11-23 08:15:24.985941: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:893] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
  2017-11-23 08:15:24.986412: I tensorflow/core/common_runtime/gpu/gpu_device.cc:955] Found device 0 with properties:
  name: Tesla V100-SXM2-16GB
  major: 7 minor: 0 memoryClockRate (GHz) 1.53
  pciBusID 0000:00:1e.0
  Total memory: 15.77GiB
  Free memory: 14.02GiB
  2017-11-23 08:15:24.986440: I tensorflow/core/common_runtime/gpu/gpu_device.cc:976] DMA: 0
  2017-11-23 08:15:24.986481: I tensorflow/core/common_runtime/gpu/gpu_device.cc:986] 0:   Y
  2017-11-23 08:15:24.986508: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1045] Creating TensorFlow device (/gpu:0) -> (device: 0, name: Tesla V100-SXM2-16GB, pci bus id: 0000:00:1e.0)
  Accuracy at step 0: 0.1416
  Accuracy at step 10: 0.7114
  Accuracy at step 20: 0.8148
  Accuracy at step 30: 0.8646
  Accuracy at step 40: 0.8782
  Accuracy at step 50: 0.8882
  Accuracy at step 60: 0.8979
  Accuracy at step 70: 0.9023
  Accuracy at step 80: 0.9081
  Accuracy at step 90: 0.9129
  2017-11-23 08:15:30.465551: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.9.0 locally
  Adding run metadata for 99
  Accuracy at step 100: 0.9126
  Accuracy at step 110: 0.9162
  Accuracy at step 120: 0.9224
  Accuracy at step 130: 0.9206
  Accuracy at step 140: 0.9232
  Accuracy at step 150: 0.9239
  Accuracy at step 160: 0.9268
  Accuracy at step 170: 0.9291
  Accuracy at step 180: 0.9313
  Accuracy at step 190: 0.9331
  ```

