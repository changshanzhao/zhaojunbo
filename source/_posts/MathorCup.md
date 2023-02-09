---
title: 第三届MathorCup
date: 2023-01-29 10:42:55
tags:
 - 竞赛
categories:
 - 教程
---

## 决策树

```python
import numpy as np
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split
from itertools import combinations

if __name__ == "__main__":
    mpl.rcParams['font.sans-serif'] = ['SimHei']
    mpl.rcParams['axes.unicode_minus'] = False

    feature = '网络覆盖与信号强度','语音通话清晰度','语音通话稳定性','是否遇到过网络问题','居民小区','办公室','高校','商业街','地铁','农村','高铁','其他，请注明','手机没有信号','有信号无法拨通','通话过程中突然中断','通话中有杂音、听不清、断断续续','串线','通话过程中一方听不见','其他，请注明.1','脱网次数','mos质差次数','未接通掉话次数','重定向次数','重定向驻留时长','ARPU（家庭宽带）','是否4G网络客户（本地剔除物联网）','当月ARPU','前3月ARPU','当月欠费金额','前第3个月欠费金额'
    path = 'f1_.csv'  # 数据文件路径
    data = pd.read_csv(path, header=None)
    x_prime = data[list(range(1,31))]
    y = pd.Categorical(data[0]).codes
    x_prime_train, x_prime_test, y_train, y_test = train_test_split(x_prime, y, test_size=0.3, random_state=0)
    pairs = [c for c in combinations(range(1,31), 30)]
    feature_pairs = []
    num = len(pairs)
    for i in range(num):
        feature_pairs.append(list(pairs[i]))

    for i, pair in enumerate(feature_pairs):
        # 准备数据
        x_train = x_prime_train[pair]
        x_test = x_prime_test[pair]

        # 决策树学习
        model = RandomForestClassifier(n_estimators=100, criterion='entropy', max_depth=10, oob_score=True)
        model.fit(x_train, y_train)
        importance = model.feature_importances_
        for j in range(0,30,3):
            print(feature[j], ':', importance[j], feature[j+1], ':', importance[j+1]
                  , feature[j+2], ':', importance[j+2])


        # 训练集上的预测结果
        y_train_pred = model.predict(x_train)
        acc_train = accuracy_score(y_train, y_train_pred)
        y_test_pred = model.predict(x_test)
        acc_test = accuracy_score(y_test, y_test_pred)
        print('OOB Score:', model.oob_score_)
        print('\t训练集准确率: %.4f%%' % (100*acc_train))
        print('\t测试集准确率: %.4f%%\n' % (100*acc_test))
```

importance里面有影响系数，如下：

![](importance1.jpg)

![](importance2.jpg)

可以看出除了前三项对整体满意度的影响较大，别的项对其影响很小。

but, 很不幸，前三项是y，不是x，大悲（）

但我头铁，死马当活马医，直接硬上。

```python
x_prime = train2
y = pd.Categorical(y2[3]).codes
x_prime_train, x_prime_test, y_train, y_test = train_test_split(x_prime, y, test_size=0.3, random_state=0)

feature_pairs = [list(range(0, 51))]
for i, pair in enumerate(feature_pairs):
    # 准备数据
    x_train = x_prime_train[pair]
    x_test = x_prime_test[pair]

    # 决策树学习
    model = RandomForestClassifier(n_estimators=100, criterion='entropy', max_depth=7, oob_score=True)
    model.fit(x_train, y_train)
    y_pre = model.predict(pre2)
    for j in range(len(y_pre)):
        y_pre[j] = y_pre[j] + 1
```

稍微小改一下，其实是一样的，就是换一下x和y

不过意料之中，准确率很低......只有50%-60%左右

然后我又尝试了一下bp神经网络

## BP神经网络

```python
# 0. 超参数设置
lr = 0.00002
epochs = 300
n_feature = 22
n_hidden = 300
n_output = 10

# 1. 数据准备
train1 = pd.read_csv('train1_.csv', header=None)
# train2 = pd.read_csv('train2_.csv', header=None)

pre1 = pd.read_csv('pre1.csv', header=None)
# pre2 = pd.read_csv('pre2.csv', header=None)

y1 = pd.read_csv('f1_.csv', header=None)
# y2 = pd.read_csv('f2_.csv', header=None)


x_prime = train1
y = pd.Categorical(y1[1]).codes
x_p_train, x_p_test, y_train, y_test = train_test_split(x_prime, y, test_size=0.2, random_state=22)
x_train = np.array(x_p_train)
x_pre = np.array(pre1)
x_test = np.array(x_p_test)

x_train = torch.FloatTensor(x_train)
y_train = torch.LongTensor(y_train)
x_test = torch.FloatTensor(x_test)
y_test = torch.LongTensor(y_test)
x_pre = torch.FloatTensor(x_pre)


# 2. 定义BP神经网络
class bpnnModel(torch.nn.Module):
    def __init__(self, n_feature, n_hidden, n_output):
        super(bpnnModel, self).__init__()
        self.hidden = torch.nn.Linear(n_feature, n_hidden)  # 定义隐藏层网络
        self.out = torch.nn.Linear(n_hidden, n_output)  # 定义输出层网络


    def forward(self, x):
        x = Fun.relu(self.hidden(x))  # 隐藏层的激活函数,采用relu,也可以采用sigmod,tanh
        out = Fun.softmax(self.out(x), dim=1)  # 输出层softmax激活函数
        return out

# 3. 定义优化器损失函数
net = bpnnModel(n_feature=n_feature, n_hidden=n_hidden, n_output=n_output)
optimizer = torch.optim.Adam(net.parameters(), lr=lr)  # 优化器选用随机梯度下降方式
loss_func = torch.nn.CrossEntropyLoss()  # 对于多分类一般采用的交叉熵损失函数

# 4. 训练数据
loss_steps = np.zeros(epochs)
accuracy_steps = np.zeros(epochs)
for epoch in range(epochs):
    y_pred = net(x_train)  # 前向过程
    loss = loss_func(y_pred, y_train)  # 输出与label对比
    optimizer.zero_grad()  # 梯度清零
    loss.backward()  # 反向传播
    optimizer.step()  # 使用梯度优化器
    loss_steps[epoch] = loss.item()  # 保存loss
with torch.no_grad():
    y_pred = net(x_test)
    y0 = net(x_pre)
    y = torch.argmax(y0, dim=1)
    correct = (torch.argmax(y_pred, dim=1) == y_test).type(torch.FloatTensor)
    accuracy_steps[epoch] = correct.mean()
print("预测准确率", accuracy_steps[-1])

# 5 绘制损失函数和精度
fig_name = '网络覆盖与信号强度(语音)'
fontsize = 15
fig, (ax1, ax2) = plt.subplots(2, figsize=(15, 12), sharex=True)
ax1.plot(accuracy_steps)
ax1.set_ylabel("test accuracy", fontsize=fontsize)
ax1.set_title(fig_name, fontsize='xx-large')
ax2.plot(loss_steps)
ax2.set_ylabel("train loss", fontsize=fontsize)
ax2.set_xlabel("epochs", fontsize=fontsize)
plt.tight_layout()
plt.savefig(fig_name + '.png')
plt.show()
for j in range(len(y)):
    y[j] = y[j] + 1
data = pd.DataFrame(y)
data.to_csv('BP网络覆盖与信号强度(语音).csv', index=None)
```

emmm, 虽然准确率稍微提高了5-10个百分点，但是过拟合太严重，结果评分都是10...

算了，就这样，直接交决策树，开摆。

期待大佬留言评论。

<script src="https://giscus.app/client.js"
        data-repo="changshanzhao/MathorCup"
        data-repo-id="R_kgDOI2ig8w"
        data-category-id="DIC_kwDOI2ig884CT3Rf"
        data-mapping="url"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>