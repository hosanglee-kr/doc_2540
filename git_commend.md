

### 1. 원격 브랜치 목록 확인하기
먼저, 원격 저장소에 어떤 브랜치들이 있는지 확인해야 합니다.<br> 
이를 위해 다음 명령어를 사용합니다:
```bash
git fetch
git branch -r
```
 
이 명령어들은 원격 저장소의 최신 정보를 가져오고,<br> 
원격 브랜치 목록을 출력합니다.

### 2. 원격 브랜치 체크아웃하기
원하는 브랜치를 확인했다면, <br>
해당 브랜치를 체크아웃하여 로컬에서 작업할 수 있습니다.<br>
체크아웃은 다음과 같이 진행합니다:

```bash
git checkout -b [브랜치명] origin/[브랜치명]
```

여기서 [브랜치명]은 체크아웃하고자 하는 원격 브랜치의 이름입니다. <br>
이 명령어는 원격 브랜치를 기반으로 새로운 로컬 브랜치를 생성하고, <br>
해당 브랜치로 전환합니다

### 3. 작업 후 변경 사항 반영하기
브랜치에서 작업을 마치고 변경 사항을 원격 저장소에 반영하려면, <br>
commit과 push를 사용합니다:
```bash
git add .
git commit -m "변경 내용에 대한 설명"
git push origin [브랜치명]
```
이렇게 하면 로컬에서 작업한 내용이 <br>
원격 저장소의 해당 브랜치에 반영됩니다


## 이미 커밋된 파일 gitignore 추적 취소 방법
### 단일 파일 추적 취소
특정 파일이 이미 커밋된 상태에서 이후로는 무시하고자 할 때, 
```bash
# 단일 파일 추적 취소
git rm --cached [파일명]
```
명령어를 사용해 파일 추적을 취소할 수 있습니다. <br>
이 명령은 파일을 시스템에서 삭제하지 않고 Git 추적에서만 제외합니다.

### .gitignore에 포함된 모든 파일 추적 취소
.gitignore에 새롭게 추가된 파일들을 일괄적으로 추적에서 제외하려면, <br>
먼저 모든 변경 사항을 커밋한 후 `git rm --cached` 명령어를 사용합니다.<br>
예시 코드
```bash
# .gitignore에 있는 모든 파일 추적 취소
git rm --cached -r .
git add .
git commit -m "Untrack files in .gitignore"
```

---
## GIT Clone으로 신규 생성 프로젝트 초기화

```
# 1. Clone 뜬 플젝 원격 끊기
git remote -v
git remote remove origin
 
# 2. 원격 끊어주고 난 뒤
rm -rf .git
git init
git add .
git commit -m "initial commit"
 

# 3. BitBucket가서 새로운 Repository 생성

# 4. Git 저장소 연결 후 강제 push
git remote add origin {git remote url}
git push -u --force origin master
```

---

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
checkout 키워드가 활용되는 명령어들이 너무 많아져서, <br>
switch 라는 키워드의 명령어를 새롭게 만들어졌다고 합니다.

### 3. 새로운 브랜치 생성후, 해당 브랜치로 바로 이동하기
```bash
git checkout -b 새로운 브랜치명
git switch -c 새로운 브랜치명
```

새로운 브렌치를 생성하면서 이동하는 명령입니다. <br>
즉, 위 명령어들은 아래 명령어를 한 줄로 줄인것입니다.
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
와 같이 명령을 입력하시면 됩니다. <br>
그러면 이제 이 브랜치는 원격에서 삭제됩니다.<br>
참고로 더 짧은 버전의 명령어도 있습니다.
```bash
git push < remote > :< branch >
```

### **Pull vs Fet**
pull 과 fetch 의 차이점은 병합(Merge) 처리 여부임<br>

- PULL : pull 은 원격 레포지토리로부터 최신 커밋들을 내려받아서, <br>
      현재 로컬 브랜치와 자동으로 병합을 진행합니다.<br>

- Fetch : 반면 fetch 는 원격 레포지토리에서 최신 commit 코드를 <br>
이름없는 임시 브랜치로 내려받고, 병합(merge)을 진행하지 않습니다.<br>

즉, 개발자가 수동으로 직접 merge 를 진행해야 합니다.<br>
이떄 브렌치는 FETCH_HEAD 의 이름으로 체크아웃이 가능합니다.<br>
> 정리해보면, 사실상 pull 명령은 내부적으로 봤을때 <br>
fetch 와 merge 의 과정을 포함하고 있는 것입니다. <br>
fetch 이후 merge 를 수행하면 pull 명령과 동일한 수행 내역이 되는 것입니다.


-------

## Git 기본 명령어

**1) 저장소 생성** 

```
git init
```

### 2) 원격 저장소로부터 복제 

```
git clone {url}
```

### 3) 변경 사항 체크

```
git status // 
```

특정 파일 스테이징

```
git add {파일명} 
```

변경된 모든 파일 스테이징

```
git add * 
```

커밋

```
git commit -m “{변경 내용}” 
```

원격으로 보내기

```
git push origin master 
```

원격저장소 추가

```
git remote add origin {원격서버주소} 
```

참고 페이지

- download(osx): http://code.google.com/p/git-osx-installer/downloads/list
- download(windows): http://git-scm.com/download/win
- 설치 메뉴얼: http://blog.outsider.ne.kr/389
- 사용 메뉴얼:http://dogfeet.github.io/articles/2012/how-to-github.html
- git 간편 안내서: http://rogerdudler.github.com/git-guide/index.ko.html
- 한장으로 핵심 기능만: http://rogerdudler.github.com/git-guide/files/git_cheat_sheet.pdf


## Commit

커밋 합치기

```
git rebase -i HEAD~4 // 최신 4개의 커밋을 하나로 합치기
```

커밋 메세지 수정

```
$ git commit --amend // 마지막 커밋메세지 수정(ref)
```

간단한 commit방법

```
$ git add {변경한 파일병}
$ git commit -m “{변경 내용}"
```

커밋 이력 확인

```
$ git log // 모든 커밋로그 확인
$ git log -3 // 최근 3개 커밋로그 확인
$ git log --pretty=oneline // 각 커밋을 한 줄로 표시
$ git reflog // reset 혹은 rebase로 없어진 과거의 커밋 이력 확인
```

커밋 취소

```
$ git reset HEAD^ // 마지막 커밋 삭제
$ git reset --hard HEAD // 마지막 커밋 상태로 되돌림
$ git reset HEAD * // 스테이징을 언스테이징으로 변경, ref
```


## Branch

master 브랜치를 특정 커밋으로 옮기기

```
git checkout better_branch
git merge --strategy=ours master    # keep the content of this branch, but record a merge
git checkout master
git merge better_branch            # fast-forward master up to the merge
```

브랜치 목록

```
$ git branch // 로컬
$ git branch -r // 리모트 
$ git branch -a // 로컬, 리모트 포함된 모든 브랜치 보기
```

브랜치 생성

```
git branch new master // master -> new 브랜치 생성
git push origin new // new 브랜치를 리모트로 보내기
```

브랜치 삭제

```
git branch -D {삭제할 브랜치 명} // local
git push origin :{the_remote_branch} // remote
```

빈 브랜치 생성

```
$ git checkout --orphan {새로운 브랜치 명}
$ git commit -a // 커밋해야 새로운 브랜치 생성됨
$ git checkout -b new-branch // 브랜치 생성과 동시에 체크아웃
```

리모트 브랜치 가져오기

```
$ git checkout -t origin/{가져올 브랜치명} // ref
```

브랜치 이름 변경

```
$ git branch -m {new name} // ref
```


## Tag


태그 생성

```
git tag -a {tag name} -m {tag message} {commit hash}
git tag {tag name} {tag name} -f -m "{new message}" // Edit tag message
```

태그 삭제

```
git tag -d {tag name}
git push origin :tags/{tag name} // remote
```

태그 푸시

```
git push origin --tags
git push origin {tag name}
git push --tags
```


## 기타 

파일 삭제

```
git rm --cached --ignore-unmatch [삭제할 파일명]
```

히스토리 삭제

- 목적: 패스워드, 아이디 같은 비공개 정보가 담긴 파일을 실수로 올렸을 때 삭제하는 방법이다. (history에서도 해당 파일만 삭제)

```
$ git clone [url] # 소스 다운로드
$ cd [foler_name] # 해당 폴더 이동
$ git filter-branch --index-filter 'git rm --cached --ignore-unmatch [삭제할 파일명]' --prune-empty -- --all # 모든 히스토리에서 해당 파일 삭제
$ git push origin master --force # 서버로 전송
```

히스토리에서 폴더 삭제:

```
git filter-branch --tree-filter 'rm -rf vendor/gems' HEAD
```

리모트 주소 추가하여 로컬에 싱크하기

```
$ git remote add upstream {리모트 주소}
$ git pull upstream {브랜치명}
```

최적화

```
$ git gc
$ git gc --aggressive
```

## 서버 설정

강제 푸시 설정

```
git config receive.denynonfastforwards false
```

## Alias

~/.gitconfig 파일을 설정하여 깃 명령어의 앨리어스를 지정할 수 있다.

~/.gitconfig > alias 부분:

```

[alias]
  br = branch
  co = checkout
  rb = rebase
  st = status
  cm = commit
  pl = pull
  ps = push
  lg = log --graph --abbrev-commit --decorate --format=format:'%C(cyan)%h%C(reset) - %C(green)(%ar)%C(reset)  %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(yellow)%d%C(reset)' --all
  ad = add
  tg = tag
  df = diff 
```

