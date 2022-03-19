# iOS(swift) 소셜 로그인 연결하기
최근 새로운 iOS 앱 개발을 시작하면서 새롭게 소셜 로그인 연결을 작업하게 되었습니다.
(기존 앱은 objective-c 코드에 타겟 버전도 낮아서 재활용이 쉽지 않아 거의 새로 구현해야 했습니다)

대부분 2022년 2월 기준 최신 버전을 차용했으니 당분간 이 포스트가 유용하게 참고 되면 좋겠습니다.

## 1. KAKAOTALK
일단 제가 개발에 참여하는 앱에서는 카카오톡의 소셜 로그인 기능의 목적만 있습니다.
SDK는 cocoapod을 이용해 설치하기로 했습니다.
버전은 2.8.5를 사용했습니다.
```sh
pod KakaoSDKUser
```

그 후에는 카카오 개발자 페이지에 애플리케이션을 등록 해야합니다.

iOS 앱의 경우 필수적으로 Xcode 프로젝트의 Bundle Id를 등록해야 합니다.

이후 발급되는 네이티브 앱키는 AppDelegate.swift 파일에서 쓰입니다.

```sh
import UIKit
import KakaoSDKAuth

class AppDelegate: UIResponder, UIApplicationDelegate {
 
     func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
     
     //카카오톡 SDK 초기화하기
        KakaoSDK.initSDK(appKey: Appkey.kakao)
     }
```

Appkey.kakao의 위치에 각자 발급 받은 네이티브 앱 키를 적으면 됩니다.

그리고 카카오톡 링크 동작을 위해 AppDelegate.swift에 코드를 또 추가해줍니다.

```sh
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        //카카오톡
        if AuthApi.isKakaoTalkLoginUrl(url) {
            return AuthController.handleOpenUrl(url: url)
        }
        
        return false
    }
```

iOS 13부터는 SceneDelegate.swift에서 해당 액션을 관리한다는군요.
```sh
import UIKit
import KakaoSDKAuth
 
 class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url {
            if AuthApi.isKakaoTalkLoginUrl(url) {
                _ = AuthController.handleOpenUrl(url: url)
            }
        }
    }
 }
```

iOS 13 이전버전을 지원한다면 Appdelegate.swift에도 작업이 필요할것 같습니다.

자 이제 직접적인 로그인 파트 입니다.


```sh
         if UserApi.isKakaoTalkLoginAvailable() {
             UserApi.shared.loginWithKakaoTalk(channelPublicIds: nil, serviceTerms: nil) { (oauthToken, error) in
                 if let error = error {
                 }
                 else {
                 //(1) 카카오 앱으로 로그인 성공
                 
                 }
             }
         }
         else {
            //(2) 카카오 계정으로 로그인
            UserApi.shared.loginWithKakaoAccount(prompts: nil, channelPublicIds: nil, serviceTerms: nil) { (oauthToken, error) in
             if let error = error {
                 print(error)
             }
             else {
                 print("Login Kakao account Success.")
                 //카카오 계정으로 로그인 성공
             }
         }
         }
```

기존 메뉴얼 대로의 코드에서 실질적으로 사용 될 수 있도록 변경했습니다.

카카오톡 앱으로 로그인이 불가능한 경우에는 카카오 계정으로 로그인 하는 방법을 시도하도록 작업한 코드입니다.

공식 페이지의 샘플 코드의 경우 2.8.5 기준으로 최신버전이 적용되어 있지 않은듯 했습니다.

prompts, channelPublicIds,serviceTerms 등에 어떤 정보를 넣어야 하는지 궁금해서 찾아봐도 작업된 블로그 들도 없어 당황하던 중에 일단 nil을 넣어도 실행에 문제가 없어 nil을 넣어 처리했습니다.

마지막으로 info.plist의 CFBundleURLTypes에 kakao url scheme을 등록해야하고(kakao$(kakaoAppkey))

LSApplicationQueriesSchemes에도 kakaokompassauth,kakaolink 을 추가해야 합니다.

이 방법까지 해준다면 카카오톡에서 성공적으로 로그인에 쓰이는 AccessToken이나 프로필 등의 정보를 받아 올 수 있습니다.
