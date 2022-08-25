---
categories:
  - 图形学
date: '2016-05-10T22:01:34'
description: ''
tags:
  - OpenCV
  - Mat
title: OpenCV的Mat类型以及基本函数使用
---



# OpenCV的Mat类型以及基本函数使用

## Mat和IplImage的区别

### Mat和IplImage的主要区别

在OpenCV中IplImage是表示一个图像的结构体，也是从OpenCV1.0到目前最为重要的一个结构；在之前的图像表示用IplImage，而且之前的OpenCV是用C语言编写的，提供的接口也是C语言接口。

Mat是后来OpenCV封装的一个C++类，用来表示一个图像，和IplImage表示基本一致，但是Mat还添加了一些图像函数。

### IplImage

IplImage数据结构的定义在opencv\build\include\opencv2\core\types_c.h文件中。

```c++
typedef struct _IplImage
{
    int  nSize;             /* sizeof(IplImage) */
    int  ID;                /* version (=0)*/
    int  nChannels;         /* Most of OpenCV functions support 1,2,3 or 4 channels */
    int  alphaChannel;      /* Ignored by OpenCV */
    int  depth;             /* Pixel depth in bits: IPL_DEPTH_8U, IPL_DEPTH_8S, IPL_DEPTH_16S,
                               IPL_DEPTH_32S, IPL_DEPTH_32F and IPL_DEPTH_64F are supported.  */
    char colorModel[4];     /* Ignored by OpenCV */
    char channelSeq[4];     /* ditto */
    int  dataOrder;         /* 0 - interleaved color channels, 1 - separate color channels.
                               cvCreateImage can only create interleaved images */
    int  origin;            /* 0 - top-left origin,
                               1 - bottom-left origin (Windows bitmaps style).  */
    int  align;             /* Alignment of image rows (4 or 8).
                               OpenCV ignores it and uses widthStep instead.    */
    int  width;             /* Image width in pixels.                           */
    int  height;            /* Image height in pixels.                          */
    struct _IplROI *roi;    /* Image ROI. If NULL, the whole image is selected. */
    struct _IplImage *maskROI;      /* Must be NULL. */
    void  *imageId;                 /* "           " */
    struct _IplTileInfo *tileInfo;  /* "           " */
    int  imageSize;         /* Image data size in bytes
                               (==image->height*image->widthStep
                               in case of interleaved data)*/
    char *imageData;        /* Pointer to aligned image data.         */
    int  widthStep;         /* Size of aligned image row in bytes.    */
    int  BorderMode[4];     /* Ignored by OpenCV.                     */
    int  BorderConst[4];    /* Ditto.                                 */
    char *imageDataOrigin;  /* Pointer to very origin of image data
                               (not necessarily aligned) -
                               needed for correct deallocation */
}
IplImage;
```

可见，IplImage是一个表示图像的结构体：C语言操作OpenCV的数据结构。地位等同于Mat，可以说是历史版本了。

<!--more-->

### Mat

Mat这个数据结构定义在opencv\build\include\opencv2\core\core.hpp这个文件。

```c++
class CV_EXPORTS Mat
{
public:
    //! default constructor
    Mat();
    //! constructs 2D matrix of the specified size and type
    // (_type is CV_8UC1, CV_64FC3, CV_32SC(12) etc.)
    Mat(int rows, int cols, int type);
    Mat(Size size, int type);
    //! constucts 2D matrix and fills it with the specified value _s.
    Mat(int rows, int cols, int type, const Scalar& s);
    Mat(Size size, int type, const Scalar& s);

    //! constructs n-dimensional matrix
    Mat(int ndims, const int* sizes, int type);
    Mat(int ndims, const int* sizes, int type, const Scalar& s);

    //! copy constructor
    Mat(const Mat& m);
    //! constructor for matrix headers pointing to user-allocated data
    Mat(int rows, int cols, int type, void* data, size_t step=AUTO_STEP);
    Mat(Size size, int type, void* data, size_t step=AUTO_STEP);
    Mat(int ndims, const int* sizes, int type, void* data, const size_t* steps=0);

    //! creates a matrix header for a part of the bigger matrix
    Mat(const Mat& m, const Range& rowRange, const Range& colRange=Range::all());
    Mat(const Mat& m, const Rect& roi);
    Mat(const Mat& m, const Range* ranges);
    //! converts old-style CvMat to the new matrix; the data is not copied by default
    Mat(const CvMat* m, bool copyData=false);
    //! converts old-style CvMatND to the new matrix; the data is not copied by default
    Mat(const CvMatND* m, bool copyData=false);
    //! converts old-style IplImage to the new matrix; the data is not copied by default
    Mat(const IplImage* img, bool copyData=false);
    //! builds matrix from std::vector with or without copying the data

   ......

protected:
    void initEmpty();
};
```

Mat是OpenCV最基本的数据结构，Mat即矩阵（Matrix）的缩写我们在读取图片的时候就是将图片定义为Mat类型，其重载的构造函数一大堆。

其中有一个构造函数可以很方便的直接将IplImage转化为Mat

```c++
Mat(const IplImage* img, bool copyData=false);
```

## 基本函数使用

### imread

功能：从一个文件中载入图片

定义：

```c++
Mat imread( const string& filename, int flags=1 );
```

■第一个参数，const string&类型的filename，这是我们需要载入的图片路径名。

在Windows操作系统下，OpenCV的imread函数支持常用的图片类型，比如bmp,jpg,jpeg,png等等。

■第二个参数，int类型的flags，为载入标识，它指定一个加载图像的颜色类型。可以看到它自带缺省值1.所以有时候这个参数在调用时我们可以忽略。如果在调用时忽略这个参数，就表示载入三通道的彩色图像。具体原因看下面的解释。

flags是int型的变量，我们可以按如下方式取值：

- flags >0返回一个3通道的彩色图像。
- flags =0返回灰度图像。
- flags <0返回包含Alpha通道的加载的图像。

需要注意的点：输出的图像默认情况下是不载入Alpha通道进来的。如果我们需要载入Alpha通道的话呢，这里就需要取负值。

 所以默认值flags=1表示载入三通道的彩色图像。

### imshow

功能：显示一个图像

定义：

```c++
void imshow(const string& winname, InputArray mat);  
```

 ■ 第一个参数，const string&类型的winname，填需要显示的窗口标识名称。

 ■ 第二个参数，InputArray 类型的mat，填需要显示的图像。

**InputArray 类型是什么类型？**

通过转到定义，我们可以在opencv\build\include\opencv2\highgui\highgui.hpp文件中找到imshow的原型：

```c++
CV_EXPORTS_W void imshow(const string& winname, InputArray mat);
```

进一步对InputArray转到定义，在opencv\build\include\opencv2\core\core.hpp文件中查到一个typedef声明：

```c++
typedef const _InputArray& InputArray;  
```

这其实一个类型声明引用，就是说`_InputArray`和`InputArray`是一个意思，然后再次对_InputArray进行转到定义，终于，在opencv\build\include\opencv2\core\core.hpp文件中发现了InputArray的真身：

```
class CV_EXPORTS _InputArray
{
public:
    enum {
        KIND_SHIFT = 16,
        FIXED_TYPE = 0x8000 << KIND_SHIFT,
        FIXED_SIZE = 0x4000 << KIND_SHIFT,
        KIND_MASK = ~(FIXED_TYPE|FIXED_SIZE) - (1 << KIND_SHIFT) + 1,

        NONE              = 0 << KIND_SHIFT,
        MAT               = 1 << KIND_SHIFT,
        MATX              = 2 << KIND_SHIFT,
        STD_VECTOR        = 3 << KIND_SHIFT,
        STD_VECTOR_VECTOR = 4 << KIND_SHIFT,
        STD_VECTOR_MAT    = 5 << KIND_SHIFT,
        EXPR              = 6 << KIND_SHIFT,
        OPENGL_BUFFER     = 7 << KIND_SHIFT,
        OPENGL_TEXTURE    = 8 << KIND_SHIFT,
        GPU_MAT           = 9 << KIND_SHIFT,
        OCL_MAT           =10 << KIND_SHIFT
    };
    _InputArray();

    _InputArray(const Mat& m);
    _InputArray(const MatExpr& expr);
    template<typename _Tp> _InputArray(const _Tp* vec, int n);
    template<typename _Tp> _InputArray(const vector<_Tp>& vec);
    template<typename _Tp> _InputArray(const vector<vector<_Tp> >& vec);
    _InputArray(const vector<Mat>& vec);
    template<typename _Tp> _InputArray(const vector<Mat_<_Tp> >& vec);
    template<typename _Tp> _InputArray(const Mat_<_Tp>& m);
    template<typename _Tp, int m, int n> _InputArray(const Matx<_Tp, m, n>& matx);
    _InputArray(const Scalar& s);
    _InputArray(const double& val);
    // < Deprecated
    _InputArray(const GlBuffer& buf);
    _InputArray(const GlTexture& tex);
    // >
    _InputArray(const gpu::GpuMat& d_mat);
    _InputArray(const ogl::Buffer& buf);
    _InputArray(const ogl::Texture2D& tex);

    virtual Mat getMat(int i=-1) const;
    virtual void getMatVector(vector<Mat>& mv) const;
    // < Deprecated
    virtual GlBuffer getGlBuffer() const;
    virtual GlTexture getGlTexture() const;
    // >
    virtual gpu::GpuMat getGpuMat() const;
    /*virtual*/ ogl::Buffer getOGlBuffer() const;
    /*virtual*/ ogl::Texture2D getOGlTexture2D() const;

    virtual int kind() const;
    virtual Size size(int i=-1) const;
    virtual size_t total(int i=-1) const;
    virtual int type(int i=-1) const;
    virtual int depth(int i=-1) const;
    virtual int channels(int i=-1) const;
    virtual bool empty() const;

#ifdef OPENCV_CAN_BREAK_BINARY_COMPATIBILITY
    virtual ~_InputArray();
#endif

    int flags;
    void* obj;
    Size sz;
};
```

可以看到，_InputArray类的里面首先定义了一个枚举，然后定了各个构造函数和虚函数。很多时候，遇到函数原型中的InputArray类型，我们把它简单地当做Mat类型就行了。

imshow 函数用于在指定的窗口中显示图像。如果窗口是用CV_WINDOW_AUTOSIZE（默认值）标志创建的，那么显示图像原始大小。否则，将图像进行缩放以适合窗口。而imshow 函数缩放图像，取决于图像的深度：

- 如果载入的图像是8位无符号类型（8-bit unsigned），就显示图像本来的样子。
- 如果图像是16位无符号类型（16-bit unsigned）或32位整型（32-bit integer），便用像素值除以256。也就是说，值的范围是[0,255 x 256]映射到[0,255]。
- 如果图像是32位浮点型（32-bit floating-point），像素值便要乘以255。也就是说，该值的范围是[0,1]映射到[0,255]。

### imwrite

功能：输出图像到文件

定义：

```c++
bool imwrite( const string& filename, InputArray img,
              const vector<int>& params=vector<int>());
```

 ■ 第一个参数，const string&类型的filename，填需要写入的文件名就行了，带上后缀，比如，“123.jpg”这样。

 ■ 第二个参数，InputArray类型的img，一般填一个Mat类型的图像数据就行了。

 ■ 第三个参数，`const vector<int>`&类型的params，表示为特定格式保存的参数编码，它有默认值`vector<int>()`，所以一般情况下不需要填写。

### cvtcolor

功能：将一个图像的颜色空间转换到另一种（Converts an image from one color space to another.）

参考：[cvtcolor](http://www.opencv.org.cn/opencvdoc/2.3.2/html/modules/imgproc/doc/miscellaneous_transformations.html?highlight=cvtcolor#cvtcolor)

定义：

```c++
void cvtColor( InputArray src, OutputArray dst, int code, int dstCn=0 );
```

 ■ 第一个参数，InputArray类型的src ,-- Source image

 ■ 第二个参数，OutputArray类型的dst，Destination image of the same size and depth as src

 ■ 第三个参数，int类型的code，颜色空间变换代码Color space conversion code。

具体的变换代码参见：opencv\build\include\opencv2\imgproc\types_c.h文件中的第87行，枚举类型。

```c++
/* Constants for color conversion */
enum
{
    CV_BGR2BGRA    =0,
    CV_RGB2RGBA    =CV_BGR2BGRA,

    CV_BGRA2BGR    =1,
    CV_RGBA2RGB    =CV_BGRA2BGR,

    CV_BGR2RGBA    =2,
    CV_RGB2BGRA    =CV_BGR2RGBA,

    CV_RGBA2BGR    =3,
    CV_BGRA2RGB    =CV_RGBA2BGR,

    CV_BGR2RGB     =4,
    CV_RGB2BGR     =CV_BGR2RGB,

    CV_BGRA2RGBA   =5,
    CV_RGBA2BGRA   =CV_BGRA2RGBA,

    CV_BGR2GRAY    =6,
    CV_RGB2GRAY    =7,
    CV_GRAY2BGR    =8,
    CV_GRAY2RGB    =CV_GRAY2BGR,
    CV_GRAY2BGRA   =9,
    CV_GRAY2RGBA   =CV_GRAY2BGRA,
    CV_BGRA2GRAY   =10,
    CV_RGBA2GRAY   =11,

    CV_BGR2BGR565  =12,
    CV_RGB2BGR565  =13,
    CV_BGR5652BGR  =14,
    CV_BGR5652RGB  =15,
    CV_BGRA2BGR565 =16,
    CV_RGBA2BGR565 =17,
    CV_BGR5652BGRA =18,
    CV_BGR5652RGBA =19,

    CV_GRAY2BGR565 =20,
    CV_BGR5652GRAY =21,

    CV_BGR2BGR555  =22,
    CV_RGB2BGR555  =23,
    CV_BGR5552BGR  =24,
    CV_BGR5552RGB  =25,
    CV_BGRA2BGR555 =26,
    CV_RGBA2BGR555 =27,
    CV_BGR5552BGRA =28,
    CV_BGR5552RGBA =29,

    CV_GRAY2BGR555 =30,
    CV_BGR5552GRAY =31,

    CV_BGR2XYZ     =32,
    CV_RGB2XYZ     =33,
    CV_XYZ2BGR     =34,
    CV_XYZ2RGB     =35,

    CV_BGR2YCrCb   =36,
    CV_RGB2YCrCb   =37,
    CV_YCrCb2BGR   =38,
    CV_YCrCb2RGB   =39,

    CV_BGR2HSV     =40,
    CV_RGB2HSV     =41,

    CV_BGR2Lab     =44,
    CV_RGB2Lab     =45,

    CV_BayerBG2BGR =46,
    CV_BayerGB2BGR =47,
    CV_BayerRG2BGR =48,
    CV_BayerGR2BGR =49,

    CV_BayerBG2RGB =CV_BayerRG2BGR,
    CV_BayerGB2RGB =CV_BayerGR2BGR,
    CV_BayerRG2RGB =CV_BayerBG2BGR,
    CV_BayerGR2RGB =CV_BayerGB2BGR,

    CV_BGR2Luv     =50,
    CV_RGB2Luv     =51,
    CV_BGR2HLS     =52,
    CV_RGB2HLS     =53,

    CV_HSV2BGR     =54,
    CV_HSV2RGB     =55,

    CV_Lab2BGR     =56,
    CV_Lab2RGB     =57,
    CV_Luv2BGR     =58,
    CV_Luv2RGB     =59,
    CV_HLS2BGR     =60,
    CV_HLS2RGB     =61,

    CV_BayerBG2BGR_VNG =62,
    CV_BayerGB2BGR_VNG =63,
    CV_BayerRG2BGR_VNG =64,
    CV_BayerGR2BGR_VNG =65,

    CV_BayerBG2RGB_VNG =CV_BayerRG2BGR_VNG,
    CV_BayerGB2RGB_VNG =CV_BayerGR2BGR_VNG,
    CV_BayerRG2RGB_VNG =CV_BayerBG2BGR_VNG,
    CV_BayerGR2RGB_VNG =CV_BayerGB2BGR_VNG,

    CV_BGR2HSV_FULL = 66,
    CV_RGB2HSV_FULL = 67,
    CV_BGR2HLS_FULL = 68,
    CV_RGB2HLS_FULL = 69,

    CV_HSV2BGR_FULL = 70,
    CV_HSV2RGB_FULL = 71,
    CV_HLS2BGR_FULL = 72,
    CV_HLS2RGB_FULL = 73,

    CV_LBGR2Lab     = 74,
    CV_LRGB2Lab     = 75,
    CV_LBGR2Luv     = 76,
    CV_LRGB2Luv     = 77,

    CV_Lab2LBGR     = 78,
    CV_Lab2LRGB     = 79,
    CV_Luv2LBGR     = 80,
    CV_Luv2LRGB     = 81,

    CV_BGR2YUV      = 82,
    CV_RGB2YUV      = 83,
    CV_YUV2BGR      = 84,
    CV_YUV2RGB      = 85,

    CV_BayerBG2GRAY = 86,
    CV_BayerGB2GRAY = 87,
    CV_BayerRG2GRAY = 88,
    CV_BayerGR2GRAY = 89,

    //YUV 4:2:0 formats family
    CV_YUV2RGB_NV12 = 90,
    CV_YUV2BGR_NV12 = 91,
    CV_YUV2RGB_NV21 = 92,
    CV_YUV2BGR_NV21 = 93,
    CV_YUV420sp2RGB = CV_YUV2RGB_NV21,
    CV_YUV420sp2BGR = CV_YUV2BGR_NV21,

    CV_YUV2RGBA_NV12 = 94,
    CV_YUV2BGRA_NV12 = 95,
    CV_YUV2RGBA_NV21 = 96,
    CV_YUV2BGRA_NV21 = 97,
    CV_YUV420sp2RGBA = CV_YUV2RGBA_NV21,
    CV_YUV420sp2BGRA = CV_YUV2BGRA_NV21,

    CV_YUV2RGB_YV12 = 98,
    CV_YUV2BGR_YV12 = 99,
    CV_YUV2RGB_IYUV = 100,
    CV_YUV2BGR_IYUV = 101,
    CV_YUV2RGB_I420 = CV_YUV2RGB_IYUV,
    CV_YUV2BGR_I420 = CV_YUV2BGR_IYUV,
    CV_YUV420p2RGB = CV_YUV2RGB_YV12,
    CV_YUV420p2BGR = CV_YUV2BGR_YV12,

    CV_YUV2RGBA_YV12 = 102,
    CV_YUV2BGRA_YV12 = 103,
    CV_YUV2RGBA_IYUV = 104,
    CV_YUV2BGRA_IYUV = 105,
    CV_YUV2RGBA_I420 = CV_YUV2RGBA_IYUV,
    CV_YUV2BGRA_I420 = CV_YUV2BGRA_IYUV,
    CV_YUV420p2RGBA = CV_YUV2RGBA_YV12,
    CV_YUV420p2BGRA = CV_YUV2BGRA_YV12,

    CV_YUV2GRAY_420 = 106,
    CV_YUV2GRAY_NV21 = CV_YUV2GRAY_420,
    CV_YUV2GRAY_NV12 = CV_YUV2GRAY_420,
    CV_YUV2GRAY_YV12 = CV_YUV2GRAY_420,
    CV_YUV2GRAY_IYUV = CV_YUV2GRAY_420,
    CV_YUV2GRAY_I420 = CV_YUV2GRAY_420,
    CV_YUV420sp2GRAY = CV_YUV2GRAY_420,
    CV_YUV420p2GRAY = CV_YUV2GRAY_420,

    //YUV 4:2:2 formats family
    CV_YUV2RGB_UYVY = 107,
    CV_YUV2BGR_UYVY = 108,
    //CV_YUV2RGB_VYUY = 109,
    //CV_YUV2BGR_VYUY = 110,
    CV_YUV2RGB_Y422 = CV_YUV2RGB_UYVY,
    CV_YUV2BGR_Y422 = CV_YUV2BGR_UYVY,
    CV_YUV2RGB_UYNV = CV_YUV2RGB_UYVY,
    CV_YUV2BGR_UYNV = CV_YUV2BGR_UYVY,

    CV_YUV2RGBA_UYVY = 111,
    CV_YUV2BGRA_UYVY = 112,
    //CV_YUV2RGBA_VYUY = 113,
    //CV_YUV2BGRA_VYUY = 114,
    CV_YUV2RGBA_Y422 = CV_YUV2RGBA_UYVY,
    CV_YUV2BGRA_Y422 = CV_YUV2BGRA_UYVY,
    CV_YUV2RGBA_UYNV = CV_YUV2RGBA_UYVY,
    CV_YUV2BGRA_UYNV = CV_YUV2BGRA_UYVY,

    CV_YUV2RGB_YUY2 = 115,
    CV_YUV2BGR_YUY2 = 116,
    CV_YUV2RGB_YVYU = 117,
    CV_YUV2BGR_YVYU = 118,
    CV_YUV2RGB_YUYV = CV_YUV2RGB_YUY2,
    CV_YUV2BGR_YUYV = CV_YUV2BGR_YUY2,
    CV_YUV2RGB_YUNV = CV_YUV2RGB_YUY2,
    CV_YUV2BGR_YUNV = CV_YUV2BGR_YUY2,

    CV_YUV2RGBA_YUY2 = 119,
    CV_YUV2BGRA_YUY2 = 120,
    CV_YUV2RGBA_YVYU = 121,
    CV_YUV2BGRA_YVYU = 122,
    CV_YUV2RGBA_YUYV = CV_YUV2RGBA_YUY2,
    CV_YUV2BGRA_YUYV = CV_YUV2BGRA_YUY2,
    CV_YUV2RGBA_YUNV = CV_YUV2RGBA_YUY2,
    CV_YUV2BGRA_YUNV = CV_YUV2BGRA_YUY2,

    CV_YUV2GRAY_UYVY = 123,
    CV_YUV2GRAY_YUY2 = 124,
    //CV_YUV2GRAY_VYUY = CV_YUV2GRAY_UYVY,
    CV_YUV2GRAY_Y422 = CV_YUV2GRAY_UYVY,
    CV_YUV2GRAY_UYNV = CV_YUV2GRAY_UYVY,
    CV_YUV2GRAY_YVYU = CV_YUV2GRAY_YUY2,
    CV_YUV2GRAY_YUYV = CV_YUV2GRAY_YUY2,
    CV_YUV2GRAY_YUNV = CV_YUV2GRAY_YUY2,

    // alpha premultiplication
    CV_RGBA2mRGBA = 125,
    CV_mRGBA2RGBA = 126,

    CV_RGB2YUV_I420 = 127,
    CV_BGR2YUV_I420 = 128,
    CV_RGB2YUV_IYUV = CV_RGB2YUV_I420,
    CV_BGR2YUV_IYUV = CV_BGR2YUV_I420,

    CV_RGBA2YUV_I420 = 129,
    CV_BGRA2YUV_I420 = 130,
    CV_RGBA2YUV_IYUV = CV_RGBA2YUV_I420,
    CV_BGRA2YUV_IYUV = CV_BGRA2YUV_I420,
    CV_RGB2YUV_YV12  = 131,
    CV_BGR2YUV_YV12  = 132,
    CV_RGBA2YUV_YV12 = 133,
    CV_BGRA2YUV_YV12 = 134,

    CV_COLORCVT_MAX  = 135
};
```

 ■ 第四个参数，int类型的dstCn，dst中的通道数（channel number ），dstCn默认为0，表示 dst中通道数自动从src和code中获取。

示例：

```c++
//将彩色图像image1变换为灰度图像gray_image1
cvtColor(image1,gray_image1,CV_RGB2GRAY);
```

## 综合示例

```
// VS2010 + OpenCV2.4.9

#include<opencv2/core/core.hpp>
#include<opencv2/highgui/highgui.hpp>

using namespace cv;


int main( )
{
	Mat girl=imread("girl.jpg"); //载入图像到Mat
	namedWindow("girl.jpg"); 
	imshow("girl.jpg",girl);

	//载入图片
	Mat image= imread("11.jpg",199);

	//载入后先显示
	namedWindow("11.jpg");
	imshow("11.jpg",image);

	//输出一张jpg图片到工程目录下
	imwrite("10.jpg",image);
	 
	waitKey();

	return 0;
}
```

