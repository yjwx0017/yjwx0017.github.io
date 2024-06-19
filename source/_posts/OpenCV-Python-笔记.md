title: OpenCV-Python 笔记
categories: 日志
tags: [Python,OpenCV]
date: 2024-05-28 02:03:00
---
# OpenCV Python 笔记

[OpenCV Tutorial in Python](https://www.geeksforgeeks.org/opencv-python-tutorial/)

## 显示图片

``` python
import cv2

img = cv2.imread('cat_dog.jpg', cv2.IMREAD_COLOR)

print(img.shape)

cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 显示图片（matplotlib）

默认情况下，OpenCV以BGR格式存储彩色图像。

``` python
import cv2
import numpy as np
import matplotlib.pyplot as plt

img = cv2.imread('cat_dog.jpg')
rgb_img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
plt.imshow(rgb_img)
plt.waitforbuttonpress()
plt.close('all')
```

## 显示灰度图

``` python
import cv2

img = cv2.imread('cat_dog.jpg', cv2.IMREAD_GRAYSCALE)

cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

<!--more-->

## 写入文件

``` python
import cv2

img = cv2.imread('cat_dog.jpg')
cv2.imwrite('cat_dog.jpg', img)
```

## 分离颜色通道

``` python
import cv2

img = cv2.imread('cat_dog.jpg')
B, G, R = cv2.split(img)

cv2.imshow('original', img)
cv2.imshow('blue', B)
cv2.imshow('green', G)
cv2.imshow('red', R)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 混合

``` python
import cv2
import numpy as np

image1 = cv2.imread('input1.jpg')
image2 = cv2.imread('input2.jpg')

weightedSum = cv2.addWeighted(image1, 0.5, image2, 0.4, 0)

# sub = cv2.substract(image1, image2)
# add = cv2.add(image1, image2)

# 位运算
# dest = cv2.bitwise_and(image1, image2, mask = None)
# dest = cv2.bitwise_or(image1, image2, mask = None)
# dest = cv2.bitwise_xor(image1, image2, mask = None)
# dest = cv2.bitwise_not(image1, mask = None)

cv2.imshow('weighted image', weightedSum)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 调整大小

``` python
import cv2
import numpy as np
import matplotlib.pyplot as plt

img = cv2.imread('cat_dog.jpg')
img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

# cv2.INTER_AREA: This is used when we need to shrink an image.
# cv2.INTER_CUBIC: This is slow but more efficient.
# cv2.INTER_LINEAR: This is primarily used when zooming is required. 
#                   This is the default interpolation technique in OpenCV.

half = cv2.resize(img, (0, 0), fx=0.2, fy=0.2)
bigger = cv2.resize(img, (600, 600))

stretch_near = cv2.resize(
    img, (700, 600), interpolation=cv2.INTER_LINEAR)

titles = ['Original', 'Half', 'Bigger', 'Interpolation Nearest']
images = [img, half, bigger, stretch_near]
count = 4

for i in range(count): 
    plt.subplot(2, 2, i + 1)
    plt.title(titles[i])
    plt.imshow(images[i])

plt.show()
```

## erode 侵蚀

``` python
import cv2
import numpy as np

img = cv2.imread('cat_dog.jpg')

# np.ones 生成的数组中的所有元素都被初始化为数值 1
kernel = np.ones((6, 6), np.uint8)

# 侵蚀
img = cv2.erode(img, kernel, cv2.BORDER_REFLECT)
cv2.imshow('image', img)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## blur 模糊

``` python
import cv2
import numpy as np

img = cv2.imread('cat_dog.jpg')

cv2.imshow('Original Image', img)

# 高斯模糊是高斯函数对图像进行模糊处理的结果。
# 它是图形软件中广泛使用的效果，通常用于减少图像噪点和减少细节。
# 在应用我们的机器学习或深度学习模型之前，它也被用作预处理阶段。
gaussian = cv2.GaussianBlur(img, (7, 7), 0)
cv2.imshow('Gaussian Blurring', gaussian)

# 中值滤波是一种非线性数字滤波技术，通常用于去除图像或信号中的噪声。
# 中值滤波在数字图像处理中得到了广泛的应用，因为在一定条件下，
# 中值滤波可以在去除噪声的同时保留边缘。它是去除椒盐噪声的最佳算法之一。
# (to remove Salt and pepper noise)
median = cv2.medianBlur(img, 5)
cv2.imshow('Median Blurring', median)

# 双边滤波器是用于图像的非线性、边缘保持和降噪平滑滤波器。
# 它将每个像素的强度替换为附近像素的强度值的加权平均值。
# 这个权重可以基于高斯分布。因此，锐利的边缘被保留，而弱的边缘被丢弃。
bilateral = cv2.bilateralFilter(img, 9, 75, 75)
cv2.imshow('Bilateral Blurring', bilateral)

cv2.waitKey(0)
cv2.destroyAllWindows()
```