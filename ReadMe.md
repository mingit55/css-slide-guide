# CSS로 슬라이드를 다루는 방법

---

## 개요

3초마다 이미지가 전환되는 슬라이드를 만들려면 어떻게 해야할까?
먼저 간단하게 HTML을 구성하여 보자.

```html
<div class="slide-wrap">
  <div class="slide"></div>
  <div class="slide"></div>
  <div class="slide"></div>
</div>
```

간단하게 슬라이드를 감싸는 slide-wrap 을 만들고, 안에 slide 를 3장 넣게끔 하였다.
이제 이 HTML에 간단한 CSS를 입혀보자.

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.slide-wrap {
  position: relative;
  margin: 50px auto;
  width: 300px;
  height: 200px;
  border: 1px solid #aaa;
  overflow: hidden;
}

.slide {
  position: absolute;
  left: -100%;
  top: 0;
  width: 100%;
  height: 100%;
}
.slide-1 { background-color: #faa; }
.slide-2 { background-color: #afa; }
.slide-3 { background-color: #aaf; }
```

지금까지 만들어진 내용을 보면 다음과 같다.

![img](./images/1.png)

이제 애니메이션을 만들어 보도록 한다.
이미지는 총 3장이라고 가정하였을 때, 3초 동안 이미지가 보여지고, 0.5초의 시간 동안 이미지가 전환된다고 생각해 보자.

| (이미지 대기시간 3초 + 이동시간 0.5초) * 3 = 10.5초

따라서 @keyframes 을 0 ~ 100%로 구분할 때, 0% 는 0s, 100% 는 10.5s 가 된다.
이를 비율로 계산해서, 3초를 퍼센트로 환산하면 다음과 같이 계산한다.

| 3초 : 10.5초 = n% : 100%
| 3 * 100 / 10.5 = 28.571%

따라서 현재 만들고자 하는 @keyframes 에서 3초는 28.571% 에 해당한다.
이와 동일한 방법으로 @keyframes 를 만들면 다음과 같다.

```css
@keyframes slide {
  /* 0s 3 */
  0% { left: -100%; }
  /* 0.5s 1 */
  4.7619% { left: 0; }
  /* 3.5s 1 */
  33.333% { left: 0; }
  /* 4s 2 */
  38.095% { left: 100%; }
  /* 7s 2 */
  66.666% { left: 100%; }
  /* 7.5s 3 */
  71.428% { left: 100%; }
  /* 10.5s 3 */
  100% { left: 100%; }
}
```

이제 만든 @keyframes 를 직접 적용해 보도록 하자

```css
.slide {
  animation-name: slide;
  animation-duration: 10.5s;
  animation-iteration-count: infinite;
}
.slide-1 { animation-delay: -0.5s; }
.slide-2 { animation-delay: -7.5s; }
.slide-3 { animation-delay: -4s; }
```

animation-delay 는 값을 음수로 설정하면, 해당 시간만큼 애니메이션의 시작 시간을 앞으로 당길 수 있다.
slide-1 은 첫번째, slide-2 는 두번째, slide-3 을 세번째로 슬라이드에 나오도록 설정한다.
위 코드까지 적용하면 기본적인 슬라이드 구성이 완료된다.

## 슬라이드 컨트롤

이제 슬라이드를 이동시키는 버튼을 추가해 보도록 하자.
HTML/CSS만으로 슬라이드를 이동시키려면, 현재 슬라이드가 몇번째인지 저장할 수단이 필요하다.
input[type="radio"]를 통해 현재 슬라이드를 저장할 수 있다. HTML의 상단에 다음과 같은 내용을 추가한다.

```html
<input type="radio" name="s" id="s1" hidden>
<input type="radio" name="s" id="s2" hidden>
<input type="radio" name="s" id="s3" hidden>

<div class="slide-wrap">
  ...
```

그리고 slide-wrap 아래에는 해당 label 들을 컨트롤 할 수 있는 control 들을 생성해 준다.

```html
<div class="control-wrap">
  <div class="control">
  </div>
  <div class="control">
  </div>
  <div class="control">
  </div>
</div>
```

```css
.control-wrap {
  justify-content: center;
  display: flex;
  gap: 5px;
}
.control {
  position: relative;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 1px solid #555;
  cursor: pointer;
}
.control .label {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}
```

## 생각해 보아야 할 문제점

### 1. 역방향이동 문제

이제 각 컨트롤을 클릭하였을 때, 해당 위치로 슬라이드가 이동할 수 있게끔 수정해주어야 하는데, 한번 생각해봐야 하는 문제에 직면하게 된다. 슬라이드는 기본적으로 정방향(1->2->3) 순서대로 움직이는데, 만약 현재 슬라이드가 3이고 슬라이드 2로 이동하려면 역방향(3->2)로 슬라이드가 움직여야만 한다.

만약 해당 과정이 @keyframes 안에 포함되게 되면 (1->2->3->2->3->1->2->..) 와 같이, 순서가 꼬여버리게 될 것이다. 이 때문에 우리는 @keyframes 를 여러개 만들어서 이전으로 움직이는 애니메이션을 하나 만들어야 한다.

### 2. animation-delay 문제

위 역방향이동 문제를 해결하고자 animation 을 여러개 등록하였을 때, 추가적으로 문제가 발생하게 된다. 예를 들어 다음과 같이 코드를 작성하였다고 가정하여 보자.

```css
@keyframes example1 {
  from { left: 0; }
  to { left: 1000px; }
}
@keyframes example2 {
  from { left: 1000px; }
  to { left: 0; }
}

.example {
  animation-name: example1, example2;
  animation-duration: 10s, 10s;
  animation-delay: 0, -5s;
  animation-iteration-count: 1, infinite;
}
```

example1 은 개체가 오른쪽으로 이동, example2는 왼쪽으로 이동하는 @keyframes 이다. 얼핏 보면, 오른쪽으로 1000px 을 이동한 후 (example1), 다시 왼쪽으로 돌아오는 애니메이션 (example2) 가 반복될 것으로 보이지만, 실상은 순차실행이 아니라 동시실행이기 때문에 animation-delay 를 통한 애니메이션 미리 시작이 불가능하다. 따라서, 역방향 슬라이드의 경우 animation-delay 를 통한 슬라이드 구분이 불가능해져서 슬라이드를 1, 2, 3번째마다 각각 따로 생성해 줄 필요가 있다.

## 3. 애니메이션 중복 문제

만약에 첫번째 control 을 클릭해서 첫번째 슬라이드로 이동하였다고 가정해 보자. 그 상태로 슬라이드가 2, 3번째가 될 때까지 기다렸다가 다시 첫번째 control 을 눌렀을 때, 동일한 radio 를 바라보고 있다면, radio 특성상 이미 선택 중인 input 을 재선택하기는 불가능하기 때문에, 이동되지 않는 버그가 발생한다. 따라서 radio 의 복사본을 만들어서, 동일한 기능을 할 수 있도록 만들어 주어야 한다.

## 컨트롤 기능 추가하기

위에서 살펴본 문제점을 해결하기 위한 해결 방안은 다음과 같다.

- 정방향 슬라이드와 역방향 슬라이드를 구분지어 애니메이션을 설정해야한다.
- 역방향 슬라이드의 경우 애니메이션을 두 가지 사용하여 역방향으로 이동할 수 있게 설정해야한다.
- 애니메이션이 중복되지 않도록, slide / radio 의 복사본을 만들어 동일한 역할을 수행하도록 해야 한다.

그럼 먼저 정방향/역방향 슬라이드를 구분지어 애니메이션을 설정할 수 있도록 radio 태그를 수정해 보자.

```html
<!-- 삭제 -->
<!-- <input type="radio" name="s" id="s1" hidden> -->
<!-- <input type="radio" name="s" id="s2" hidden> -->
<!-- <input type="radio" name="s" id="s3" hidden> -->

<input type="radio" name="s" id="move-2-1" checked>
<input type="radio" name="s" id="move-3-1">
<input type="radio" name="s" id="move-1-2">
<input type="radio" name="s" id="move-3-2">
<input type="radio" name="s" id="move-1-3">
<input type="radio" name="s" id="move-2-3">
```

슬라이드 이미지가 총 3장이기 때문에, 정방향 3개(1-2, 2-3, 3-1), 역방향 3개(1-3, 2-1, 3-2) 총 6개의 radio 가 필요하다. 이렇게 radio 를 모두 생성하였다면, 정방향과 역방향을 구분지을 수 있게 되어, 문제점 1을 해결할 수 있게 된다.
이제 문제점 2를 해결하기 위해 slide 를 1~3으로 구분지어 보자.

```css

/* slide-1 은 위에 있는 @keyframes slide를 그대로 사용한 것이다.  */
@keyframes slide-1 {
  /* 0s 3 */
  0% { left: -100%; }
  /* 0.5s 1 */
  4.7619% { left: 0; }
  /* 3.5s 1 */
  33.333% { left: 0; }
  /* 4s 2 */
  38.095% { left: 100%; }
  /* 7s 2 */
  66.666% { left: 100%; }
  /* 7.5s 3 */
  71.428% { left: 100%; }
  /* 10.5s 3 */
  100% { left: 100%; }
}
@keyframes slide-2 {
  /* 0s 2 */
  0% { left: -100%; }
  /* 0.5s 3 */
  4.7619% { left: -100%; }
  /* 3.5s 3 */
  33.333% { left: -100%; }
  /* 4s 1 */
  38.095% { left: 0%; }
  /* 7s 1 */
  66.666% { left: 0%; }
  /* 7.5s 2 */
  71.428% { left: 100%; }
  /* 10.5s 2 */
  100% { left: 100%; }
}
@keyframes slide-3 {
  /* 0s 1 */
  0% { left: 0%; }
  /* 0.5s 2 */
  4.7619% { left: 100%; }
  /* 3.5s 2 */
  33.333% { left: 100%; }
  33.333333% { left: -100%; }
  /* 4s 3 */
  38.095% { left: -100%; }
  /* 7s 3 */
  66.666% { left: -100%; }
  /* 7.5s 1 */
  71.428% { left: 0%; }
  /* 10.5s 1 */
  100% { left: 0%; }
}
```

slide-3의 경우 left가 100% 에서 -100%로 단숨에 이동시키기 위해 특별히 100% ~ -100% 로 이동하는 그 순간을 퍼센티지로 짧게 추가 설정해서 사용자가 보는 영역에 왼쪽으로 이동하는 과정이 보여지지 않도록 하였다.
이렇게 3종류의 @keyframes 까지 모두 완성하였다면, 각 radio와 연결되는 CSS 를 만들어주어야 한다. CSS 에서는 radio 가 활성화될 때, :checked 선택자를 통해 알 수 있다. 이를 이용해서 :checked 가 되었을 때 형제 선택자를 이용해서 각 slide 에 적절한 애니메이션이 들어가도록 수정해 준다.

```css
#move-1-2:checked ~ .slide-wrap .slide-1 { animation-name: slide-3; }
#move-1-2:checked ~ .slide-wrap .slide-2 { animation-name: slide-1; }
#move-1-2:checked ~ .slide-wrap .slide-3 { animation-name: slide-2; }

#move-2-3:checked ~ .slide-wrap .slide-1 { animation-name: slide-2; }
#move-2-3:checked ~ .slide-wrap .slide-2 { animation-name: slide-3; }
#move-2-3:checked ~ .slide-wrap .slide-3 { animation-name: slide-1; }

#move-3-1:checked ~ .slide-wrap .slide-1 { animation-name: slide-1; }
#move-3-1:checked ~ .slide-wrap .slide-2 { animation-name: slide-2; }
#move-3-1:checked ~ .slide-wrap .slide-3 { animation-name: slide-3; }
```

예를 들어 move-1-2의 경우 슬라이드가 1에서 2로 이동하는 애니메이션이다. 따라서 결과적으로 2가 보여지면서 2번째부터 슬라이드가 시작되기 때문에, 2번째 슬라이드가 @keyframes slide-1 부터 부여된다.

마찬가지로 move-2-3의 경우는 2에서 3으로 이동하는 애니메이션이고, 3번째 슬라이드부터 시작하기 때문에 3번재 슬라이드가 @keyframes slide-1 부터 시작하는 것이다.

위 내용은 정방향 슬라이드(1-2, 2-3, 3-1)을 생성한 것이다. 이제부턴 역방향 슬라이드를 만들어야 한다. 역방향 슬라이드(1-3, 2-1, 3-2)는 위에서 한번 언급했듯이, 슬라이드 애니메이션이 2개가 필요하다.

1. 역방향으로 한번 움직이는 동작
2. 이후 정방향으로 다시 움직이는 동작

2번 정방향 동작은 위에서 이미 만들었으니, 1번 역방향으로 한번 움직이는 동작을 @keyframes 로 만들어 보자.

```css
/*
해당 예제는 정방향 @keyframes 가 한번 움직인 뒤 시작하기 때문에,
이에 맞춰 한번 움직인 후(0.5s) 정지하는 움직임(3s) 으로 역방향 애니메이션을 제작한다.

총 애니메이션의 길이: 3.5s
움직이는 동작: 0.5s
기다리는 동작: 3s
*/
@keyframes be-hide {
  /* 0s */
  0% { left: 0; }
  /* 0.5s */
  14.2857% { left: -100%; }
  /* 3.5s */
  100% { left: -100%; }
}
@keyframes be-show {
  /* 0s */
  0% { left: 100%; }
  /* 0.5s */
  14.2857% { left: 0; }
  /* 3.5s */
  100% { left: 0; }
}
```

이렇게 역방향 이동 애니메이션을 만들었다면, 이어서 역방향도 정방향과 동일한 방식으로 만들어 준다.

```css

#move-2-1:checked ~ .slide-wrap .slide-1 { animation-name: be-show, slide-3; }
#move-2-1:checked ~ .slide-wrap .slide-2 { animation-name: be-hide, slide-1; }
#move-2-1:checked ~ .slide-wrap .slide-3 { animation-name: slide-2; }

#move-3-2:checked ~ .slide-wrap .slide-1 { animation-name: slide-2; }
#move-3-2:checked ~ .slide-wrap .slide-2 { animation-name: be-show, slide-3; }
#move-3-2:checked ~ .slide-wrap .slide-3 { animation-name: be-hide, slide-1; }

#move-1-3:checked ~ .slide-wrap .slide-1 { animation-name: be-hide, slide-1; }
#move-1-3:checked ~ .slide-wrap .slide-2 { animation-name: slide-2; }
#move-1-3:checked ~ .slide-wrap .slide-3 { animation-name: be-show, slide-3; }

#move-2-1:checked ~ .slide-wrap .slide-1,
#move-2-1:checked ~ .slide-wrap .slide-2,
#move-3-2:checked ~ .slide-wrap .slide-2,
#move-3-2:checked ~ .slide-wrap .slide-3,
#move-1-3:checked ~ .slide-wrap .slide-1,
#move-1-3:checked ~ .slide-wrap .slide-3 {
  animation-duration: 3.5s, 10.5s;
  animation-delay: 0s, 3.5s;
  animation-iteration-count: 1, infinite;
}

#move-2-1:checked ~ .slide-wrap .slide-3,
#move-3-2:checked ~ .slide-wrap .slide-1,
#move-1-3:checked ~ .slide-wrap .slide-2 {
  animation-delay: 3.5s;
}

```

예를 들어 move-2-1의 경우 2가 사라지고 1이 나타나는 것이기 때문에 2에 be-hide, 1에 be-show 가 적용된다. 이후, 그 다음 슬라이드로 이동해야 하므로 2가 slide-1, 3이 slide-2, 1이 slide-3 이 되어 적용된다. 애니메이션이 2가지 적용되는 slide 의 경우 duration, delay, iteration-count 를 각각 적용해줄 필요가 있는데 be-show, be-hide 의 경우 움직였다가(0.5s) 멈추는(3s) 동작 합해서 3.5s의 duration 을 지니고, slide-n 의 경우 정방향과 동일하게 10.5s 로 설정하되, delay 를 be-show/hide 시간만큼 기다려줄 수 있도록 3.5s 로 설정한다.
나머지 애니메이션이 하나만 포함되는 슬라이드의 경우 다른 슬라이드의 be-show/hide 시간을 기다려주기 위해, 3.5s 의 delay를 설정한다.

여기까지 완성하였다면, 각 radio 를 클릭했을 때 해당 방향으로 이동하는 것까지 완성되었을 것이다. radio 를 클릭해서 move-1-2 라면 첫번째 슬라이드에서 두번째로 이동하는지 확인해 보자.

그 다음은 각 control 을 클릭하였을 때 올바른 방향으로 이동시킬 수 있도록 label을 삽입해야 한다. label-1 은 첫번째 슬라이드가 화면에 보이는 경우, label-2는 두번째 슬라이드, label-3은 세번째 슬라이드가 화면에 보이는 경우 활성화 시킬 것이다.

```html
<div class="control-wrap">
    <div class="control">
      <label for="move-2-1" class="label label-2"></label>
      <label for="move-3-1" class="label label-3"></label>
    </div>
    <div class="control">
      <label for="move-1-2" class="label label-1"></label>
      <label for="move-3-2" class="label label-3"></label>
    </div>
    <div class="control">
      <label for="move-1-3" class="label label-1"></label>
      <label for="move-2-3" class="label label-2"></label>
    </div>
  </div>
```

move-2-1 은 2에서 1로 이동하기 때문에, 2번째 슬라이드가 화면에 보여지게 되었을 때 첫번째 control 에서 활성화 되어야 한다. 따라서 label-2 클래스로, 첫번째 control 에 삽입된다. 동일한 방법으로 나머지 label들을 적절하게 삽입하여 주면, label 쪽 HTML은 완성된다. 이어서 CSS 로 각 label 들이 슬라이드가 움직일 때마다 적절하게 활성화될 수 있도록, z-index 를 사용한 @keyframes 를 만들어 준다. 해당 @keyframes 의 시간은 슬라이드와 동일하므로, 똑같이 복사해서 z-index 로 바꿔주기만 하면 된다.

```css
@keyframes label-1 {
  /* 0s 3 */
  0% { z-index: -1; }
  /* 0.5s 1 */
  4.7619% { z-index: 1; }
  /* 3.5s 1 */
  33.333% { z-index: 1; }
  /* 4s 2 */
  38.095% { z-index: -1; }
  /* 7s 2 */
  66.666% { z-index: -1; }
  /* 7.5s 3 */
  71.428% { z-index: -1; }
  /* 10.5s 3 */
  100% { z-index: -1; }
}

@keyframes label-2 {
  /* 0s 3 */
  0% { z-index: -1; }
  /* 0.5s 1 */
  4.7619% { z-index: -1; }
  /* 3.5s 1 */
  33.333% { z-index: -1; }
  /* 4s 2 */
  38.095% { z-index: 1; }
  /* 7s 2 */
  66.666% { z-index: 1; }
  /* 7.5s 3 */
  71.428% { z-index: -1; }
  /* 10.5s 3 */
  100% { z-index: -1; }
}

@keyframes label-3 {
  /* 0s 3 */
  0% { z-index: -1; }
  /* 0.5s 1 */
  4.7619% { z-index: -1; }
  /* 3.5s 1 */
  33.333% { z-index: -1; }
  /* 4s 2 */
  38.095% { z-index: -1; }
  /* 7s 2 */
  66.666% { z-index: -1; }
  /* 7.5s 3 */
  71.428% { z-index: 1; }
  /* 10.5s 3 */
  100% { z-index: 1; }
}
```

이렇게 하면 label 에 대한 @keyframes 가 완성되었다. label-1, label-2, label-3 은 각각 slide-1, slide-2, slide-3 과 동일하게 해당 슬라이드에 맞춰 label에 적용시켜주면 정상적으로 작동할 것이다.

```css

.control .label {
  animation-duration: 10.5s;
  animation-iteration-count: infinite;
}


#move-2-1:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-2-1:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-2-1:checked ~ .control-wrap .label-3 { animation-name: label-3; }
#move-3-1:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-3-1:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-3-1:checked ~ .control-wrap .label-3 { animation-name: label-3; }

#move-1-2:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-1-2:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-1-2:checked ~ .control-wrap .label-3 { animation-name: label-2; }
#move-3-2:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-3-2:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-3-2:checked ~ .control-wrap .label-3 { animation-name: label-2; }

#move-1-3:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-1-3:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-1-3:checked ~ .control-wrap .label-3 { animation-name: label-1; }
#move-2-3:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-2-3:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-2-3:checked ~ .control-wrap .label-3 { animation-name: label-1; }

```

label 과 slide 의 차이점은 label 의 경우 z-index 가 사용자에게 시각적으로 변화가 보여지지 않기 때문에, 정방향인지 역방향인지를 신경 쓸 필요가 없다는 것이다. 따라서 move-2-1 이던 move-3-1 이던 결국 보여지기 시작하는 건 첫번째 슬라이드이기 때문에, label-1 이 첫번째가 되도록 설정하고, move-1-2이던, move-3-2이던 2부터 시작하기 때문에 label-2가 첫번째가 되도록 설정하면 되는 것이다.

여기까지 진행했다면, 각 버튼을 클릭하였을 때 정상적으로 각 슬라이드로 이동하는 것을 확인할 수 있을 것이다. 다만 아직 문제점 3이 해결되지 않았는데, 두번째 control 을 클릭해서 두번째 슬라이드로 이동한 상태에서 다시 두번째 control 을 클릭하면 동일한 애니메이션으로 변경되는 것이기 때문에 아무런 작동을 하지 않는다는 문제점이 있다.

해당 문제를 해결하기 위하여 각 @keyframes 의 복사본을 제작하고, radio 도 복사본을 만들어, 복사본 radio 가 checked 가 되었다면 복사본 @keyframes 가 지정되도록 수정해주면 된다.

기존 radio 아래에 다음과 같이 copy본을 추가한다.

```html
<input type="radio" name="s" id="move-2-1-copy">
<input type="radio" name="s" id="move-3-1-copy">
<input type="radio" name="s" id="move-1-2-copy">
<input type="radio" name="s" id="move-3-2-copy">
<input type="radio" name="s" id="move-1-3-copy">
<input type="radio" name="s" id="move-2-3-copy">
```

또한, 기존 label 아래에도 마찬가지로 copy 본을 만들어 연결해준다.

```html
<div class="control">
  <label for="move-2-1" class="label label-2"></label>
  <label for="move-3-1" class="label label-3"></label>

  <!-- 추가된 부분 -->
  <label for="move-2-1-copy" class="label label-2-copy"></label>
  <label for="move-3-1-copy" class="label label-3-copy"></label>
  <!-- //추가된 부분 -->
</div>
<div class="control">
  <label for="move-1-2" class="label label-1"></label>
  <label for="move-3-2" class="label label-3"></label>

  <!-- 추가된 부분 -->
  <label for="move-1-2-copy" class="label label-1-copy"></label>
  <label for="move-3-2-copy" class="label label-3-copy"></label>
  <!-- //추가된 부분 -->
</div>
<div class="control">
  <label for="move-1-3" class="label label-1"></label>
  <label for="move-2-3" class="label label-2"></label>

  <!-- 추가된 부분 -->
  <label for="move-1-3-copy" class="label label-1-copy"></label>
  <label for="move-2-3-copy" class="label label-2-copy"></label>
  <!-- //추가된 부분 -->
</div>
```

이제 모든 애니메이션 @keyframes를 복사해서, 복사한 radio 가 선택된 경우 각 슬라이드의 @keyframes 의 복사본이 선택되도록 하면 된다.

```css
@keyframes slide-1-copy {
  /* 0s 3 */
  0% { left: -100%; }
  /* 0.5s 1 */
  4.7619% { left: 0; }
  /* 3.5s 1 */
  33.333% { left: 0; }
  /* 4s 2 */
  38.095% { left: 100%; }
  /* 7s 2 */
  66.666% { left: 100%; }
  /* 7.5s 3 */
  71.428% { left: 100%; }
  /* 10.5s 3 */
  100% { left: 100%; }
}

@keyframes slide-2-copy {
  /* 0s 3 */
  0% { left: -100%; }
  /* 0.5s 1 */
  4.7619% { left: -100%; }
  /* 3.5s 1 */
  33.333% { left: -100%; }
  /* 4s 2 */
  38.095% { left: 0%; }
  /* 7s 2 */
  66.666% { left: 0%; }
  /* 7.5s 3 */
  71.428% { left: 100%; }
  /* 10.5s 3 */
  100% { left: 100%; }
}

@keyframes slide-3-copy {
  /* 0s 3 */
  0% { left: 0%; }
  /* 0.5s 1 */
  4.7619% { left: 100%; }
  /* 3.5s 1 */
  33.333% { left: 100%; }
  33.333333% { left: -100%; }
  /* 4s 2 */
  38.095% { left: -100%; }
  /* 7s 2 */
  66.666% { left: -100%; }
  /* 7.5s 3 */
  71.428% { left: 0%; }
  /* 10.5s 3 */
  100% { left: 0%; }
}

#move-1-2-copy:checked ~ .slide-wrap .slide-1 { animation-name: slide-3-copy; }
#move-1-2-copy:checked ~ .slide-wrap .slide-2 { animation-name: slide-1-copy; }
#move-1-2-copy:checked ~ .slide-wrap .slide-3 { animation-name: slide-2-copy; }

#move-2-3-copy:checked ~ .slide-wrap .slide-1 { animation-name: slide-2-copy; }
#move-2-3-copy:checked ~ .slide-wrap .slide-2 { animation-name: slide-3-copy; }
#move-2-3-copy:checked ~ .slide-wrap .slide-3 { animation-name: slide-1-copy; }

#move-3-1-copy:checked ~ .slide-wrap .slide-1 { animation-name: slide-1-copy; }
#move-3-1-copy:checked ~ .slide-wrap .slide-2 { animation-name: slide-2-copy; }
#move-3-1-copy:checked ~ .slide-wrap .slide-3 { animation-name: slide-3-copy; }

@keyframes be-hide-copy {
  0% { left: 0; }
  14.2857% { left: -100%; }
  100% { left: -100%; }
}
@keyframes be-show-copy {
  0% { left: 100%; }
  14.2857% { left: 0; }
  100% { left: 0; }
}

#move-2-1-copy:checked ~ .slide-wrap .slide-1 { animation-name: be-show-copy, slide-3-copy; }
#move-2-1-copy:checked ~ .slide-wrap .slide-2 { animation-name: be-hide-copy, slide-1-copy; }
#move-2-1-copy:checked ~ .slide-wrap .slide-3 { animation-name: slide-2-copy; }

#move-3-2-copy:checked ~ .slide-wrap .slide-1 { animation-name: slide-2-copy; }
#move-3-2-copy:checked ~ .slide-wrap .slide-2 { animation-name: be-show-copy, slide-3-copy; }
#move-3-2-copy:checked ~ .slide-wrap .slide-3 { animation-name: be-hide-copy, slide-1-copy; }

#move-1-3-copy:checked ~ .slide-wrap .slide-1 { animation-name: be-hide-copy, slide-1-copy; }
#move-1-3-copy:checked ~ .slide-wrap .slide-2 { animation-name: slide-2-copy; }
#move-1-3-copy:checked ~ .slide-wrap .slide-3 { animation-name: be-show-copy, slide-3-copy; }

#move-2-1-copy:checked ~ .slide-wrap .slide-1,
#move-2-1-copy:checked ~ .slide-wrap .slide-2,
#move-3-2-copy:checked ~ .slide-wrap .slide-2,
#move-3-2-copy:checked ~ .slide-wrap .slide-3,
#move-1-3-copy:checked ~ .slide-wrap .slide-1,
#move-1-3-copy:checked ~ .slide-wrap .slide-3 {
  animation-duration: 3.5s, 10.5s;
  animation-delay: 0s, 3.5s;
  animation-iteration-count: 1, infinite;
}

#move-2-1-copy:checked ~ .slide-wrap .slide-3,
#move-3-2-copy:checked ~ .slide-wrap .slide-1,
#move-1-3-copy:checked ~ .slide-wrap .slide-2 {
  animation-delay: 3.5s;
}

```

해당 작업까지 완료되었다면, 이제 각 radio 가 checked 되었을 때 보여질 label 을 컨트롤 할 시간이다. 같은 control 을 클릭했을 때 서로 중복된 애니메이션이 선택되지 않도록, 원본 radio 가 checked 가 되면 복사본을, 복사본 radio 가 checked 되면 원본이 클릭될 수 있게끔 CSS 를 수정한다.

```css
/* 
기존에 있던 애니메이션 설정은 삭제한다.
#move-2-1:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-2-1:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-2-1:checked ~ .control-wrap .label-3 { animation-name: label-3; }
#move-3-1:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-3-1:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-3-1:checked ~ .control-wrap .label-3 { animation-name: label-3; }

#move-1-2:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-1-2:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-1-2:checked ~ .control-wrap .label-3 { animation-name: label-2; }
#move-3-2:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-3-2:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-3-2:checked ~ .control-wrap .label-3 { animation-name: label-2; }

#move-1-3:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-1-3:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-1-3:checked ~ .control-wrap .label-3 { animation-name: label-1; }
#move-2-3:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-2-3:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-2-3:checked ~ .control-wrap .label-3 { animation-name: label-1; } */


#move-2-1:checked ~ .control-wrap .label-1-copy { animation-name: label-1; }
#move-2-1:checked ~ .control-wrap .label-2-copy { animation-name: label-2; }
#move-2-1:checked ~ .control-wrap .label-3-copy { animation-name: label-3; }
#move-3-1:checked ~ .control-wrap .label-1-copy { animation-name: label-1; }
#move-3-1:checked ~ .control-wrap .label-2-copy { animation-name: label-2; }
#move-3-1:checked ~ .control-wrap .label-3-copy { animation-name: label-3; }

#move-1-2:checked ~ .control-wrap .label-1-copy { animation-name: label-3; }
#move-1-2:checked ~ .control-wrap .label-2-copy { animation-name: label-1; }
#move-1-2:checked ~ .control-wrap .label-3-copy { animation-name: label-2; }
#move-3-2:checked ~ .control-wrap .label-1-copy { animation-name: label-3; }
#move-3-2:checked ~ .control-wrap .label-2-copy { animation-name: label-1; }
#move-3-2:checked ~ .control-wrap .label-3-copy { animation-name: label-2; }

#move-1-3:checked ~ .control-wrap .label-1-copy { animation-name: label-2; }
#move-1-3:checked ~ .control-wrap .label-2-copy { animation-name: label-3; }
#move-1-3:checked ~ .control-wrap .label-3-copy { animation-name: label-1; }
#move-2-3:checked ~ .control-wrap .label-1-copy { animation-name: label-2; }
#move-2-3:checked ~ .control-wrap .label-2-copy { animation-name: label-3; }
#move-2-3:checked ~ .control-wrap .label-3-copy { animation-name: label-1; }


#move-2-1-copy:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-2-1-copy:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-2-1-copy:checked ~ .control-wrap .label-3 { animation-name: label-3; }
#move-3-1-copy:checked ~ .control-wrap .label-1 { animation-name: label-1; }
#move-3-1-copy:checked ~ .control-wrap .label-2 { animation-name: label-2; }
#move-3-1-copy:checked ~ .control-wrap .label-3 { animation-name: label-3; }

#move-1-2-copy:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-1-2-copy:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-1-2-copy:checked ~ .control-wrap .label-3 { animation-name: label-2; }
#move-3-2-copy:checked ~ .control-wrap .label-1 { animation-name: label-3; }
#move-3-2-copy:checked ~ .control-wrap .label-2 { animation-name: label-1; }
#move-3-2-copy:checked ~ .control-wrap .label-3 { animation-name: label-2; }

#move-1-3-copy:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-1-3-copy:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-1-3-copy:checked ~ .control-wrap .label-3 { animation-name: label-1; }
#move-2-3-copy:checked ~ .control-wrap .label-1 { animation-name: label-2; }
#move-2-3-copy:checked ~ .control-wrap .label-2 { animation-name: label-3; }
#move-2-3-copy:checked ~ .control-wrap .label-3 { animation-name: label-1; }

```

여기까지 완료되었다면 슬라이드 구현이 완성되었다.
