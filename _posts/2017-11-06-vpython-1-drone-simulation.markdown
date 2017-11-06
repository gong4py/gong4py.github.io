---
layout: post
title: "Vpython 1 - Drone Simulation"
date: 2017-11-06T18:16:27+09:00
---

vpython 활용기
===

## 김태훈

---
# vpython

- 3d object의 표현이나 움직임을 쉽게 나타낼 수 있도록 도와주는 패키지 (http://www.vpython.org/)
- 이전에 택호형이 한번 다룬적 있음

---

# 찾고 쓰게 된 계기...

- 이전에 한 번 들은 바가 있음 (Thx to 택호형)
- 드론 시뮬레이터를 만들어달라는 의뢰를 받음
  - 드론이 물류를 배송하는데 있어서, 조건들이 달라질때 상황이 어떻게 변하는지 간단하게 보여주기 위한 시뮬레이션 모델
  - GPS성능, 풍속, 장애물 유무에 따라 지표가 어떻게 되는지,...
- 로직은 문제가 없는데 보여주는게 문제
  - 로직은 3d 경로 상에서 겹치지 않게만 짜주면 문제가 없지만!
  - 산업공학과는 기본적으로 다 할 수 있습니다 ㅎㅎ
- 기본 파이선 라이브러리는 3d 오브젝트 생성, 카메라 앵글 회전, 경로 나타내기 등.. 어려움

---

# 기본적인 vpython 활용법 (1/2)

- import vpython
- define canvas and scean
  - 캔버스는 우리의 그림이 보여지는 환경을 뜻함
  - 기본적인 제목, 그림 프레임의 가로세로, 그림 상의 시작 위치(center), 배경 색깔 등 기본적인 정보를 정의

``` python
scean1 = canvas(title ="title", width=640,height=360, center=vector(100,30,30),background = vector(0.75,1,1))
```

- add objects in the scean
  - 제목, 캡션, 구, 박스, 선, 점, 면 등 3d 객체 생성이 가능
  - text의 경우, 폰트,사이즈,색상 등도 지정 가능
  - 3d object의 경우, 위치, 크기, 색상 등 지정 가능

``` python
small_block = box(pos=vector(25,15,0),size=vector(10,30,10))
```

---

# 기본적인 vpython 활용법 (2/2)

- define and add control object
  - 값 변경, 세팅 등을 위한 드랍다운, 슬라이더, 입력창, 라디오버튼, 버튼 등을 생성
  - 그리고 해당 이벤트 발생시 이를 핸들링할 함수를 만들어주면 된다!

``` python
menu(choices=['No Block', 'Small Block', 'Large Block'], index=1, bind=set_block_type)
# 항목이 No Block/Small Block/Large Block 인 드랍다운 메뉴 생성
# 첫 화면에서의 값은 1번 인덱스(즉 Small Block)
# bind는 어떤 함수와 연결할 것인지를 지정
def set_block_type(m):
    global block_type
    block_type = m.selected
    # 이런식으로 binding 된 객체에서 값을 불러올 수 있다
    if m.selected == 'No Block':
        pass 
    elif m.selected == 'Small Block':
        pass
    elif m.selected == 'Large Block':
        pass
```

---

# 그림을 다 그리고 나서 

## (사실 하고싶었던 얘기는...)

- 결국엔 시뮬레이션이라는 것은 움직임
  - 고정된 물체는 큰 의미가 없다
  - 결국 움직이는 것이 중요
- vector: vpython에서 객체를 조정할때 사용하는 기본 단위
  - pos = vector 를 활용하여 위치를 바꿔줄수도 있고
  - delta 같은 느낌으로 활용할 수도 있음!  
    - pos = pos + del_vec 하면 del_vec 만큼 움직이는것이 가능

---

# 기본적인 vector 활용법

```python
v1 = vector(10,10,10)
v2 = vector(5,10,15)
v1_norm = v1.norm() # 크기가 1인 normal vector 리턴
v1_mag = v1.mag # 이 벡터의 크기를 리턴 해줌, 괄호 없음에 주의..
v3 = v1 + v2 # 벡터의 합 가능
dpv = v1.dot(v2) # 벡터의 내적
cpv = v1.cross(v2) # 벡터의 외적
prj = v1.proj(v2) # v2로 v1을 투영
rad_angle = v1.diff_angle(v2) # 두 벡터간의 각도
rv1 = rotate(vector=v1, angle=theta, axis=v2) 
# v1을 v2를 축으로 theta만큼 회전 시켜라 (반시계 방향)
```

---

# 자 그럼 이걸로...

- 드론끼리 만났을때 비껴가는 그림을 그려보자 ![ROTATION_Imgur](https://i.imgur.com/49iuRPJ.gif)
  - 경로가 겹치거나 충돌이 예상될 때, 이를 비껴가는 로직 및 그림을 그리자
  - 같은 축을 기준으로 원형으로 회전하듯 비껴가면 pseudo 최적임

![ROTATION2_Imgur](https://i.imgur.com/p4zLYHV.gif)

---

# 자 그럼 이걸로...

1. 공통된 plane의 축을 찾는다
2. 두 안전반원이 만나는 지점을 찾는다
3. plane과 원의 중심을 기준으로 rotation 좌표들을 찾는다
4. 다시 축과 만나는 좌표를 찾는다
5. SUCCESS!

```python
r = 5
d1_pos = vector(0,0,0)
d1_vec = vector(1,0,0).norm()
d2_pos = vector(20,5,0)
d2_vec = vector(-1,0,0).norm()
d_c_vec = d2_pos - d1_pos
theta = math.asin(1/2)
r_axis = vector(0,0,1) # 축찾기 생략
# ((d2-a) + (d1+a))^2 = 100 + 25
# a = (20-sqrt(125))/2
a = (20-math.sqrt(125))/2 #간략히..
cv = -vector(r*math.cos(theta),r*math.sin(theta),0)
cp = vector(a+r*math.cos(theta), r*math.sin(theta), 0) 
#만나는 지점 찾기 간략
tmp_v = cv
v1_rots = []
for i in range(30):
    tmp_v = tmp_v.rotate(angle=(1.0/float(r)), axis = r_axis)
    v1_rots.append(tmp_v)

v1_detour = [j_th+cp for j_th in v1_rots]
v1_feasible = [j_th for j_th in v1_detour if j_th.y <= 0]
```

```python
[<4.992818, -0.810430, 0.000000>,
<5.725192, -1.488883, 0.000000>,
<6.577755, -2.008313, 0.000000>,
<7.516519, -2.348010, 0.000000>,
<8.504057, -2.494432, 0.000000>,
<9.501000, -2.441742, 0.000000>,
<10.467602, -2.192040, 0.000000>,
<11.365329, -1.755282, 0.000000>,
<12.158390, -1.148879, 0.000000>,
<12.815169, -0.397006, 0.000000>]
```

---
# 우선은 끝...!