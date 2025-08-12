# 의도

**어댑터 패턴**의 의도는 아주 명확하다. **서로 호환되지 않는 인터페이스를 가진 클래스들이 함께 작동할 수 있도록 '중간 다리' 역할을 하는 클래스를 만드는 것**이다.

쉽게 말해, **코드를 직접 수정할 수 없는 기존 클래스(또는 외부 라이브러리)를 우리가 원하는 방식의 인터페이스로 '포장'해서 사용하는 기술**이다.

# 문제 상황 (사용하면 좋을 상황)

개발을 하다 보면 이런 상황을 자주 마주하게 된다.

- 우리가 만든 앱은 자체적으로 정의한 `NewLogging` 프로토콜에 맞춰 로그를 남기고 있다.
- 그런데 팀에서 갑자기 외부의 유명한 로깅 라이브러리인 `FantasticLogger`를 도입하기로 했다.
- 문제는 이 `FantasticLogger`의 메서드 이름과 방식이 우리가 사용하던 `NewLogging` 프로토콜과 완전히 다르다는 것이다. 예를 들어, 우리는 `log(message: String)`를 쓰는데, 라이브러리는 `recordLog(text: String, timestamp: Date)`를 사용한다.

이때, 우리 앱의 모든 로그 코드를 `FantasticLogger`에 맞게 전부 수정해야 할까? 그렇게 하면 나중에 다른 로깅 라이브러리로 교체할 때 또 모든 코드를 수정해야 하는 끔찍한 일이 반복될 것이다.

# 해결책

이때 **어댑터(Adapter)**를 도입한다.

1. `NewLogging` 프로토콜을 따르는 **`LoggingAdapter`**라는 새로운 클래스를 만든다.
2. `LoggingAdapter`는 내부에 `FantasticLogger`의 인스턴스를 가지고 있다.
3. `LoggingAdapter`의 `log(message: String)` 메서드가 호출되면, 내부적으로 `FantasticLogger`가 알아들을 수 있는 `recordLog(text: message, timestamp: Date())` 형식으로 **변환해서 대신 호출**해준다.

이제 우리 앱은 `FantasticLogger`의 존재를 전혀 알 필요 없이, 예전처럼 `NewLogging` 프로토콜에 맞춰 로그를 남기기만 하면 된다. 모든 변환 작업은 어댑터가 알아서 처리해준다.

# 구조

어댑터 패턴은 주로 4개의 참여자로 구성된다.

1. **클라이언트 (Client)**: 어댑터를 사용하는 객체이다. 클라이언트는 **타겟 인터페이스**에 맞춰서만 작동한다.
2. **타겟 인터페이스 (Target Interface)**: 클라이언트가 사용하고자 하는 목표 인터페이스이다. (우리 예시에서는 `NewLogging` 프로토콜이 이에 해당한다.)
3. **어댑터 (Adapter)**: **타겟 인터페이스**를 구현하고, **어댑티(Adaptee)**의 인스턴스를 감싸고 있다. 클라이언트의 요청을 어댑티가 이해할 수 있는 형태로 변환하여 전달하는 핵심적인 역할을 한다.
4. **어댑티 (Adaptee) 또는 서비스(Service)**: 기존에 존재하지만 호환되지 않는 인터페이스를 가진 클래스이다. 우리 예시에서는 외부 라이브러리인 `FantasticLogger`가 해당된다.

<img width="80%" alt="image" src="https://github.com/user-attachments/assets/7d14ef68-6496-49b2-93cb-26ae98e4c618" />

# 수도코드

```swift
// 2. Target Interface: 우리 앱이 기대하는 인터페이스
protocol NewLogging {
    func log(message: String)
}

// 4. Adaptee: 외부 라이브러리 (우리가 수정할 수 없음)
class FantasticLogger {
    func recordLog(text: String, timestamp: Date) {
        print("[FantasticLogger] \\(timestamp): \\(text)")
    }
}

// 3. Adapter: 중간 변환기
class LoggingAdapter: NewLogging {
    // 내부에 Adaptee 인스턴스를 가지고 있다.
    private let adaptee: FantasticLogger

    init(adaptee: FantasticLogger) {
        self.adaptee = adaptee
    }

    // Target Interface의 메서드를 구현한다.
    func log(message: String) {
        // 클라이언트의 요청을 Adaptee가 알아들을 수 있는 형태로 변환하여 호출한다.
        adaptee.recordLog(text: message, timestamp: Date())
    }
}

// 1. Client: 어댑터를 사용하는 코드
class App {
    let logger: NewLogging // Client는 오직 Target Interface만 알고 있다.

    init(logger: NewLogging) {
        self.logger = logger
    }

    func start() {
        logger.log(message: "앱이 시작되었습니다.")
    }
}

// --- 실행 ---
let fantasticLogger = FantasticLogger()
let adapter = LoggingAdapter(adaptee: fantasticLogger)
let app = App(logger: adapter)
app.start() // 출력: [FantasticLogger] 2025-08-12 11:11:00 +0000: 앱이 시작되었습니다.
```

# 적용 가능성

아래와 같은 경우에 어댑터 패턴을 떠올리면 좋다.

- **기존 클래스를 사용해야 하지만 인터페이스가 맞지 않을 때**: 가장 전형적인 사용 사례이다.
- **재사용 가능한 클래스를 만들고 싶은데, 미래에 호환되지 않을 수 있는 클래스들과의 관계를 미리 차단하고 싶을 때**: 우리 코드가 특정 라이브러리에 직접적으로 종속되는 것을 막아준다.
- **여러 서브클래스를 만들어야 하지만, 각 서브클래스에서 약간씩 다른 외부 기능을 재사용해야 할 때**: 각 서브클래스마다 다른 어댑터를 사용하여 동일한 부모 클래스의 로직을 재사용할 수 있다.

# 구현 방법

- 클라이언트가 필요로 하는 **타겟 인터페이스(프로토콜)**가 존재하는지 확인한다. 없다면 새로 정의한다.
- **어댑터 클래스**를 만들고, 이 클래스가 타겟 인터페이스를 준수하도록 한다.
- 어댑터 클래스 내부에 **어댑티 객체**를 저장할 필드(프로퍼티)를 만든다. 보통 생성자를 통해 외부에서 주입받는다.
- 타겟 인터페이스의 모든 메서드를 어댑터 클래스에서 구현한다. 이 메서드 내부에서, 어댑티 객체의 메서드를 호출하도록 **요청을 변환**하는 로직을 작성한다.
- 클라이언트는 이제 타겟 인터페이스를 통해 어댑터 객체를 사용한다.

# 장단점

- **장점 (Pros)**
    - **단일 책임 원칙 (SRP)**: 클라이언트의 핵심 로직과 인터페이스 변환 로직을 분리할 수 있다.
    - **개방-폐쇄 원칙 (OCP)**: 기존 클라이언트 코드를 전혀 수정하지 않고도 새로운 어댑터를 통해 새로운 클래스를 시스템에 통합할 수 있다.
- **단점 (Cons)**
    - **코드 복잡도 증가**: 단순히 기능을 추가하는 것이 아니라, 인터페이스와 어댑터 클래스 등 추가적인 클래스들을 만들어야 하므로 전체적인 코드 구조가 복잡해질 수 있다.

# 다른 패턴과의 관계성

- **브릿지 (Bridge)**: 브릿지는 보통 시스템을 설계하는 초기 단계부터 기능과 구현을 분리하기 위해 의도적으로 설계되지만, 어댑터는 이미 만들어진 시스템에 새로운 클래스를 통합하기 위해 나중에 추가되는 경우가 많다.
- **데코레이터 (Decorator)**: 데코레이터는 객체의 인터페이스를 유지하면서 새로운 기능을 '덧씌우는' 반면, 어댑터는 기존 객체의 인터페이스를 **다른 인터페이스로 '변환'**한다.
- **프록시 (Proxy)**: 프록시는 대상 객체와 동일한 인터페이스를 사용하며 접근을 제어하는 역할을 하지만, 어댑터는 다른 인터페이스를 제공한다.

# 예제 코드 (In Swift)

iOS 개발, 특히 SwiftUI 시대에 **어댑터 패턴**이 가장 빛을 발하는 순간 중 하나는 **UIKit의 컴포넌트를 SwiftUI에서 사용**해야 할 때이다.

```swift
import SwiftUI

// 1. Target Interface 역할을 하는 SwiftUI의 View 프로토콜과
//    이를 감싸는 UIViewControllerRepresentable 프로토콜
struct ImagePicker: UIViewControllerRepresentable {
    
    // 3. Adapter 역할: SwiftUI와 UIKit 사이를 연결하는 구조체
    
    @Binding var selectedImage: UIImage?
    @Environment(\\.presentationMode) private var presentationMode

    // Adaptee인 UIImagePickerController를 생성하고 설정
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator // Coordinator가 통신을 담당
        return picker
    }

    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {
        // SwiftUI 뷰가 업데이트될 때 UIKit 뷰 컨트롤러에 변경사항을 전달
    }
    
    // Coordinator는 UIKit의 Delegate 패턴을 SwiftUI에서 처리하기 위한 중간 다리 역할
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    final class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        var parent: ImagePicker

        init(_ parent: ImagePicker) {
            self.parent = parent
        }

        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let image = info[.originalImage] as? UIImage {
                parent.selectedImage = image
            }
            parent.presentationMode.wrappedValue.dismiss()
        }
    }
}

// --- Client View ---
struct MyProfileView: View {
    @State private var image: UIImage?
    @State private var showingImagePicker = false

    var body: some View {
        VStack {
            if let image {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
            }
            Button("프로필 사진 선택") {
                showingImagePicker = true
            }
        }
        .sheet(isPresented: $showingImagePicker) {
            // SwiftUI 뷰는 ImagePicker(어댑터)를 호출할 뿐,
            // 내부에서 UIImagePickerController가 동작하는지는 모른다.
            ImagePicker(selectedImage: $image)
        }
    }
}
```

이 코드에서 `ImagePicker` 구조체는 **어댑터**의 완벽한 예시이다. SwiftUI의 언어(`View`, `@Binding`)를 UIKit의 `UIImagePickerController`가 이해할 수 있는 방식(`delegate`, `UIViewController`)으로 변환해주고 있다.

# 현업에서의 상황

- **레거시 코드 연동**: 새로운 모듈을 개발했지만, 구형 시스템의 데이터 모델(예: `XML` 데이터)을 사용해야 할 때, 이 `XML`을 우리 시스템이 사용하는 `JSON`이나 `Codable` 객체로 변환해주는 '어댑터' 클래스를 자연스럽게 만들게 된다.
- **서드파티 SDK 연동**: 외부 결제 SDK나 지도 SDK를 앱에 통합할 때, 해당 SDK의 API를 직접 호출하는 대신 우리 앱의 인터페이스에 맞게 한 번 감싸주는 `PaymentManager`나 `MapService` 같은 클래스를 만든다. 이것이 바로 어댑터 패턴이다.
- **SwiftUI와 UIKit 연동**: 위 예시처럼, 아직 SwiftUI로 완전히 구현되지 않은 UIKit의 컴포넌트나 뷰 컨트롤러를 사용해야 할 때 `UIViewRepresentable`이나 `UIViewControllerRepresentable`을 사용하는 것은 iOS 개발자에게 거의 일상적인 어댑터 패턴의 활용이다.

> **다른 코드와 우리 코드 사이의 '경계'를 만들고, 의존성을 낮추며, 시스템을 유연하게 만들기 위한 매우 실용적이고 필수적인 기술**

# 출처

- **Refactoring Guru**: [Adapter Design Pattern](https://refactoring.guru/design-patterns/adapter)
- **Apple Developer Documentation**: [Interfacing with UIKit](https://developer.apple.com/tutorials/swiftui/interfacing-with-uikit)
