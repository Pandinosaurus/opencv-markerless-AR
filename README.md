##http://meiroo.github.io/#explorer

###基于takmin项目

原项目地址： https://github.com/takmin/OpenCV-Marker-less-AR

###AR：使用摄像头识别特定图像目标并基于目标附加3D展示动画。

takmin的开源项目使用了OpenCV识别目标，再用OpenGL绘制增强现实效果。但只有Windows版本。本项目打算重写模型和渲染部分，并移植到Android版本。
目前基于takmin的代码编译Windows版本成功,修复部分bug，并替换OpenCV版本为249。
搭建了本项目的Android版本，目前在Android版本上成功移植识别部分。即使用Android Camera进行OpenCV识别后绘制基本的OpenGL图形。 下一步调研继续优化此工程的基本算法，并完善Android版本。

###编译以及Debug
 * OpenCV249
 * GLUT
 * OpenGL11
 * VS2010

###Android改造
 * 基于OpenCV Sample tutorial-2-mixedprocessing， 此sample采用Java-JNI-C++混合开发方式。其中Java负责界面显示。C++负责主要业务。
 * 基于Sample，Java部分增加GLSurfaceView / Render，使用FrameLayout叠加到org.opencv.android.JavaCameraView前面。OpenGL部分设置alpha可以透过背景。从而完成CV和GL图案叠加。
 * 重新定制JNI部分，两个View均通过JNI调用C++部分。
 * 由于OpenCV249的Android库不包含 nonfree 这个库。 所以增加了一个工程通过windows代码，编译出此so 。
 * 移植识别部分代码，OpenCV的JavaCameraView捕获的图像传递到C++，识别后返回匹配的图像ID到Java部分。
 * Java部分将识别结果通知GLSurfaceView，OpenGL绘制3D图案。
 * Camera矩阵部分还没有处理。
 
 ###技术点
 
 CV和GL的叠加
 
 ```
 
 <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    
    
    <meiroo.cvgl.GlBufferView
	   android:id="@+id/glbuffer_view"
	   android:layout_width="fill_parent"
	   android:layout_height="fill_parent"/>
	    

    <org.opencv.android.JavaCameraView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:id="@+id/tutorial2_activity_surface_view" />
    
    

</FrameLayout>
 ```
 
 
 识别部分，
 
 ```
 
 //java部分将Mat的地址发送
 public Mat onCameraFrame(CvCameraViewFrame inputFrame) {
      // input frame has RGBA format
      mRgba = inputFrame.rgba();
      mGray = inputFrame.gray();
  		result = native_FindFeatures(mGray.getNativeObjAddr(), mRgba.getNativeObjAddr());
}

//native部分
int native_FindFeatures(JNIEnv *env,jclass clazz,jlong addrGray, jlong addrRgba)
{
	
	//LOGI("native_FindFeatures  gray:%ld rgba:%ld",addrGray,addrRgba);
	Mat& frame  = *(Mat*)addrGray;
	...
	//先缩放
	cv::resize(frame, query_image, query_image.size());
  //然后特征点比对
	vector<cvar::resultInfo> recog_result = ctrlOR.queryImage(query_image);
	if(!recog_result.empty()){
		return recog_result[0].img_id;
	}
	
}
 ```
 
 native接口
 ```
 //opengl surfaceview interface
 void native_start(JNIEnv *env, jclass clazz);
void native_gl_resize(JNIEnv *env, jclass clazz, jint w, jint h);
void native_gl_render(JNIEnv *env, jclass clazz);
void native_key_event(JNIEnv *env,jclass clazz,jint key,jint status);
void native_touch_event(JNIEnv *env,jclass clazz,jfloat x,jfloat y,jint status);
//opencv cameraview interface 
int native_FindFeatures(JNIEnv *env,jclass clazz,jlong addrGray, jlong addrRgba);
 ```
 
 ###问题点：
  * 慢， Windows版本release很流畅。debug都比较慢。（台式机i3处理器）
  * Android版本也比较慢。但是还能接受。（大黄蜂）
  * 现在只有两个样本。对比样本增大后速度如何保证。算法识别部分需要分析下复杂度。





###以下为原作者readme

2012/01/10
Marker less AR is using natural images as markers instead of monochromatic markers.   This program includes object recognition, tracking, and overlay of 3D model functions.
You can use this program source code under MIT license.
http://www.opensource.org/licenses/mit-license.php

This program was written in C++ using OpenCV 2.3.1 and GLUT 3.7.6.  You should install these libraries before you compile this source code. 

GLUT
http://www.opengl.org/resources/libraries/glut/

OpenCV
https://sourceforge.net/projects/opencvlibrary/

This program also includes "GLMetaseq" which had been developed by Sunao Hashimoto and Keisuke Konishi.
GLMetaseq is 3D model loader of mqo format.

GLMetaseq
http://kougaku-navi.net/ARToolKit.html

The Windows demo program "ARengine.exe" includes a 3D models "mikuX.mqo", I obtained these files created by Zusa-san from:
http://nanoha.kirara.st/3dcg/file/dlrank.php

This program consists of 3 parts: Object Recognition, Tracking, and Overlay.  You can use Object Recognition function apart from tracking and overlay.

If you are a Windows user, I strongly recommend to begin with demo program.  This demo program is build on Windows 7 (32bit).  You can find how to use it in "HowToUse.pdf" in WinDemo.zip.
This source code is still alpha version and I've not written enough documents yet.  
This code is still on the way of refactoring, so API interface design will be changed later.  I will write API documents after refactoring.
At the current version, the only document "HowToUse.pdf" may be helpful to understand this application.

If you have any question, please contact here:
Takuya MINAGAWA (z.takmin@gmail.com)
