# AWS EC2 ssh 원격 접속과 scp를 통한 파일 업로드

[EC2 인스턴스 생성](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_create_instance.md)이 성공적으로 끝났다면, 다음과 같이 CLI환경에서 ssh를 통해서 EC2에 원격 접속을 할 수 있고, scp를 통해 간단한 파일 업/다운로드를 할 수 있다.



## 접속

git bash를 실행시켜서 Teminal에서 다음 명령으로 접속한다. ec2-user 계정명은 ubuntu의 경우 ubuntu이다. 각자 선택한 인스턴스 OS 이미지가 다를 수 있으므로 EC2 콘솔 왼쪽 메뉴중 INSTANCES -> Instances 로 이동하여 상단에 있는 Connect 버튼을 누르면 외부에서 인스턴스에 접속하기 위한 가이드가 있으니 참고하면 된다.

``` ssh -i [pem파일경로] []```