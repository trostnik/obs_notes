To create custom view we need to extend either View or ViewGroup classes. Then, we need to override methods:
## onDraw()   
This method delivers Canvas, which we can use to draw anything we want
## onMeasure()
This method is used to calculate width and height of our view. It provides `widthMeasureSpec` and `heightMeasureSpec` parameters which are meant to be used as restrictions of size of the component. Then, we calculate size based on restrictions (we can exceed them) and call setMeasuredDimension() method with calculated measurements.

## Here's a summary of other standard methods that the framework calls on views:

Creation	Constructors	There is a form of the constructor that is called when the view is created from code and a form that is called when the view is inflated from a layout file. The second form parses and applies attributes defined in the layout file.

onFinishInflate()	Called after a view and all of its children are inflated from XML.

Layout	onMeasure(int, int)	Called to determine the size requirements for this view and all of its children.

onLayout(boolean, int, int, int, int)	Called when this view must assign a size and position to all of its children.

onSizeChanged(int, int, int, int)	Called when the size of this view is changed.

Drawing	onDraw(Canvas)	Called when the view must render its content.

Event processing	onKeyDown(int, KeyEvent)	Called when a key down event occurs.
onKeyUp(int, KeyEvent)	Called when a key up event occurs.
onTrackballEvent(MotionEvent)	Called when a trackball motion event occurs.
onTouchEvent(MotionEvent)	Called when a touchscreen motion event occurs.

Focus	onFocusChanged(boolean, int, Rect)	Called when the view gains or loses focus.

onWindowFocusChanged(boolean)	Called when the window containing the view gains or loses focus.

Attaching	onAttachedToWindow()	Called when the view is attached to a window.

onDetachedFromWindow()	Called when the view is detached from its window.

onWindowVisibilityChanged(int)	Called when the visibility of the window containing the view is changed.

## Compound controls 

If you don't want to create a completely customized component but instead are looking to put together a reusable component consisting of a group of existing controls, then creating a compound component (or compound control) might be best. In summary, this brings together a number of more atomic controls or views into a logical group of items that can be treated as a single thing. 
Create XML with the root of some layout and elements you, then create class that extends this layout and provide methods for interacting with this view.

## Modify an existing view type

If there is a component that is similar to what you want, you can extend that component and override the behavior that you want to change. You can do all the things you do with a fully customized component, but by starting with a more specialized class in the `View` hierarchy, you can get some behavior that does what you want for free.

## Subclass a view

All the view classes defined in the Android framework extend `[View](https://developer.android.com/reference/android/view/View)`. Your custom view can also extend `View` directly, or you can save time by extending one of the existing view subclasses, such as `[Button](https://developer.android.com/reference/android/widget/Button)`.

To allow Android Studio to interact with your view, at a minimum you must provide a constructor that takes a `[Context]` and an `[AttributeSet]` object as parameters. This constructor allows the layout editor to create and edit an instance of your view.
```kotlin
class PieChart(context: Context, attrs: AttributeSet) : View(context, attrs)
```
## Custom attributes
To define custom attributes, add `<declare-styleable>` resources to your project. It's customary to put these resources into a `res/values/attrs.xml` file. Here's an example of a `attrs.xml` file:

```xml
<resources>   
	<declare-styleable name="PieChart">       
		<attr name="showText" format="boolean" />       
		<attr name="labelPosition" format="enum">           
			<enum name="left" value="0"/>           
			<enum name="right" value="1"/>       
		</attr>   
	</declare-styleable>  
</resources>
```
How to use the attributes defined for `PieChart`:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"   xmlns:custom="http://schemas.android.com/apk/res-auto"> 
<com.example.customviews.charting.PieChart     
	custom:showText="true"     
	custom:labelPosition="left" />  
</LinearLayout>
```
In order to retrieve attributes pass the `AttributeSet` to `[obtainStyledAttributes()]`. This method passes back a `[TypedArray]`  array of values that are already dereferenced and styled.

The Android resource compiler does a lot of work for you to make calling `obtainStyledAttributes()` easier. For each `<declare-styleable>` resource in the `res/` directory, the generated `R.java` defines both an array of attribute IDs and a set of constants that define the index for each attribute in the array. You use the predefined constants to read the attributes from the `TypedArray`. Here's how the `PieChart` class reads its attributes:



``` kotlin
init {    
	context.theme.obtainStyledAttributes(attrs,R.styleable.PieChart,            0, 0).apply {        
		try { 
			mShowText = getBoolean(R.styleable.PieChart_showText, false)                textPos = getInteger(R.styleable.PieChart_labelPosition, 0)            } finally {
			recycle()
		}    
	}  
}  
```

Note that `TypedArray` objects are a shared resource and must be recycled after use.

## invalidate() and requestLayout(). 
These calls are crucial to ensure that the view behaves reliably. You need to invalidate the view after any change to its properties that might change its appearance, so that the system knows it needs to be redrawn. Likewise, you need to request a new layout if a property changes in a way that might affect the size or shape of the view. Forgetting these method calls can cause hard-to-find bugs.

### invalidate
public void invalidate ()
Invalidate the whole view. If the view is visible, `[onDraw(android.graphics.Canvas)]` will be called at some point in the future.
This must be called from a UI thread. To call from a non-UI thread, call `[postInvalidate()]`

### requestLayout
public void requestLayout ()
Call this when something has changed which has invalidated the layout of this view. This will schedule a layout pass of the view tree.

Метод `View.invalidate()` шедулит перерисовку view. Результат вызова этого метода – асинхронный вызов [onDraw()](https://itsobes.ru/AndroidSobes/kak-realizovat-metod-view-ondraw).  
`invalidate()` используется, когда нужно перерисовать view без изменения размеров, например когда изменяется цвет бэкграунда.  
  
Метод `View.requestLayout()` асинхронно вызывает методы [onMesure()](https://itsobes.ru/AndroidSobes/kak-realizovat-metod-view-onmeasure) и `onLayout()` на текущей view и на всех родителях.  
Этот метод используется, чтобы обновить view после изменения размеров.  
  
На картинке изображен жизненный цикл view и то, как на него влияют `invalidate()` и `requestLayout()`, но есть одна неточность: `requestLayout()` **не гарантирует** вызов `onDraw()`.  
  
Чтобы обновить view, `requestLayout()` следует вызывать вместе с `invalidate()`.

![[Pasted image 20230917125655.png]]
При вызове метода [requestLayout()](https://itsobes.ru/AndroidSobes/v-chem-raznitsa-mezhdu-invalidate-i-requestlayout) на `ViewGroup`, новое измерение дочерних view не гарантируется. Вместо этого используются закэшированные значения.  
  
Метод `View.forceLayout()` инвалидирует закэшированные размеры view, чем провоцирует измерение, при вызове `requestLayout()` на родительском `ViewGroup`.  
  
`forceLayout()` не вызывает `requestLayout()` на view, или ее родителях. Этот метод используется только в связке с `requestLayout()`.

## Create Drawing Objects

The `[android.graphics]` framework divides drawing into two areas:

- _What_ to draw, handled by `[Canvas]`
- _How_ to draw, handled by `[Paint]`.

For instance, `[Canvas]` provides a method to draw a line, while `[Paint]` provides methods to define that line's color. `[Canvas]` has a method to draw a rectangle, while `[Paint]` defines whether to fill that rectangle with a color or leave it empty. Simply put, `[Canvas]` defines shapes that you can draw on the screen, while `[Paint]` defines the color, style, font, and so forth of each shape you draw.
So, before you draw anything, you need to create one or more `[Paint]` objects
Creating objects ahead of time is an important optimization. Views are redrawn very frequently, and many drawing objects require expensive initialization. Creating drawing objects within your `[onDraw()]` method significantly reduces performance and can make your UI appear sluggish.

## Handle size calculations
`[onSizeChanged()]` is called when your view is first assigned a size, and again if the size of your view changes for any reason. Calculate positions, dimensions, and any other values related to your view's size in `[onSizeChanged()]`, instead of recalculating them every time you draw. In the `PieChart` example, `[onSizeChanged()]` is where the `PieChart` view calculates the bounding rectangle of the pie chart and the relative position of the text label and other visual elements.

When your view is assigned a size, the layout manager assumes that the size includes all of the view's padding. You must handle the padding values when you calculate your view's size. Here's a snippet from `PieChart.onSizeChanged()` that shows how to do this:
```kotlin
// Account for padding  
var xpad = (paddingLeft + paddingRight).toFloat()  
val ypad = (paddingTop + paddingBottom).toFloat()  
  
// Account for the label  
if (showText) xpad += textWidth
val ww = w.toFloat() - xpad
val hh = h.toFloat() - ypad
// Figure out how big we can make the pie.  
val diameter = Math.min(ww, hh)
```

```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {    // Try for a width based on our minimum    
val minw: Int = paddingLeft + paddingRight + suggestedMinimumWidth    
val w: Int = View.resolveSizeAndState(minw, widthMeasureSpec, 1)    
// Whatever the width ends up being, ask for a height that would let the pie    
// get as big as it can    
val minh: Int = View.MeasureSpec.getSize(w) - textWidth.toInt() + paddingBottom + paddingTop    
val h: Int = View.resolveSizeAndState(minh, heightMeasureSpec, 0)    setMeasuredDimension(w, h)  
}
```
There are three important things to note in this code:
- The calculations take into account the view's padding. As mentioned earlier, this is the view's responsibility.
- The helper method resolveSizeAndState() is used to create the final width and height values. This helper returns an appropriate View.MeasureSpec value by comparing the view's desired size to the spec passed into onMeasure().
- onMeasure() has no return value. Instead, the method communicates its results by calling setMeasuredDimension(). Calling this method is mandatory. If you omit this call, the View class throws a runtime exception.

## Draw
Once you have your object creation and measuring code defined, you can implement onDraw(). Every view implements onDraw() differently, but there are some common operations that most views share:
- Draw text using drawText(). Specify the typeface by calling setTypeface(), and the text color by calling setColor().
- Draw primitive shapes using drawRect(), drawOval(), and drawArc(). Change whether the shapes are filled, outlined, or both by calling setStyle().
- Draw more complex shapes using the Path class. Define a shape by adding lines and curves to a Path object, then draw the shape using drawPath(). Just as with primitive shapes, paths can be outlined, filled, or both, depending on the setStyle().
- Define gradient fills by creating LinearGradient objects. Call setShader() to use your LinearGradient on filled shapes.
- Draw bitmaps using drawBitmap().
Example:
```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)

    canvas.apply {
        // Draw the shadow
        drawOval(shadowBounds, shadowPaint)

        // Draw the label text
        drawText(data[mCurrentItem].mLabel, textX, textY, textPaint)

        // Draw the pie slices
        data.forEach {
            piePaint.shader = it.mShader
            drawArc(bounds,
                    360 - it.endAngle,
                    it.endAngle - it.startAngle,
                    true, piePaint)
        }

        // Draw the pointer
        drawLine(textX, pointerY, pointerX, pointerY, textPaint)
        drawCircle(pointerX, pointerY, pointerSize, mTextPaint)
    }
}
```
## Speed up your view
To speed up your view, eliminate unnecessary code from routines that are called frequently. Start with onDraw(), which gives you the biggest payback. In particular, eliminate allocations in onDraw(), because allocations might lead to a garbage collection that causes a stutter. Allocate objects during initialization or between animations. Never make an allocation while an animation is running.

In addition to making onDraw() leaner, make sure it's called as infrequently as possible. Most calls to onDraw() are the result of a call to invalidate(), so eliminate unnecessary calls to invalidate().

Another very expensive operation is traversing layouts. When a view calls requestLayout(), the Android UI system traverses the entire view hierarchy to find how big each view needs to be. If it finds conflicting measurements, it might traverse the hierarchy multiple times. UI designers sometimes create deep hierarchies of nested ViewGroup objects. These deep view hierarchies cause performance problems, so make your view hierarchies as shallow as possible.

If you have a complex UI, consider writing a custom ViewGroup to perform its layout. Unlike the built-in views, your custom view can make application-specific assumptions about the size and shape of its children and therefore avoid traversing its children to calculate measurements.

For example, if you have a custom ViwGroup that doesn't adjust its own size to fit all its child views, you avoid the overhead of measuring all the child views. This optimization isn't possible if you use the built-in layouts that cater to a wide range of use-cases.

![[Pasted image 20230917140847.png]]

## Saving state
![[Pasted image 20230917141453.png]]
![[Pasted image 20230917141511.png]]
## Animations

![[Pasted image 20230917141732.png]]

![[Pasted image 20230917141831.png]]
![[Pasted image 20230917142006.png]]
## Gestures and touch events
![[Pasted image 20230917142659.png]]
![[Pasted image 20230917142733.png]]
![[Pasted image 20230917142811.png]]
![[Pasted image 20230917143133.png]]
