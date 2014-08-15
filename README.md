AppMessageText
==============

Android角标-反编译QQ-apk所得, 为系统广播/Launcher提供(未深入研究)


1. Launcher上显示的
  
    Intent intent = new Intent(
				"android.intent.action.APPLICATION_MESSAGE_UPDATE");
		// replace the extra value with your_package_name/your_main_activity_name
		intent.putExtra(
				"android.intent.extra.update_application_component_name",
				"android.intclub.net./.MainActivity");
		// replase the extra value with the text your want to be displayed
		intent.putExtra("android.intent.extra.update_application_message_text",
				"999");
		sendBroadcast(intent);




2. 关于应用内部控件角标的实现:
github上有项目: 
	@jgilfelt/android-viewbadger
	@stefanjauker/BadgeView


下面也有实例:
	@https://bitbucket.org/droidwolf/superscriptview
		
如下图，角标在移动设备中是比较常见的ui元素。各种“最新”、“vip”、“最热”之类的层出不穷。

   

   在展现上最简单的做法是让ui同学ps一张角标图片输入“最新”、“vip”、“最热”等盖在要特别醒目提醒的控件上面即可。当然偷懒是没有一劳永逸的做法的，图片实现带文字的角标在当下android设备如此繁荣的情形下，码工们必然会为千奇百怪的适配而劳碌成大牛的，如果频繁更换图片中的文字ui设计师也会烦滴。
    下文探讨第一种角标的代码实现方式，其他三种还有比较少见的右下角、左下角的角标也可以照着做了。
   把第一种角标直观化卸妆，那么她是这样的
       

   而这个图形我们可以看作是一个小长方形旋转某个角度后，底边两点刚好和大长方形左边和顶边链接叠加所成的直角三角形。

   在android中TextView、LinearLayout是方形的控件。那么，如果我们用TextView替代小长方形、LinearLayout替代大长方形，在TextView靠顶部左对齐的情况下，由下图看到这个几何图形可从A->B->C变换而成。

  

   最终的图形那么是

  

  其中点d坐标即是TextView的位移点，而∠bah则是TextView反方向旋转的角度。

  剩下的就是中学几何计算了。忘了吧!我们复习下要用到的公式：

  sinA=对边/斜边

  cosA=邻边/斜边

  A角角度=arcsinA/π*180&deg;

  勾股定理 c^2=a^2+b^2

  好了，这几个公式够我们用了。

 

   根据UI同学的设计图，那么eb、ea两三角边是可以量出来滴(需要注意的是eb不一定=ea)。

通过垂直三角形、长方形的各种垂直、平衡关系

可得∠bah=∠ebf=∠deg=∠gad

ab=√(eb^2+ab^2)

ef=sin(∠ebf)*eb

dg=sin(∠gad)*da=sin(∠ebf)*ef

ga=cos(∠gad)*da=cos(∠ebf)*ef

eg=ea-ga

  到这里就是怎么在代码中对TextView做移动和旋转动作了。由于多次尝试直接对Canvas做移动旋转失败，所以我采用Animation来完成这一系列动作。下面是实现控件的代码。
	
package com.droidwolf.superscript;
 
import android.content.Context;
import android.graphics.Matrix;
import android.util.AttributeSet;
import android.util.DisplayMetrics;
import android.util.TypedValue;
import android.view.View;
import android.view.animation.Animation;
import android.view.animation.Transformation;
import android.widget.TextView;
 
public class SuperscriptView extends TextView {
    private float mDegress,mX,mY;
    private int mHeight,mWidth;
     
    public SuperscriptView(Context context) {
        super(context);
        init(context, null);
    }
 
    public SuperscriptView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }
 
    public SuperscriptView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init(context, attrs);
    }
    @Override
    public void setVisibility(int visibility) {
        setAnimation(visibility == View.VISIBLE? mAnimation: null);
        super.setVisibility(visibility);
    }
 
    private void init(Context context, AttributeSet attrs) {
        DisplayMetrics dm = getResources().getDisplayMetrics();
        int topEdge = Math.round(TypedValue.applyDimension( TypedValue.COMPLEX_UNIT_DIP, 41.333f, dm));
        int  leftEdge= Math.round(TypedValue.applyDimension( TypedValue.COMPLEX_UNIT_DIP, 41.333f, dm));
        calc(leftEdge, topEdge);
 
        mAnimation.setFillBefore(true);
        mAnimation.setFillAfter(true);
        mAnimation.setFillEnabled(true);
        startAnimation(mAnimation);
    }
 
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mHeight < 1 || mWidth < 1) {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        } else {
            setMeasuredDimension(mWidth, mHeight);
        }
    }
 
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        int vi= getVisibility();
        setAnimation(null);
        setVisibility(vi);
    }
     
    private void calc(int leftEdge, int topEdge) {
        final double ab = Math.sqrt(Math.pow(topEdge, 2d)+ Math.pow(leftEdge, 2d));
        final double sinB = leftEdge / ab;
        mDegress = -(float) Math.toDegrees(Math.asin(sinB));
 
        // ef=da=sin(∠ebf)*eb
        mHeight = Math.round((float) (sinB * topEdge));
 
        // de=sin(∠ead)*ea=sin(∠ebf)*ea
        final double de = sinB * leftEdge;
 
        // dg=cos(∠ead)*de=cos(∠ebf)*de
        mX = -(float) ((topEdge / ab) * de);
 
        // eg==sin(∠edg)*de=sin(∠ebf)*de
        mY = (float) (sinB * de);
        mWidth = Math.round((float) ab);
    }
 
    private Animation mAnimation = new Animation() {
        protected void applyTransformation(float interpolatedTime,Transformation t) {
            if (mHeight < 1 || mWidth < 1) {
                return;
            }
            Matrix tran = t.getMatrix();
            tran.setTranslate(mX, mY);
            tran.postRotate(mDegress, mX, mY);
        }
    };
}

 

   实现demo。我还实现了第一张图的另外三种角标到一个控件，请移步到我的bitbucket。
