---
layout: post
title:  "[iOS]번역 Handling button tap inside UITableView Cell without using tag"
subtitle: 
categories: ios
tags: ios swift
comments: true
---

https://fluffy.es/handling-button-tap-inside-uitableviewcell-without-using-tag/

앱에서 사용자가 구독 버튼을 탭했을 때 "[유튜버 이름]을 구독했습니다"를 뜨게하고 싶다. 이 기능은 어떻게 구현해야할까? 각 셀이 탭되는 것을 계속 추적하고 인덱스를 사용해 배열에서 유튜버의 이름을 가져와야 할 것이다. 

구글에서 'button click in uitableviewcell'의 [top result](https://stackoverflow.com/questions/20655060/get-button-click-inside-uitableviewcell) 를 확인해보자

답변은 버튼의 태그 속성을 사용하라고 추천한다.

```swift
let youtubers = ["Brian Voong", "Seth Everman", "Dave Lee", "Cybershell", "Bill Wurtz"]

func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
  let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseIdentifier, for: indexPath) as! YoutuberTableViewCell
  cell.youtuberLabel.text = youtubers[indexPath.row]
  
  // assign the index of the youtuber to button tag	
  cell.subscribeButton.tag = indexPath.row
  
  // call the subscribeTapped method when tapped
  cell.subscribeButton.addTarget(self, action: #selector(subscribeTapped(_:)), for: .touchUpInside)
        
  return cell
}

@objc func subscribeTapped(_ sender: UIButton){
  // use the tag of button as index
  let youtuber = youtubers[sender.tag]
  let alert = UIAlertController(title: "Subscribed!", message: "Subscribed to \(youtuber)", preferredStyle: .alert)
  let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
  alert.addAction(okAction)
        
  self.present(alert, animated: true, completion: nil)
}
```



## tag를 사용하면 안되나요?

[Tag property](https://developer.apple.com/documentation/uikit/uiview/1622493-tag)는 원래 앱에서 뷰를 고유하게 식별하기 위해 만들어졌다.(뷰를 식별하기 위해IBOutlet을 만드는 것과 비슷하다. 예: self.view.viewWithTag(42)) 데이터를 저장하기 위해 사용되는 것이 아니다. (이 경우에서는, item의 row / index 를 저장한다.) 잘못된 tag의 사용은 multiple section이 있는 경우에 악몽으로 이어진다😱

```swift
cell.likeButton.tag = indexPath.row + 100
...
// I have seen production code which uses tag 101, 102, 103 etc as row in first section; 201, 202, 203 etc as row in second section..
self.tableView.scrollToRow(at: IndexPath(row: sender.tag, section: sender.tag / 100), at: .top, animated: true)

```

**/100** 이 뭘 의미하는 지 어떻게 알 수 있습니까!? 이런 수학적 방법보다 좀 더 직관적인 방법을 사용할 순 없을까? 한 섹션에 100개 이상의 row 가 있으면? 1000을 사용하나?? 



스택 오버플로우의 포스트는 섹션이 여러개일 때 테이블 뷰를 다루는  [다른 post](https://stackoverflow.com/questions/31649220/detect-button-click-in-table-view-ios-xcode-for-multiple-row-and-section)의 링크가 걸려있다. 인덱스를 알아내기 위해 터치된 셀의 좌표를 감지하는 것과 관련되어 있다..대체왜..😱



인덱스 데이터를 전달하는 것은 그렇게 어렵지 않다! 이를 위해 여러가지 방법이 있다. 이 포스트는 delegate와 closure를 사용해 테이블 뷰 셀 안의 버튼을 클릭하는 방법을 알아볼 것이다.



## 델리게이트를 사용한 방법

델리게이트와 친숙하지 않다면, [how delegate works here](https://fluffy.es/eli-5-delegate/)을 여기서 읽을 수 있다. 델리게이트 방법을 사용하려면 **index** 프로퍼티 (인덱스를 계속 추적하기 위해) 와 **delegate** 프로퍼티를 셀 클래스에 추가해야한다.

```swift
//YoutuberTableViewCell.swift

class YoutuberTableViewCell: UITableViewCell {

  @IBOutlet weak var youtuberLabel: UILabel!
    
  @IBOutlet weak var subscribeButton: UIButton!
    
  // the youtuber (Model), you can use your custom model class here
  var youtuber : String?
    
  // the delegate, remember to set to weak to prevent cycles
  weak var delegate : YoutuberTableViewCellDelegate?
    
  override func awakeFromNib() {
    super.awakeFromNib()
    // Initialization code
        
    // Add action to perform when the button is tapped
    self.subscribeButton.addTarget(self, action: #selector(subscribeButtonTapped(_:)), for: .touchUpInside)
  }

  override func setSelected(_ selected: Bool, animated: Bool) {
    super.setSelected(selected, animated: animated)

    // Configure the view for the selected state
  }
    
  @IBAction func subscribeButtonTapped(_ sender: UIButton){
    // ask the delegate (in most case, its the view controller) to 
    // call the function 'subscribeButtonTappedFor' on itself.
    if let youtuber = youtuber,
       let delegate = delegate {
        self.delegate?.youtuberTableViewCell(self, subscribeButtonTappedFor: youtuber)
    }
  }
    
}

// Only class object can conform to this protocol (struct/enum can't)
protocol YoutuberTableViewCellDelegate: AnyObject {
  func youtuberTableViewCell(_ youtuberTableViewCell: YoutuberTableViewCell, subscribeButtonTappedFor youtuber: String)
}
```



`delegate` 는 `YoutuverTableViewCellDelegate`프로토콜을 준수하는 any object 이다. 이는 객체가  `subscribeButtonTappedFor` 함수를 구현해야 델리게이트가 될 수 있다.



테이블 뷰 데이터 소스에서, 우리는 인덱스와 델리게이트 프로퍼티를 `cellForRowAt` 메서드에서 할당해줄 것이다.

```swift
extension ViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseIdentifier, for: indexPath) as! YoutuberTableViewCell
    cell.youtuberLabel.text = youtubers[indexPath.row]
    
    // assign the youtuber model to the cell
    cell.youtuber = youtubers[indexPath.row]
    
    // the 'self' here means the view controller, set view controller as the delegate
    cell.delegate = self
        
    return cell
}
    
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return youtubers.count
  }
}
```



뷰 컨트롤러가 YoutuberTableViewCellDelegate를 준수해야하므로, 다음의 코드를 추가해준다.

```swift
extension ViewController : YoutuberTableViewCellDelegate {
  func youtuberTableViewCell(_ youtuberTableViewCell: YoutuberTableViewCell, subscribeButtonTappedFor youtuber: String) { {
    // directly use the youtuber saved in the cell
    // show alert
    let alert = UIAlertController(title: "Subscribed!", message: "Subscribed to \(youtuber)", preferredStyle: .alert)
    let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
    alert.addAction(okAction)
    
    self.present(alert, animated: true, completion: nil)
  }
}
```

위의 코드는 유저가 각 셀의 구독 버튼을 탭했을 때 실행된다. 인덱스는 이 메서드를 통해 전달된다.



모델, (ie. Youtuber(string), 자신의 코드에서 커스텀 모델 클래스를 사용할 수 있음)을 테이블 뷰 셀에 직접 전달한다. 따라서 row 삽입 / 삭제 가 있는 경우에 indexPath의 업데이트를 걱정할 필요가 없다.



## 클로저를 사용한 방법

아직 클로저를 사용하는 방법과 옵셔널에 대해 잘 모르겠다면 이 글을 읽고 오면 된다.

클로저를 사용하려면, 셀 클래스에 클로저 프로퍼티(subscribeButtonAction)를 추가해야한다.

```swift
//YoutuberTableViewCell.swift

class YoutuberTableViewCell: UITableViewCell {

  @IBOutlet weak var youtuberLabel: UILabel!
    
  @IBOutlet weak var subscribeButton: UIButton!
    
  /*
  No need to keep track the index since we are using closure to store the function that will be executed when user tap on it.
  */
  
  // the closure, () -> () means take no input and return void (nothing)
  // it is wrapped in another parentheses outside in order to make the closure optional
  var subscribeButtonAction : (() -> ())?
    
  override func awakeFromNib() {
    super.awakeFromNib()
    // Initialization code
        
    // Add action to perform when the button is tapped
    self.subscribeButton.addTarget(self, action: #selector(subscribeButtonTapped(_:)), for: .touchUpInside)
  }

  override func setSelected(_ selected: Bool, animated: Bool) {
    super.setSelected(selected, animated: animated)

    // Configure the view for the selected state
  }
    
  @IBAction func subscribeButtonTapped(_ sender: UIButton){
    // if the closure is defined (not nil)
    // then execute the code inside the subscribeButtonAction closure
    subscribeButtonAction?()
  }
    
}
```

input이 없고 void를 반환하는 클로저 타입 변수 `subscribeButtonAction` 을 사용해 유저가 구독 버튼을 탭했을 때 실행되는 코드를 저장한다. 이는 변수에 함수를 저장하고 변수명 뒤에 () 괄호를 붙여서 함수를 실행하는 것과 같다. 

![closure](https://iosimage.s3.amazonaws.com/2018/33-uibutton-uitableviewcell-tap/closure.png)

그런 다음 사용자가 버튼을 누를 때 실행될 코드를  `cellForRowAt`메소드에 추가한다.

```swift
extension ViewController : UITableViewDataSource {
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: cellReuseIdentifier, for: indexPath) as! YoutuberTableViewCell
    cell.youtuberLabel.text = youtubers[indexPath.row]
    
    // the code that will be executed when user tap on the button
    // notice the capture block has [unowned self]
    // the 'self' is the viewcontroller
    cell.subscribeButtonAction = { [unowned self] in
      let youtuber = self.youtubers[indexPath.row]
      let alert = UIAlertController(title: "Subscribed!", message: "Subscribed to \(youtuber)", preferredStyle: .alert)
      let okAction = UIAlertAction(title: "OK", style: .default, handler: nil)
      alert.addAction(okAction)
            
      self.present(alert, animated: true, completion: nil)
    }
        
    return cell
}
    
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return youtubers.count
  }
}
```



`[unowned self]` 가 `cell.subscribeButtonAction` 클로저의 시작 부분에 들어가 있는 것을 볼 수 있다. 이는 뷰 컨트롤러가 테이블 뷰를 소유하고, 테이블 뷰는 셀을 소유하고, 셀은 subscribeButtonAction 클로저를 소유하는 retain 싸이클을 방지해준다. 만약 클로저를weak/unowned reference로 만들지 않고 내부에 self 키워드를 사용하게 되면, 다음과 같은 싸이클이 생길 것이다.

![cycle](https://iosimage.s3.amazonaws.com/2018/33-uibutton-uitableviewcell-tap/cycle.png)

테이블 뷰 셀의 버튼을 탭했을 때 뷰컨트롤러가 여전히 메모리에 있음을 확신할 수 있기 때문에  unowned를 사용해줄수 있다. Hector가  [excellent article about weak/unowned and retain cycle](https://krakendev.io/blog/weak-and-unowned-references-in-swift) 에 대한 글을 썼으니 관심있으면 읽어보길 바란다.



클로저를 이용한 접근 방식은 더 멋있어보이지만 각각의 셀이 클로저 변수(버튼이 탭됐을 때 실행할 행동)를 저장하기 위해 메모리에 할당되어야 한다는 것을 기억해야한다. 이 접근방법은 함수가 커지면 꽤 많은 메모리를 차지할 수 있다.



## Notes

이 포스트는 uitableview cell의 버튼에서 delegate / closure 를 사용하는 방법이지만, 이 방법들을  [passing data back to the previous view controller](https://fluffy.es/3-ways-to-pass-data-between-view-controllers/#delegate) , 기타의 경우에도 사용할 수 있다. 