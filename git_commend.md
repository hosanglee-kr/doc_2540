

### 1. 원격 브랜치 목록 확인하기
먼저, 원격 저장소에 어떤 브랜치들이 있는지 확인해야 합니다. 이를 위해 다음 명령어를 사용합니다:
```bash
git fetch
git branch -r
```
 
이 명령어들은 원격 저장소의 최신 정보를 가져오고, 
원격 브랜치 목록을 출력합니다.

### 2. 원격 브랜치 체크아웃하기
원하는 브랜치를 확인했다면, 해당 브랜치를 체크아웃하여 로컬에서 작업할 수 있습니다.
체크아웃은 다음과 같이 진행합니다:

```bash
git checkout -b [브랜치명] origin/[브랜치명]
```

여기서 [브랜치명]은 체크아웃하고자 하는 원격 브랜치의 이름입니다. 이 명령어는 원격 브랜치를 기반으로 새로운 로컬 브랜치를 생성하고, 해당 브랜치로 전환합니다

### 3. 작업 후 변경 사항 반영하기
브랜치에서 작업을 마치고 변경 사항을 원격 저장소에 반영하려면, commit과 push를 사용합니다:
```bash
git add .
git commit -m "변경 내용에 대한 설명"
git push origin [브랜치명]
```
이렇게 하면 로컬에서 작업한 내용이 원격 저장소의 해당 브랜치에 반영됩니다


## 이미 커밋된 파일 gitignore 추적 취소 방법
### 단일 파일 추적 취소
특정 파일이 이미 커밋된 상태에서 이후로는 무시하고자 할 때, 
```bash
# 단일 파일 추적 취소
git rm --cached [파일명]
```
명령어를 사용해 파일 추적을 취소할 수 있습니다. 이 명령은 파일을 시스템에서 삭제하지 않고 Git 추적에서만 제외합니다.

###.gitignore에 포함된 모든 파일 추적 취소
.gitignore에 새롭게 추가된 파일들을 일괄적으로 추적에서 제외하려면, 먼저 모든 변경 사항을 커밋한 후 git rm --cached 명령어를 사용합니다.
예시 코드
```bash
# .gitignore에 있는 모든 파일 추적 취소
git rm --cached -r .
git add .
git commit -m "Untrack files in .gitignore"
```

### git checkout 사용 예시:
```bash
# develop 브랜치로 전환
git checkout develop

# 새 브랜치를 생성하고 그곳으로 전환
git checkout -b new-feature

# 특정 커밋으로 HEAD를 이동
git checkout 5d3a123

# 특정 파일을 마지막 커밋 상태로 복원
git checkout -- file.txt
```
### git switch 사용 예시:
```bash
# develop 브랜치로 전환
git switch develop

# 새 브랜치를 생성하고 그곳으로 전환
git switch -c new-feature
```


## Branch 관리를 위한 명령
cf) HEAD : 여러 브랜치 중에서 현재 작업중인 브랜치가 무엇인지 알려주는 포인터

### 1 브렌치 생성 및 조회
```bash
# 모든 브랜치 조회
git branch
#새로운 브랜치 생성
git branch 브랜치명
```
### 2. 브렌치 전환하기
```bash
git checkout 브렌치명
git switch 브렌치명
```
checkout 키워드가 활용되는 명령어들이 너무 많아져서, switch 라는 키워드의 명령어를 새롭게 만들어졌다고 합니다.

### 3. 새로운 브랜치 생성후, 해당 브랜치로 바로 이동하기
```bash
git checkout -b 새로운 브랜치명
git switch -c 새로운 브랜치명
```

새로운 브렌치를 생성하면서 이동하는 명령입니다. 즉, 위 명령어들은 아래 명령어를 한 줄로 줄인것입니다.
```bash
git checkout my_branch
git checkout my_branch
```
### 4. 브렌치 히스토리 조회 (git log 관련)
```bash
# git log 를 한줄로 간략히 조회
git log --oneline

# 모든 브랜치의 커밋 상황을 확인
git log --oneline --branches

# 모든 브랜치의 커밋 상황을 시각적으로 표현
# 각 브랜치들의 커밋 생성순서 및 관계를
# 그래프 형태로 확인 가능
git log --oneline --branches --graph
```

###4. 두 브렌치간의 Commit 차이 조회
```bash
git log 브렌치1 ..브렌치2

# 브렌치 사이의 차이점을 확인합니다.
# master와 safecal 브랜치의 차이 확인
git log master ..safecal

# 브렌치1에는 없고
# 브렌치2에만 있는 커밋의 log 확인
git log 브렌치1..브렌치2 

# 브렌치2에는 없고 
# 브렌치1에만 있는 커밋의  log 확인
git log 브렌치2..브렌치1
```
### 5. 원격 레포지토리에서 브랜치 삭제하기
```bash
git push < remote >--delete < branch >

# 예를 들어서 삭제하고 싶은 
# 원격 브랜치 이름이
# fix/aut hentication 이라면

git push origin --delete fix/authentication
```
와 같이 명령을 입력하시면 됩니다. 그러면 이제 이 브랜치는 원격에서 삭제됩니다.
참고로 더 짧은 버전의 명령어도 있습니다.
```bash
git push < remote > :< branch >
```

### Pull vs Fetch
pull 과 fetch 의 차이점은 병합(Merge) 처리 여부입니다.

PULL : pull 은 원격 레포지토리로부터 최신 커밋들을 내려받아서, 현재 로컬 브랜치와 자동으로 병합을 진행합니다.

Fetch : 반면 fetch 는 원격 레포지토리에서 최신 commit 코드를 이름없는 임시 브랜치로 내려받고, 병합(merge)을 진행하지 않습니다.

즉, 개발자가 수동으로 직접 merge 를 진행해야 합니다.
이떄 브렌치는 FETCH_HEAD 의 이름으로 체크아웃이 가능합니다.
=> 정리해보면, 사실상 pull 명령은 내부적으로 봤을때 fetch 와 merge 의 과정을 포함하고 있는 것입니다. fetch 이후 merge 를 수행하면 pull 명령과 동일한 수행 내역이 되는 것입니다.

