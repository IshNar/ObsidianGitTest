GitHub Personal Access Token(PAT) 발급 방법을 단계별로 정리해 드립니다. 2021년 8월부터 GitHub는 보안 강화를 위해 패스워드 대신 PAT 사용을 의무화했습니다.

현재 GitHub에는 **Classic** 방식과 보안이 더 강화된 **Fine-grained**(세분화된 권한 설정 가능) 방식 두 가지가 있습니다. 일반적으로 범용적인 사용을 위해 **Classic** 방식을 많이 사용하므로, 이를 기준으로 설명하겠습니다.

---

## 1. GitHub PAT(Classic) 발급 단계

1. **GitHub 로그인 및 설정 진입**
    
    - GitHub 우측 상단의 **[프로필 아이콘]** 클릭 → **[Settings]** 선택.
        
2. **개발자 설정 이동**
    
    - 왼쪽 사이드바 가장 하단의 **[Developer settings]** 클릭.
        
3. **토큰 메뉴 선택**
    
    - **[Personal access tokens]** → **[Tokens (classic)]** 선택.
        
    - 우측 상단의 **[Generate new token]** 드롭다운 클릭 → **[Generate new token (classic)]** 선택.
        
4. **토큰 정보 설정**
    
    - **Note**: 토큰의 용도 입력 (예: `My Laptop CLI`, `Visual Studio`)
        
    - **Expiration**: 만료 기간 설정 (보안상 30일~90일을 권장하지만, 번거롭다면 'No expiration' 선택 가능)
        
    - **Select scopes**: 필요한 권한 체크
        
        - 일반적인 Git Push/Pull 용도라면 **`repo`** 항목 전체를 체크하면 충분합니다.
            
5. **토큰 생성 및 복사**
    
    - 가장 아래의 **[Generate token]** 버튼 클릭.
        
    - **⚠️ 주의**: 생성된 토큰 문자열(예: `ghp_...`)을 즉시 복사하여 안전한 곳에 저장하세요. **이 페이지를 벗어나면 다시는 확인할 수 없습니다.**
        

---

## 2. Classic vs Fine-grained 토큰 차이

|**구분**|**Fine-grained (Beta)**|**Classic**|
|---|---|---|
|**보안성**|매우 높음 (특정 레포지토리 지정 가능)|보통 (계정 내 모든 레포 접근)|
|**유효 기간**|최대 1년 (필수)|무제한 설정 가능|
|**추천 용도**|특정 프로젝트 협업, 보안이 중요한 서버|개인 PC의 CLI 환경, 범용 개발 도구|

---

## 3. 발급받은 토큰 사용 방법

터미널(Git Bash, CMD 등)에서 `git push` 등을 할 때 비밀번호를 물어보면, 기존 계정 비밀번호 대신 **복사해둔 PAT 값을 붙여넣기** 하면 됩니다.

### 💡 팁: 매번 입력하기 번거롭다면?

Windows 개발 환경이라면 다음 명령어를 통해 토큰을 로컬에 저장하여 매번 입력하는 번거로움을 줄일 수 있습니다.

Bash

```
git config --global credential.helper store
```

이후 한 번만 토큰을 입력하면 이후에는 자동으로 인증됩니다. (리눅스/Mac의 경우 `osxkeychain` 또는 `cache` 모드 사용 권장)

에이앤아이에서 반도체 검사 프로그램을 개발하시다 보면 빌드 자동화나 CI/CD 환경 구축 시 PAT가 자주 필요하실 텐데, 위 과정 중 막히는 부분이 있다면 말씀해 주세요.

**혹시 발급받은 토큰을 Visual Studio나 특정 IDE에 바로 적용하는 방법이 궁금하신가요?**