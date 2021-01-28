# Git 초기 설정 명령 예시 

> git 수동 사용 시 초기 설정 명령어 셋에 대해서 설명한다. 
>
> 참고자료 : Test-Driven Development with Python, 2nd Edition



- 프로젝트 디렉토리로 이동하여 git 초기화 명령 수행

  ~~~
  $ cd [project dir]
  $ git init .
  
  # git에 현재 폴더 파일들 추가 
  $ git add .
  ~~~

- 불필요한 파일 git 관리 대상에서 제외

  ~~~
  #제외 대상 파일 혹은 디렉토리 이름 추가 
  $ echo "db.sqlite3" >> .gitignore
  $ echo "venv/" >> .gitignore
  ~~~

- 기존에 git 추가된 파일 삭제 

  ~~~
  $ git rm -r --cached [타킷 디렉토리 or 파일]
  #예시
  $ git rm -r --cached superlists/__pycache__
  ~~~

- .gitignore 파일 추가 및 커밋 하기 

  ~~~
  $ git add .gitignore
  $ git commit
  ~~~

  