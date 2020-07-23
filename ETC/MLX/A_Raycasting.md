# A. Ray Casting

> [이 링크의 자료](https://lodev.org/cgtutor/raycasting.html)를 대략적으로 번역하였습니다.

[🏠Home](https://github.com/batboy118/Study_Note)

[◀Previous page ](./README.md)

---

<!-- TOC -->

- [1. Introduction](#1-introduction)
- [2. The Basic Idea](#2-the-basic-idea)
- [3. Untextured Raycaster](#3-untextured-raycaster)

<!-- /TOC -->

## 1. Introduction

레이캐스팅은 2D 지도에서 3D 관점을 만드는 렌더링 기법이다. 컴퓨터가 느렸던 시절에는, 실시간 3D 엔진을 가동할 수 없었고, 레이캐스팅이 그 첫 번째 해결책이었다. 레이캐스팅은 화면의 모든 수직선에 대해 계산만 하면 되기 때문에 매우 빠르 빠른 렌더링 기법이다. 이 기술을 사용한 가장 잘 알려진 게임은 당연히 울펜슈타인 3D이다.

![image](https://user-images.githubusercontent.com/53181778/76977795-b6d3cc80-692d-11ea-8aea-f3b4de99e9d3.png)

울펜슈타인 3D의 레이캐스팅 엔진은 286 컴퓨터에서도 작동할 수 있었다. 모든 벽은 높이가 같고 2D 그리드의 직교 사각형이다.

![image](https://user-images.githubusercontent.com/53181778/76977801-b9362680-692d-11ea-9e92-943001e045f1.png)

이 엔진으로는 계단, 점프, 높이 차이 같은 것들을 만들 수 없다. 이후 둠과 듀크누켐 3D 같은 게임도 레이캐스팅을 사용했지만, 훨씬 더 진보된 엔진으로 경사진 벽, 다른 높이, 질감 있는 바닥과 천장, 투명한 벽 등이 가능했다. 스프라이트(적, 물체 및 굿즈)는 2D 이미지지만, 스프라이트는 이 튜토리얼에서 논의하지 않을 것이다.

레이캐스팅은 레이트레이싱과 같지 않다! 레이캐스팅은 4MHz 그래픽 계산기에서도 실시간으로 작동하는 빠른 semi-3D 기법으고, 반면에 레이트레이싱은 실제 3D 장면에서 반사와 그림자를 지원하는 현실적인 렌더링 기법이다. 레이트레이싱은 최근에 와서야 충분히 빠른 컴퓨터에서 사용해야 납득할만한 해상도와 복잡한 장면을 실시간으로 렌더링 할 수 있다. (대충 레이트레이싱이 더 고사양을 필요로 한다는 의미)

질감이 없고 질감이 없는 레이캐스터 코드와, 질감이 있는 레이캐스터의 코드가 이 문서에 완전히 주어지지만, 길다. 너는 대신 아래 코드를 다운로드 할 수 있다.

[raycaster_flat.cpp](https://lodev.org/cgtutor/files/raycaster_flat.cpp)
[raycaster_textured.cpp](https://lodev.org/cgtutor/files/raycaster_textured.cpp)

## 2. The Basic Idea

레이캐스팅의 기본 개념은 다음과 같다 : 지도는 2D 사각형 격자로, 각 정사각형은 0(= 벽 없음) 또는 양수 값(= 특정 색이나 질감을 가진 벽)이 될 수 있다.

화면의 모든 x에 대해(즉, 격자의 모든 수직 줄 에 대해) 플레이어가 서있는 곳에서 시작되어, 플레이어가 보는 방향과 화면의 x좌표와 연관되어 광이 뻗어 나간다. 그런 다음, 이 광선이 벽에 해당하는 정사각형에 부딪힐 때까지 2D 지도에서 앞으로 나아가게 한다. 만약 그것이 벽에 부딪히면, 플레이어와 부딪힌 점의 거리를 계산하고, 이 거리를 이용하여 이 벽이 화면에 얼마나 높이 그려져야 하는가를 계산한다: 벽이 멀리 떨어져 있을수록 화면에 더 작아지고, 가까이 있을수록 더 높게 나타난다(원근감 표현). 이것들은 모두 2차원 계산이다. 아래 이미지는 플레이어에서 시작하여 파란색 벽에 부딪히는 두 개의 그러한 광선(빨간색)에 대한 하향 개요를 보여준다.

![image](https://user-images.githubusercontent.com/53181778/76979351-b76d6280-692f-11ea-8129-20f8e46f35b6.png)

광선이 직진하다가 도중에 마주치는 첫 번째 벽을 찾으려면, 플레이어의 위치에서 광선이 출발한 후로 광선이 벽 안에 있는지 여부를 항상 확인해야 한다. 만약 광선이 벽 안에 있다면, 광선이 벽안에 있는지 확인하는 루프는 멈추고, 플레이어와 광선의 거리를 계산해서 벽을 정확한 높이로 그릴 수 있다. 만약 광선 위치가 벽에 있지 않다면, 더 추적해야 한다: 이 광선의 방향으로 해당 위치에 특정한 값을 더해주어야 한다. 그리고 새로운 위치에 대해서는 다시 그것이 벽 안에 있는지 여부를 확인한다. 벽에 부딪칠 때까지 이것을 계속해라.

인간은 그 광선이 벽에 부딪히는 곳을 바로 볼 수 있지만, 컴퓨터는 그 광선 위의 제한된 수의 위치들만 확인할 수 있기 때문에, 그 광선이 하나의 공식으로 어떤 정사각형을 바로 맞히는지는 찾을 수 없다. 많은 레이캐스터들은 각 단계마다 일정한 값을 광선에 더하지만, 그러면 벽을 놓칠 가능성이 있다! 예를 들어, 아래 적색선을 사용하여 모든 적색 지점에서 위치가 확인되었다.

![image](https://user-images.githubusercontent.com/53181778/76979367-bc321680-692f-11ea-9370-073d56f7d370.png)

보시다시피 광선은 파란 벽을 뚫고 직진하지만 컴퓨터가 이를 감지하지 못한 것은 빨간 점으로 위치만 확인했기 때문이다. 위치를 더 많이 확인할수록 컴퓨터가 벽을 감지하지 못할 가능성은 작아지지만 계산은 더 필요하다. 아래 그림에서는 각 단계의 거리는 반감되었으므로, 지금은 위치가 완전히 정확하지는 않지만, 광선이 벽을 통과했음을 감지한다.

![image](https://user-images.githubusercontent.com/53181778/76979418-cfdd7d00-692f-11ea-90a1-eeaa992456e2.png)

이 방법의 무한정 정밀도를 위해서는 무한히 작은 스텝 사이즈가 필요하며, 따라서 무한정 많은 계산이 필요할 것이다! 하지만, 다행히도, 아주 적은 계산만을 가지고 모든 벽을 감지할 수 있는 더 나은 방법이 있다: 그 아이디어는 그 광선이 마주칠 벽의 모든 면을 확인하는 것이다. 우리는 각 정사각형 폭 1을 주므로 벽의 각 측면은 정수 값이고 그 사이의 위치는 점 뒤에 값을 가진다. 이제 스텝 크기는 일정하지 않고, 다음 사각형의 변까지의 거리에 따라 달라진다.

![image](https://user-images.githubusercontent.com/53181778/76979443-d9ff7b80-692f-11ea-95ea-7bf809217396.png)

위의 이미지에서 볼 수 있듯이, 그 광선은 우리가 원하는 곳에 정확하게 벽에 부딪힌다. 본 자습서에 제시된 방식에서, DDA(Digital Differential Analysis)에 근거한 알고리즘을 사용한다. DDA는 일반적으로 사각형 그리드에 사용되는 고속 알고리즘으로 선이 어떤 정사각형이 맞는지(예: 사각형 픽셀의 그리드인 화면에 선을 긋는 경우) 찾아낸다. 그래서 우리는 이것을 이용해서 우리의 광선이 어떤 정사각형을 치는지 찾아내고, 벽인 정사각형을 치게 되면 알고리즘을 정지시킬 수 있다.

일부 raytracer는 유클리드 각도를 이용하여 플레이어가 보는 방향과 광선을 나타내고 다른 각도로 시야를 결정한다. 하지만 나는 벡터와 카메라로 대신 일하는 것이 훨씬 쉽다는 것을 알아차렸다: 플레이어의 위치는 항상 벡터(x와 y 좌표)이지만, 거기에 더해 이제 우리는 방향을 벡터로 만든다: 그래서 방향은 방향의 x와 y 좌표라는 두 가지 값으로 정할 수 있다. 방향 벡터는 다음과 같이 볼 수 있다: 플레이어의 위치로 플레이어가 보는 방향으로 선을 그으면 라인의 모든 지점은 플레이어의 위치의 좌표 + 벡터의 배수이다. 방향 벡터의 길이는 실제로 중요하지 않고, 방향만 중요하다. 어떤 벡터의 x와 y에 같은 값으로 곱하면 길이는 변경되지만 방향은 동일하게 유지된다.

벡터를 이용한 이 방법은 또한 하나의 벡터가 더 필요한데, 그 벡터는 카메라 평면 벡터이다. 실제 3D 엔진에는 카메라 평면이 있고, 3D 평면이기 때문에 카메라 평면을 나타내기 위해서는 두 개의 벡터(u와 v)가 필요하다. 하지만, 레이캐스팅은 2D 지도에서 일어나기 때문에 여기 카메라 평면은 실제로는 평면이 아니라 `선`이며 `단일 벡터`로 표현된다. 카메라 평면은 항상 방향 벡터에 수직이어야 한다. 카메라 평면은 컴퓨터 화면의 표면을 나타내는 반면, 방향 벡터는 컴퓨터 화면의 표면에 수직이며 화면 내부를 가리키고 있다. 하나의 점으로 표현된 플레이어의 위치가 카메라 평면 앞에 있는 점이다. 화면의 x-좌표 상의 광선은, 플레이어 위치에서 시작하여 화면(카메라 평면)의 해당 위치를 통과하는 레이입니다.

![image](https://user-images.githubusercontent.com/53181778/77065911-0bcd1c80-69da-11ea-9419-c670f7a176ae.png)

위의 이미지는 일종의 2D 카메라를 나타낸다. 녹색 점은 `위치(벡터 "pos")`이다. 검은 점에서 끝나는 검은 선은 `방향 벡터(벡터 "dir")`를 나타내고, 검은 점의 위치는 `pos+dir`이다. 파란색 선은 전체 카메라 평면을 나타내고, 검은색 점부터 오른쪽 파란색 점까지의 벡터는 `벡터 "plane"`을 나타내며, 따라서 오른쪽 파란색 점의 위치는 `pos+dir+plane`이고, 왼쪽 파란색 점의 포지션은 `pos+dir-plane`이다(이들은 모두 벡터 덧셈이다).

> 위치 : pos
>
> 방향 벡터 (카메라 벡터) : dir
>
> 평면 벡터 (카메라 평면) : plane

이미지의 빨간선은 몇개의 광선을 의미한다. 광선의 방향은 카메라를 이용해 쉽게 계산된다:카메라 벡터와 카메라의 평면 벡터의 합이다 : 예를 들면 3번째 빨간 광선은 카메라 평면의 오른쪽 1/3길이 지점을 통과한다. 그렇기 때문에, 3번 째 광선의 방향은 `dir  + plane/3` 으로 구할 수 있다. 이 광선의 방향은 `벡터 rayDir`이고, 이 벡터의 X와 Y요소는 DDA알고리즘에 활용된다.

> 광선의 방향 : rayDir

두 개의 바깥선(스크린의 좌우 가장 자리)과 이 두 선 사이의 각도를 `시야` 또는 `FOV`라고 한다. FOV는 방향 벡터의 길이와 평면 벡터 길이의 비로 결정된다. 다음은 다양한 FOV의 몇 가지 예들이다.

방향 벡터와 카메라 평면 벡터의 길이가 같은 경우 FOV는 90°가 된다.

![image](https://user-images.githubusercontent.com/53181778/77065912-0d96e000-69da-11ea-9573-e40d021bd101.png)

방향 벡터가 카메라 평면보다 훨씬 길면 FOV는 90°보다 훨씬 작고 시야도 매우 좁을 겁니다. 하지만 모든 것이 보다 상세하게 보일 것이고 깊이가 줄어들 것이다. 따라서 이것은 확대와 같다.

![img](https://lodev.org/cgtutor/images/raycastingFOV0.gif)

방향 벡터가 카메라 평면보다 짧으면 FOV가 90°(방향 벡터가 0에 가까우면 180°가 최대 값)보다 커지며, 다음과 같이 시야가 훨씬 넓어진다.

![image](https://user-images.githubusercontent.com/53181778/77065922-112a6700-69da-11ea-9a15-a1b1111b8d9e.png)



플레이어가 회전할 때, 카메라도 회전하기 때문에, 방향 벡터와 평면 벡터 또한 회전하게 된다. 그러면, 광선들도 자동적으로 화전하게 된다.

![image](https://user-images.githubusercontent.com/53181778/77065927-14255780-69da-11ea-97dc-b109d11243c6.png)

벡터를 회전하려면 회전 행렬과 함께 벡터를 곱하면된다.

> 회전 벡터

|      | 1열    | 2열     |
| ---- | ------ | ------- |
| 1행  | cos(a) | -sin(a) |
| 2행  | sin(a) | cos(a)  |


벡터 및 매트릭스에 대해 모르는 경우, 이 튜토리얼에 대한 부록을, Google을 사용하여 찾아보세요.

방향 벡터와 수직이 아닌 카메라 평면을 사용하는 것을 금지하지는 않지만, 그 결과는 마치 '왜곡된' 세계처럼 보일 것이다.

## 3. Untextured Raycaster

Download the source code here: [raycaster_flat.cpp](https://lodev.org/cgtutor/files/raycaster_flat.cpp)

기초부터 시작하기 위해 질감이 없는 레이캐스터부터 시작한다. 이 예에는 fps 카운터(초당 프레임)와 이동 및 회전을 위한 충돌 감지 입력 키도 포함된다.

세계의 지도는 2D 배열로, 각 값은 정사각형을 나타낸다. 값이 0이면 정사각형은 비어 있는, 즉 걸어다닐 수 있는 정사각형을 의미하고, 값이 0보다 크면 일정한 색이나 질감을 가진 벽을 의미한다. 여기서 선언된 지도는 매우 작으며, 24x24의 제곱에 불과하며, 코드에 직접 정의되어 있다. 울펜슈타인 3D와 같은 실제 게임의 경우, 당신은 더 큰 지도를 사용하고 파일로 부터 그것을 로드한다. 격자 안의 0은 모두 빈 공간이기 때문에 기본적으로 매우 큰 방을 볼 수 있는데, 그 둘레에는 벽(값 1)이 있고, 그 안에 작은 방(값 2)이 있고, 몇 개의 털로 덮인(값 3)것이 있고, 방이 있는 복도(값 4)가 있다. 이 맵 코드는 아직 어떤 함수에도 포함되어 있지 않으므로, main 함수가 시작되기 전에 맵을 넣으십시오.

```cpp
#define mapWidth 24
#define mapHeight 24
#define screenWidth 640
#define screenHeight 480

int worldMap[mapWidth][mapHeight]=
{
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,2,2,2,2,0,0,0,0,3,0,3,0,3,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,3,0,0,0,3,0,0,0,1},
  {1,0,0,0,0,0,2,0,0,0,2,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,2,2,0,2,2,0,0,0,0,3,0,3,0,3,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,4,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,0,0,0,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,0,0,0,5,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,0,0,0,0,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,4,4,4,4,4,4,4,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
  {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1}
};
```

처음 몇 가지 변수가 선언된다: `posX`와 `posY`는 플레이어의 위치 벡터를 나타내고, `dirX`와 `dirY`는 플레이어의 방향을 나타내며, `PlaneX`와 `PlanY`는 플레이어의 카메라 평면을 나타낸다. 카메라 평면은 플레이어의 방향에 수직으로 해야하지만, 길이는 변경해도 된다. 방향의 길이와 카메라 평면의 길이의 비가 FOV를 결정하는데, 여기서 방향 벡터는 카메라 평면보다 약간 길기 때문에 FOV는 90°보다 작을 것이다. 더 정확히 말하면 `FOV는 2 * atan(0.66/1.0)=66°`이다. 이는 1인칭 슈터 게임에 안성맞춤이다. 나중에 키를 입력하여 회전할 때 `dir`과 `plane`은 변하지만, 항상 수직으로 유지되고 동일한 길이를 유지할 것이다.

`time`과 `oldTime` 변수는 현재 프레임과 이전 프레임의 시간을 저장하는 데 사용되게 되된다. 이 둘의 시간차이는 플레이어가 특정한 키를 눌렀을 때 얼마만큼 이동시켜야 할지 결정할 수 있게 한다. (프레임의 계산이 얼마나 오려걸리는지와 상관없이 일정한 속도로 움직이기 위해서) 그리고, FPS(초당 프레임 수) 카운터에도 사용된다.

```cpp
int main(int /*argc*/, char */*argv*/[])
{
  double posX = 22, posY = 12;  //x and y start position
  double dirX = -1, dirY = 0; //initial direction vector
  double planeX = 0, planeY = 0.66; //the 2d raycaster version of camera plane

  double time = 0; //time of current frame
  double oldTime = 0; //time of previous frame
```

나머지 main 함수는 지금부터 시작한다. 첫째로, 선택된 해상도로 화면이 만들어 진다. 만약, 1280*1024와 같이 큰 해상도를 선택하면 꽤 느려질 수도 있다. 이것은 레이케이팅 알고리즘이 느려서가 아니라, 단순히 CPU에서 비디오 카드로 전체 화면을 업로드하는 것이 너무 느리기 때문이다.

```cpp
screen(screenWidth, screenHeight, 0, "Raycaster");
```


화면을 설정한 후, 게임의 루프가 시작되는데, 이것은 매번 전체 프레임을 그리고 입력을 읽는 루프다.

```cpp
while(!done())  {
```

여기서 실제 레이캐스팅이 시작된다. 레이캐스팅 루프는 모든 x만을 통과하는 for문이기 때문에 화면의 모든 픽셀에 대해 계산을 하는 것이 아니라, 모든 수직 줄무늬에 대한 계산만 하고(x만 따지고 y는 따지지 않는 다는 뜻), 이것은 전혀 많지 않은 계산량이다. 레이캐스팅 루프를 시작하기 위해 일부 변수를 선언하고 계산한다.

ray는 플레이어의 위치(posX, posY)에서 시작한다:

`cameraX`는 화면의 현재 x-좌표가 나타내는 카메라 평면의 x-좌표로서,  1이면 오른쪽을, 0이면 중앙을, -1이면 왼쪽을 의미한다.

이 와중에, 광선의 방향은 앞에서 설명한 대로 계산할 수 있다: `방향 벡터`와 `평면벡터의 일부`의 합으로 구할 수 있다. 이것은 벡터의 x와 y 좌표에 대해 모두 수행되어야 한다(두 벡터를 더하면 x 좌표와  y좌표가 모두 더해진다.)

```cpp
for(int x = 0; x < w; x++)
{
//calculate ray position and direction
	double cameraX = 2 * x / double(w) - 1; //x-coordinate in 	camera space
	double rayDirX = dirX + planeX * cameraX;
	double rayDirY = dirY + planeY * cameraX;
```

다음 코드 조각에서는 더 많은 변수가 선언되고 계산되며, 이것들은 DDA 알고리즘과 관련이 있다.

mapX와 mapY는 광선이 위치한 지도의 현재 정사각형을 의미한다. 광선 위치 자체는 부동 소수점 숫자로, 우리가 어느 정사각형에 있는지와 그 정사각형 안 어디에 있는지의 대한 정보를 모두 포함하지만, mapX와 mapY는 그 정사각형의 좌표일 뿐이다.

`sideDistX`와 `sideDistY`는 처음 시작 지점에서 first x-side와 first y-side까지 광선이 가야할 거리이다. 나중에 코드에서는 이것들의 의미가 살짝 변하게 된다. (즉, 다음 정사각형을 만나는데 x축이나 y축 좌표에 만날 것이다. 이때 처음 만나는 y 좌표까지 가야하는 거리가 `sideDistY`이고, 처음 만나는 x 좌표까지의 거리가 `sideDistX` 이다.)

`deltaDistX` 와 `deltaDistY` 는 광선이 가야할 거리인데, `deltaDistX` 는 첫 번째 x-side부터 다음 x-side 까지의 거리이고, `deltaDistY`  첫 번째 y-side부터 다음 y-side 까지의 거리를 의미한다.

다음 이미지는 초기 sideDistX, sideDistY 및 deltaDistX 및 deltaDistY를 보여준다.:

![image](https://user-images.githubusercontent.com/53181778/77065939-1982a200-69da-11ea-8261-31c049f030ff.png)

`deltaDistX` 기하학적으로 도출하면, 피타고라스 정리로다음과 같은 공식들을 얻을 수 있다.

`deltaDistX = sqrt(1 + (rayDirY * rayDirY) / (rayDirX * rayDirX))`

=> x축 길이를 1

`deltaDistY = sqrt(1 + (rayDirX * rayDirX) / (rayDirY * rayDirY))`

=> y축 길이를 1

그리고 더 간단하게 아래와 같이 나타낼 수 있다.

`deltaDistX = abs(|v| / rayDirX)`
`deltaDistY = abs(|v| / rayDirY)`

`|v|`는 ` vector rayDirX, rayDirY`의 길이(`sqrt(rayDirX * rayDirX + rayDirY * rayDirY)`)이다. 그러나, `|v|`대신 `1`을 사용 할 수 있다. 왜냐하면, 단지 `deltaDistX `와 `deltaDistY` 사이의 비만 DDA 알고리즘에 중요하기 때문이다. 그래서 아래와 같이 다시 정리가 가능하다.

`deltaDistX = abs(1 / rayDirX)`
`deltaDistY = abs(1 / rayDirY)`

변수 `perpWallDist`는 나중에 광선 길이를 계산하는 데 사용될 것이다.

DDA 알고리즘은 항상 각 루프마다 x 방향의 사각형 또는 y 방향의 사각형을 정확하게 점프한다. x의 음의 방향으로 가는지 양의 방향으로 가는지,  y의 양의 방향으로 가는지 음의 방향으로 가는지는 광선의 방향에 따라 달라지며, 이 것은 `stepX`와 `stepY`에 저장될 것이다. 이 두 변수는 항상 -1 또는 +1이다.

마지막으로, 히트(`hit`)는 루프의 종료 여부를 결정하기 위해 사용되며, 측면에는 벽의 x 측이나 y 측이 부딪혔는지 여부를 저장한다. x-side를 맞았을 경우, 측면은 `0`으로 설정되며, y-side를 맞았을 경우, 측면은 `1`이 된다. x-side와 y-side는 두 정사각형 사이의 경계선인 격자선을 의미한다.

```cpp
//which box of the map we're in
int mapX = int(posX);
int mapY = int(posY);

//length of ray from current position to next x or y-side
double sideDistX;
double sideDistY;

//length of ray from one x or y-side to next x or y-side
double deltaDistX = std::abs(1 / rayDirX);
double deltaDistY = std::abs(1 / rayDirY);
double perpWallDist;

//what direction to step in x or y-direction (either +1 or -1)
int stepX;
int stepY;

int hit = 0; //was there a wall hit?
int side; //was a NS or a EW wall hit?
```

참고 : `layDirX` 또는 `layDirY`가 0이면 위 코드에서 `DeltaDistX` 또는 `DeltaDistY`를 0으로 나누게 되어 무한대 값이 된다. 만약 시스템에서 IEEE 754 부동 소수점 표준을 사용하고 있고, 이에 대한 예외를 적용하지 않는 경우 괜찮다.(예: C++, Java 또는 JS를 사용하는 경우는 괜찮지만 Python은 이를 허용하지 않는다.) : 무한대는 아래 DDA 단계의 비교에서 올바르게 사용될 것이다. 그러나 이것을 허용하지 않는 프로그래밍 언어를 사용하면 다음과 같이 사용할 수 있으며, 이 코드는 유한한 것을 0으로 설정함으로써 DDA 루프도 올바르게 동작하게 된다.

```cpp
// Alternative code for deltaDist in case division through zero is not supported
double deltaDistX = (rayDirY == 0) ? 0 : ((rayDirX == 0) ? 1 : abs(1 / rayDirX));
double deltaDistY = (rayDirX == 0) ? 0 : ((rayDirY == 0) ? 1 : abs(1 / rayDirY));
```

이제 실제 DDA를 시작하기 전에 첫 번째 `stepX`, `stepY` 및 초기 `sideDistX` 및 `sideDistY`를 계산해야 한다.

광 방향의 X 성분이 음수인 경우, `stepX`는 -1이고,  광방향에 양의 x 성분이 있는 경우 +1이 된다. x-구성 요소가 0이면 stepX가 어떤 값을 가지는지는 중요하지 않다. 그 이유는 사용하지 않을 것이기 때문이다. y 성분도 마찬가지다.

광방향에 음의 x 성분이 있는 경우, `sideDistX`는 광선의 시작 위치에서 왼쪽으로 첫 번째 측면까지의 거리가 되며, 광선 방향이 양의 x-구성 요소를 가지고 있는 경우 오른쪽의 첫 번째 측면을 대신 사용한다.

y-구성 요소도 마찬가지로, 첫 번째 면의 위나 아래를 까지의 거리이다.

`sideDistX`와 `sideDistY`의 값을 구하기 위해, 정수값 `mapX`를 사용하여 그 값에서 실제 위치를 빼며, 상단 또는 하단의 왼쪽 또는 오른쪽 측면에 따라 일부 경우에 1.0을 추가한다. 그런 다음 측면까지의 수직 거리를 얻고, `deltaDistX` 또는 `deltaDistY`로 곱하여 실제 유클리드 거리를 구한다.

```c++
//calculate step and initial sideDist
if (rayDirX < 0)
{
    stepX = -1;
    sideDistX = (posX - mapX) * deltaDistX;
}
else
{
    stepX = 1;
    sideDistX = (mapX + 1.0 - posX) * deltaDistX;
}
if (rayDirY < 0)
{
    stepY = -1;
    sideDistY = (posY - mapY) * deltaDistY;
}
else
{
    stepY = 1;
    sideDistY = (mapY + 1.0 - posY) * deltaDistY;
}
```

이제 실제 DDA가 시작된다. 벽에 부딪힐 때까지 매번 한개의 정사각형만큼 광선을 증가시키는 반복문이다. 매번 X 방향(`stepX` )의 사각형이나 y 방향(`stepY` )의 사각형을 점프할 때마다 항상 한 번에 한 칸씩 점프한다. 광선의 방향이 x 방향이라면, 광선은 결코 y 방향을 바꾸지 않기 때문에 루프는 매번 x 방향의 사각형만 점프하면 된다. 만약 그 광선이 y방향으로 약간 기울어진다면, X방향으로 여러번 점프를 할 때, y방향으로는 한 칸씩 점프를 해야 할 것이다. 만약 그 광선이 정확히 y방향이라면, x방향으로 뛰어들 필요가 전혀 없다 등...

sideDistX와 sideDistY는 deltaDistX와 deltaDistY만큼 점프할 때 마다 증가하며, mapX와 mapY는 각각 stepX와 stepY로 증가된다.

광선이 벽에 부딪히면 루프가 끝나고, 그 다음 우리는 변수 "side"를 통해 벽의 x측면이나 y측면이 맞았는지 알게되고, 그리고 mapX와 mapY로 어떤 벽이 부딪혔는지 알 수 있을 것이다. 하지만 우리는 정확히 벽의 어디에 부딪혔는지 알 수는 없지만, 현재로서는 질감 있는 벽을 사용하지 않을 것이기 때문에 이 경우에는 알 필요가 없다.

```cpp
//perform DDA
while (hit == 0)
{
    //jump to next map square, OR in x-direction, OR in y-direction
    if (sideDistX < sideDistY)
    {
        sideDistX += deltaDistX;
        mapX += stepX;
        side = 0;
    }
    else
    {
        sideDistY += deltaDistY;
        mapY += stepY;
        side = 1;
    }
    //Check if ray has hit a wall
    if (worldMap[mapX][mapY] > 0) hit = 1;
}
```

DDA가 끝난 후에는 벽에 대한 광선 거리를 계산하여, 벽이 얼마나 높이 그려져야 하는지 계산할 수 있다.

우리는 플레이어를 나타내는 지점까지의 유클리드 거리를 사용하지 않고, 대신 fisheye 효과를 피하기 위해 카메라 평면까지의 거리(또는 플레이어에 대한 카메라 방향에 투영된 지점의 거리)를 사용한다. fisheye 효과는 모든 벽이 둥글게 되는 효과로, 실제 거리를 사용하면 볼 수있고, 회전하게 되면 어지러울 수 있다.

다음 이미지는 왜 우리가 플레이어 대신에 카메라 평면까지 거리를 측정하는지를 보여준다. P 플레이어와 검은색 라인 카메라 평면: 플레이어의 왼쪽에넌 벽의 hit point에서 플레이어 까지 적색선이 몇 개 나타나며 이는 유클리드 거리를 나타낸다. 플레이어의 오른쪽에는 플레이어가 아닌 벽의 hit point에서 카메라 평면으로 가는 몇 개의 녹색 광선이 보인다. 녹색 선의 길이는 우리가 직접 유클리드 거리 대신 사용할 수직 거리의 예 이다.

이미지에서 플레이어는 벽을 직접 보고 있는데, 그럴 경우 벽의 바닥과 윗부분이 화면에 완벽하게 수평선을 형성할 것으로 예상할 수 있을 것이다. 그러나, 적색 광선은 모두 다른 관절을 가지고 있기 때문에 다른 수직 줄무늬에 대해 다른 벽 높이를 계산할 수 있고, 따라서 둥글게 보이는 효과를 얻을 수 있다. 오른쪽의 녹색 광선은 모두 길이가 같으므로 정확한 결과를 얻을 것이다.

플레이어가 회전할 때 역시 동일하게 적용된다. (그러면 카메라 평면이 더 이상 수평이 아니고 녹색 라인이 서로 다른 길이를 가지지만, 여전히 서로 상수적인 변화만 있을 뿐이다.) 벽은 대각선이 되지만 화면에선 직선이 된다.

![perpWallDist](https://lodev.org/cgtutor/images/raycastdist.png)

코드의 이부분이 "fisheye correction"이 아니라는 점에 유의하십시오. 이러한 보정은 여기서 사용하는 raycasting의 방식에 필요하지 않으므로 fisheye 효과는 여기에서 거리가 계산되는 방식으로 간단히 방지할 수 있다. 실제 거리보다 이 수직 거리를 계산하는 것이 훨씬 쉬워, 우리는 벽이 부딪힌 정확한 위치조차 알 필요가 없다.

아래의 코드, `(1-stepX)/2`는 `stepX = -1`이면 `1`이고 `stepX = +1`이면 `0`인데, 이것은 우리가  `rayDirX < 0`일 때 길이에 `1`을 더해야 하기 때문에 필요한 것이다. 이것은 1.0이 초기 `sideDistX`의 한쪽에는 추가되었지만 다른 경우에는 추가되지 않은 것과 같은 이유 때문이다.

그 후 거리는 다음과 같이 계산된다: x측면이 부딪혔을 경우, `mapX-posX+(1-stepX)/2)`는 광선이 X방향으로 교차한 정사각형 수이다(이것이 반드시 전체 수는 아니다).

광선이 X측에 수직인 경우, 이것은 이미 정확한 값이지만, 대부분의 경우 광선의 방향이 다르기고 실제 수직 거리는 더 커지기 때문에, 우리는 rayDir 벡터의 X 좌표를 통해 이 값을 나눈다. y측면이 부딪히는 경우도 유사하다. `mapX-posX`는 `rayDirX`가 음수인 경우에만 음수이기 때문에, 계산된 거리는 결코 음성이 아니다. 그리고 우리는 이 둘을 서로 나눈다.

```cpp
//Calculate distance projected on camera direction (Euclidean distance will give fisheye effect!)
if (side == 0) perpWallDist = (mapX - posX + (1 - stepX) / 2) / rayDirX;
else           perpWallDist = (mapY - posY + (1 - stepY) / 2) / rayDirY;
```

`perpWallDist`를 계산하는 또 다른 방법은 벽의 적중점과 플레이어의 카메라 평면을 사용하여 `2차원에서 점과 선의 거리 공식` 사용하는 것이다. 그러나, 그것은 단순한 공식보다 계산적으로 더 비쌀 것이다. 아래 이미지는 간단한 공식을 도출하는 방법을 보여준다. 상황은 2D의 오버헤드 뷰에서 y(`side == 1`) 케이스에 대해 표시된다. `side == 0` 설명도 같다.

이미지에 표시되는 항목:

- P: 플레이어 위치
- H: 광선이 맞은 벽의 지점
- `perpWallDist`: 이 선의 길이는 지금 계산해야 할 값이며, 플레이어 포인트까지의 유클리드 거리 대신 벽의 적중점에서 카메라 평면으로 수직인 거리를 사용하여 직선 벽이 둥근 것처럼 보이지 않도록 한다.
- `yDist` 는 위의 코드에서 (mapY - posY + (1 - stepY) / 2)와 일치하며, 이것은 좌표계에서 유클리드 거리 벡터의 y 좌표다.
- `Euclidean` 플레이어 P에서 정확한 적점 H까지의 유클리드 거리를 말한다. 방향은 `rayDir`이지만, 길이는 벽까지 이른다.
- rayDir: the direction of the ray marked "Euclidean", matching the rayDirX and rayDirY variables in the code. Note that its length |rayDir| is not 1 but slightly higher, due to how we added a value to (dirX,dirY) (the dir vector, which *is* normalized to 1) in the code.
- `rayDir`: 코드의 `rayDirX` 및 `rayDirY` 변수에 일치하는 '유클리드'로 표시된 광선의 `방향`. 코드에서 `(dirX,dirY)`에 값을 추가한 방법 때문에 `|rayDir|`의 길이가 `1`이 아니라 약간 더 높다는 점에 유의하십시오.
- `rayDirX` and `rayDirY`: 코드의 'rayDirX' 및 'rayDirY' 변수와 일치하는 'rayDir'의 X 및 Y 성분.
- `dir`: dirX와 dirY에 의해 주어진 플레이어가 바라보는 방향을 의미한다. 이 벡터의 길이는 항상 정확히 1이다. 이것은 화면의 중앙이 바라보는 방향과 정확히 일치하고, 현재 광선의 방향과는 대조적이다. 그것은 카메라 평면에 수직이고, `perpWallDist`는 이것과 평행하다.
- 주황 점선 (잘 안보일 수 있는데 확대해서 봐라): `dir`에 더해져서 `rayDir`을 얻을 수 있게 해주는 값이다. 중요한 것은, 카메라 평면과 평행하고, `dir`과 수직이라는 점이다.
- `camera plane`: 이것은 카메라 평면, 즉 `CameraX`와 `CameraY`에 의해 주어진 **선**이며, 플레이어의 시선 방향과 수직이다.
- A: H에 가장 가까운 카메라 평면의 점, `perpWallDist`와 카메라 평면이 교차하는 지점
- B: H에 가장 가까운 플레이어가 위치한 X축의 점. yDist가 X축을 교차하는 점
- C: 플레이어의 위치 + rayDirX
- D: 플레이어의 위치 + rayDir.
- E: 이것은 D - dir 인 지점이다. 다시 말해서, E + dir = D.
- 점 A, B, C, D, E, H, P는 아래의 설명에 사용된다: 그들이 형성하는 삼각형은 의미를 가진다: BHP, CDP, AHP, DEP.

![perpWallDist](https://lodev.org/cgtutor/images/raycastperpwalldist.png)

그리고 위의 `perpWallDist` 계산의 유도는 다음과 같다.

- 1: `(mapY - posY + (1 - stepY) / 2) / rayDirY` 은  `yDist / rayDirY` 이다.
- 2: 삼각형 `PBH` 와`PCD`는 같은 모양을 가지고 있고 사이즈만 다르다. 그렇기 때문에 같은 각도들을 가진다.
- 3: 2번을 통해, 삼각형들의 비는 같기 때문에 `yDist / rayDirY`과 `Euclidean / |rayDir|` 이 같다는 것을 알 수 있고, 대신 `perpWallDist = Euclidean / |rayDir|` 로 쓸수 있다.
- 4: 삼각형 `AHP` 와 `EDP`  같은 모양을 가지고 있고 사이즈만 다르다. 그렇기 때문에 같은 각도들을 가진다. ED 모서리의 길이, 즉 `|ED|`는 `dir`의 길이인  `|dir|`와 같기 이것은  `1` 이다. 비슷하게 `|DP|`와 `|rayDir|`는 같다.
- 5: 4번을 통해,  `Euclidean / |rayDir|` = `perpWallDist / |dir|` = `perpWallDist / 1` 인것을 알 수 있다.
- 6: 3번과 5번을 같이 보면, `perpWallDist = yDist / rayDirY` 인 것을 확인할 수 있고, 위 코드에서 이 수식이 사용되었다.

이제 계산된 거리`(perpWallDist)`를 갖게 되었으므로 화면에 그려져야 할 선의 높이를 계산할 수 있다:이것은 `perpWallDist`이고 역수이고, 픽셀로 표현하기 위해 h를 곱한다. (h는 화면의 픽셀의 높이 이다.) 벽이 더 높거나 더 낮기를 원한다면 물론 다른 값(예: 2*h)으로 곱할 수도 있다. h 값은 벽을 높이, 너비 및 깊이가 동일한 큐브처럼 보이게 하고, 큰 값은 (모니터에 따라) 더 높은 상자를 만들 것이다.

그러면 이 `lineHeight` (그려져야할 수직선의 높이)로, 우리가 정말로 그려야 할 곳의 시작과 끝 위치가 계산되었다. 벽의 중심은 화면 중앙에 위치해야 하며, 시작과 끝점이 화면 밖에 있다면, 그 것들을 각각 0과 h-1로 되도록 한다.

```cpp
//Calculate height of line to draw on screen
int lineHeight = (int)(h / perpWallDist);

//calculate lowest and highest pixel to fill in current stripe
int drawStart = -lineHeight / 2 + h / 2;
if(drawStart < 0)drawStart = 0;
int drawEnd = lineHeight / 2 + h / 2;
if(drawEnd >= h)drawEnd = h - 1;
```

마지막으로 맞은 벽이 몇번에 해당하는지에 따라 색을 선택한다. 만약 y측면이 부딪혔다면, 색이 더 진해지며, 이것은 더 좋은 효과를 준다. 그리고 나서 수직선은 `verLine` 명령으로 그려진다. 이것은 적어도 모든 x에 대해 이것을 수행한 후에, 레이캐스팅 루프를 종료한다.

```cpp
//choose wall color
ColorRGB color;
switch(worldMap[mapX][mapY])
{
    case 1:  color = RGB_Red;  break; //red
    case 2:  color = RGB_Green;  break; //green
    case 3:  color = RGB_Blue;   break; //blue
    case 4:  color = RGB_White;  break; //white
    default: color = RGB_Yellow; break; //yellow
}

//give x and y sides different brightness
if (side == 1) {color = color / 2;}

//draw the pixels of the stripe as a vertical line
verLine(x, drawStart, drawEnd, color);
}
```

레이캐스팅 루프가 완료된 후, 현재 시간과 이전 프레임이 계산되고, FPS(초당 프레임)가 계산되어 인쇄되며, 화면이 다시 그려져서 모든 것(모든 벽, fps 카운터 값)이 보이게 된다. 그 후 `backbuffer `백버퍼는 `cls()`로 지워져 다음 프레임에서 다시 벽을 그릴 때 이전 프레임의 픽셀을 계속 포함하는 대신 바닥과 천장이 다시 검은색으로 변하게 된다.

speed modifiers는 `frameTime`과 상수 값을 사용하여 입력 키의 이동 및 회전 속도를 결정한다. 프레임타임(frameTime)을 이용한 덕분에, 우리는 이동과 회전 속도를 프로세서 속도와 상관없이 독립적으로 할 수 있다.

```cpp
//timing for input and FPS counter
oldTime = time;
time = getTicks();
double frameTime = (time - oldTime) / 1000.0; //frameTime is the time this frame has taken, in seconds
print(1.0 / frameTime); //FPS counter
redraw();
cls();

//speed modifiers
double moveSpeed = frameTime * 5.0; //the constant value is in squares/second
double rotSpeed = frameTime * 3.0; //the constant value is in radians/second
```

마지막 부분은 입력 부분이고, 키를 읽는다.

위쪽 화살표를 누르면 플레이어가 앞으로 이동한다: `posX`에 `dirX`를 추가하고` dirY`를 `posY`에 추가한다. 이것은 `dirX`와 `dirY`가 정규화된 벡터(길이는 1)라고 가정한다. 처음에는 이렇게 설정되었으므로 괜찮다. 또한 단순한 충돌 감지 장치가 내장되어 있는데, 즉 새로운 위치에 벽이 있으면 움직이지 않을 겁니다. 그러나 이러한 충돌 감지 기능은 플레이어 주위의 원이 벽 안으로 들어가지 않는지 확인함으로써 향상될 수 있다.

아래쪽 화살표를 누를 때도 마찬가지지만, 그 다음 방향은 대신 뺄셈된다.

회전하기 위해 왼쪽 또는 오른쪽 화살표를 누르면 방향 벡터와 평면 벡터가 회전 행렬( 회전각 `rotSpeed`가 적용된)과의 곱셈 공식이 사용되어 회전한다.

```cpp
readKeys();
//move forward if no wall in front of you
if (keyDown(SDLK_UP))
{
    if(worldMap[int(posX + dirX * moveSpeed)][int(posY)] == false) posX += dirX * moveSpeed;
    if(worldMap[int(posX)][int(posY + dirY * moveSpeed)] == false) posY += dirY * moveSpeed;
}
//move backwards if no wall behind you
if (keyDown(SDLK_DOWN))
{
    if(worldMap[int(posX - dirX * moveSpeed)][int(posY)] == false) posX -= dirX * moveSpeed;
    if(worldMap[int(posX)][int(posY - dirY * moveSpeed)] == false) posY -= dirY * moveSpeed;
}
//rotate to the right
if (keyDown(SDLK_RIGHT))
{
    //both camera direction and camera plane must be rotated
    double oldDirX = dirX;
    dirX = dirX * cos(-rotSpeed) - dirY * sin(-rotSpeed);
    dirY = oldDirX * sin(-rotSpeed) + dirY * cos(-rotSpeed);
    double oldPlaneX = planeX;
    planeX = planeX * cos(-rotSpeed) - planeY * sin(-rotSpeed);
    planeY = oldPlaneX * sin(-rotSpeed) + planeY * cos(-rotSpeed);
}
//rotate to the left
if (keyDown(SDLK_LEFT))
{
    //both camera direction and camera plane must be rotated
    double oldDirX = dirX;
    dirX = dirX * cos(rotSpeed) - dirY * sin(rotSpeed);
    dirY = oldDirX * sin(rotSpeed) + dirY * cos(rotSpeed);
    double oldPlaneX = planeX;
    planeX = planeX * cos(rotSpeed) - planeY * sin(rotSpeed);
    planeY = oldPlaneX * sin(rotSpeed) + planeY * cos(rotSpeed);
}
}
}
```


이것으로 질감이 없는 레이캐스터의 코드가 끝나며, 결과는 이와 같으며, 지도에서 걸어다닐 수 있다.

![img](https://lodev.org/cgtutor/images/raycasteruntextured.gif)

카메라 평면이 방향 벡터에 수직이 아닌 경우, 세계는 기울어진 것처럼 보인다.

![img](https://lodev.org/cgtutor/images/raycastingskewed.gif)

## 4. Textured Raycaster

