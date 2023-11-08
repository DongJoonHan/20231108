# 20231108
Git_Jenkins 과정


# File의 고유값(Checksum) 확인하기 (Windows)
명령어
```
certutil -hashfile 파일명 SHA256|MD5
```

예제
```
certutil -hashfile java-11-openjdk-11.0.15.9-1.windows.ojdkbuild.x86_64.msi SHA256

```

# 7zip 압축 명령
```shell
"C:\Program Files\7-Zip\7z.exe" a bin.zip .
```

## Jenkins Job 으로 포함하는데, 환경변수인 "BUILD_ID"를 사용하는 경우
```shell
"C:\Program Files\7-Zip\7z.exe" a bin_%BUILD_ID%.zip Bin
```

# OpenJDK 다운로드 위치(11버전)
- 홈페이지: https://github.com/ojdkbuild/ojdkbuild
- 실제 다운 위치: https://github.com/ojdkbuild/ojdkbuild/releases/download/java-11-openjdk-11.0.15.9-1/java-11-openjdk-11.0.15.9-1.windows.ojdkbuild.x86_64.msi


# VS2022 MSBuild.exe 위치
```
C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe
```

## CMD에서 빌드하기
```
"C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" DirectXTK_Desktop_2022_Win10.vcxproj
```
따옴표를 해준 이유는, 폴더에 공백이 있기 때문



# CppCheck
## 커맨드라인 실행
```
"C:\Program Files\Cppcheck\cppcheck"  --enable=all --inconclusive --xml --xml-version=2 src 2> cppcheck.xml
```

## MISRA 분석을 위한 실행
### Misra_2012.txt 파일 복사
- 위치: C:\DevTools\misra

### 설정 파일 생성: misra.json
- 위치: C:\DevTools\misra
- 파일 구성
```
{
    "script": "misra.py",
    "args": [
        "--rule-texts=C:\\DevTools\\misra\\Misra_2012.txt"
    ]
}
```

### 실행명령
```
cppcheck.exe --addon="C:\DevTools\misra\misra.json"  --xml --xml-version=2 . 2> cppcheck.xml
```

# Lizard
## 설치
CMD를 관리자 권한으로 실행 필요
```
pip install lizard
```

## 실행 명령(Command-Line for CSV)
```
lizard ./src --csv > lizard.csv
```
## CSV 출력 결과 예제
![image](https://github.com/DongJoonHan/20231106/assets/8405564/9f543137-e757-4e5e-96d7-dbd606005662)



## 실행 명령(Jenkins)
```
lizard ./src --xml > lizard.xml
```
or
```
"C:\Program Files\Python312\Scripts\lizard" ./src --xml > lizard.xml
```

# CPD
## 실행 명령
실행 시, 소스코드 폴더(분석 대상 폴더)의 대소문자가 동일해야 함!!

```
cpd --minimum-tokens 100 --files ./src --language cpp --format xml > cpd.xml || exit 0
```
or
```
C:\DevTools\pmd\bin\cpd --minimum-tokens 100 --files ./src --language cpp --format xml > cpd.xml || exit 0
```

뒤에 ||exit 0 이 있는 이유는, Jenkins는 exit 0을 정상이라고 보고, 나머지는 빌드 실패라 판단하는데 CPD는 기본으로 exit 1과 같은 방식으로 종료해서, 정상 처리해도 빌드 실패가 발생함. 이를 예방하기 위해 exit 0을 추가함.

# CLOC (라인수, 주석라인수 확인)
## 다운로드
https://github.com/AlDanial/cloc

## 사용법
```
cloc.exe .
```

# Pipeline
```
pipeline {
    agent any 
    
    environment {
        MSBUILD_PATH = 'C://Program Files//Microsoft Visual Studio//2022//Community//MSBuild//Current//Bin//MSBuild.exe'  
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    submoduleCfg: [],
                    userRemoteConfigs: [[url: 'https://github.com/microsoft/DirectXTK12.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                bat "\"${env.MSBUILD_PATH}\" DirectXTK_Desktop_2022_Win10.vcxproj"  
            }
        }
        
        stage('Verification') {
            steps {
                bat "\"C://Program Files//Cppcheck//cppcheck.exe\"  --enable=all --inconclusive --xml --xml-version=2 src 2> cppcheck.xml"
                recordIssues enabledForFailure: true, aggregatingResults: true, tool: cppCheck(pattern: 'cppcheck.xml')
            }
        }
    }
}
```
# Jenkins Node
## Windows 시작 시 연결을 위한 bat 파일 생성
가정: runNode.bat 이라 가정
```
cd c:\Jenkins
java -jar agent.jar -jnlpUrl http://172.16.2.26/computer/Win10_VM/jenkins-agent.jnlp -secret 90b6ee882eaddf4404c12db7db2a59ab30c56a283126d6da93d1edc7caf9a3cd -workDir "C:\Jenkins"
```

## 시작프로그램 등록
1. 시작+R 키로 "실행"을 실행
2. shell:startup 입력해서 실행
3. 시작 프로그램에 runNode.bat 복사

# Virtual Box CLI 명령
## 주의 사항
Jenkins가 서비스로 실행되는 경우, 로그인 사용자가 VirtualBox 설치 사용자와 다르면 VM 실행이 안됨.
Jenkins의 서비스 실행을 VirtualBox 설치 사용자와 동일하게 변경 필요
- 단, 사용자 Password가 필요함 (없는 경우 생성 필요)
### 방법
Service -> Jenkins -> 로그온 탭 -> 사용자 지정 선택 -> 사용자를 현재 사용자로 설정
![image](https://github.com/DongJoonHan/20231106/assets/8405564/0f605cee-3ae9-40b8-ad03-37ebb744f518)



## win10Pure VM을 OfficialBuild 스냅샷으로 복원
```
VBoxManage snapshot "win10Pure" restore "OfficialBuild"
```

## win10Pure VM 시작
```
VBoxManage startvm "win10Pure"
```

## win10Pure VM 시작 (GUI 없이 백그라운드로 시작)
```
VBoxManage startvm "win10Pure" --type=headless
```

# Node_DX12 Job에서 자동으로 Windows 종료 명령
```
shutdown /s /t 60
```
/s: 종료 (/r은 재부팅)
/t 초: 설정 시간(초) 이후에 종료

# Bamboo
## 다운로드
https://www.atlassian.com/software/bamboo/download-archives

# GitHub - PR - Jenkins 빌드 연동 참고자료
https://forl.tistory.com/139



