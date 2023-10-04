# CG-homework2

### 在代码框架中我共修改了以下两个函数
```c++

cv::Point2f recursive_bezier(const std::vector<cv::Point2f> &control_points, float t)

void bezier(const std::vector<cv::Point2f>& control_points, cv::Mat& window)

```

### 具体内容

#### 代码：
```C++
cv::Point2f recursive_bezier(const std::vector<cv::Point2f> &control_points, float t) 
{
     // TODO: Implement de Casteljau's algorithm
     //如果仅剩下一个点，那么就直接返回即可;
     if (control_points.size() == 1)   return control_points[0];
     //对于每一次迭代有两个点及以上的情况进行下述操作;
     //创建一个新的向量vector,用来存放每两个点经过一次计算后得到的新的点;
     std::vector<cv::Point2f> new_control_points;
     for (int i = 0; i < control_points.size() - 1; i++)
     { 
         new_control_points.push_back(control_points[i] * (1 - t) + control_points[i + 1] * t);
     }
      //再对上面算出的点进行递归;
      return recursive_bezier(new_control_points, t);
    //return cv::Point2f();
}
```
- 本函数实现的是de Casteljau 算法，返回Bézier曲线上的对应点坐标.<br>
- de Casteljau 算法的流程大概如下：<br>
1、根据控制点序列，将相邻的点连接起来，形成线段.<br>
2、用t:(1-t)比例分割每个线段，并找到其分割点.<br>
3、把每个线段得到的分割点形成一个新的控制点序列.<br>
4、如果控制点序列中只有一个点，那么就返回该点，递归终止。否则将新的控制点序列继续进行递归求解，继续执行步骤1，直到序列中只有一个点为止.<br>
- de Casteljau 算法的实现即上述代码，首先判断控制点序列是否只有一个坐标：是，即返回，否，则继续执行。利用存放二维point的vector存储分割点，再将分割点集作为新的控制点序列进行递归.<br>
- 这段代码即为利用de Casteljau 算法，根据传入的t的不同而返回贝塞尔曲线上不同的点坐标.<br>

#### 代码：
```C++
void bezier(const std::vector<cv::Point2f>& control_points, cv::Mat& window)
{
    // TODO: Iterate through all t = 0 to t = 1 with small steps, and call de Casteljau's 
    // recursive Bezier algorithm.
    for (float t = 0.0; t <= 1.0; t = t + 0.001)
    {
        //接收每一个t所求得的点
        cv::Point2f point = recursive_bezier(control_points, t);
        //找到该点所对应的最近的四个像素点坐标
        cv::Point2f point_1 = cv::Point2f(std::floor(point.x + 0.5), std::floor(point.y + 0.5));
        cv::Point2f point_2 = cv::Point2f(std::floor(point.x + 0.5), std::floor(point.y - 0.5));
        cv::Point2f point_3 = cv::Point2f(std::floor(point.x - 0.5), std::floor(point.y + 0.5));
        cv::Point2f point_4 = cv::Point2f(std::floor(point.x - 0.5), std::floor(point.y - 0.5));
        //创建一个vector容器，存储求得的四个像素点坐标;
        std::vector<cv::Point2f> pixel_dot{ point_1,point_2,point_3,point_4 };

        float MaxDistance = sqrt(2.0);//点到临近四个像素点包含的四方体的最远距离;
        float ratio = 0.0f;
 
        for (int i = 0; i < 4; i++)
        {
            cv::Point2f point_center(pixel_dot[i].x + 0.5, pixel_dot[i].y + 0.5);//记录像素中心点;
            //距离point越远的点的颜色占比应该较少
            ratio = 1 - sqrt(std::pow(point.x - point_center.x, 2) + std::pow(point.y - point_center.y, 2))/ MaxDistance;
            
            window.at<cv::Vec3b>(pixel_dot[i].y, pixel_dot[i].x)[1] = std::fmax(window.at<cv::Vec3b>(pixel_dot[i].y, pixel_dot[i].x)[1],255.f * ratio);
        }
    }
}
```
- 本函数实现的是绘制Bézier曲线的功能，根据recursive_bezier函数返回的Bézier曲线上t处的点，将该点绘制在OpenCV::Mat对象window上，并对Bézier曲线进行2×2的反走样技术.<br>
- 2×2反走样的思路：为进行反走样技术，不能把recursive_bezier函数返回的点只对应一个像素，需要根据到像素中心的距离来考虑相邻像素的颜色。所以根据返回的点point到周围2×2个像素中心的距离来为这些像素进行填色，以实现平滑过渡的效果。像素中心离point距离越远，颜色应该越暗。同时，point到周围2×2个像素中心的距离最大值为根号2。因此遍历周围4个像素中心时，利用1 - (point到当前像素中心的距离/最大距离)作为比率ratio，将255*ratio对当前像素填色。其中，一个像素中心可能会反复填色，应取最大的一次填色，防止贝塞尔曲线中出现暗点的情况。
- 实现流程：<br>
1、首先令t在0到1中进行迭代，每次迭代都只加上一个微小的值.<br>
2、其次根据当前迭代轮次的t，送入recursive_bezier函数中，得到Bézier曲线上t处的点.<br>
3、求出返回点point周围四个像素点的坐标.<br>
4、利用上述2×2反走样的思路实现2×2反走样：先对每个像素求出point到像素中心的距离distance，在根据比率ratio = 1 - distance / max_distance，用255*ratio对像素进行填色。注意一点，每次填色取得是最大值即可，因为题目要求绘制成绿色，所以对绿色通道进行修改.



