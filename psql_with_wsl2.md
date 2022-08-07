# WSL2를 이용하여 psql 사용하기

## wsl2 를 사용하시며 윈도우에서 postgreSQL을 이용하기

1. 윈도우에서 cmd -> ipconfig -> Ethernet adapter vEthernet (WSL): 의 IPv4 주소를 복사하신 후, 프로젝트의 'app.module.ts' 파일의 TypeOrmModule.forRoot 의 attribute 중 localhost 대신 써주시면 됩니다.

2. 윈도우에서 postgreSQL 이 설치된 폴더 로 가신 후 ~/data/ 폴더에 가시면 pg_hba.conf 파일이 있는데 메모장으로 열어주신 후 하단의 # TYPE DATABASE USER ADDRESS METHOD 밑에 host all all 0.0.0.0/0 md5 를 적고 저장하시면 됩니다.

3. 설정 -> firewall -> advanced setting -> 좌측 패널 inbound rules 클릭 -> 맨 우측 패널 New Rule 로 추가 -> Port -> 포트번호 입력 -> rule 이름 넣고 저장하시면 됩니다.

이거 그대로 따라하면 wsl nest + window sql 될거에요

## Linux용 Windows 하위 시스템 데이터베이스 시작

https://docs.microsoft.com/ko-kr/windows/wsl/tutorials/wsl-database