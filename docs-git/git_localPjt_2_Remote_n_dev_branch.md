

아래는 명령어별로 상세 설명을 추가한 요약 명령어입니다. 

---

### 1. **Git 초기화 및 사용자 설정**
#### **Git 저장소 초기화**
```bash
git init
```
- 현재 디렉토리를 Git 저장소로 초기화합니다.  
- `.git` 폴더가 생성되며, Git은 이 폴더를 통해 파일 변경 사항을 추적합니다.

#### **사용자 이름 및 이메일 설정**
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```
- Git을 사용할 때 커밋 작성자 정보를 설정합니다.  
- `--global` 옵션은 시스템 전역적으로 설정하며, 특정 프로젝트에만 설정하려면 `--local` 옵션을 사용합니다.

---

### 2. **파일 추가 및 첫 커밋**
#### **Git에 파일 추가**
```bash
git add .
```
- 현재 디렉토리의 모든 파일을 **Staging Area**에 추가합니다.  
- 특정 파일만 추가하려면 `git add 파일명`을 사용합니다.

#### **커밋 생성**
```bash
git commit -m "Initial commit"
```
- Staging Area에 있는 파일들을 저장소에 커밋합니다.  
- `-m` 옵션 뒤에 커밋 메시지를 작성합니다. 커밋 메시지는 변경 내용을 요약하여 기록합니다.

---

### 3. **GitHub 원격 레포지토리 연결 및 푸시**
#### **원격 레포지토리 연결**
```bash
git remote add origin https://github.com/YourUsername/YourRepository.git
```
- 로컬 저장소를 GitHub의 새 레포지토리에 연결합니다.  
- `origin`은 원격 저장소 이름이며, 기본적으로 원격 저장소를 가리키는 이름으로 사용됩니다.

#### **연결 확인**
```bash
git remote -v
```
- 연결된 원격 저장소를 확인합니다.  
- `fetch`와 `push` URL이 표시되며, 올바르게 연결되었는지 확인합니다.

#### **로컬 브랜치 이름 변경**
```bash
git branch -M main
```
- 기본 브랜치를 `main`으로 변경합니다.  
- GitHub는 기본 브랜치로 `main`을 사용하므로, 로컬 저장소의 브랜치 이름도 동일하게 맞춥니다.

#### **GitHub로 푸시**
```bash
git push -u origin main
```
- 로컬 `main` 브랜치를 GitHub의 `main` 브랜치에 업로드합니다.  
- `-u` 옵션은 추후 `git push` 명령어를 간단히 사용할 수 있도록 원격 브랜치를 기본값으로 설정합니다.

---

### 4. **새 브랜치 생성 및 전환**
#### **새 브랜치 생성**
```bash
git branch dev1
```
- `dev1`이라는 이름의 새 브랜치를 생성합니다.  
- 브랜치를 생성만 하고 전환하지는 않습니다.

#### **현재 브랜치 확인**
```bash
git branch
```
- 현재 브랜치 목록과 활성화된 브랜치(앞에 `*` 표시)를 확인합니다.

#### **브랜치 전환**
```bash
git checkout dev1
```
- 작업 중인 브랜치를 `dev1`으로 변경합니다.  
- 변경 후에는 해당 브랜치에서 작업이 저장됩니다.

#### **브랜치 생성과 전환을 동시에 수행**
```bash
git checkout -b dev1
```
- `dev1` 브랜치를 생성하고 바로 전환합니다.

---

### 5. **현재 브랜치 확인**
#### **활성화된 브랜치 확인**
```bash
git branch
```
- 현재 활성화된 브랜치와 모든 브랜치를 확인합니다.  
- `*` 기호가 현재 활성화된 브랜치를 나타냅니다.

---

### 6. **요약**
```bash
# Git 초기화 및 사용자 설정
git init
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"

# 파일 추가 및 첫 커밋
git add .
git commit -m "Initial commit"

# 원격 레포지토리 연결 및 푸시
git remote add origin https://github.com/YourUsername/YourRepository.git
git branch -M main
git push -u origin main

# 브랜치 생성 및 전환
git branch dev1
git checkout dev1
# 또는 생성과 전환을 동시에 수행
git checkout -b dev1

# 현재 브랜치 확인
git branch
```

위 명령어와 설명을 참고하여 GitHub로 로컬 프로젝트를 올리고 브랜치를 관리할 수 있습니다. VS Code를 사용하면 UI를 통해 작업을 간소화할 수도 있습니다.
