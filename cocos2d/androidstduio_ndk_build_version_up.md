#### Cocos2D-X Android Studio NDK 빌드 하기 

왜 이런짓을 해야 하는건가?

- 3.16 버전 이하에서 스튜디오에서 직접 빌드가 안되고, cocos 콘솔을 이용해서 빌드 해야되는 문제가 있음
- Android SDK 빌드툴 26 버전이 되면서 cocos 콘솔 명령으로 빌드 되는게 더이상 되지 않음. 

이렇게 함으로써 얻을수 있는 장점은?

- 그 동안 하기 힘들었던 C++ 디버깅이  안드로이드 스튜디오에서 가능해진다. 

그렇다면 단점은?

- NDK 빌드까지 한번에 이루어지기 때문에 빌드 시간이 많이 늘어난다. 
- 더이상 cocos 콘솔로 컴파일이 되지 않기 때문에 자동 빌드 하는 다른 방법을 찾아봐야 한다. (gradlew 를 이용해서 빌드 해야됨. )

하는 방법은 

- app/build.gradle 과  global.properties , Applycation.mk  Android.mk  파일만 수정하면 된다.

일단  global.properties

```python
PROP_APP_ABI=armeabi # 여긴 적당히 수정해도 된다. 

PROP_BUILD_TYPE=ndk-build

android.enableAapt2=false   # only gradle 3.0 이후로 적용... 최근 버전에서는 필요 없어졌기에 2018년 이후로 폐기 될 예정...
```



app/build.gradle

```json
	// defaultConfig 하위에 추가 
        externalNativeBuild {
            ndkBuild {
                targets 'MyGame'
                arguments 'NDK_TOOLCHAIN_VERSION=clang'
                arguments '-j' + Runtime.runtime.availableProcessors()
            }
            ndk {
                abiFilters = []
                abiFilters.addAll(PROP_APP_ABI.split(':').collect{it as String})
            }
        }
```

```json
// android 하위에 추가
    externalNativeBuild {
        ndkBuild {
            path "jni/Android.mk"
        }
    }
```

```json
    // buildTypes은 아래와 같이 적절히 수정 
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
            debuggable true
            jniDebuggable true
            renderscriptDebuggable true
 
 
            externalNativeBuild {
                ndkBuild {
                    arguments 'NDK_DEBUG=1'
                }
            }
 
 
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            debuggable false
 
            externalNativeBuild {
                ndkBuild {
                    arguments 'NDK_DEBUG=0'
                }
            }
 
 
        }
    
```



Android.mk  (추가)

```makefile
LOCAL_SHORT_COMMANDS := true  
```



Applycation.mk (추가)

```makefile
LOCAL_SHORT_COMMANDS := true 
```






