# 小球颜色识别
## 关键字
* 模糊滤波
* 霍夫圆检测
* 框出小球
* 文字绘制
## 所需配件
* powersensor传感器
* 各种颜色小球
## 原理讲解
**霍夫圆变换**：的基本原理和上个教程中提到的霍夫线变换类似, 只是点对应的二维极径极角空间被三维的圆心点$x,y$还有半径$r$空间取代。
**原理**：从平面坐标圆上的点到极坐标转换的三个参数$C(x_0,y_0,r)$其中$x_0,y_0$是圆心，$r$取一固定值时扫描360度，$x,y$ 跟着变化， 若多个边缘点对应的三维空间曲线交于一点,则他们在共同圆上，在圆心处有累积最大值，也可以用同样的阈值的方法来判断一个圆是否被检测到。
## 相关API - HoughCircles
因为霍夫圆检测对噪声比较敏感，所以首先要对图像做滤波（比如椒盐噪声用中值滤波，其他的也可以用高斯模糊）。基于效率考虑，Opencv中实现的霍夫变换圆检测是基于图像梯度（霍夫梯度法, 也叫2-1霍夫变换(21HT)）的实现，分为两步（已封装到HoughCircles）：
（1）Canny检测边缘，发现可能的圆心。圆心一定是在圆上的每个点的模向量上, 这些圆上点模向量的交点就是圆心, 霍夫梯度法的第一步就是找到这些圆心, 这样三维的累加平面就又转化为二维累加平面。
（2）基于第一步的基础上从候选圆心开始计算最佳半径大小。第二步根据所有候选中心的边缘非0像素对其的支持程度来确定半径。
## 函数介绍
```
cv2.HoughCircles(image, method, dp, minDist[, circles[, param1[, param2[,minRadius[, maxRadius]]]]]) → circles
# image,必须，原图,要求二值化后的图片；
# method,必须,检测方法，目前仅支持CV_HOUGH_GRADIENT，基于21HT实现；
# dp,必须，检测的缩放比例参数，如果为1，就是检测原图，为2就是缩小一半检测，影响计算效率；
# minDist，为霍夫变换检测到的圆的圆心之间的最小距离，即让我们的算法能明显区分的两个不同圆之间的最小距离。这个参数如果太小的话，多个相邻的圆可能被错误地检测成了一个重合的圆。反之，这个参数设置太大的话，某些圆就不能被检测出来了。
# param1，有默认值100。它是method设置的检测方法的对应的参数。对当前唯一的方法霍夫梯度法，它表示传递给canny边缘检测算子的高阈值，而低阈值为高阈值的一半。
# param1 , 可 选 ， 低 阈 值 ， 传 递 给 canny() 检 测 器 的 低 阈 值 ；
# minRadius，默认值0，表示圆半径的最小值。
# maxRadius，也有默认值0，表示圆半径的最大值
```

```
 cv2.putText(src, text, place, Font, Font_Size, Font_Color, Font_Overstriking)
 # src，输入图像
 # text，需要添加的文字
 # place，左上角坐标
 # Font，字体类型
 # Font_Size，字体大小
 # Font_Color，文字颜色
 # Font_Overstriking，字体粗细
```

## 参考例程
```
#指定编码方式
fourcc = cv2.VideoWriter_fourcc(*'MJPG')
#设置保存的文件名，图像的格式
out1 = cv2.VideoWriter('output3.avi',fourcc, 20.0, (320,240))
#小球颜色识别bgr
def colour2(img_b,img_g,img_r) :
    if ((img_b>=60 and img_b<=130 )  and (img_g>=60 and img_g<=130) and (img_r>=140 and img_r<=220) ):
        cv2.putText(origin,'red', (i[0]-i[2],i[1]-i[2]), cv2.FONT_HERSHEY_PLAIN, 2.0, (0, 0, 255), 2)
    elif ((img_b>=70 and img_b<=140) and (img_g>=120 and img_g<=190) and (img_r>=170 and img_r<=240) ):
        cv2.putText(origin,'yellow',(i[0]-i[2],i[1]-i[2]), cv2.FONT_HERSHEY_PLAIN, 2.0, (0, 0, 255), 2)
    elif ((img_b>=70 and img_b<=140) and (img_g>=150 and img_g<=230) and (img_r>=100 and img_r<=170) ):
        cv2.putText(origin,'green', (i[0]-i[2],i[1]-i[2]), cv2.FONT_HERSHEY_PLAIN, 2.0, (0, 0, 255), 2) 
    elif ((img_b>=50 and img_b<=120) and (img_g>=60 and img_g<=120) and (img_r>=70 and img_r<=120) ):
        cv2.putText(origin,'brown', (i[0]-i[2],i[1]-i[2]), cv2.FONT_HERSHEY_PLAIN, 2.0, (0, 0, 255), 2)
        
while(True):
    clear_output(wait=True)  
    imgMat = cam1.read_img_ori()
    origin = cv2.resize(imgMat, (320,240))
    start = time.time() 
    # 转换为灰度图 
    img_gray = cv2.cvtColor(origin, cv2.COLOR_BGR2GRAY)
    # medianBlur 平滑（模糊）处理
    img_gray = cv2.medianBlur(img_gray, 7)
    #圆检测
    circles = cv2.HoughCircles(img_gray, cv2.HOUGH_GRADIENT, 1, 40, param1=50,param2=35, minRadius=0, maxRadius= 300)    
    if circles is None:
        pass
    else:
        circles = np.uint16(np.around(circles))
        for i in circles[0,:]:
            # 勾画正方形，origin图像、i[2]*2是边长
            cv2.rectangle(origin,(i[0]-i[2],i[1]-i[2]),(i[0]+i[2],i[1]+i[2]),(255,0,0), 2)
            #取球心一小块区域
            roi = origin[i[0]:i[1] , i[0]:i[1]+1] 
            #分离bgr通道
            img_b = np.uint16(np.mean(roi[:,:,0]))
            img_g = np.uint16(np.mean(roi[:,:,1]))
            img_r = np.uint16(np.mean(roi[:,:,2]))
            #判断小球颜色
            colour2(img_b,img_g,img_r)

# 计 算 消 耗 时 间
    end = time.time()   
    ps.CommonFunction.show_img_jupyter(origin)
    out1.write(origin)
    print(end - start)
#time.sleep(0.1) 
out1.release()   
```
## 结果展示
结果如图所示
![小球颜色识别](%E5%B0%8F%E7%90%83%E9%A2%9C%E8%89%B2%E8%AF%86%E5%88%AB.png)
框出小球并识别到三个小球颜色
## 小结一下
本文主要介绍了基于霍夫圆检测方法以及如何框出小球并识别颜色，霍夫变换的原理就是利用图像全局特征将边缘像素连接起来组成区域封闭边界，它将图像空间转换到参数空间，在参数空间对点进行描述，达到检测图像边缘的目的。识别小球颜色主要难点在调整RGB参数方面，利用TakeColor取色器选取参数范围，设置正确后便可以准确判断出小球颜色。