---
layout: post
title: AWS kinesis streaming
category: AWS
---
## 1 설치 환경

1. windows 10 64bit


## 2 chocolatey 설치

1. powershell에서 진행  
	1. window key + r  
	2. powershell 입력  
	3. 관리자 권한으로 열기 위해 crtl+shift+enter

2. https://chocolatey.org/install 에서 설치 명령어를 복붙
ex) 
```bash
$ Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```


## 3 kinesis에 필요한 패키지 설치

1. 마찬가지로 powershell 관리지 권한으로 진행  
2. nasm, perl, gstreamer 설치  
```bash
$ choco install nasm strawberryperl gstreamer
$ choco install gstreamer-devel --version=1.16.2
```
	* gstreamer가 C 드라이브가 아닌 D, E 등에 설치될 수도 있다. 꼭 C 드라이브로 옮길 것.  
	ex) C:\gstreamer
	* 이 경우, chocolatey 자동으로 설정해주는 환경변수 GSTREAMER_1_0_ROOT_X86_64의 경로도 바꿔줘야 한다.  
	gstreamer 내의 bin 폴더의 경로로 다시 설정해줄 것  
	ex) C:\gstreamer\1.0\x86_64
	* strawberryperl, nasm도 chocolatey 자동으로 환경변수 path에 경로를 추가해주지만 혹시 모르니 한 번씩 확인 할 것.
		1. strawberryperl
			- C:\Strawberry\c\bin
			- C:\Strawberry\perl\site\bin
			- perl.exe 경로 : C:\Strawberry\perl\bin
		2. nasm
			- nasm.exe 경로 : C:\Program Files\NASM

3. pkg config 설치
```bash
$ choco install pkgconfiglite
```
	* pkg-config.exe의 경로를 환경변수 path에 추가해야 한다.  
	ex) C:\ProgramData\chocolatey\lib\pkgconfiglite\tools\pkg-config-lite-0.28-1\bin
	* 앞서 설치한 perl, gsteamer 내의 pkg-config.exe 때문에 제대로 인식 되지 않을 수 있다.  
path에서 pkgconfiglite의 pkg-config.exe 경로를 제일 상단으로 올린다.


## 4 kinesis cpp sdk 다운로드

* 여기부터는 cmd에서 실행
```bash
$ git clone --recursive https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp.git
```


## 5 kinesis cpp sdk build

1. build_windows.bat 수정
	* .github 폴더의 build_windows.bat 파일 내에 `call "C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\VC\Auxiliary\Build\vcvars64.bat"` 부분이 있을텐데,  
visual studio가 설치 되어 있다면 비슷한 경로에 vcvars64.bat이 있을 것이다. 해당 경로로 수정한다.  
만약 visual studio가 없다면 설치 후 진행한다.  
AWS의 가이드를 보면 MinGW로도 가능한 모양이지만 계속 빌드 에러가 나는 통에 그냥 비주얼로 진행했다... 

2. build_windows.bat 실행
	* CMakeLists.txt 때문에 sdk의 루트 디렉토리에서 실행해야 한다.
```bash
$ .github/build_windows.bat
```
	* 이후 약 30분 간 빌드가 완료되기를 기다린다.


## 6 샘플 실행

1. kinesis 환경변수 설정
	* kinesis sdk readme에도 명시되어 있지만, 반드시 루트 디렉토리에서 실행할 것.
```bash
$ set GST_PLUGIN_PATH=%CD%\build
$ set PATH=%PATH%;%CD%\open-source\local\bin;%CD%\open-source\local\lib
```

2. 플러그인 확인
	* 아래 명령어를 실행했을 때 무언가 오류가 나거나 `No such element or plugin 'kvssink'` 메세지가 뜬다면 경로 설정 오류 혹은 빌드 실패 등 뭔가가 잘못된 것이다.
```bash
$ gst-inspect-1.0 kvssink
```

3. 실행
	* 웹캠으로 스트리밍하는 예제이다.
```bash
$ gst-launch-1.0 ksvideosrc do-timestamp=TRUE ! video/x-raw,width=640,height=480,framerate=30/1 ! videoconvert ! x264enc bframes=0 key-int-max=45 bitrate=512 ! video/x-h264,profile=baseline,stream-format=avc,alignment=au,width=640,height=480,framerate=30/1 ! kvssink stream-name="your-stream-name" access-key=your_accesskey_id secret-key=your_secret_access_key
```

4. 스트리밍 확인
	* 아래의 hls 플레이어 예제를 통해 자신이 스트리밍 하고 있는 영상을 확인할 수 있다.  
	<https://github.com/aws-samples/amazon-kinesis-video-streams-media-viewer>


## 7 AWS kinesis sdk - cpp github

* 더 자세한 정보는 아래에서 확인. 빌드 도중의 에러 등은 issue에서 찾아보면 왠만하면 나온다.
<https://github.com/awslabs/amazon-kinesis-video-streams-producer-sdk-cpp>