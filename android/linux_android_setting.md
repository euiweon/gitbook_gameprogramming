#### 리눅스 안드로이드 sdk 설정(Docker) 



안드로이드 sdk 설치 ...

```bash

$ apt-get update
$ apt-get install default-jdk
$ wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip
$ unzip sdk-tools-linux-3859397.zip
$ export PATH=/home/euiweon/tools:/home/euiweon/tools/bin:$PATH
$ sdkmanager "platforms;android-25"

```



셋팅이 끝난후에 sdkmanager 로 각종 tool 들 설치 



ndk는 해당 버전  linux로 받은뒤 sdk 폴더에 압축을 풀고 ndk-bundle이라는 이름으로 바꾼다. 
