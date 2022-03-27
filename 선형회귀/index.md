# 선형회귀


선형 회귀는 한 개 이상의 독립 변수 x와 y의 선형 관계를 모델링한다.

가중치 행렬을 W, 편향을 b, 실제값을 y라고 할 때, 선형회귀는 비용함수 MSE를 최소화하는 W와 b를 추정해 나가는 과정이다.

## 계산 과정

편차를 $WX_{i}+b-y_{i}$라고 할 때, 비용함수는 다음과 같다.

$$\sum_{i=1}^{m} (WX_{i}+b-y_{i})^2 = \sum_{i=1}^{m}(X_{i}^2W^2+2X_{i}bW-2by_{i}-2X_{i}y_{i}W+b^2+yi^2)$$


비용함수를 W로 각각 편미분 하면 다음과 같다.

$$\partial{W} = \cfrac{\partial{cost(W,b)}} {\partial{W}} = \sum_{i=1}^{m}(2X_{i}^2W+2X_{i}b-2X_{i}y_{i})\cdot \cfrac{1}{m} $$

$$= 2X_{i}\sum_{i=1}^{m} (WX_{i}+b-y_{i})\cdot \cfrac{1}{m}$$

비용함수를 b로 각각 편미분 하면 다음과 같다.

$$\partial{b} = \cfrac{\partial{cost(W,b)}} {\partial{b}} = \sum_{i=1}^{m}(2X_{i}W-2y_{i}+2b)\cdot \cfrac{1}{m}$$

$$= 2\sum_{i=1}^{m} (WX_{i}+b-y_{i})\cdot \cfrac{1}{m}$$


W와 b를 Gradient Descent로 추정해나가는 과정은 다음과 같다.

$$W:=W-lr\cfrac{\partial{cost(W,b)}} {\partial{W}}$$

$$b:=b-lr\cfrac{\partial{cost(W,b)}} {\partial{b}}$$

## 코드로 구현

```python
import numpy as np

xy = np.array([[1., 2., 3., 4., 5., 6.], [3., 6., 9., 12., 15., 18.]])

x_train = xy[0]
y_train = xy[1]

weight = np.random.rand(1)
bias = np.random.rand(1)

learning_rate = 0.001

for i in range(10000):
    pred = weight * x_train + bias
    error = pred - y_train
    cost = (error ** 2).mean()
    weight -= learning_rate * (error * 2 * x_train).mean()
    bias -= learning_rate * 2 * error.mean()

    if i % 100 == 0:
        print('Epoch ({:10d}/1000) error: {:10f}, beta_gd: {:10f}, bias: {:10f}'.format(i, cost, weight.item(), bias.item()))
```

결과는 다음과 같다.

```
Epoch (         0/1000) error:  79.680243, beta_gd:   0.628488, bias:   0.666181
Epoch (       100/1000) error:   0.359263, beta_gd:   2.657885, bias:   1.096474
Epoch (       200/1000) error:   0.222940, beta_gd:   2.745611, bias:   1.074800
Epoch (       300/1000) error:   0.207077, beta_gd:   2.757658, bias:   1.036960
Epoch (       400/1000) error:   0.192499, beta_gd:   2.766458, bias:   0.999819
Epoch (       500/1000) error:   0.178947, beta_gd:   2.774833, bias:   0.963984
Epoch (       600/1000) error:   0.166349, beta_gd:   2.782904, bias:   0.929432
Epoch (       700/1000) error:   0.154637, beta_gd:   2.790685, bias:   0.896118
Epoch (       800/1000) error:   0.143751, beta_gd:   2.798188, bias:   0.863999
Epoch (       900/1000) error:   0.133631, beta_gd:   2.805421, bias:   0.833030
Epoch (      1000/1000) error:   0.124223, beta_gd:   2.812395, bias:   0.803172
Epoch (      1100/1000) error:   0.115477, beta_gd:   2.819120, bias:   0.774384
Epoch (      1200/1000) error:   0.107348, beta_gd:   2.825603, bias:   0.746628
Epoch (      1300/1000) error:   0.099790, beta_gd:   2.831854, bias:   0.719867
Epoch (      1400/1000) error:   0.092765, beta_gd:   2.837881, bias:   0.694064
Epoch (      1500/1000) error:   0.086234, beta_gd:   2.843692, bias:   0.669187
Epoch (      1600/1000) error:   0.080163, beta_gd:   2.849294, bias:   0.645202
Epoch (      1700/1000) error:   0.074520, beta_gd:   2.854696, bias:   0.622076
Epoch (      1800/1000) error:   0.069273, beta_gd:   2.859904, bias:   0.599779
Epoch (      1900/1000) error:   0.064396, beta_gd:   2.864925, bias:   0.578281
Epoch (      2000/1000) error:   0.059863, beta_gd:   2.869767, bias:   0.557554
Epoch (      2100/1000) error:   0.055648, beta_gd:   2.874435, bias:   0.537569
Epoch (      2200/1000) error:   0.051731, beta_gd:   2.878935, bias:   0.518301
Epoch (      2300/1000) error:   0.048089, beta_gd:   2.883275, bias:   0.499724
Epoch (      2400/1000) error:   0.044703, beta_gd:   2.887459, bias:   0.481812
Epoch (      2500/1000) error:   0.041556, beta_gd:   2.891492, bias:   0.464543
Epoch (      2600/1000) error:   0.038631, beta_gd:   2.895382, bias:   0.447892
Epoch (      2700/1000) error:   0.035911, beta_gd:   2.899131, bias:   0.431838
Epoch (      2800/1000) error:   0.033383, beta_gd:   2.902747, bias:   0.416360
Epoch (      2900/1000) error:   0.031033, beta_gd:   2.906233, bias:   0.401436
Epoch (      3000/1000) error:   0.028848, beta_gd:   2.909594, bias:   0.387048
Epoch (      3100/1000) error:   0.026817, beta_gd:   2.912834, bias:   0.373175
Epoch (      3200/1000) error:   0.024929, beta_gd:   2.915958, bias:   0.359799
Epoch (      3300/1000) error:   0.023174, beta_gd:   2.918971, bias:   0.346903
Epoch (      3400/1000) error:   0.021542, beta_gd:   2.921875, bias:   0.334469
Epoch (      3500/1000) error:   0.020026, beta_gd:   2.924675, bias:   0.322481
Epoch (      3600/1000) error:   0.018616, beta_gd:   2.927375, bias:   0.310922
Epoch (      3700/1000) error:   0.017305, beta_gd:   2.929978, bias:   0.299778
Epoch (      3800/1000) error:   0.016087, beta_gd:   2.932488, bias:   0.289033
Epoch (      3900/1000) error:   0.014955, beta_gd:   2.934908, bias:   0.278673
Epoch (      4000/1000) error:   0.013902, beta_gd:   2.937241, bias:   0.268684
Epoch (      4100/1000) error:   0.012923, beta_gd:   2.939490, bias:   0.259054
Epoch (      4200/1000) error:   0.012013, beta_gd:   2.941659, bias:   0.249769
Epoch (      4300/1000) error:   0.011168, beta_gd:   2.943750, bias:   0.240816
Epoch (      4400/1000) error:   0.010381, beta_gd:   2.945766, bias:   0.232185
Epoch (      4500/1000) error:   0.009650, beta_gd:   2.947710, bias:   0.223863
Epoch (      4600/1000) error:   0.008971, beta_gd:   2.949585, bias:   0.215839
Epoch (      4700/1000) error:   0.008339, beta_gd:   2.951392, bias:   0.208102
Epoch (      4800/1000) error:   0.007752, beta_gd:   2.953134, bias:   0.200643
Epoch (      4900/1000) error:   0.007207, beta_gd:   2.954814, bias:   0.193452
Epoch (      5000/1000) error:   0.006699, beta_gd:   2.956433, bias:   0.186518
Epoch (      5100/1000) error:   0.006228, beta_gd:   2.957995, bias:   0.179833
Epoch (      5200/1000) error:   0.005789, beta_gd:   2.959500, bias:   0.173387
Epoch (      5300/1000) error:   0.005382, beta_gd:   2.960952, bias:   0.167172
Epoch (      5400/1000) error:   0.005003, beta_gd:   2.962352, bias:   0.161180
Epoch (      5500/1000) error:   0.004651, beta_gd:   2.963701, bias:   0.155403

```

