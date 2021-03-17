# ios-juice-maker
iOS 쥬스 메이커 재고관리 시작 저장소

Step 1 

Fruit.swift

```swift
import Foundation

class Fruit {
    private let name: String
    private let origin: String
    private let price: Int

    init(name: String, origin: String, price: Int) {
        self.name = name
        self.origin = origin
        self.price = price
    }
}

class Strawberry: Fruit {}

class Banana: Fruit {}

class Kiwi: Fruit {}

class Pineapple: Fruit {}

class Mango: Fruit {}

```
* Fruit클래스는 과일의 타입을 정의해주는 클래스입니다. 
* 과일의 속성만 포함하는 타입이고 상속을 위하여 class로 작성하였습니다.
* 각각의 Strawberry, Banana, Kiwi, Pineapple, Mango 클래스는 Fruit 클래스를 상속받습니다.

FruitStorage.swift

```swift
import Foundation

typealias stock = [ObjectIdentifier : Int]

class FruitsStorage {
    static let sharedInstance = FruitsStorage()
    private(set) var fruitsStock: stock = [ObjectIdentifier(Banana.self): 10, ObjectIdentifier(Strawberry.self): 10,
                                           ObjectIdentifier(Kiwi.self): 10, ObjectIdentifier(Pineapple.self): 10,
                                           ObjectIdentifier(Mango.self): 10]

    private init() {
    }
    
    func addFruit(_ fruit: stock.Key, amount: Int) {
        if let remainedFruitStock = self.fruitsStock[fruit] {
            self.fruitsStock[fruit] = remainedFruitStock + amount
        }
    }
    
    func reduceFruit(_ fruit: stock.Key, amount: Int) {
        if let remainedFruitStock = self.fruitsStock[fruit] {
            self.fruitsStock[fruit] = remainedFruitStock - amount
        }
    }
}
```
* typealias를 이용하여 타입 식별자 ObjectIdentifier를 키로 갖고 Int형을 value값으로 갖는 딕셔너리타입을 정의했습니다.
* fruitStock 프로퍼티는 창고의 과일재고를 관리하는 stock으로 명명된 딕셔러니 프로퍼티입니다.
* 과일재고는 유일하게 관리되어야하므로 싱글턴패턴으로 구현했습니다.
* addFruit은 재고를 추가하는 메서드 reduceFruit은 재고를 감소시키는 메서드입니다.

Juice.swift

```swift
import Foundation

typealias Recipe = (stock: ObjectIdentifier, requiredAmount: Int)

class Juice {
    private let name: String
    private let recipe: [Recipe]

    init(name: String, recipe: [Recipe]) {
        self.name = name
        self.recipe = recipe
    }
}

class StrawberryJuice: Juice {}

class BananaJuice: Juice {}

class KiwiJuice: Juice {}

class PineappleJuice: Juice {}

class MangoJuice: Juice {}

class StrawberryBananaJuice: Juice {}

class MangoKiwiJuice: Juice {}
```
* Fruit타입을 상속받은 과일타입들과 마찬가지로 Juice타입을 상속받아 쥬스타입들을 정의했습니다.
* Recipe는 튜플타입으로써 stock: Objectidentifier, requiredAmount: Int로 이루어져있습닏. 과일과 그 과일의 요구량을 저장하는 튜플입니다.

JuiceMaker.swift

```swift
import Foundation

typealias juiceRecipe = [ObjectIdentifier : [Recipe]]

class JuiceMaker {
    private let recipes: juiceRecipe = [ObjectIdentifier(StrawberryJuice.self) : [(ObjectIdentifier(Strawberry.self), 16)],
                             ObjectIdentifier(BananaJuice.self) : [(ObjectIdentifier(Banana.self), 2)],
                             ObjectIdentifier(KiwiJuice.self) : [(ObjectIdentifier(Kiwi.self), 3)],
                             ObjectIdentifier(PineappleJuice.self) : [(ObjectIdentifier(Pineapple.self), 2)],
                             ObjectIdentifier(MangoJuice.self) : [(ObjectIdentifier(Mango.self), 3)],
                             ObjectIdentifier(StrawberryBananaJuice.self) :  [(ObjectIdentifier(Strawberry.self), 10),(ObjectIdentifier(Banana.self), 1)],
                             ObjectIdentifier(MangoKiwiJuice.self) : [(ObjectIdentifier(Mango.self), 2), (ObjectIdentifier(Kiwi.self), 1)]]
    
    var fruitsStorage = FruitsStorage.sharedInstance
    
    private func hasEnoughFruitStock(_ juice: juiceRecipe.Key) -> Bool {
        if let juiceIngredients = recipes[juice] {
            for juiceIngredient in juiceIngredients {
                if let remainedFruitStock = fruitsStorage.fruitsStock[juiceIngredient.stock] {
                    if juiceIngredient.requiredAmount > remainedFruitStock {
                        return false
                    }
                }
            }
        }
        return true
    }
    
    func makeJuice(_ juice: juiceRecipe.Key) {
        if hasEnoughFruitStock(juice) {
            if let juiceIngredient = recipes[juice]{
                for juiceIngredient in juiceIngredient {
                    fruitsStorage.reduceFruit(juiceIngredient.stock, amount: juiceIngredient.requiredAmount)
                    print(fruitsStorage.fruitsStock)
                }
            }
            return
        }
    }
}
```
* juiceRecipe는 딕셔너리타입으로써 ObjectIdentifier을 키값으로 가지고 [Recipe]의 value값을 가집니다.(Recipe는 위에서 정의한 튜플형타입입니다)
* JuiceMaker클래스의 recipes 프로퍼티는 juiceRecipe타입이며 각 쥬스들을 식별하는 ObjectIdentifier가 키값이며 재료에 필요한 과일타입을 식별하는 ObjectIdentifier와 소요개수를 묶은 튜플이 Value값을 나타냅니다.
* fruitsStorage는 FruitsStorgae의 인스턴스이며 JuiceMaker에서 makejuice와 hasEnoughFruitStock에서 쥬스재고를 참조하기위하여 선언했습니다.
* hasEnoughFruitStock메서드는 현재제고가 주문한 쥬스를 만들때 필요한 재료만큼있는지 확인하는 메서드입니다.
* makeJuice는 쥬스를 만드는 메서드입니다.
