@[toc]
## 一、前言
今天分享一下Alembic插件的使用教程，这个插件的主要作用就是将.abc文件导入到Unity，然后进行播放。
.abc文件主要是影像业界使用的数据格式，用于存储巨大的顶点缓存数据。
Alembic插件就是转化这些影像资料和动力学等的模拟结果转换为顶点缓存数据为Unity可以使用的文件。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417142130750.png)
abc文件转化为Unity可以识别的Prefabs文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417142155374.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=,size_16,color_FFFFFF,t_70)


## 二、参考网站及下载
参考网站：
[Alembic官方网站](http://www.alembic.io/index.html)
Github地址：
https://github.com/Unity-Technologies/AlembicForUnity
UnityPackge包下载链接：
https://download.csdn.net/download/q764424567/12333549

## 三、正文
### abc文件导入
首先我们要把这个包导入到场景中：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417150502958.png)
然后我们将.abc动画文件导入到Unity的Assets任意文件夹中，会发现文件导入之后就变成了Unity可识别的prefabs文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417150628190.png)
在StreamingAssets文件夹中会同步生成一个abc格式的文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004171507497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=,size_16,color_FFFFFF,t_70)
这是因为为了从文件中流传送数据，即使是build后也需要保留abc文件。

### abc导入Unity之后的格式

接着我们看一下导入的abc文件格式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417150845363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=,size_16,color_FFFFFF,t_70)
- Normals ：是使用.abc文件的法线，这是根据顶点位置来设定计算。默认的“Compute If Missing”是.abc文件如果有法线就使用，没有的话就计算，大部分情况下这样应该没有问题。
- Tangents：是计算切线的设定。因为abc文件没有切线，所以是计算还是不计算有两种选择。但是，切线的计算需要法线和UV，如果欠缺这些，“Tangents”和“Compute”也不能进行计算。
虽然默认是有效的，但是切线的计算是麻烦的过程，不需要的情况下可以设置成Compute可以更加高效
- Camera Aspect Ratio：设置相机的纵横比。是使用abc文件的相机参数，还是使用Unity侧画面的纵横比。
- Scale Factor：缩放因子，模型的等比例缩放
- Swap Handedness：将X方向反转，并且四边形分割成三角形时，三角形的排列也会反转。
- Interpolate Samples：是进行动画片的插值运算的设定。如果这是有效的，Transform、Camera和顶点不变化(=顶点数和索引不变)的Mesh就会得到动画的插值。

如果Interpolate Samples有效，或者如果abc文件中包含velocity数据，可以将velocity数据传递给着色器。
Alembic/Standard着色器是在普通的Standard着色器的基础上添加基于上述velocity的motion vector生成的着色器。
在需要motion vector的情况下会有帮助，比如后期效果的MotionBlur。

如果你想在自己的整形器中添加motion vector生成功能，可以修改SubShader中usepass " hidden/alembi/c/motionvectors motionvectors "这一行的代码。
内部的想知道详细情况,请参照alembicmotionvectors.cginc。(因为向第4个UV传递velocity数据，以此为基础计算出1帧前的顶点位置)

左边是未加工的，右边是输出motion vector并加上Post Processing Stack的MotionBlur的状态。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417152333889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=,size_16,color_FFFFFF,t_70)

### AlembicStreamPlayer组件
由插件生成的prefab有一个叫做AlembicStreamPlayer的组件，它负责播放。
移动Time参数可以确认Mesh的移动。
控制Timeline播放动画。
Vertex Motion Scale是计算velocity时的倍率。
越大的velocity越大，在后效果MotionBlur中会出现激烈的模糊。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417153531393.gif)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417153538342.gif)
代码：

```csharp
using UnityEngine;
using UTJ.Alembic;

public class Test_ABC : MonoBehaviour
{
    public GameObject m_AbcObjcect;
    AlembicStreamPlayer m_AlembicSP;
    float m_TempTime = 0;
    
    void Start()
    {
    	m_AlembicSP = m_AbcObjcect.GetComponent<AlembicStreamPlayer>();   
    }
    void Update()
    {
        m_TempTime += Time.deltaTime;
        m_AlembicSP.currentTime = m_TempTime;
        if (m_TempTime>3)
        {
            m_TempTime = 0;
        }
    }
}

```

### AlembicExporter组件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200417154149204.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E3NjQ0MjQ1Njc=,size_16,color_FFFFFF,t_70)
- Output Path：指定输出路径。
- Archive Type：指定Archive的格式，一般使用Ogawa就可以
- Xform Type：选择单独记录对象的位置、旋转、标度(TRS)还是矩阵记录(Matrix)。TRS应该没什么问题。
- Time Sampling Type：指定捕获的间隔。Alembic一帧间隔总是恒定的(1 / Frame Rate秒)。如果设置为Uniform那么就可以在Fix DeltaTime开始俘获，改写Time. maxdeltatime Unity方面也固定Delta时间。
在不影像制作的情况下，这应该是可取的行为，但是如果是独自管理Time.maxDeltaTime的话，就需要注意了。在Acyclic的情况下，Unity侧的delta时间就那样变成Alembic侧的帧间间隔。当然间隔不是一定的，但是对游戏进行的影响是最小的。主要是设想游戏的3d录像的模式。Start Time是Alembic一侧的开始时间。Frame Rate是Time Sampling类型为Uniform时的Alembic侧的帧间间隔。
- Swap Handedness：使之有效的话，夹入右手坐标系/左手坐标系改变的处理。很多DCC工具都是与Unity相反的坐标系，所以大部分都是有效的。
- Swap Faces：反转面的正反面。
- Scale Factor：缩放因子，缩放模型的比例
- Scope：捕捉场景内可捕捉的全部对象。目前的Branch只捕获带有Alembic Exporter组件的GameObject以下的树。