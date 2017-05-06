+++
date = "2017-04-22T11:33:30+09:00"
description = "AppCode를 이용해 유닛테스트를 해보자"
title = "Unit testing with AppCode"
name = "Masher Shin"
tags = ["appcode", "unit test", "swift", "ios"]
images = ["/unit-test-with-appcode-01.png", "/unit-test-with-appcode-02.png", "/unit-test-with-appcode-03.png", "/unit-test-with-appcode-04.png", "/unit-test-with-appcode-05.png"]

+++

# Add Configuration
우선 Xcode 프로젝트에서 unit test 설정이 되어 있어야 한다. 프로젝트 타겟에 유닛테스트 타겟이 었어야 한다는 의미.
이미 Xcode상에서 unit test를 한번이상 실행하였다면 AppCode를 실행하자.
AppCode상에서

**Run | Edit Configurations...**

으로 들어가서 **Test Configuration**을 추가한다.
나같은 경우는 **XCTest/Kiwi** 라고 되어있었다.
<img src="/unit-test-with-appcode-01.png">

- **Name** 에 적당한 이름을 지어주고(Xcode상의 타겟 이름과 같이 하면 좋다)
- **Class** 는 비워두면 알아서 모든 Test파일을 참조한다.
- **Target** 과 **Configuration** 을 Xcode 와 동일하게 지정해 준다.

# Run
상단 툴바의 **Configurations** 선택에서 방금 추가한 Test configuration 을 선택하고,
<img src="/unit-test-with-appcode-02.png">
평소에 앱을 실행 할때와 같이 녹색 플레이 버튼인 **Run** 을 눌러 실행한다.

# Results
테스트가 실행되면 IDE 의 바닦면에서 Run Tool Window 가 팝업되면서 진행상태가 표시 된다.
<img src="/unit-test-with-appcode-03.png">
테스트가 끝나고 나면 왼편에 실패한 테스트가 표시되고
<img src="/unit-test-with-appcode-04.png">
프로그래에서스 바에는 최종 결과를 확인할 수 있다.
<img src="/unit-test-with-appcode-05.png">
