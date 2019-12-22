# View的事件体系
[toc]
## MotionEvent(动作事件)
典型的事件类型有：
- ACTION_DOWN：手指刚接触屏幕；
- ACTION_MOVE：手指在屏幕上移动；
- ACTION_UP：手机从屏幕上离开的一瞬间。

TouchSlop：这是系统所能识别出的被认为是滑动的最小距离。当手指在屏幕上滑动时，如果两次滑动之间的距离小于这个常量，则系统就不认为是在进行滑动操作。

VelocityTracker：速度追踪，用于追踪手指在滑动过程中的速度，包括水平和竖直方向的速度。

GestureDetector：手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。

Scroller：弹性滑动对象，用于实现View的弹性滑动。当使用View的scrollTo/scrollBy方法来进行滑动时，其过程是瞬间完成的，所以用户体验效果不好。