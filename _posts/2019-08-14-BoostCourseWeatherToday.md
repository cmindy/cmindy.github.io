---
layout: post
title:  "[iOS] 기상정보 애플리케이션"
subtitle:   "부스트코스 iOS 프로젝트 3 기상정보 애플리케이션"
categories: ios
tags: ios boostcourse retrospective
comments: true
---



나는 부스트코스 에이스 2019 1기로 활동중이다. 이 글은 그 중에서 iOS의 세번째 프로젝트인 기상정보 애플리케이션에 대한 회고라고 쓰고 거의 질문과 리뷰어님의 답에 대한 정리이다 😅



### Q. prepare(for:) vs didSelectRowAt

프로젝트의 요구 사항에 셀을 클릭하면 다음 뷰 컨트롤러로 정보를 넘겨주면서 화면을 띄워주는 사항이 있었다.

강의에서는 `prepare(for:)` 를 사용해 값을 전달해주는 방법을 알려주었는데, 

해당 요구사항을 봤을 때,  tableView의 `didSelectRowAt` 메서드를 사용해서도 같은 결과물을 만들 수 있었다. 

그래서 두 방법이 어떤 차이점이 있는지 질문을 드렸다.

#### A. 두 방법 모두 사용 가능합니다

리뷰어님께서는 그때 그때 적용이 편한 쪽을 고르시고 계시다고 하셨는데, 

-  `prepare(for:)`:  보통 스토리 보드를 이용해 빠르게 화면 설계 후 다음 화면에 전달할 화면을 segue를 활용해 구현한 경우 
- `didSelectRowAt`: 프로젝트가 견고해지고 storyboard의 UI 설계보다 세부적인 화면 설계를 코딩으로 구현해 storyboard를 사용하고 있지 않은 경우



### Q. prepare(for: )에서 값 전달

강의대로 하면 `prepare(for:) `에서 segue의 destination과 cell을 타입캐스팅을 해서 값을 전달하는데, 이렇게 하면
`weatherViewController.navigationItem.title = cell.countryNameLabel.text`이런 식으로 cell의 label의 text에 접근하게 된다. 
값을 전달할 때 이런식으로 UI에 표시된 값을 가져와서 전달하는 것이 이상하다는 생각이 들었다.
그래서 나는 `WeatherViewController`의 ` prepare(for:)`에서는

```swift
guard let detailViewController = segue.destination as? DetailViewController,
         let index = tableView.indexPathForSelectedRow?.row else {
            return
         }
detailViewController.city = cities[index]
```
이런 식으로 배열과 indexpath를 이용해 구현을 하고 리뷰어님께 질문을 남겼다.

#### A. 어떤 방법이든 선택된 정확한 데이터 모델을 찾아 다음 화면으로 전달하는 목표를 이룰 수만 있다면 상관 없습니다

대신, 화면에서 선택한 **리스트의 정렬 기준과 데이터 목록이 정확히 일치하는 경우**에는 index로 참조하는게 훨씬 효율적일 수도 있다. 예를 들어, 셀 내부의 문자열 값보다 데이터 목록에서 가져온 데이터 모델 객체가 필요한 경우가 있다. 

셀을 선택하면 선택한 데이터 index key로 db에 저장하거나 네트워크 통신을 하거나 등등 다양한 작업을 해야하기 때문에 실제 현업에서도 index로 구현하는 경우가 대부분이라고 한다.



### Q. cell 객체 생성시 캐스팅 방법 `as?` vs `as!`


 tableView cell을 생성할 때 보통

```swift
let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseID, for: indexPath) as! CustomTableViewCell
```
 처럼 `as!` 강제 캐스팅을 하는지

```swift
guard let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseID, for: indexPath) as? CustomTableViewCell else {
    return .init()
}
```

 처럼 `as?` 를 이용해 cell을 생성하는지 궁금했다.
나는 `as!` 보다는 `as?` 가 안전하다고 생각해서 `as?`를 주로 사용해왔는데 현업에서는 어떤 방법을 많이 사용하는지 궁금했다.

#### A. `as?`를 사용합니다

앱 크래시는 사용자 UX 경험에 최악의 시나리오이다.

개발을 하다보면 `as!` 로 작성했을 때 이상 없던 코드가 코드 변경에 의해 크래시를 유발하는 코드로 돌아올 수 있다.

iOS 개발자는 빠르게 배포 등록을 하더라도 애플 심사 때문에 사용자에게 도달하는데까지 며칠간의 시간이 소요된다.

배포를 하더라도 사용자가 업데이트를 하지 않으면 그 사람은 계속 크래시 나는 앱을 사용해야 한다.

앱을 한번 설치하고 업데이트를 안하는 사람도 꽤 많기 떄문에... 가능하면 최대한 안전하게 타입 캐스팅하도록 코드를 작성하는 것이 좋다.



### Q. 셀 내부 관리(?)

두번째 화면의 테이블 뷰의 셀에서 설정해야할 정보와 로직이 많아서 코드가 길어지고 나중에 변경에 대비하기 어려울 것 같아 방법을 찾다가 ViewModel을 이용해서 cell을 설정했다.

현업에서는 한 셀에 레이블 색이나 이미지, 로직 등등 설정해야할 것이 더 많을 것 같은데, 어떤 방식으로 효율적으로 이 코드들을 관리하는지 궁금했다.

#### 여러 디자인 패턴을 사용합니다

뷰는 자주 바뀔 수 있는 경우 custom View로 분리한다. 

그리고 로직 부분은 초기 개발 시점에는 MVC 모델로도 간단히 해결되는데, ViewController에 화면을 그려주는 코드가 비슷한 로직이 생기고 점점 거대해지게 되기때문에 별도로 분리할 필요성이 생긴다. (Massive View Controller를 말씀하시는 듯!)

그래서 현업에서는 viewModel 패턴을 활용하기도 하고 VIPER 패턴으로 분리해서 구현하기도 한다.

디자인 패턴을 정하는 이유는 여러 사람이 구현할 때 어느정도 규칙을 통일하기 위해서이기도 하다.

우리가 원하는 목표인 코드 복잡도를 낮추고 반복되는 코드를 줄이고 성격별로 나누는 리팩토링을 한다.



### Q. 모델과 뷰의 역할에 따른 코드 분리

강의에서는 `"섭씨: \(celsius) / 화씨: \(fahrenheit)"` 처럼 뷰에 보여지는 텍스트들을 모델단에서 생성을 했다. 

이렇게 하면 모델에서 뷰에 보여지는 값들을 아는 것이라 역할이 제대로 나눠지지 못한 것 같다는 생각이 들었다. 

출력되는 문자열 같은 것은 변경의 가능성이 많기 때문에 모델에서는 최대한 celsius나 fahrenheit 같은 기본적인(?) 값을 갖고 있고 뷰나 뷰모델에서 이 값들을 조합하는 것이 맞다는 생각을 했다.

#### A. 코드를 역할에 따라 분리해야합니다

특정 view에 보여지는 커스텀한 UI 데이터는 해당 view를 위한 viewModel을 만들어서 처리해주는것이 변경이 용이한 코드이다.



### Q. Model에서 변수의 옵셔널 처리

`let name: String?` 이런식으로 옵셔널 처리를 하는 경우는 언제인지 궁금했다.

 이 프로젝트에서는 json파일에 모든 값이 확실하게 있어서 옵셔널로 하지는 않았다.

그런데 실제 API에서 받아올 때는 값이 있을 수도 있고 없을 수도 있을 때 옵셔널 처리를 하는지 아니면 안정성이나 변경등을 위해서 필수적으로 들어오는 값일 때도 옵셔널로 하는지 궁금했다.

#### A. 옵서널 또는 기본값을 사용합니다.

이 프로젝트와 같은 스펙이면 옵셔널을 사용할 필요가 없다.

그렇지만 현업에서는 서버에서 전달해주는 API 결과 모델이 일부 바뀌거나 없는 상황이 생긴다면 옵셔널로 선언해주거나 아니면 옵셔널 대신 기본값을 넣는다.



이번 프로젝트를 하면서 전체적으로는 아니지만 viewModel을 처음 적용해봤는데 코드가 훨씬 깔끔해지고 역할이 명확해지는 것 같아서 좋았다. view model과 다른 디자인 패턴들에 대해 좀 더 공부해봐야겠다는 생각이 들었다... 실제로는 어떻게 사용하는지 너무 궁금한것,,ㅜ

