# Swift Dependency Injection


## Singleton as a default value for parameters
```
class Manager {
  let userDefaults: UserDefaults

  init(userDefaults: UserDefaults = .standard) {
    self.userDefaults = userDefaults
  }
}

let testInstance = Manager(userDefaults: mockUserDefaults)
```

## Using functions
- Avoid tight coupling with singletons

```
struct DebugManager {
  static let shared = DebugManager()
  let isDebugMode: Bool
}
  
class SomethingManager {
  let isDebugMode: () -> Bool

  init(isDebugMode: @escaping () -> Bool = { return DebugManager.shared.isDebugMode } ) {
    self.isDebugMode = isDebugMode
  }
}

let instance = SomethingManager()
let testInstance = SomethingManager(isDebugMode: { true } ) //isDebugMode is always true
```


## Using protocols
```
protocol Reachability {
  func isNetworkAvailable()
}

class ReachabilityManager: Reachability {
  func isNetworkAvailable() {
  }
}

class MockReachabilityManager: Reachability {
  func isNetworkAvailable() {
  }
}


class TestViewController: UIVIewController {
  let reachabilityManager: Reachability

  init(reachabilityManager: Reachability = ReachabilityManager()) {
    self.reachabilityManager = reachabilityManager
  }
}

let instance = TestViewController()
let testInstance = TestViewController(reachabilityManager: MockReachabilityManager())
```

## Using Environment ("Super/God" Singleton)
```
protocol Networking {
  func call()
}

struct MainNetworkManager: Networking {
  func call() { print("--Making networking request") }
}

struct FakeNetworkManager: Networking {
  func call() { print("--🙅🏻‍♂️") }
}

struct Environment {
  static var current = Environment()
  var date: () -> Date = Date.init
  var networkManager: Networking = MainNetworkManager()
}

let main = Environment()
let mock = Environment(
  date: { return Date(timeIntervalSince1970: 0) },
  networkManager: FakeNetworkManager()
)

func printDate() {
  print("Timestamp: \(Environment.current.date())")
  Environment.current.networkManager.call()
}


Environment.current = main
print("----MAIN----")
printDate()
sleep(1)
printDate()
sleep(1)


print("\n----MOCK----")
Environment.current = mock
printDate()
printDate()


// Log
/*
----MAIN----
Timestamp: 2020-05-28 17:36:15 +0000
--Making networking request
Timestamp: 2020-05-28 17:36:16 +0000
--Making networking request

----MOCK----
Timestamp: 1970-01-01 00:00:00 +0000
--🙅🏻‍♂️
Timestamp: 1970-01-01 00:00:00 +0000
--🙅🏻‍♂️
*/
```

