### Firebase 유니티 설정

문제점

- 회사에서 인증서 문제로 aar 파일이 다운로드가 되지 않는다. (회사 자체 인증서로 인터넷 접속)
- ios의 경우 cocoapod를 이용해서 다운로드를 하기에는 괜찮은듯. 
- 집에서 할 경우 제대로 됨. 



해결 방안

- aar이 다운로드가 제대로 이루어지지 않기에 플러그인의 실 소스 코드를 수정 하자. 
- 일단 gradle의 maven의 https 인증서 체크를 무시하도록 해야 함

```bash
maven -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true
```

https://www.lesstif.com/pages/viewpage.action?pageId=7635187

- Asset/Firebase/Editor/AppDendencies.xml 아래와 같이 수정해본다. Https 는 쓰지 말것. (이게 해결 방법)

```xml
    <androidPackage spec="com.google.firebase:firebase-core:16.0.1">
      <repositories>
        <repository>http://maven.google.com</repository>
      </repositories>
    </androidPackage>
```



