# Less painful Android development with Scala

Scala is cool. Writing Android application with Scala is also a cool idea. Because Android exposes Java API, we need some wrapper and utility library to leverage full power of Scala. android-scala-common is an initial attempt to provide this.

## Implicit conversions
This library employs several implicit conversions. 

```
helpButton.setOnClickListener {
  new OnClickListener {
	def onClick(dialog: DialogInterface, which: Int) {
	  Log.i("pocorall", "pressed!")
	}
  }
}
```

is reduced to:

    helpButton.setOnClickListener(Log.i("pocorall", "pressed!"))

For a multi-line block:

```
helpButton.setOnClickListener {
  (v: View) =>
    Log.i("pocorall", "pressed!")
    Log.i("pocorall", v.toString())
}
```

## Context as an implicit parameter
Many methods in Android API requires an instance of a class `Context`. Providing this for every method calls results a clumsy code. We employs implicit parameter to elliminate this. Just declare an implicit value that represents current context:

    implicit val context = ...

or just extend trait `ContextUtil`, which defines it for you. Then the codes that required `Context`, for example:

```
new Intent(context, classOf[MyActivity])
startService(new Intent(context, classOf[MyService]))
Toast.makeText(context, "hi, there!", Toast.LENGTH_SHORT).show()
ProgressDialog.show(context, "Dialog", "working...", true)
PendingIntent.getActivity(context, 0, new Intent(context, classOf[MyActivity]), 0)
PendingIntent.getService(context, 0, new Intent(context, classOf[MyService]), 0)
```

is reduced to:

```
newIntent[MyActivity]
startService[MyService]
toast("hi, there!")
spinnerDialog("Dialog", "working...")
pendingActivity[MyActivity]
pendingService[MyService]
```

   
## Traits

### Trait ContextUtil

Trait `ContextUtil` includes several shortcuts for frequently used android idioms.

**System services**

Getting system service objects become much simpler.

```
val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE).asInstanceOf[NotificationManager]
notificationManager.notify(R.string.someString, notification)
val vibrator = getSystemService(Context.VIBRATOR_SERVICE).asInstanceOf[Vibrator]
vibrator.vibrate(500)
```

is reduced to:

```
notificationManager.notify(R.string.someString, notification)
vibrator.vibrate(500)
```

All of the system service accessors described in Android API level 10 are defined.

**Play ringtones**

Just play default notification ringtone:

    play()
	
, specify ringtone resources:

    play("content://media/internal/audio/media/50")
	
, or specify default resource Uris:
	
	play(alarmSound)

	
### Trait RunOnUiThread

Android API provides `runOnUiThread()` only for class `Activity`. Trait `RunOnUiThread` provides Scala version of `runOnUiThread()` for anywhere other than `Activity`.

Instead of:

```
activity.runOnUiThread {
  new Runnable() {
	def run() {
	  Log.i("I am running", "only for Activity class")
	}
  }
}
```

, extend trait `RunOnUiThread` and use it like this:

    runOnUiThread(Log.i("I am running", "for any context"))

	
### Trait UnregisterReceiverService

When you registered `BroadcastReceiver` with `Context.registerReceiver()` you have to unregister it to prevent memory leak. Trait UnregisterReceiverService handles these chores for you by just extend it for your Service.

```
class MyService extends Service with UnregisterReceiverService {
  def func() {
    // ...
	registerReceiver(receiver, intentFilter)
	// Done! automatically unregistered at UnregisterReceiverService.onDestroy()
  }
}
```

## Implicit conversions for resource Ids

Android API provides two versions of methods for string resources; One is for CharSequence, another is for Int as a parameter ID. If you write a middleware classes for Android, you also have to expose methods for every combinations of two versions of resources:

```
def alert(titleId:Int, textId:Int)(implicit context:Context) = {
  alert(context.getText(titleId), context.getText(textId))
}

def alert(titleId:Int, text:CharSequence)(implicit context:Context) = {
  alert(context.getText(titleId), text)
}

def alert(titleId:CharSequence, textId:Int)(implicit context:Context) = {
  alert(title, context.getText(textId))
}

def alert(title:CharSequence, text:CharSequence) = ...
```

This is not a smart way. Use type `TextResource` instead. 

    def alert(title:TextResource, text:TextResource) = ...
	
Then implicit conversions take care about these resource type conversions.


## Import it to your project

For now, android-scala-common is a single-file project. Just copy `common.scala` and paste it to your project and `import net.pocorall.android.common._`. Enjoy!

## Let's make it together!

This project is very early stage, and I will grow it constantly. If you have any idea to improve this library, feel free to open issues or post patches.

### License

This software is licensed under the Apache 2 license.

### Recommended resources

* To set up a Maven project that build Android app written in Scala, please refer https://github.com/rohansingh/android-scala-test.

* A delicate problem related to implicit conversion for mult-line block is discussed at  http://stackoverflow.com/questions/12774840/scala-passing-function-parameter-which-does-not-have-parameter.
