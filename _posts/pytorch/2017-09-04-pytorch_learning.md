---
layout: post
title: pytorch examples
category: 科研
tags: 深度学习
keywords: pytorch
description: 
---

# 安装pytorch

[github](https://github.com/pytorch/pytorch)

我选择了binaries安装，在[pythoch](http://pytorch.org/)上选择自己的环境，运行给出的command

```
pip install http://download.pytorch.org/whl/cu80/torch-0.2.0.post3-cp27-cp27mu-manylinux1_x86_64.whl 
pip install torchvision
```

# 使用教程


```
import torch
import numpy as np

x = torch.Tensor(5, 3) #定义一个未初始化的tensor
y = torch.rand(5, 3) #定义一个随机初始化的tensor

torch.add(x, y)

result = torch.Tensor(5, 3)
torch.add(x, y, out=result) #给定一个output tensor

y.add_(x) # in-place adds x to y

xn = x.numpy() #Converting torch Tensor to numpy Array， numpy和tensor共享内存，改变其中一个另一个也会改变

a = np.ones(5)
b = torch.from_numpy(a) #Converting numpy Array to torch Tensor

x = x.cuda() #Tensors can be moved onto GPU
```

```
import torch
from torch.autograd import Variable

x = Variable(torch.ones(2, 2), requires_grad=True)
y = x + 2
z = y * y * 3
out = z.mean()
out.backward()
print(x.grad)
```

```
import torch
from torch.autograd import Variable
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()
        # 1 input image channel, 6 output channels, 5x5 square convolution
        # kernel
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)
        # an affine operation: y = Wx + b
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        # Max pooling over a (2, 2) window
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        # If the size is a square you can only specify a single number
        x = F.max_pool2d(F.relu(self.conv2(x)), 2)
        x = x.view(-1, self.num_flat_features(x))
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

    def num_flat_features(self, x):
        size = x.size()[1:]  # all dimensions except the batch dimension
        num_features = 1
        for s in size:
            num_features *= s
        return num_features


net = Net()
print(net)

params = list(net.parameters())
print(len(params))
print(params[0].size())

#torch.nn only supports mini-batches The entire torch.nn package only supports inputs that are a mini-batch of samples, and not a single sample.
#For example, nn.Conv2d will take in a 4D Tensor of nSamples x nChannels x Height x Width.
#If you have a single sample, just use input.unsqueeze(0) to add a fake batch dimension.
input = Variable(torch.randn(1, 1, 32, 32))
out = net(input)
target = Variable(torch.arange(1, 11))  # a dummy target, for example
criterion = nn.MSELoss()

loss = criterion(output, target)
net.zero_grad()
loss.backward()

learning_rate = 0.01
for f in net.parameters():
    f.data.sub_(f.grad.data * learning_rate)
	
	
import torch.optim as optim

# create your optimizer
optimizer = optim.SGD(net.parameters(), lr=0.01)

# in your training loop:
optimizer.zero_grad()   # zero the gradient buffers
output = net(input)
loss = criterion(output, target)
loss.backward()
optimizer.step()    # Does the update
```

# 训练一个分类器

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import torch
import torchvision
import torchvision.transforms as transforms

# torchvision数据集的输出是在[0, 1]范围内的PILImage图片。
# 此处使用归一化的方法将其转化为Tensor，数据范围为[-1, 1]
transform=transforms.Compose([transforms.ToTensor(),transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5)),])
trainset = torchvision.datasets.CIFAR10(root='./data', train=True, download=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=4,shuffle=True, num_workers=2)
testset = torchvision.datasets.CIFAR10(root='./data', train=False, download=True, transform=transform)
testloader = torch.utils.data.DataLoader(testset, batch_size=4, shuffle=False, num_workers=2)
classes = ('plane', 'car', 'bird', 'cat','deer', 'dog', 'frog', 'horse', 'ship', 'truck')
'''注：这一部分需要下载部分数据集 因此速度可能会有一些慢 同时你会看到这样的输出
Downloading http://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz to ./data/cifar-10-python.tar.gz
Extracting tar file
Done!
Files already downloaded and verified
'''
# functions to show an image
import matplotlib
matplotlib.use('Agg')
from matplotlib import pyplot as plt
import numpy as np

def imshow(img):
    img = img / 2 + 0.5 # unnormalize
    npimg = img.numpy()
    plt.imshow(np.transpose(npimg, (1,2,0)))
    plt.savefig('./show.jpg')
    plt.close()

# show some random training images
dataiter = iter(trainloader)
images, labels = dataiter.next()

# print images
imshow(torchvision.utils.make_grid(images))
# print labels
print(' '.join('%5s'%classes[labels[j]] for j in range(4)))


import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.autograd import Variable

class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 32, 5)
        self.pool  = nn.MaxPool2d(2,2)
        self.conv2 = nn.Conv2d(32, 64, 5)
        self.fc1   = nn.Linear(64*5*5, 256)
        self.fc2   = nn.Linear(256, 128)
        self.fc3   = nn.Linear(128, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 64*5*5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

net = Net()
criterion = nn.CrossEntropyLoss() # use a Classification Cross-Entropy loss
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)
net.cuda()

for epoch in range(5): # loop over the dataset multiple times
    
    running_loss = 0.0
    for i, data in enumerate(trainloader, 0):
        # get the inputs
        inputs, labels = data
        
        # wrap them in Variable
        inputs, labels = Variable(inputs.cuda()), Variable(labels.cuda())
        
        # zero the parameter gradients
        optimizer.zero_grad()
        
        # forward + backward + optimize
        outputs = net(inputs)
        loss = criterion(outputs, labels)
        loss.backward()        
        optimizer.step()
        
        # print statistics
        running_loss += loss.data[0]
        if i % 2000 == 1999: # print every 2000 mini-batches
            print('[%d, %5d] loss: %.3f' % (epoch+1, i+1, running_loss / 2000))
            running_loss = 0.0
print('Finished Training')

class_correct = list(0. for i in range(10))
class_total = list(0. for i in range(10))
for data in testloader:
    images, labels = data
    outputs = net(Variable(images.cuda()))
    _, predicted = torch.max(outputs.data, 1)
    c = (predicted == labels.cuda()).squeeze()
    for i in range(4):
        label = labels[i]
        class_correct[label] += c[i]
        class_total[label] += 1

for i in range(10):
    print('Accuracy of %5s : %2d %%' % (classes[i], 100 * class_correct[i] / class_total[i]))
```



