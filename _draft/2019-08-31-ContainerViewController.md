---
layout: post
title:  "[iOS] Container ViewController"
subtitle:   "Container ViewController"
categories: ios
tags: ios swift
comments: true
---

# Container ViewController

- 컨테이너 뷰 컨트롤러는 여러 뷰 컨트롤러들의 컨텐츠를 묶어서 하나의 UI로 보여주는 방법이다.
- 다른 뷰컨트롤러들을 포함하는 뷰컨트롤러
  UITabBarController, UINavigationController, UISplitViewController 등이 대표적이다.
- 컨테이너 뷰 컨트롤러도 컨텐트 뷰 컨트롤러와 같이 루트 뷰와 컨텐츠를 관리하는 방법은 같다. 대신 컨테이너 뷰 컨트롤러는 그 **컨텐츠를 다른 뷰 컨트롤러에게서 받아와 보여준다**는 것이다.
- 컨테이너 뷰 컨트롤러를 만들 때는 항상 **자식 뷰 컨트롤러와의 관계를 염두**에 두어야 한다.



## Navigation Controller

[![VCPG_structure-of-navigation-interface_5-1_2x](https://github.com/cmindy/TIL/raw/master/iOS/assets/VCPG_structure-of-navigation-interface_5-1_2x.png)](https://github.com/cmindy/TIL/blob/master/iOS/assets/VCPG_structure-of-navigation-interface_5-1_2x.png)

- 내비게이션 스택(navigation stack)**을 사용하여 다른 뷰 컨트롤러를 관리한다.

- 컨텐트 뷰 컨트롤러(content view controller)

  - 내비게이션 스택(navigation stack)에 담겨서 콘텐츠를 보여주게 되는 뷰 컨트롤러들

- 내비게이션 컨트롤러는 두 개의 뷰를 화면에 표시한다.

  - 내비게이션 스택뷰에 포함된 최상위 컨텐트 뷰 컨트롤러의 콘텐츠를 나타내는 뷰
  - 내비게이션 컨트롤러가 직접 관리하는 뷰(내비게이션바 또는 툴바)
  - 한 번에 하나의 자식 뷰 컨트롤러를 보여준다.

- 내비게이션

   

  델리게이트

   

  객체 사용

  - 인터페이스의 변화에 따른 특정 액션을 동작하도록 한다.

 

#### 네비게이션 컨트롤러가 표시하는 뷰

[![68_1](https://github.com/cmindy/TIL/raw/master/iOS/assets/68_1.png)](https://github.com/cmindy/TIL/blob/master/iOS/assets/68_1.png)

 

### 화면 전환

**네비게이션 스택에 뷰 컨트롤러를 추가/삭제** 해서 화면을 전환할 수 있다.

- `UINavigationController` 클래스의 메서드, `segue`를 이용한다.

### 네비게이션 스택 (Navigation Stack)

- **뷰 컨트롤러를 담을 수 있는 배열**과 같다.
- 네비게이션 컨트롤러에 의해 관리된다.
- **push**: 새로운 `UIViewController` 인스턴스를 생성해 네비게이션 스택에 추가한다.
- **pop**: UIViewController` 의 인스턴스가 다른 곳에서 참조되고 있지 않다면 메모리에서 해제하고 네비게이션 스택에서 제거한다.
- Root VC: 네비게이션 스택의 가장 하위에 있는(가장 먼저 스택에 추가된) 뷰 컨트롤러
  - 루트 뷰 컨트롤러는 내비게이션 스택에서 팝(pop)되지 않는다.
- Top VC: 네비게이션 스택의 가장 상위에 있는(가장 마지막에 푸시(push) 된) 뷰 컨트롤러
  - 화면에 보여지는 뷰

[![image-20190723223637592](https://github.com/cmindy/TIL/raw/master/iOS/assets/image-20190723223637592.png)](https://github.com/cmindy/TIL/blob/master/iOS/assets/image-20190723223637592.png)

 

### 네비게이션바

- 네비게이션바는 네비게이션 컨트롤러에 의해 생성된다.
- 네비게이션바는 네비게이션 컨트롤러의 관리를 받는 모든 뷰 컨트롤러의 상단에 표시된다.
- 현재 데이터 계층에서 어느 위치에 있는지 보여준다.
- Top 뷰 컨트롤러가 변경될 때마다 네비게이션 컨트롤러는 네비게이션바를 업데이트 한다.

## Custom Container View Controller

### 스토리보드에서 추가

[![container_view_embed_2x](https://github.com/cmindy/TIL/raw/master/iOS/assets/container_view_embed_2x.png)](https://github.com/cmindy/TIL/blob/master/iOS/assets/container_view_embed_2x.png)

인터페이스 빌더가 자동으로 부모-자식 관계를 설정해준다

### 프로그래밍 방식으로 추가

#### Adding a Child View Controller to Your Content

자식 뷰 컨트롤러를 프로그래밍 방식으로 컨텐츠에 통합하려면, 다음 단계를 통해 관련 뷰컨트롤러들 간에 부모-자식 관계를 만든다.

1. 컨테이너 뷰컨트롤러에서 `addChild(_:)` 를 호출한다.

   이 메소드는 UIKit에게 내 컨테이너 뷰컨트롤러가 이제 자식 뷰컨트롤러들을 관리할거야! 라는 것을 말해준다.

2. 자식의 루트 뷰를 컨테이너 뷰 계층에 추가한다.

   이 과정에서 자식 뷰의 프레임의 크기와 위치를 항상 설정해야한다.

3. 자식의 루트뷰의 크기와 사이즈를 관리하는 제약을 추가한다.

4. 자식 뷰컨트롤러의 `didMove(toParent:)` 메소드를 호출한다.

```swift
func displayContentController(_ content: UIViewController?) {
    if let content = content {
        addChild(content)
    }
    content?.view.frame = frameForContentController()
    view.addSubview(currentClientView)
    content?.didMove(toParent: self)
}
```

예제에서 자식의 `didMove(toParent:) `메서드만 호출한다. 이는  `addChild(_:)` 메서드가 자식의 `willMove(toParent:)` 메서드를 호출하기 때문이다.

`didMove(toParent:)` 메서드를 직접 호출해야하는 이유는 자식 뷰를 컨테이너의 뷰 계층 구조에 포함해야만 메서드를 호출 할 수 있기 때문이다.

오토 레이아웃을 사용하는 경우 **자식 뷰 컨트롤러를 컨테이너의 뷰 계층에 추가 한 후 컨테이너와 자식 사이에 제약 조건을 설정**해야한다. 이 제약 조건은 자식의 루트뷰의 크기와 위치에만 영향을 주어야한다. 자식 뷰 계층에서 루트뷰 또는 다른뷰의 내용을 변경하면 안 된다.

#### Removing a Child View Controller

자식 뷰 컨트롤러를 제거하면 부모-자식 관계는 영구적으로 끊어지기 때문에 더 이상 자식 뷰 컨트롤러를 참조할 필요가 없을 때 제거해야 한다.

부모-자식 관계를 끊을 때는 다음 단계를 거친다.

1. 자식의 `willMove(toParent:)` 메서드를 `nil` 값과 함께 호출한다.

2. 자식의 루트뷰에 설정된 제약들을 모두 제거한다.

3. 컨테이너의 뷰 계층에서 자식의 루트 뷰를 제거한다.

4. Call the child’s `removeFromParentViewController` method to finalize the end of the parent-child relationship.

   자식의 `removeFromParent()` 메서드를 호출해 부모-자식 관계를 끝낸다.

`willMove(toParent:)` 메서드를  `nil` 값과 함께 호출해서 자식 뷰 컨트롤러에게 변화를 준비할 기회를 준다. `removeFromParent()` 메서드 또한 자식의 `didMove(toParent:)` 메서드를 호출해 해당 메서드에 `nil` 값을 전달한다. 부모 뷰 컨트롤러를 `nil` 로 설정하면 컨테이너에서 자식 뷰의 제거가 완료된다.

```swift
func hideContentController(_ content: UIViewController?) {
    content?.willMove(toParent: nil)
    content?.view.removeFromSuperview()
    content?.removeFromParent()
}
```

#### Suggestions for Building a Container View Controller

Designing, developing, and testing a new container view controller takes time. Although the individual behaviors are straightforward, the controller as a whole can be quite complex. Consider the following tips when implementing your own container classes:

- 자식 뷰 컨트롤러의 루트뷰에만 접근해한다.
  - 자식의 `view` 프로퍼티를 통해 리턴되는 루트뷰에만 접근을 해야한다.
- 자식 뷰 컨트롤러는 자신이 속한 컨테이너에 대해 최소한의 정보만 알고 있어야 한다.
  - 자식 뷰 컨트롤러는 자신의 콘텐츠에 집중해야한다.
  - 둘의 상호작용이 필요하다면 delegation 디자인 패턴을 사용한다.
- 첫번째로 일반적인 뷰를 이용해 컨테이너를 디자인한다.
  - 자식 뷰 컨트롤러의 뷰 대신 일반적인 뷰를 사용하면 단순한 환경에서 레이아웃 제약과 애니메이션 트랜지션을 테스트 할 수 있다.

#### Delegating Control to a Child View Controller

A container view controller can delegate some aspects of its own appearance to one or more of its children. You can delegate control in the following ways:

컨테이너 뷰 컨트롤러는 자신의 모습 중 일부를 자식에게 위임할 수 있다.

- 자식 뷰 컨트롤러가 상태 바 스타일을 결정하게 한다.
  - 컨테이너 뷰 컨트롤러 안에서 `childViewControllerForStatusBarStyle` , `childViewControllerForStatusBarHidden` 메서드를 오버라이드 한다.
- 자식이 자신의 선호 사이즈를 정할 수 있다.
  - 유연한 레이아웃을 가진 컨테이너는 자식의 `preferredContentSize` 속성을 사용하여 자식의 크기를 결정할 수 있다.