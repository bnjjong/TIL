# VS Code 2일차 가이드: 파일 & 코드 작업 마스터하기 

- **⌘**: 커맨드 (Command) - Windows의 Ctrl과 유사
- **⌥**: 옵션 (Option) - Windows의 Alt와 유사  
- **⌃**: 컨트롤 (Control) - 보조 기능키
- **⇧**: 쉬프트 (Shift)
- **⇪**: 캡스락 (Caps Lock)

## 파일 탐색과 검색의 기본
### 빠른 파일 열기와 탐색
VS Code의 **Quick Open** 기능은 IntelliJ의 `⌘ + ⇧ + N`과 유사하지만 더 간단합니다.[2][3][4]

- **⌘ + P**: 파일명 검색 및 빠른 이동 (가장 자주 사용하는 단축키)
- **?** 입력: 사용 가능한 명령어 확인
- **@** 입력: 현재 파일의 심볼 검색
- **:** 입력: 특정 줄 번호로 이동

### 전체 프로젝트 검색
대규모 Spring 프로젝트에서 코드를 찾을 때 필수적인 기능들입니다:[2][3]

- **⌘ + ⇧ + F**: 전체 프로젝트 검색 (IntelliJ와 동일)
- **⌘ + F**: 현재 파일 검색
- **⌘ + G**: 다음 검색 결과로 이동
- **⇧ + ⌘ + G**: 이전 검색 결과로 이동

정규식을 사용하여 `callback|false`와 같이 여러 단어를 동시에 검색할 수 있으며, 특정 파일 타입만 검색하는 필터링 기능도 제공합니다[2].

## 멀티 커서와 다중 선택 활용
macOS VS Code의 멀티 커서 기능은 변수명 일괄 변경이나 반복 작업에서 매우 강력합니다.[2][4][5]

### 기본 멀티 커서 기능
- **⌥ + Click**: 원하는 위치에 커서 추가
- **⌥ + ⌘ + ↑/↓**: 현재 커서 기준 위아래로 커서 추가
- **ESC**: 멀티 커서 해제[2][4]

### 같은 단어 선택 및 편집
Spring 개발 시 변수명이나 메서드명을 일괄 변경할 때 매우 유용합니다:

- **⌘ + D**: 현재 단어와 같은 다음 단어를 하나씩 선택
- **⌘ + ⇧ + L**: 현재 단어와 같은 모든 단어를 한 번에 선택
- **⌘ + U**: 마지막 선택 취소[2][4][5]

### 세로 선택 (Column Selection)
- **⇧ + ⌥ + 드래그**: 박스 형태로 세로 선택
- **⇧ + ⌥ + ⌘ + ↑/↓**: 키보드로 세로 선택
- 여러 줄의 동일한 위치를 한 번에 편집할 때 유용[3][4]

## 코드 편집 효율성 높이기### 줄 단위 편집IntelliJ에서 익숙한 줄 편집 기능들을 macOS VS Code에서도 사용할 수 있습니다:[1][2][5]

- **⌥ + ↑/↓**: 현재 줄을 위아래로 이동
- **⇧ + ⌥ + ↑/↓**: 현재 줄을 위아래로 복사
- **⇧ + ⌘ + K**: 현재 줄 삭제
- **⌘ + Enter**: 커서 아래에 빈 줄 생성

### 주석 처리- **⌘ + /**: 줄 주석 토글 (IntelliJ와 동일)
- **⇧ + ⌥ + A**: 블록 주석 토글
- 선택된 여러 줄에 대해서도 동시 적용 가능[1][2]

### 코드 정렬과 포맷팅- **⇧ + ⌥ + F**: 자동 코드 정렬 (전체 문서)
- **⌘ + K, ⌘ + F**: 선택된 영역만 정렬[2][4]

## 워크스페이스 통합 관리### .code-workspace 파일 활용VS Code의 워크스페이스 기능은 여러 프로젝트를 효율적으로 관리할 수 있는 강력한 도구입니다.[6][7][8]

#### 워크스페이스 생성 과정

1. **File → Add Folder to Workspace**: 여러 프로젝트 폴더를 하나의 워크스페이스에 추가
2. **File → Save Workspace As**: `.code-workspace` 파일로 저장
3. **프로젝트별 설정**: `.vscode` 폴더에 `settings.json` 등 저장[7][8]

#### 워크스페이스 파일 구조 예시

```json
{
  "folders": [
    {
      "name": "backend-api",
      "path": "./backend"
    },
    {
      "name": "frontend-react", 
      "path": "./frontend"
    }
  ],
  "settings": {
    "java.home": "/opt/homebrew/opt/op```dk@11"
  }
}
```

### 화면 분할 관리대형 Spring 프로젝트에서는 여러 파일을 동시에 보는 것이 중요합니다:

- **⌘ + \\**: 에디터를 좌우로 분할
- **⌘ + K, ⌘ + \\**: 에디터를 상하로 분할
- **⌘ + 1/2/3**: 분할된 창 간 이동
- **드래그 앤 드롭**: 탭을 원하는 위치로 끌어서 분할[2][4]

### 사이드바와 패널 관리- **⌘ + B**: 왼쪽 사이드바 토글
- **⌃ + `**: 통합 터미널 토글 (Windows와 다름!)
- **⌘ + J**: 하단 패널 토글
- **⌘ + ⇧ + E**: 파일 탐색기로 포커스 이동[1][2]

## 터미널에서 VS Code 실행하기macOS에서 개발할 때 터미널에서 `code .` 명령으로 현재 디렉토리를 VS Code에서 바로 열 수 있습니다.[9][10][11]

### Shell Command 설치 방법1. VS Code 실행
2. **⇧ + ⌘ + P** (명령 팔레트 열기)
3. "shell command" 입력
4. **Shell Command: Install 'code' command in PATH** 선택
5. 권한 확인 후 완료

설치 후 터미널에서 다음과 같이 사용할 수 있습니다:
- `code .`: 현재 디렉토리를 VS Code에서 열기
- `code myproject/`: 특정 폴더 열기
- `code file.txt`: 특정 파일 열기[9][10][11]

## macOS Java 개발 환경 설정### Homebrew를 통한 JDK 설치macOS에서 Java 개발 환경을 구축하는 가장 효율적인 방법입니다:[12][13][14]

```bash
# OpenJDK 설치
brew install openjdk@11

# 심볼릭 링크 생성
sudo ln -sfn /opt/homebrew/opt/openjdk@11/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-11.jdk
```

### VS Code Java 확장팩 설치**Extension Pack for Java**를 설치하면 다음 확장팩들이 자동으로 설치됩니다:
- Language Support for Java by Red Hat
- Debugger for Java
- Test Runner for Java  
- Maven for Java
- Project Manager for Java[12][13][15]

### JAVA_HOME 환경 변수 설정macOS에서는 `.zshrc` 파일 (Catalina 이후) 또는 `.bash_profile`에 설정합니다:

```bash
# ~/.zshrc 파일에 추가
export JAVA_HOME=/opt/homebrew/opt/openjdk@11
export PATH=$JAVA_HOME/bin:$PATH
```

설정 후 `source ~/.zshrc` 명령으로 적용하고 `java -version`으로 확인.[12][14][15]

## Breadcrumb 네비게이션 활용VS Code의 **Breadcrumb** 기능은 현재 파일의 위치와 구조를 한눈에 파악할 수 있게 해줍니다.[16][17]

### Breadcrumb 기본 사용법- **⌘ + ⇧ + .**: Breadcrumb에 포커스하고 드롭다운 열기
- **화살표 키**: Breadcrumb 항목 간 이동
- **Enter**: 선택된 항목 열기
- **타이핑**: 필터링으로 빠른 검색

## IntelliJ 사용자를 위한 전환 팁### 주요 단축키 비교IntelliJ에서 VS Code로 전환할 때 자주 사용하는 단축키들의 차이점을 정리했습니다:[18]### 단축키 커스터마이징**⌘ + K, ⌘ + S**로 키보드 설정을 열어 IntelliJ와 유사한 단축키로 변경할 수 있습니다. 또는 **IntelliJ IDEA Keybindings** 확장팩을 설치하여 기존 단축키를 그대로 사용할 수도 있습니다.[18]

### 프로젝트 구조 관리IntelliJ의 모듈 개념과 달리, VS Code는 워크스페이스 내에서 여러 프로젝트를 플랫하게 관리합니다. Maven 멀티모듈 프로젝트의 경우 루트 `pom.xml`이 있는 폴더를 워크스페이스로 설정하는 것이 좋습니다.[19]

## 터미널 활용과 출력 관리### macOS 터미널 설정Spring Boot 프로젝트 개발 시 터미널 사용이 빈번하므로, 효율적인 터미널 설정이 중요합니다:

- **⌃ + `**: 터미널 토글 (Windows와 다름!)
- **⌃ + ⇧ + `**: 새 터미널 열기
- **터미널 프로필**: zsh, bash, fish 등 선택 가능

### 출력 버퍼 확장로그가 많은 Spring 애플리케이션의 경우 터미널 출력 버퍼를 늘려야 합니다:

Settings에서 `scrollback` 검색 후 **Terminal › Integrated: Scrollback** 값을 기본 1000에서 10000 이상으로 변경하는 것이 좋습니다.

## macOS 전용 추가 기능### 시스템 통합 기능- **⌘ + ,**: VS Code 설정 열기
- **⌃ + ⌘ + F**: 전체화면 토글
- **⌘ + ⇧ + P**: 명령 팔레트 (F1로도 접근 가능)
- **⌘ + W**: 현재 탭 닫기
- **⌘ + Q**: VS Code 완전 종료

### Spotlight 통합macOS의 Spotlight 검색에서 `.code-workspace` 파일을 바로 검색하여 열 수 있어, 프로젝트 전환이 매우 빠릅니다.

## 마무리VS Code 2일차에서는 macOS 환경에 특화된 파일 탐색, 멀티 커서, 코드 편집, 워크스페이스 관리 등 핵심 기능들을 익혔습니다. 특히 macOS의 키보드 심볼과 단축키 체계를 이해하는 것이 중요합니다.IntelliJ에서 VS Code로 전환하는 과정에서 가장 중요한 것은 각 도구의 철학적 차이를 이해하는 것입니다. IntelliJ는 무거운 IDE로서 모든 기능이 내장되어 있지만, VS Code는 가벼운 에디터에서 시작하여 확장팩으로 기능을 추가하는 방식입니다.

다음 단계에서는 이러한 기본기를 바탕으로 Java와 Kotlin 개발에 특화된 확장팩 설정, 디버깅 환경 구축, 그리고 실제 Spring 프로젝트 개발 워크플로우를 다룰 예정입니다. macOS 환경의 장점을 살려 효율적인 개발 환경을 구축해보시길 바랍니다.

[1] https://zkvn1103.tistory.com/4
[2] https://iworldt.tistory.com/37
[3] https://juntcom.tistory.com/80
[4] https://skuld2000.tistory.com/204
[5] https://leeaain.tistory.com/entry/VScodeMac-%ED%95%9C-%EC%A4%84-%EC%82%AD%EC%A0%9C-%ED%95%9C-%EC%A4%84-%EB%B3%B5%EC%A0%9C-%EB%8B%A4%EC%A4%91-%EC%84%A0%ED%83%9D-%EB%8B%A8%EC%B6%95%ED%82%A4-%EB%93%B1%EB%A1%9D%ED%95%98%EA%B8%B0
[6] https://jakedevdiray.tistory.com/9
[7] https://code-algo.tistory.com/178
[8] https://heyoonow.tistory.com/54
[9] https://www.freecodecamp.org/korean/news/how-to-open-visual-studio-code-from-your-terminal/
[10] https://shanepark.tistory.com/50
[11] https://archivers.tistory.com/175
[12] https://geminihoroscope.tistory.com/157
[13] https://will-behappy.tistory.com/34
[14] https://velog.io/@cjyooong/Java-%EB%A7%A5%EB%B6%81%EC%97%90%EC%84%9C-Java-%ED%99%98%EA%B2%BD%EC%84%B8%ED%8C%85VSCODE
[15] https://www.varofla.com/2c6307e1-a558-4e45-920e-4c054d75be0a
[16] https://code.visualstudio.com/docs/editing/editingevolved
[17] https://www.youtube.com/watch?v=y04X_sawKMg
[18] https://glasslego.tistory.com/210
[19] https://csj000714.tistory.com/711
[20] https://zuyo.tistory.com/902
[21] https://velog.io/@gillog/IDE-VSCode-Mac-OS-%EB%8B%A8%EC%B6%95%ED%82%A4-%EC%A0%95%EB%A6%AC
[22] https://sonny6786.tistory.com/1
[23] https://likedev.tistory.com/entry/MAC%EB%A7%A5%EB%B6%81-Visual-Studio-Code%EC%97%90%EC%84%9C-%ED%84%B0%EB%AF%B8%EB%84%90-%EC%8B%A4%ED%96%89-%EB%B0%A9%EB%B2%95
[24] https://altongmon.tistory.com/1055
[25] https://developer88.tistory.com/entry/VisualStudio-Code-%EC%9E%90%EC%A3%BC%EC%93%B0%EB%8A%94-%EB%A7%A5-%EB%8B%A8%EC%B6%95%ED%82%A4-%EC%A0%95%EB%A6%AC-VSCode
[26] https://make-some-wave.tistory.com/entry/vscode-terminal-mac
[27] https://parkjh7764.tistory.com/entry/VScode-Mac-%EB%8B%A8%EC%B6%95%ED%82%A4-%EB%B9%84%EC%A3%BC%EC%96%BC%EC%8A%A4%ED%8A%9C%EB%94%94%EC%98%A4-%EC%BD%94%EB%93%9C-%EB%A7%A5-%EB%A7%A5%EB%B6%81-%EB%8B%A8%EC%B6%95%ED%82%A4-%EB%AA%A8%EC%9D%8C
[28] https://webroadcast.tistory.com/30
[29] https://velog.io/@roadzmoon76/vscode-%EB%8B%A8%EC%B6%95%ED%82%A4
[30] https://inpa.tistory.com/entry/VS-Code-%E2%8F%B1%EF%B8%8F-%EC%9C%A0%EC%9A%A9%ED%95%9C-%EB%8B%A8%EC%B6%95%ED%82%A4-%EC%A0%95%EB%A6%AC
[31] https://kim-dragon.tistory.com/137
[32] https://justdoitproject.tistory.com/31
[33] https://geonlee.tistory.com/246
[34] https://computer-science-student.tistory.com/381
[35] https://my-adventure-book.tistory.com/22
[36] https://pinkwink.kr/1484
[37] https://joshwon.tistory.com/21
[38] https://bitkunst.tistory.com/entry/%EB%A7%A5%EB%B6%81MacOS-%ED%84%B0%EB%AF%B8%EB%84%90-%EC%84%B8%ED%8C%85%ED%95%98%EA%B8%B0-feat-Developer
[39] https://study-ce.tistory.com/66
[40] https://www.youtube.com/watch?v=ix7K3xMudgc