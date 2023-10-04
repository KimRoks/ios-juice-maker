# ios-juice-maker

# 팀원
[@KimRoks](https://github.com/KimRoks)


[@dpwns1234](https://github.com/dpwns1234)

# 구현 영상

![2023-10-04_5 53 13](https://github.com/KimRoks/ios-juice-maker/assets/113083860/0f7ab558-2578-4eac-a08a-30f18308b614)

사용자가 음료를 주문하면 레시피에 따라 과일을 소모하여 음료를 제공하는 앱

# 핵심 구현 사항

### Delegate Pattern

화면 전환 시 데이터 전달을 위해 Delegate Pattern 채택

```swift
//StockEditonDelegate.swift
protocol StockEditDelegate {
    func sendChangedStock(_ fruitStock: [Fruit : Int])
}

//JuiceMakerViewController.swift
class JuiceMakerViewController: UIViewController, StockEditDelegate {
...
    private func presentStockManagerViewController() {
        guard let stockManagerViewController = self.storyboard?.instantiateViewController(identifier: "StcokManagerViewController") as? StockManagerViewController else { return }
        stockManagerViewController.fruitStock = juiceMaker.fruitStore.getFruitStock()
        stockManagerViewController.stockEditionDelegate = self
        self.present(stockManagerViewController, animated: true, completion: nil)
    }
}

//StockManagerViewController.swift
class StockManagerViewController: UIViewController {
  weak var stockEditionDelegate: StockEditDelegate?

  @IBAction private func pressCloseButton(_ sender: UIBarButtonItem) {
          stockEditionDelegate.sendChangedStock(fruitStock)
          dismiss(animated: true)
      }
}
```

### Factory Pattern


Alert의 확장성과 결합도를 줄이기위해 Factory Pattern 채택

```swift
//AlertFactoryService.swift

protocol AlertFactoryService {
    func build(alertData: AlertViewData) -> UIAlertController
}

//AceeptAlertActionDelegate.swift
protocol AceeptAlertActionDelegate {
    func okAction()
}

//AlertFactory.swift
struct AcceptAlertFactory: AlertFactoryService {
    var alertActionDelegate: AceeptAlertActionDelegate
    
    func build(alertData: AlertViewData) -> UIAlertController {
        let alert = UIAlertController(
            title: alertData.title,
            message: alertData.message,
            preferredStyle: alertData.style
        )
        let okAction = UIAlertAction(
            title: alertData.okActionTitle,
            style: .default,
            handler: { _ in
                alertActionDelegate.okAction()
            })
        alert.addAction(okAction)
        
        return alert
    }   
}

//JuiceMakerViewController.swift
class JuiceMakerViewController: UIViewController, AceeptAlertActionDelegate {
    private var acceptAlertFactory: AlertFactoryService?

    override func viewDidLoad() {
        super.viewDidLoad()
        acceptAlertFactory = AcceptAlertFactory(alertActionDelegate: self)
    }

    func okAction() {
        // 추후에 ok버튼이 눌렸을 때 동작 추가 가능
    }
}
```



# 트러블 슈팅

- 개발자가 코드가 아닌 Storyboard의 Inspector에 설정된 값을 확인해야하는 코드(tag 등)가 좋은 코드라고 할 수 있을까?

   
```
  최초 객체들을 구분하기 위해 `tag`를 사용하며 들었던 고민

  tag는 런타임에 동적으로 생성되는 뷰들을 구분짓기 위해 쓰이기때문에 옳지 않다고 판단하였고

  Button과 Label의 상속을 활용하여 객체들을 구분짓도록 코드를 개선했다.
```
   
   
- Alert Controller를 사용하며 중복되는 코드가 늘어났다
```
  명확한 구분을 위해 코드의 양이 늘어나는것과
  가독성이 떨어지더라도 최대한 중복된 코드를 줄이는것에 대한 고민을 했다

  이를 해결하기 위해 Factory Pattern을 채택하였고 그로인해
  코드의 중복을 줄이고 결합도를 낮췄다.
```
