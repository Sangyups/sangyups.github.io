---
title: "[C++] Mac에서 VSCode C++ PS 환경 세팅하기"
categories:
  - PS
tags:
  - C++
  - VSCode
date: 2021-12-30 23:07:00+0900
last_modified_at: 2022-01-18 00:53:00+0900
---
진짜 오랜만에 다시 PS를 하려고 백준 온라인 저지 사이트를 키고 코드를 실행시킬 IDE를 열심히 찾았다. 실제 코딩에 들어가기 전 이런 세팅을 먼저 완벽히 끝내놓고 하는 타입이라 우선 나에게 맞는 환경을 찾아보기로 했다. Mac에서는 다음과 같은 IDE 옵션들이 있다.

- XCode
- Visual Studio
- CLion

등등...

솔직히 다 써봤지만 나한테 있어서 가장 큰 불만은 위의 IDE 들을 이용하게 된다면 하나의 프로젝트 단위로 관리해야 하며 프로젝트 생성에 의해 같이 생성되는 잡다한 파일, 폴더 등이 너무 많아진다는 점이다. 내가 쓰는 것은 오직 cpp 소스파일 하나 뿐인데 배보다 배꼽이 더 커지는 것이다. 그래서 결국 VSCode에다가 PS 환경 세팅을 하도록 했다. 물론 VSCode를 가장 자주 써서 제일 편한 것도 있다.

## 빌드 세팅

### 기본 환경

```bash
g++ -v
lldb -v
```

Mac에 가장 기본적으로 깔려 있는 컴파일 및 디버깅 관련 명령어 도구들이다. 제대로 깔려 있는지 버전 체크를 해본 후 만약 설치되어 있지 않다면 `xcode-select --install` 명령어를 통해 Xcode 명령어 도구를 설치한다.

### Microsoft C/C++ Extension

![Untitled](https://user-images.githubusercontent.com/80937237/149800257-052bbe5b-c8f2-4af6-bb7f-278d03f304a3.png)

VSCode에서 C/C++을 이용하고 싶다면 가장 필수적으로 깔아줘야 하는 설정이다.

### Code Runner

![Untitled 1](https://user-images.githubusercontent.com/80937237/149800214-efc37580-d6e6-47fa-89f7-c192b5878168.png)

VSCode 상에서 설정해주어 명령어를 통한 컴파일이 가능하긴 하지만 Code Runner를 깔면 굉장히 간편하게 내가 작성한 Source Code를 빌드할 수 있다.

![Untitled 2](https://user-images.githubusercontent.com/80937237/149800226-b8a87a51-4f92-4357-b797-44b562c0366b.png)

 필수적으로 해야되는 것은 아니지만 나는 터미널에서 작업하는 것이 훨씬 편하기 때문에 VSCode 설정을 열고 아래와 같은 설정을 체크해서 Output 패널 대신 Terminal에서 작업할 수 있게 해주었다.

![Untitled 3](https://user-images.githubusercontent.com/80937237/149800229-cdc85453-5bc6-4393-a89d-205c45bfa57b.png)

그리고 위의 버튼을 누르거나 혹은 직접 `settings.json`에 들어가서 아래와 같이 Executor Map 설정을 해준다. 

```json
"code-runner.executorMap": {
  "cpp": "cd $dir && clang++ -std=c++17 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt"
},
```
{: file="settings.json"}

`-std=c++17` 옵션을 통해 c++의 최신 문법을 사용할 수 있도록 해준다. 이것을 해주지 않는다면 최신 문법을 컴파일러가 읽지 못한다.

![Untitled 4](https://user-images.githubusercontent.com/80937237/149800232-1c732dbe-1a61-46b1-bcd9-f601c750bbb7.png)

내가 사용하고자 하는 폴더를 열어주고 *Terminal-Configure Default Build Task...* 를 눌러준다.

![Untitled 5](https://user-images.githubusercontent.com/80937237/149800236-07062abd-9c61-439f-bb9c-75bde83cc94f.png)

그리고 위와 같이 clang++ 를 선택해준다.

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "cppbuild",
      "label": "C/C++: clang++ build active file",
      "command": "/usr/bin/clang++",
      "args": [
				"-std=c++17",
        "-fdiagnostics-color=always",
        "-g",
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}"
      ],
      "options": {
        "cwd": "${fileDirname}"
      },
      "problemMatcher": ["$gcc"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "detail": "compiler: /usr/bin/clang++"
    }
  ]
}
```
{: file="tasks.json" }

그러면 위와 같이 `tasks.json` 이 생성되는데 `"args"` 의 첫 번째 아이템으로 `“-std=c++17",` 를 추가해주면 된다. 그러면 기본적으로 빌드 시 위의 `tasks.json` 에 적혀진 정보를 이용하게 된다. 여기까지 한다면 Code-runner를 통해 c++ 소스코드를 빌드하고 실행시킬 수 있다.

![Untitled 6](https://user-images.githubusercontent.com/80937237/149800238-28357bb5-817b-47f5-a5dd-68c461667415.png)

나는 위와 같이 keyboard shortcut을 바꿔주어서 이용 중이다.

## 디버그 세팅

이후에는 디버깅 세팅을 완료할 차례이다.

![Untitled 7](https://user-images.githubusercontent.com/80937237/149800241-f0c1e123-0793-453f-a3fd-0895eb6f0ef1.png)

F5를 눌러 디버깅 설정을 시작하자. C++ (GDB/LLDB) 를 눌러준다.

![Untitled 8](https://user-images.githubusercontent.com/80937237/149800248-340c5d49-3169-497f-be7e-17efa552e1b1.png)

위의 두 개 중 아무거나 한 개를 눌러준다.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "clang++ - Build and debug active file",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "lldb",
      "preLaunchTask": "C/C++: clang++ build active file"
    }
  ]
}
```
{: file="launch.json"}
그렇다면 위와 같이 `launch.json` 파일이 생성된다. 여기까지 왔으면 breakpoint를 설정해서 디버깅을 바로 진행 가능하다. 

❗️**cin, cout 등이 안될 때**

`launch.json` 의 설정에서 `"externalConsole"` 의 설정을 `false` 에서 `true` 로 바꿔준다. 그러면 `cin`, `cout` 처럼 사용자와 상호작용을 하는 코드가 외부의 새로운 터미널에서 실행이 된다.

### LLDB 설치

여기서부터는 선택사항이다 (*M1 맥북의 경우 이 부분도 진행해주어야 한다고 들었지만 M1 맥북이 없는 관계로 테스트 할 수가 없다. 위까지 했는데 잘 안된다면 이 부분도 진행해보자*). 나는 `cin` , `cout` 등을 새로운 터미널 창에서 해야된다는 사실이 굉장히 맘에 들지 않았다. 항상 내부 터미널로만 해결해왔기 때문에 이것을 해결하려고 계속 구글링해보았지만 도저히 찾을 수가 없어서 결국 새로운 확장프로그램을 설치하기로 했다.

![Untitled 9](https://user-images.githubusercontent.com/80937237/149800249-65246b46-ed7b-4406-bc50-a7815a12a17c.png)

위의 확장 프로그램을 설치해주도록 한다. VSCode 상에서 LLDB를 이용할 수 있도록 해주는 확장 프로그램이다. 해당 확장 프로그램을 설치하고 `launch.json` 에서 `"type": "cppdbg"` 에서  `"type": "lldb"` 로 바꿔주면 끝이다.

## 추가 설정 (깨끗한 폴더 유지)

사실 이 글을 쓴 가장 큰 이유이기도 한데 나는 백준을 풀면서 문제 번호 별로 소스코드를 나눠서 저장하고 싶었다. 그래서 PS 폴더 내부에 문제_번호.cpp 로 이루어진 소스코드들로만 있었으면 좋겠다고 생각했다. 그렇기 때문에 위의 IDE들을 다 포기하고 이렇게 VSCode를 이용하여 PS 환경을 세팅한 것이다.

![Untitled 10](https://user-images.githubusercontent.com/80937237/149800253-9189a299-c590-4275-9b66-1f8f7034fdab.png)

(위의 문제 번호는 예시이다.)

그런데 이렇게 해도 디버깅 관련 파일과 실행파일이 어쩔 수 없이 남게 된다. 그래서 해당 소스코드를 빌드하고 실행이 끝나거나 디버깅이 끝나면 해당 파일들을 다 없애주고 싶어서 추가적으로 설정해주려고 한다.

```bash
rm -rf ./*.dSYM
ls -ar | grep -v "\." | xargs rm
```
{: file="clean.sh"}

같은 폴더 내에 다음과 같은 쉘 스크립트 파일을 작성한다. 간단하게 설명하면 첫 번째 줄은 .dSYM 으로 끝나는 디버깅 관련 폴더들을 다 지우고 두 번째 줄은 확장자가 없는 executable 파일들을 지워준다. 그리고 해당 쉘 스크립트를 각 action이 끝났을 때마다 실행되도록 해주면 된다.

```json
{
  "version": "2.0.0",
  "tasks": [
    ...,
    {
      "type": "shell",
      "label": "clean",
      "command": "${fileDirname}/clean.sh"
    }
  ]
}
```
{: file="tasks.json"}
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "clang++ - Build and debug active file",
      "type": "lldb",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "lldb",
      "preLaunchTask": "C/C++: clang++ build active file",
      "postDebugTask": "clean" // 이 부분 추가
    }
  ]
}
```
{: file="launch.json"}
`tasks.json` 에 build task 밑에 추가적으로 위와 같은 새로운 task를 만들어준다. 그리고 `launch.json` 에서 `"postDebugTask": "clean"` 을 밑에 추가해줌으로써 Debuging이 끝나면 clean이라는 label을 가진 task가 실행될 수 있도록 한다. 그렇다면 자동적으로 해당 task에 의해 `clean.sh` 가 실행될 것이다. 여태까지 디버깅과 관련된 clean 작업이었고 이제 code-runner를 통해 디버깅 없이 실행만 시켰을 때 없애는 방법도 설정해보자.

```json
"code-runner.executorMap": {
  "cpp": "cd $dir && clang++ -std=c++17 $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt && $dir/clean.sh"
},
```
{: file="settings.json"}
vscode의 `settings.json`에 들어가서 Executor Map 설정 끝에 위와 같이 `&& $dir/clean.sh` 를 붙여준다. `&&` 은 다중 명령어의 일종으로 `&&` 앞의 명령어가 정상적으로 실행 후 종료된 뒤 그 뒤의 명령어가 실행되도록 해준다. 그래서 실행이 끝나면 해당 디렉토리의 `clean.sh` 가 실행되는 것이다. 이것으로 모든 설정이 끝났고 vscode를 통해 PS를 하면서 이제 깨끗한 작업공간을 유지할 수 있다.