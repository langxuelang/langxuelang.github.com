---
layout: post
title: "blog-cocos2d"
date: 2013-03-01 15:02
comments: true
categories: 
---
#一个实现了多点触控的CCMenu类#
今天在做东西的时候，有一个问题一直没有解决，我的界面上有两个按钮，两个按钮分别绑定了不同的函数，但是，问题出现了，我想同时按下两个按钮的时候，只能响应一个，而另一个必须等待这个按钮抬起之后才能按下。

后来想了想也只能用多点触控了。
<!-- more -->
于是我自己写了一个类，继承了CCMenu。由于CCMenu继承了CCLayer，而CCMenu中又重写了
>virtual void registerWithTouchDispatcher()

>{

>CCDirector* pDirector = CCDirector::sharedDirector()；

>pDirector->getTouchDispatcher()->addTargetedDelegate(this, kCCMenuHandlerPriority, true);

>}

方法，这个方法就是用来开启单点触控的。
于是我们找到了CCMenu只能单点击的原因。
下面就开始写代码了。

	#include "cocos2d.h"
	class CCMultiTouchMenu:public cocos2d::CCMenu
	{
		public:
		virtual void ccTouchesBegan(cocos2d::CCSet *pTouches, cocos2d::CCEvent *pEvent);
		virtual void ccTouchesMoved(cocos2d::CCSet *pTouches, cocos2d::CCEvent *pEvent);
		virtual void ccTouchesEnded(cocos2d::CCSet *pTouches, cocos2d::CCEvent *pEvent);
		virtual void ccTouchesCancelled(cocos2d::CCSet *pTouches, cocos2d::CCEvent *pEvent);
		static CCMultiTouchMenu * create();
		void onEnter();
		void onExit();
		//virtual void registerWithTouchDispatcher();
		bool init();
		protected:
		private:
	};

上面是我写的类的头文件，大概解释一下。我重写了
>ccTouchesBegan

>ccTouchesMoved

>ccTouchesEnded

>ccTouchesCancelled

这四个函数，用来做自己的逻辑判断。同时我再自己的init函数里面做了如下操作
>CCMenu::init();

>CCDirector::sharedDirector()->getTouchDispatcher()->addStandardDelegate(this,kCCMenuHandlerPriority);

>addStandardDelegate()函数的作用就是重新开启CCLayer中的多点触控（CCLayer默认开启的是这个）


然后简单介绍一下我的cpp文件
	void CCMultiTouchMenu::ccTouchesBegan(CCSet *pTouches, CCEvent *pEvent)
	{
		CC_UNUSED_PARAM(pEvent);
		if (m_eState != kCCMenuStateWaiting || ! m_bIsVisible || !m_bEnabled)
		{
			return ;
		}
	
		for (CCNode *c = this->m_pParent; c != NULL; c = c->getParent())
		{
			if (c->isVisible() == false)
			{
				return ;
			}
		}
	
		for(CCSetIterator it = pTouches->begin();it!=pTouches->end();it++)
		{
			CCTouch *touchCur = (CCTouch *)*it;
			touchCur->locationInView();
			m_pSelectedItem = this->itemForTouch(touchCur);
			if (m_pSelectedItem)
			{
				m_eState = kCCMenuStateWaiting;
				m_pSelectedItem->selected();
				m_pSelectedItem->activate();
				return;
			}
		}
	}

原理就是遍历所有的触控点，然后分别让按钮进行响应其操作。

下面我附上我的源码，有需要的同学可以下载哦。
[CCMultiTouchMenu](http://pan.baidu.com/share/link?shareid=330180&uk=2970884173)

转载请标明出处原文地址：[]()