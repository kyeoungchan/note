# ✊ 응집도
- 우연적 응집도(Coincidental Cohesion): 모듈 내부의 각 구성요소가 연관이 없을 경우의 응집도
- 논리적 응집도(Logical Cohesion): **유사한** 성격을 갖거나 특정 형태로 분류되는 처리 요소들이 한 모듈에서 처리되는 경우의 응집도
- 시간적 응집(Temporal Cohesion): 연관된 기능이라기보다는 **특정 시간**에 처리되어야 하는 활동들을 한 모듈에서 처리할 경우의 응집도
- 절차적 응집도(Procedural Cohesion): 모듈이 **다수의 관련 기능**을 가질 때 모듈 안의 구성요소들이 그 기능을 **순차적으로 수행**할 경우의 응집도
- 통신적 응집도(Communication Cohesion): **동일한 입력과 출력**을 사용하여 다른 기능을 수행하는 활동들이 모여있을 경우의 응집도
- 순차적 응집도(Sequential Cohesion): 모듈 내에서 한 활동으로부터 나온 출력값을 다른 활동이 사용할 경우의 응집도
- 기능적 응집도(Functional Cohesion): 모듈 내부의 모든 기능이 단일한 목적을 위해 수행되는 경우의 응집도

> 아래로 내려갈수록 응집도가 높고 좋은 품질이다.