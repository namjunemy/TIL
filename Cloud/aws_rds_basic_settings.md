# RDS 기본 설정

## 운영환경에 맞는 파라미터 설정하기

* 파라미터 그룹 탭에서 운영환경 파라미터 그룹 새로 생성

* 타임존 설정
    * 파라미터 필터링을 통해서 time_zone을 Asia/Seoul로 변경
* Character Set 변경
    * 이모지 저장을 지원하는 utf8mb4 형식으로 아래의 8개 항목들 변경
    * character_set_client : utf8mb4
    * character_set_connection : utf8mb4
    * character_set_database : utf8mb4
    * character_set_results : utf8mb4
    * character_set_server : utf8mb4
    * collation_connection : utf8mb4_unicode_ci
    * collation_server : utf8mb4_unicode_ci 
* Max Connetion 변경
    * RDS의 Max Connection은 인스턴스 사양에 따라 자동으로 정해진다.
    * 프리티어 사양에서는 60개의 커넥션이 기본값으로 설정됨
    * 테스트용으로 150개로 임의로 설정

* 위의 변경사항 저장 후, RDS의 파라미터 그룹을 기본 그룹에서 새로 생성한 그룹으로 변경

* 적용 후 재부팅을 통해서 완전히 적용

* 추가 확인 사항

    * 위에서 설정한 character-set, collation 설정 적용 확인

        ```sql
        show variables like 'c%';
        ```

    * rds 파라미터 설정에서 적용했지만, 일부 기본값인 latin1로 설정 되어있는 항목 존재

    * character_set_database, collation_connection 두가지

    * 직접 쿼리 실행

        ```sql
        alter database db명
        character set = 'utf8mb4'
        collate = 'utf8mb4_general_ci';
        ```

    * 타임존 확인

        ```sql
        select @@time_zone, now();
        ```

        