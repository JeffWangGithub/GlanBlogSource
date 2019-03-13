---
date: 2016-01-27 11:50
status: public
title: Android布局动画(LayoutTransition容器转换动画)--下
categories: Android
---

Android出现属性动画之后，提供了一套对ViewGroup中View的显示隐藏添加删除等操作的转换动画。
我们可以通过LayoutTransition对ViewGroup中的view改变进行动画处理。
### ViewGroup转换动画类型
- 触发条件：`当你添加或者移除ViewGroup中的View时，或者你调用View的setVisibility()方法来控制其显示或消失时，就处于一个转换状态。这种事件就有可能会激发动画。`当前被增加或者移除的View可以经历一个出现的动画或者一个消失的动画。而且不止是当前要控制的View，ViewGroup中的其他View也可以随之进行变动，比如经历一个动画移动到新的位置。
- 动画类型（四种）
    1. View本身出现的动画；APPEARING
    2. View消失的动画；DISAPPEARING    
    3. 由于新增了其他View而需要改变位置的View的动画；CHANGE_APPEARING    
    4. 由于移除了其他View而需要改变位置的动画；CHANGE_DISPPEARING（如果增加或移除了其他View之后，当前View的位置不需要改变，则无动画）
- API    
    1. ViewGroup.setLayoutTransition(LayoutTransition transition) : 给容器设置转换动画控制器LayoutTransition对象。
    2. LayoutTransition.setAnimator(int transitionType, Animator animator):给LayoutTransition    设置动画，并指定动画类型。transitionType有四中类型，即上边所说的四中类型。
    3. LayoutTransition.setStartDelay(int transitionType, long delay):给指定的动画类型设置启动延迟。
    4. LayoutTransition.setDuration(int transitionType, long duration):给指定的动画类型设置时间。
    -- 属性动画相关API    
    1. ObjectAnimator.ofPropertyValuesHolder(Object target,PropertyValuesHolder... values)：根据PropertyValuesHolder讲多个属性合并成一个动画。
    2. PropertyValuesHolder.ofFloat()/ofInt()/ofKeyframe():一系列获取PropertyValuesHolder的方法。
PropertyValuesHolder可以代表某种需要执行的属性动画，ObjectAnimator.ofPropertyValuesHolder（）可以将多个属性都有的动画进行合并成一个。
    3. Keyframe.ofFloat（）/ofInt()等获取Keyframe的方法。Keyframe标识再某个指定的帧上做动作，即是关键帧的意思。
    
### 实例

![](Android布局动画(LayoutAnimation)/animation.gif)

```java
public class TestLayoutTransitionActivity extends Activity {

    private LinearLayout mContainer;
    private Button mAddViewBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_layout_transition);

        mContainer = (LinearLayout) findViewById(R.id.container);
        mAddViewBtn = (Button) findViewById(R.id.add_view_btn);
        //1.创建LayoutTransition对象
        LayoutTransition layoutTransition = new LayoutTransition();
        //2.把LayoutTransition对象transitioner设置给这个外层容器
        mContainer.setLayoutTransition(layoutTransition);

        setCustomAnimations(layoutTransition);
        mAddViewBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                ImageView img = new ImageView(TestLayoutTransitionActivity.this);
                img.setImageResource(R.mipmap.ic_launcher);
                mContainer.addView(img, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT));
                img.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        mContainer.removeView(v);
//                        v.setVisibility(View.GONE);
                    }
                });
            }
        });

    }

    private void setCustomAnimations(LayoutTransition layoutTransition) {

        PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofFloat("scaleX",
                0f, 1f);
        PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofFloat("scaleY",
                0f, 1f);
        PropertyValuesHolder pvhScaleR = PropertyValuesHolder.ofFloat("rotation", 0, 360);

        //Keyframe标识再某个指定的帧上做动作，即是关键帧的意思
        Keyframe kf1 = Keyframe.ofFloat(0,0f);
        Keyframe kf2 = Keyframe.ofFloat(0.99f,2f);
        Keyframe kf3 = Keyframe.ofFloat(1f,1f);
        PropertyValuesHolder pvhKF = PropertyValuesHolder.ofKeyframe("scaleY",kf1,kf2,kf3);

        long changeAppearingTime = layoutTransition.getDuration(LayoutTransition.CHANGE_APPEARING);//获取默认时间

        //属性动画
        ObjectAnimator customChangingAppearingAnim = ObjectAnimator.ofPropertyValuesHolder(
                this, pvhScaleX, pvhScaleY, pvhScaleR,pvhKF).setDuration(changeAppearingTime);
//        customChangingAppearingAnim.setInterpolator(new OvershootInterpolator(0.8f));//设置插值器

        //3.设置动画给LayoutTransition对象，病指定动画的类型
        //设置APPEARING时的动画
        layoutTransition.setAnimator(LayoutTransition.APPEARING, customChangingAppearingAnim);

        //设置DISAPPEARING时的动画
        layoutTransition.setAnimator(LayoutTransition.DISAPPEARING, customChangingAppearingAnim);
    }
}
```
