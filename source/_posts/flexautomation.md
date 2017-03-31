---
title: FlexAutomation与FlexMonkey流程
date: 2014-1-8 11:02:07
tags: [FlexAutomation,FlexMonkey]
categories: Flex
---
## 一、Mixin元数据标签
Mixin元数据标签：主要作用是为应用程序初始化工作创建一个静态代码块，该代码块会在Flex程序运行的最初阶段调用，即SystemManager初始化完毕，而Application尚未初始化时调用。执行[Mixin]元标签的类的
```javascript
public static function init(fbs:IFlexModuleFactory):void
```

## 二、工作流程
1. SystemManager初始化完毕后，都会调用所Mixin的Init方法。
在所有代理委托类中的都会有一段Mixin的Init方法，用于注册控件对应的代理类
```javascript
[Mixin]
/**
 1.
 2.  Defines methods and properties required to perform instrumentation for the
 3.  Button control.
 4.
 5.  @see mx.controls.Button
 6.  */
public class ButtonAutomationImpl extends UIComponentAutomationImpl
{
    include "../../../core/Version.as";

    //--------------------------------------------------------------------------
    //
    //  Class methods
    //
    //--------------------------------------------------------------------------
    /**
     *  Registers the delegate class for a component class with automation manager.
     *
     *  @param root The SystemManger of the application.
     */
    public static function init(root:DisplayObject):void
    {
        Automation.registerDelegateClass(Button, ButtonAutomationImpl);
    }
```
2. SystemManager在加载每一个UIComponent的时候，都会触发Event.ADDED事件。
3. AutomationManager也是元数据[Mixin]标识的。
AutomationManager的init方法中监听了Event.ADDED事件。
root.addEventListener(Event.ADDED,childAddedHandler, false, 0, true);
在childAddedHandler处理函数中，为每一个可视化控件都创建了一个委托实例，并把控件和委托类进行了关联（根据第一步的结果）。
Automation.delegateClassMap[componentClass] = Automation.delegateClassMap[className];
AutomationManager中包含重要的recordAutomatableEvent()方法，方法内部主要是创建并触发了AutomationRecordEvent.RECORD事件。当进行录制操作的时候会触发这个事件
4. 以Button的Click事件举例：
- 用户单击了Button钮，SystemManager会触发一个MouseEvent.CLICK事件
-  ButtonAutomationImpl是Button控件的自动化委托类，委托类的构造函数中监听了Button钮的CLICK事件。
obj.addEventListener(MouseEvent.CLICK, clickHandler, false, EventPriority.DEFAULT+1, true);
```javascript
/**
 *  Constructor.
 * @param obj Button object to be automated.
 */
public function ButtonAutomationImpl(obj:Button)
{
    super(obj);
    obj.addEventListener(KeyboardEvent.KEY_UP, btnKeyUpHandler, false, EventPriority.DEFAULT+1, true);
    obj.addEventListener(MouseEvent.CLICK, clickHandler, false, EventPriority.DEFAULT+1, true);
}
/**
 *  @private
 *  storage for the owner component
 */
protected function get btn():Button
{
    return uiComponent as Button;
}
/**
 *  @private
 */
private var ignoreReplayableClick:Boolean;
//----------------------------------
//  automationName
//----------------------------------
/**
 *  @private
 */
override public function get automationName():String
{
    return btn.label || btn.toolTip || super.automationName;
}
//----------------------------------
//  automationValue
//----------------------------------
/**
 *  @private
 */
override public function get automationValue():Array
{
    return [ btn.label || btn.toolTip ];
}
/**
 *  @private
 */
protected function clickHandler(event:MouseEvent):void
{
    if (!ignoreReplayableClick)
        recordAutomatableEvent(event);
    ignoreReplayableClick = false;
}
/**
 *  @private
 */
private function btnKeyUpHandler(event:KeyboardEvent):void
{
    if (!btn.enabled)
        return;
    if (event.keyCode == Keyboard.SPACE)
    {
        // we need to ignore recording a click being dispatched here
        ignoreReplayableClick = true;
        recordAutomatableEvent(event);
    }
}
```
CLICK事件的处理函数clickHandler内部，调用了AutomationManager的recordAutomatableEvent方法，该方法又触发了AutomationRecordEvent.RECORD事件。

5. MonkeyLink的init方法中对FlexMonkey和Flex Application通过LocalConnection进行了关联，并且监听了AutomationRecordEvent.RECORD事件。
MonkeyAutomationManager.instance.addEventListener(AutomationRecordEvent.RECORD, recordEventHandler, false, 0, true);

6. MonkeyLink捕获到了AutomationRecordEvent.RECORD事件，记录下了事件信息，并创建了UIEventMonkeyCommand命令。
```javascript
     public function recordEventHandler(event:AutomationRecordEvent):void {
         var id:String;
         var idProp:String;
         var obj:IAutomationObject = event.automationObject;
         var container:UIComponent = null;
    idProp = "automationID";
id=MonkeyAutomationManager.instance.createID(obj).toString();
         var uiEventCommand:UIEventMonkeyCommand = new UIEventMonkeyCommand();
         uiEventCommand.value = id;
         uiEventCommand.prop = idProp;
         uiEventCommand.command = event.name;
         uiEventCommand.args = event.args;
         if (MonkeyAutomationState.monkeyAutomationState.state != MonkeyAutomationState.SNAPSHOT) {
             FMHub.instance.dispatchEvent(new CommandRecordedEvent(uiEventCommand));
```
7. FlexMonkium The FlexMonkey/Selenium Bridge
中处理UIEventMonkeyCommand命令，生成XML之类似的
```javascript
   [Mixin]
    public class FlexMonkium {
        // I'm totally static. No need to instance me.
        public function FlexMonkium() {
        }
        public static function init(root:DisplayObject):void {
            var link:MonkeyLink; // Dummy reference to cause class to load
            root.addEventListener(FlexEvent.APPLICATION_COMPLETE, function():void {
                // If we're in MonkeyLink, the external interface will be available. In AirMonkey, it won't be.
                if (ExternalInterface.available) {
                    ExternalInterface.addCallback("playFromSelenium", FlexMonkium.playFromSelenium);
                    ExternalInterface.addCallback("verifyFromSelenium", FlexMonkium.verifyFromSelenium);
                    ExternalInterface.addCallback("getForSelenium", FlexMonkium.getForSelenium);
                    ExternalInterface.addCallback("getCellForSelenium", FlexMonkium.getCellForSelenium);
                    AQAdapter.aqAdapter.beginRecording();
                    FMHub.instance.listen(CommandRecordedEvent.COMMAND_RECORDED, function(ev:CommandRecordedEvent):void {
                        sendToSelenium(ev.command);
                    }, this);
//                    FMHub.instance.listen(VerifyCreatedEvent.VERIFY_CREATED, function(ev:VerifyCreatedEvent):void {
//                        sendVerifyToSelenium(ev.verify);
//                    }, this);
                    if (ExternalInterface.available)
                    {
                        var js:String = 'function() {window["_Selenium_IDE_Recorder"].record("waitForFlexMonkey")}';
                        ExternalInterface.call(js);
                    } else {
                        trace(new Error("FlexMonkium: Unexpectedly found ExternalInterface to be unavailable").getStackTrace());
                    }
                }
            }, false, -1); // Set to -1 priority so that aqadapter can initialize first
        }
        static public function sendToSelenium(event:UIEventMonkeyCommand):void {
            if (ExternalInterface.available) {
                //trace("FlexMonkium: Send to selenium: " + event.xml.toXMLString());
                var js:String = 'function(xml) {window["_Selenium_IDE_Recorder"].record("flexMonkey",xml)}';
                ExternalInterface.call(js, XmlConversionFactory.encode(event, true).toXMLString());
            } else {
                trace(new Error("FlexMonkium: Unexpectedly found ExternalInterface to be unavailable").getStackTrace());
            }
        }
```

## 三、回放操作
```
/**
 *  @private
 *  Replays click interactions on the button.
 *  If the interaction was from the mouse,
 *  dispatches MOUSE_DOWN, MOUSE_UP, and CLICK.
 *  If interaction was from the keyboard,
 *  dispatches KEY_DOWN, KEY_UP.
 *  Button's KEY_UP handler then dispatches CLICK.
 *
 *  @param event ReplayableClickEvent to replay.
 */
override public function replayAutomatableEvent(event:Event):Boolean
{
    var help:IAutomationObjectHelper = Automation.automationObjectHelper;

    if (event is MouseEvent && event.type == MouseEvent.CLICK)
        return help.replayClick(uiComponent, MouseEvent(event));
    else if (event is KeyboardEvent)
        return help.replayKeyboardEvent(uiComponent, KeyboardEvent(event));
    else
        return super.replayAutomatableEvent(event);
}
```

