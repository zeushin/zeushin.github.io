+++
name = "Masher Shin"
title = "Should we use [weak self] in GCD"
description = "GCD 안에서 반드시 weak self 를 사용해야 할까?"
imgaes = ["/should-we-use-weak-self-in-GCD-01.png"]
date = "2017-09-26T14:04:23+09:00"
+++

<img src="/should-we-use-weak-self-in-GCD-01.png" style="margin: auto">
# GCD 안에서 반드시 [weak self] 를 사용해야 할까?
라는 의문을 갖고 검색을 해보니 [iOS — GCD what the __weak is going on?](https://medium.com/rocknnull/ios-gcd-what-the-weak-is-going-on-d5a10fc682a) 이라는 제목의 좋은 글을 발견했다. 다만 해당 포스트는 Objective-C로 되어 있어 Swift로 컨버팅을 해 보았다.
Objective-C와 기본적으로는 같지만 약간의 차이점이 있어 공유한다.

## GCD는 retain cycle 을 유발하는가?
위에 링크한 [medium의 포스트](https://medium.com/rocknnull/ios-gcd-what-the-weak-is-going-on-d5a10fc682a)는 궁극적으로 이러한 의문을 갖고 출발한다. 결론부터 이야기 하면 글로별 변수로 store 하지 않으면 어떻게 사용해도 retain cycle을 유발하지 않는다. (위에 링크한 [medium 포스트](https://medium.com/rocknnull/ios-gcd-what-the-weak-is-going-on-d5a10fc682a)를 참고하자!)
하지만, 그럼에도 불구하고 `[weak self]` 를 이용해야 하는 이유는 충분해 보인다.

## closure 안에서 self에 접근하는 3가지 방법
[medium의 포스트](https://medium.com/rocknnull/ios-gcd-what-the-weak-is-going-on-d5a10fc682a)에서 처럼, 네비게이션 푸시로 한뎁스 들어간 어떤 뷰 컨트롤러에 버튼을 만들어 간단히 GCD를 사용해 보았다.
버튼을 누르면 다른 스레드에서 글로벌 변수의 배열에 접근해 아이템을 추가하고 10초를 슬립한 후 다시 메인 스레드(UI thread)로 넘어와 배열에 접근하고 UI 객체에 접근하는 코드이다.
핵심은 버튼을 누른 후 10초가 되기 전에 네비게이션 바의 백 버튼을 눌러 해당 뷰 컨트롤러를 스택에서 뽑아 냈을 때 어떤 결과와 차이가 있는지 보는 것이다.

### self를 그냥 사용
```
    DispatchQueue.global(qos: .userInitiated).async {
        self.proArray.append("2")
        self.proArray.append("3")
        Thread.sleep(forTimeInterval: 10)

        DispatchQueue.main.async {
            self.proArray.removeLast()
            print(self.proArray ?? "")
            self.indicator.stopAnimating()
        }
    }
```
10초가 지난 후 메인 스레드 클로져가 실행된다. 당연히 클로져가 실행 된 후에 뷰 컨트롤러가 메모리에서 해제(deinit)된다.

### self를 weak 로 캡쳐한 후 optional로 사용
```
    DispatchQueue.global(qos: .userInitiated).async { [weak self] in
        self?.proArray.append("2")
        self?.proArray.append("3")
        Thread.sleep(forTimeInterval: 10)

        DispatchQueue.main.async {
            self?.proArray.removeLast()
            print(self?.proArray ?? "")
            self?.indicator.stopAnimating()
        }
    }
```
백 버튼을 누른 즉시 메모리에서 해제 되고, 10초가 지난 후에 아무일도 일어나지 않는다.

### self를 weak 로 챕쳐한 후 guard let 으로 사용
```
    DispatchQueue.global(qos: .userInitiated).async { [weak self] in
        guard let sSelf = self else { return }
        sSelf.proArray.append("2")
        sSelf.proArray.append("3")
        Thread.sleep(forTimeInterval: 10)

        DispatchQueue.main.async {
            guard let sSelf = self else { return }
            sSelf.proArray.removeLast()
            print(sSelf.proArray)
            sSelf.indicator.stopAnimating()
        }
    }
```
10초 뒤에 뷰컨트롤러가 메모리에서 해제 되며 메인 스레드 클로져는 실행되지 않는다.
다만!!
메인 스레드 클로져 안의 가드문(`guard let sSelf = self else { return }`, 위 코드에서 두번째 가드문)을 생략하게 되면 `self`를 그냥 사용 사용하였을 때와 같은 결과가 나타난다.

## 결론
반드시 `[weak self]`를 사용해야 할까? 에 대한 답은 *반드시는 아니다* 이다. `self`의 라이프 사이클에 따라서 적절히 사용해 주면 된다. 위의 뷰 컨트롤러 처럼 `self`가 즉시 없어져야 하는 경우에는 `[weak self]`로 사용하고, 그 외의 경우에는 굳이 그러지 않아도 된다. 다만 마지막 케이스 처럼 이중 클로져 안에서의 가드문 사용은 각별히 주의 하도록 하자.
