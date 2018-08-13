# 회사 자체 https 인증서를 쓰는 곳에서 Firebase for Unity 설정법

문제점

* 회사에서 인증서 문제로 aar 파일이 다운로드가 되지 않는다. \(회사 자체 인증서로 인터넷 접속\)
* ios의 경우 cocoapod를 이용해서 다운로드를 하기에는 괜찮은듯. 
* 집에서 할 경우 제대로 됨. 

해결 방안

* aar이 다운로드가 제대로 이루어지지 않기에 플러그인의 실 소스 코드를 수정 하자. 
* 일단 gradle의 maven의 https 인증서 체크를 무시하도록 해야 함

```bash
maven -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true
```

[https://www.lesstif.com/pages/viewpage.action?pageId=7635187](https://www.lesstif.com/pages/viewpage.action?pageId=7635187)

* Asset/Firebase/Editor/AppDendencies.xml 아래와 같이 수정해본다. Https 는 쓰지 말것. \(이게 해결 방법1\)

```markup
    <androidPackage spec="com.google.firebase:firebase-core:16.0.1">
      <repositories>
        <repository>http://maven.google.com</repository>
        <repository>http://jcenter.bintray.com</repository>
        <repository>http://repo.maven.apache.org/maven2</repository>
      </repositories>
    </androidPackage>
```

구글만으로 부족할수 있으니 Jcenter와 MavenCentral의 주소도 추가 시켜준다.

* 위 방법으로 해결이 안될 경우, 일단 집에서는 정상적으로 받아진다고 했으므로, 관련 파일을 모조리 받아서 m2repository에 넣어준뒤 로컬 경로로 변경해준다. \(해결방법 2\)

```markup
    <androidPackage spec="com.google.firebase:firebase-core:16.0.1">
      <repositories>
        <repository>Assets/Plugin/....</repository>
      </repositories>
    </androidPackage>
```

* 위 방법도 안 될경우에는 별수 없이

  [https://github.com/googlesamples/unity-jar-resolver](https://github.com/googlesamples/unity-jar-resolver)

  에 가서 Dll 파일 지우고 소스코드로 안 받아지는 파일들 디버깅 해보자. \(해결방법 3\)

