# 🤦🏻‍♂️ gitignore에 파일 추가해서 추적 안하려면
.gitignore에 파일을 추가하더라도
```shell
git rm -r --cached .
```
이 명령어를 통해서 캐시에서 삭제를 해주어야 한다.