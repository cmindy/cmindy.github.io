---
layout: post
title:  "[iOS] Main Run Loop"
subtitle:   ""
categories: ios
tags: ios
comments: true

---



#### The Main Function

- C 기반의 모든 앱의 시작점은 `main`. iOS도 마찬가지로 시작점은 `main` 이다.

- But, iOS에서는 `main` 을 작성하지 않아도 XCode가 기본 프로젝트의 일부로 이 함수를 생성한다.

- ```objective-c
  #import <UIKit/UIKit.h>
  #import "AppDelegate.h"
  int main(int argc, char * argv[])
  {
      @autoreleasepool {
          return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate
  class]));
  } }
  ```

- `main` 함수는 UIKit 프레임워크에 컨트롤을 넘긴다.

- `UIApplicationMain`

  - 앱의 핵심 객체 생성
  - 스토리보드에서 앱의 UI를 로드
  - 사용자 지정 코드를 호출해 초기 설정
  - 앱의 런 루프를 실행해 이 프로세스를 처리

#### The Structure of an App

- 앱 시작시  `UIApplicationMain` 함수는 핵심 객체들을 생성하고 앱을 실행한다.
- 모든 iOS 앱의 핵심은 `UIApplication` 객체이다.
  - 시스템과 앱 내의 다른 객체들 간의 상호작용을 원활하게 해 준다.
  - 이벤트 루프와 다른 높은 수준의 앱의 동작을 관리한다.

[![image-20190926183245333](https://camo.githubusercontent.com/82197afbf2c4949a5664c770683e88717711a571/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303679386d4e3667793167376431773678696e756a333133333075306167662e6a7067)](https://camo.githubusercontent.com/82197afbf2c4949a5664c770683e88717711a571/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303679386d4e3667793167376431773678696e756a333133333075306167662e6a7067)

#### The Main Run Loop

- 앱의 main run loop는 유저와 관련된 모든 이벤트를 처리한다.
- `UIApplication` 객체는 앱 시작 시, 메인 런 루프를 설정한다. 이를 이벤트 처리와 뷰 기반 인터페이스 갱신에 사용한다.
- 앱의 메인 스레드에서 동작한다.
- 관련 이벤트가 들어온 순서대로 순차적으로 처리된다.

[![image-20190926183618302](https://camo.githubusercontent.com/1bb98c54266789952c34cdfc26bafa56285554bd/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303679386d4e36677931673764323031657634706a3331616e307530307a6d2e6a7067)](https://camo.githubusercontent.com/1bb98c54266789952c34cdfc26bafa56285554bd/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303679386d4e36677931673764323031657634706a3331616e307530307a6d2e6a7067)

- 유저 상호 작용과 관련된 이벤트가 OS에 의해 생성된다.
- UIKit에 의해 설정된 특별한 포트를 통해 앱에 전달된다.
- 앱 안의 이벤트 큐에 이벤트들이 넣어진다.
- 실행을 위해 메인 런 루프에 넣어진다.
- `UIApplication` 객체는 이벤트를 받아서 무엇이 행해져야 하는지 결정을 내리는 첫번째 객체이다.