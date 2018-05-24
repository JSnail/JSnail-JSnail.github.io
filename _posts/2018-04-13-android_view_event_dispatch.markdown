---
layout: post
title: Android事件分发
categories: Android
description: Android事件分发
keywords: Android, 事件分发
---



#### 首先简单的看一下有什么相关方法
![avatar](https://github.com/JSnail/JSnail.github.io/raw/master/img/event_dispatch.png)

### 1 基本知识点

#### 1.1 什么是事件
 - 当用户点击屏幕的时候(View或者ViewGroup)就会产生点击事件，并且会被封装成MotionEvent对象	

#### 1.2 什么是事件分发

- 事件分发实际上就是MotionEvent的传递以及消费过程

#### 1.3 事件会在什么对象之间传递

 Activity、ViewGroup、View

- Activity 最早获取到事件
- ViewGroup 除了判断自身是否消费以外还需要进行事件的分发
- View 事件传递的最终端，没有OnInterptTounchEvent()方法

#### 1.4 事件从上至下的传递顺序
Activity --> ViewGroup --> View

#### 1.5 事件由哪些方法协作完成

- dispatchTouchEvent()  用于事件的分发，当事件需要传递给View的时候就需要调用这个方法
- onInterceptTouchEvent() 判断事件是否拦截，只存在于ViewGroup中，在dispatchTouchEvent()这个方法中调用
- onTouchEvent() 处理点击事件 在dispatchTouchEvent()中调用

### 2 源码分析
#### 2.1 activity的事件相关

当一个时间发送的时候首先由activity获取到，所以我们首先查看Activity中的相关方法

	 public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        //如果getWindow().superDispatchTouchEvent(ev)返回为true表示事件被自己消费，事件结束，如果是false就继续往下传递
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
 

getWindow().superDispatchTouchEvent(ev)中getWindow()涉及到Activity的构成，activity的页面中最外层是PhoneWindow然后是DecorView 接着分为上下两部分在同一层，上面的是TitlView即平时页面里的标题栏，一个是ContentView,就是我们自己添加的布局存在的位置。

这里的getWindow()实际上就是获取到PhoneWindow的实现类，superDispatchTouchEvent()就是Window类的抽象方法，其实以上这部分的事件分发都不用在意，一般的在意的事件传递都由顶层的ViewGroup开始。	

#### 2.2 ViewGroup

只关心关键代码，一下为ViewGroup中的dispatchTouchEvent()方法

	 @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
    
      if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {//注释1
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;//注释2
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); 
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
            
            ````省略部分源码
            
              final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {//注释3
                            final int childIndex = getAndVerifyPreorderedIndex(
                                    childrenCount, i, customOrder);
                            final View child = getAndVerifyPreorderedView(
                                    preorderedList, children, childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {//注释4
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {//注释5
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
    
    }


- 注释1 首先判断事件是不是DOWN事件，如果是就出事换点击事件，并且设置mFirstTouchTarget为null,因为一个点击事件是由DOWN开始的。
mFirstTouchTarget表示当前ViewGroup是否拦截了时间，如果拦截了就为空，反之需要传递给子View处理则不为空。如果不是ACTION_DOWN事件，则设置intercepted = true，拦截事件，自身处理所有的事件。
- 注释2 FLAG_ DISALLOW_INTERCEPT 表示禁制ViewGroup拦截除DOWN之外的所有方法，一般通过子view的requestDisallowInterceptTounchEvent() 来设置
- 注释3 首先倒序由最上层的view开始向内层遍历，判断子view是否能接受点击事件
- 注释4 判断view是否在范围之内，或者view是否正在播放动画。
- 注释5 看看dispatchTransformedTouchEvent()方法内部在做什么

		private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        		final boolean handled;

      
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }
        ·····
        }

该方法内部判断ViewGroup是否有子View,如果有就调用他的dispatchTouchEvent()方法，如果没有就调用父类即View的dispatchTouchEvent()方法， 那么看看View的该方法内部实现了什么

	       if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

该方法内部判断了如果OnTouchListener不为null且ontouch返回true就表示时间被消费，否则就执行onTouchEvent()方法，这里也就能判断出ontouch的调用层级高于onTouchEvent，然后终于到了事件最终处理的地方

    public boolean onTouchEvent(MotionEvent event) {
      final int action = event.getAction();

        final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;//注释1
      if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
          
            return clickable;
        }
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
    
    ····
    
          switch (action) {
                case MotionEvent.ACTION_UP:
                            boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                    if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                       
                        boolean focusTaken = false;
                        if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                            focusTaken = requestFocus();
                        }

                        if (prepressed) {
                               setPressed(true, x, y);
                        }

                        if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                            // This is a tap, so remove the longpress check
                            removeLongPressCallback();

                            // Only perform take click actions if we were in the pressed state
                            if (!focusTaken) {
                                // Use a Runnable and post this rather than calling
                                // performClick directly. This lets other visual state
                                // of the view update before click actions start.
                                if (mPerformClick == null) {
                                    mPerformClick = new PerformClick();
                                }
                                if (!post(mPerformClick)) {
                                    performClick();
                                }
                            }
                        }                            }    }

从上面的注释1和注释2的地方可以看到只要view设置了CLICKABLE或LONG_CLICKABLE中的一个为true，那么onTouchEvent(event)就会消费掉整个事件。这两个参数可以通过view的setClickable和setLongClickable方法来设置，也可以通过设置监听方法来实现。然后在MotionEvent.ACTION_UP中会执行performClick()方法，

	    public boolean performClick() {
        final boolean result;
        final ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);
            result = true;
        } else {
            result = false;
        }

        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        notifyEnterOrExitForAutoFillIfNeeded(true);

        return result;
    }
  

  从代码中可以看出只要设置了OnClickListener，那么就会执行onClick方法。自此view的事件分发就分析完成

### 3 总结   
当事件产生之后会有Activity来处理，然后传递给PhoneWindow，再传递给DercorView,最后传递给顶层的ViewGroup。对于顶层ViewGroup，点击事件首先由他的dispatchTounchEvent()方法接收处理，如果他的onInterceptTouchEvent()方法范围true，表示事件被拦截，由他的onTouchEvent()方法处理；如果返回为false，表示他不拦截整个事件，那么事件就会被分发到他的子元素dispatchTounchEvent()来处理，由此重复进行下去，，最终传递到view中，由于他没有dispatchEvent()的方法，一般是由view的onTouchEvent()消费掉，如果没有处理，就会传回去，到最顶层去最后被消费掉

	



	