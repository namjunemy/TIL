# AWS EC2 인스턴스 생성하기

AWS 프리티어 계정을 생성 한 후, EC2(Elastic Compute Cloud) 인스턴스를 어렵지 않게 만들 수 있었다. 워낙 공식 메뉴얼이 가이드는 물론 한글화도 잘 되어 있었기 때문에 따로 정리하지 않고, 링크를 남겨둔다.

>  시간과 여유를 가지고 빠짐없이 정말 그대로 따라하기만 하면 EC2인스턴스를 생성 할 수 있다!



* [Amazon Ec2란 무엇인가?](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/concepts.html)

* [EC2 인스턴스 생성 전에 해야되는 설정](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)

  * [AWS가입](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#sign-up-for-aws)

  * [IAM 사용자 생성](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-an-iam-user)

  * [키 페어 생성](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-a-key-pair)

    * **중요** -> 이 단계에서 저장되는 private key 파일을 꼭 별도의 위치에 저장해 두고, SSH 클라이언트를 사용하여 Linux 인스턴스에 연결할 때 사용자만 키 파일을 읽을 수 있도록 아래의 명령어로 권한 설정을 꼭 해준다. 해당 파일 위치로 이동하여 Git bash를 통해서, 또는 유사한 방법으로.

      ```
      chmod 400 your_user_name-key-pair-region_name.pem
      ```

  * [Virtual Private Cloud(VPC) 생성](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-a-vpc)

  * [보안 그룹 생성](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html#create-a-base-security-group)

* [EC2 Linux 인스턴스 시작하기](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

  * [개요](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-get-started-overview)
  * [사전 조건](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-getstarted-prereqs)
  * [1단계: 인스턴스 시작](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance)
  * [2단계: 인스턴스에 연결](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-connect-to-instance-linux)
  * [3단계: 인스턴스 정리](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-clean-up-your-instance)
  * [다음단계](http://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-next-steps)



위의 설정이 끝난 후, 인스턴스가 Launch될 때 시간이 조금 걸린다. 성공적이라면 Instance State 표시 란에 초록색의 running State를 볼수 있다. 위 단계를 거치고 error가 발생한다면 분명히 생략하거나 지나친 단계가 있을 것이다. 처음 한다면, 차근차근 여유를 가지고 해야한다. 

다음으로 ssh client를 통해서 ec2 인스턴스에 접근 하는 법에 대해 알아보려고 한다.