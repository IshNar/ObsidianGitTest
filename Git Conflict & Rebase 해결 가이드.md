## 📝 Git Conflict & Rebase 해결 가이드 (LeDos_SVI 프로젝트)

### 1. 상황 발생

원격 저장소(GitHub)의 최신 커밋과 로컬의 작업 내용이 겹칠 때 `git pull --rebase`를 시도했으나, 특정 파일에서 **Conflict(충돌)** 발생.

- **충돌 파일:** * `SVI_InspLib/Insp_ManagerLib.cpp`
    
    - `SVI_InspLib/include/Define.h`
        
- **상태:** `(master|REBASE 1/1)`
    

---

### 2. 해결 프로세스

#### STEP 1: 충돌 지점 확인 및 수정

에디터(VS Code 등)로 해당 파일을 열어 Git이 표시한 충돌 마커를 제거하고 최종 코드를 결정합니다.

C++

```
<<<<<<< HEAD
// 원격 저장소의 코드 (예: 구형 변수명)
=======
// 내가 수정한 코드 (예: UnderKill 관련 신규 변수명)
>>>>>>> e3ef48f... 260130
```

- **Action:** 위 기호들을 모두 삭제하고, 실제 프로젝트에 반영할 최종 로직만 남깁니다.
    

#### STEP 2: 수정 완료 알리기 (Stage)

수정한 파일들을 Git이 인식할 수 있도록 추가합니다.

Bash

```
git add SVI_InspLib/Insp_ManagerLib.cpp
git add SVI_InspLib/include/Define.h
```

#### STEP 3: 리베이스 계속 진행

Bash

```
git rebase --continue
```

- 이때 커밋 메시지 수정을 위한 **Vim 에디터**가 뜨면 `:wq`를 입력하여 저장하고 나옵니다.
    

#### STEP 4: 원격 저장소 반영 (Push)

리베이스로 인해 히스토리가 재정렬되었으므로, 강제 푸시가 필요할 수 있습니다.

Bash

```
git push origin master --force
```

---

### 💡 주요 팁

- **작업 전 Pull:** 출근 직후나 작업 시작 전에 `git pull`을 먼저 수행하면 충돌을 최소화할 수 있습니다.
    
- **안전한 강제 푸시:** `--force` 대신 `--force-with-lease`를 사용하면 다른 사람이 올린 코드를 실수로 지우는 것을 방지할 수 있습니다.
    
- **상태 확인:** 현재 어떤 단계인지 헷갈릴 때는 항상 `git status`를 입력해 보세요.
    

---

**Markdown 파일로 저장하는 법:**

1. 위 내용을 복사합니다.
    
2. 메모장이나 VS Code에서 새 파일을 만듭니다.
    
3. 내용을 붙여넣고 `Git_Conflict_Resolve.md`라는 이름으로 저장합니다.
    

혹시 이 정리된 내용 외에 특정 명령어의 **'본질적인 동작 원리'**나 추가적인 설명이 필요하신가요? 구체적으로 알고 싶은 부분이 있다면 말씀해 주세요!