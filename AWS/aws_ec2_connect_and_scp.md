# AWS EC2 ssh 원격 접속과 scp를 통한 파일 전송

[EC2 인스턴스 생성](https://github.com/namjunemy/TIL/blob/master/AWS/aws_ec2_create_instance.md)이 성공적으로 끝났다면, 다음과 같이 CLI환경에서 ssh를 통해서 EC2에 원격 접속을 할 수 있고, scp를 통해 간단한 파일 업/다운로드를 할 수 있다.



## 접속

git bash를 실행시켜서 Teminal에서 다음 명령으로 접속한다. ec2-user 계정명은 ubuntu의 경우 ubuntu이다. 각자 선택한 인스턴스 OS 이미지가 다를 수 있으므로 EC2 콘솔 왼쪽 메뉴중 INSTANCES -> Instances 로 이동하여 상단에 있는 Connect 버튼을 누르면 외부에서 인스턴스에 접속하기 위한 가이드가 있으니 참고하면 된다.

```
ssh -i [pem파일경로] [ec2-user계정명]@[ec2 instance의 public IP 또는 public DNS]
```

```
ex)

ssh -i "D:/document/aws/keypair/namjunemy_key_pair_ldcc_notebook.pem" ubuntu@ec2~~~~~~.ap-northeast-2.compute.amazonaws.com
```



## 파일 업로드

업로드 하고 싶은 파일이 있는 경로에서 아래의 명령을 수행한다. 경로 앞의 :~ 는 계정의 root path를 의미한다.

```
scp -i [pem파일경로] [업로드할 파일 이름] [ec2-user계정명]@[ec2 instance의 public DNS]:~/[경로]
```

```
ex)

scp -i "D:/document/aws/keypair/namjunemy_key_pair_ldcc_notebook.pem" test.txt ubuntu@ec2~~~~~~.ap-northeast-2.compute.amazonaws.com:~/test/

```



## 파일 다운로드

반대로 EC2인스턴스에서 다운로드 해야 할 파일이 있을 경우 반대로 수행하면 된다.

```
scp -i [pem파일경로] [ec2-user계정명]@[ec2 instance의 public DNS]:~/[경로] [다운로드 파일의 로컬 경로] 
```

```
ex)

scp -i "D:/document/aws/keypair/namjunemy_key_pair_ldcc_notebook.pem" ubuntu@ec2~~~~~~.ap-northeast-2.compute.amazonaws.com:~/test/test.txt D:/document/
```

