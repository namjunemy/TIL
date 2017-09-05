# gradle 프로젝트 eclipse  import

[gradle 설치와 프로젝트 생성](https://github.com/namjunemy/TIL/blob/master/Tools/gradle_project_create_windows10_eclipse.md)을 끝냈다면, 이클립스와 연동을 한다.



- 이클립스 project explorer에서
- new -> import -> import
- select메뉴 -> General -> Existing Projects into Workspace 선택 후 next
- select root directory -> browse -> 새로만든 gradle project 루트 선택 후 확인
- import project창의 중간 Projects: 메뉴에 추가한 gradle 프로젝트가 생성됨
- Finish


`$ gradle eclipse `명령을 수행하기 전까지 import할 수 없었던 gradle 프로젝트를 드디어 이클립스와 연동할 수 있다.

