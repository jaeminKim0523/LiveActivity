# LiveActivityWidgetExtension
### 목차
1. 위젯 추가
2. 위젯 초기 작업


## 위젯 추가
<img width="160" alt="스크린샷 2025-03-15 오전 8 38 44" src="https://github.com/user-attachments/assets/31660501-8c12-422f-9afc-ebe94a489214" /><br/>
프로젝트 - Targets 하단의 + 버튼을 누른다.<br/><br/>

<img width="722" alt="스크린샷 2025-03-15 오전 8 39 16" src="https://github.com/user-attachments/assets/9e3a312e-912c-479f-8807-4a5d2e7b3ac8" /><br/>
우리가 추가해야 하는 것은 Widget Extension이다.  
해당 Extension을 선택한다.<br/><br/>

![스크린샷 2025-03-15 오전 8 52 31](https://github.com/user-attachments/assets/41fb89bc-ea08-4a05-a67a-675afdf2fd34)<br/>
우리는 LiveActivity만을 사용 할 것이기 때문에 Include Live Activity만 선택해서 마무리한다.<br/><br/>

![스크린샷 2025-03-15 오전 8 59 30](https://github.com/user-attachments/assets/cc7b6a31-87f3-4fe0-9b50-836b585f40a8)<br/>
프로젝트 파일 목차에 해당 위젯 폴더들이 생성돼있는것을 볼 수 있다.<br/>

기본적으로 생성되는 파일은
- xxxx + Widget
- xxxx + WidgetBundle
- xxxx + WidgetLiveActivity
이렇게 있다면, 정상적으로 위젯은 추가가 된 것이다.

## 위젯 초기 작업
### WidgetLiveActivity와 App을 연결
우리는 보통 LiveActivity를 앱에서 특정 상황에 실행되게 하기 때문에, 앱에서 위젯 실행을 위해 앱과 위젯을 연결 지어줘야 한다.<br/><br/>
![스크린샷 2025-03-15 오전 9 08 43](https://github.com/user-attachments/assets/9e4bb61d-1918-4e45-98d5-7716e2aed102)<br/>
xxxx + WidgetLiveActivity의 우측에 있는 Inspector를 보면, Target Membership이 있다.<br/>
여기에 + 버튼을 눌러서 아래와 같이 체크를 해주자.<br/>
![스크린샷 2025-03-15 오전 9 10 21](https://github.com/user-attachments/assets/335aa65b-8cab-402a-9f1f-8af768248a55)<br/><br/>
![스크린샷 2025-03-15 오전 9 10 48](https://github.com/user-attachments/assets/16ec4efa-e5e1-4f44-8801-493e5d0a4ac9)<br/>
위 스크린샷과 같이 앱 프로젝트와 연결을 해준다.<br/>

### 메인 프로젝트의 info.plist설정
![스크린샷 2025-03-15 오전 9 14 42](https://github.com/user-attachments/assets/1c607b59-ca09-44eb-b6cf-3ebb0319bddd)<br/>
info.plist에 추가 할 수 있는 LiveActivity의 두가지 항목이 있다.<br/>
- Supports Live Activities: 우리가 추가 할 LiveActivity를 지원 할 지 여부
- Supports Live Activities Frequent Updates: LiveActivity의 업데이트 빈도가 잦을 때

원하는 항목들을 추가 후 값을 YES로 변경해주면 된다.

## 위젯 테스트 코드
LiveActivity.swift 파일의 내용에 다들 비슷하게 추가되었을 것이다.
```
struct KakaoVXBookingLiveActivityAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        // Dynamic stateful properties about your activity go here!
        var text: String
    }

    // Fixed non-changing properties about your activity go here!
    var name: String
}

struct KakaoVXBookingLiveActivityLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: KakaoVXBookingLiveActivityAttributes.self) { context in
            // Lock screen/banner UI goes here
            LiveActivityView(state: context.state)

        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded UI goes here.  Compose the expanded UI through
                // various regions, like leading/trailing/center/bottom
                DynamicIslandExpandedRegion(.leading) {
                    Text("Leading \(context.state.text)")
                }
                DynamicIslandExpandedRegion(.trailing) {
                    Text("Trailing \(context.state.text)")
                }
                DynamicIslandExpandedRegion(.bottom) {
                    Text("Bottom \(context.state.text)")
                    // more content
                }
            } compactLeading: {
                Text("L \(context.state.text)")
            } compactTrailing: {
                Text("T \(context.state.text)")
            } minimal: {
                Text("M \(context.state.text)")
            }
            .widgetURL(URL(string: "http://www.apple.com"))
            .keylineTint(Color.red)
        }
    }
}
```

## 위젯 실행
```
guard ActivityAuthorizationInfo().areActivitiesEnabled else { return }

let initialContentState = LiveActivityAttributes.ContentState(text: "Temp")
let activityAttributes = LiveActivityAttributes(name: "Francesco")

do {
    
    let activity = try Activity.request(attributes: activityAttributes,
                                        contentState: initialContentState,
                                        pushType: .token)
    
    Task {
        for await pushToken in activity.pushTokenUpdates {
            let pushTokenString = pushToken.reduce("") {
                $0 + String(format: "%02x", $1)
            }
            // pushTokenString - 해당 토큰으로 푸시 메세지 전달
        }
    }
} catch (let error) {
    print("Error requesting Lockscreen Live Activity.")
}
```
위젯이 실행 될 위치에 해당 코드를 넣어주면 된다.
