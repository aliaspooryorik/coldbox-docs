# Scheduled Tasks

## Requirements

The [async](./) package is what powers scheduled tasks and it can be available to any CFML application by using any of our standalone libraries and frameworks:

* [CacheBox](https://forgebox.io/view/cachebox)
* [ColdBox](https://forgebox.io/view/coldbox)
* [LogBox](https://forgebox.io/view/logbox)
* [WireBox](https://forgebox.io/view/wirebox)

{% hint style="danger" %}
**YOU DON'T NEED COLDBOX TO RUN ANY SCHEDULED TASKS OR ANY FEATURES OF THE ASYNC PACKAGE. YOU CAN USE ANY OF THE STANDALONE LIBRARIES ABOVE.**
{% endhint %}

However, if you use ColdBox, you get enhanced features and new functionality.  For example, the [ColdBox Scheduled Tasks](../scheduled-tasks.md) are an enhanced implementation of the core scheduled tasks we will be reviewing in this document.

## Introduction

The async package offers you the ability to schedule tasks and workloads via the[ Scheduled Executors](executors.md#executor-types) that you can register in the async manager.  We also provide you with a lovely `Scheduler` class that can keep track of all the tasks you would like to be executing in a `ScheduledExecutor`.  In essence, you have two options when scheduling tasks:

1. **Scheduler Approach**: Create a scheduler and register tasks in it
2. **Scheduled Executor Approach**: Create a `ScheduledExecutor` and send task objects into it

![Async Scheduler & Tasks](../../.gitbook/assets/AsyncScheduler.png)

{% hint style="info" %}
With our scheduled tasks you can run either one-off tasks or periodically tasks.
{% endhint %}

## Scheduler Approach

To create a new scheduler you can call the Async Managers' `newScheduler( name )` method.  This will create a new `coldbox.system.async.tasks.Scheduler` object with the specified name you pass. It will also create a `ScheduledExecutor` for you with the default threads count inside the scheduler..  It will be then your responsibility to persist that scheduler so you can use it throughout your application process.

{% embed url="https://s3.amazonaws.com/apidocs.ortussolutions.com/coldbox/6.4.0/coldbox/system/async/tasks/Scheduler.html" %}
Scheduler API Docs
{% endembed %}

{% embed url="https://s3.amazonaws.com/apidocs.ortussolutions.com/coldbox/6.4.0/coldbox/system/async/executors/ScheduledExecutor.html" %}
ScheduledExecutor API Docs
{% endembed %}

```javascript
application.scheduler = asyncmanager.newScheduler( "appScheduler" );
```

Once you get an instance to that scheduler you can begin to register tasks on it.  Once all tasks have been registered you can use the `startup()` method to startup the tasks and the `shutdown()` method to shutdown all tasks and the linked executor.

{% hint style="success" %}
The name of the `ScheduledExecutor` will be `{schedulerName}-scheduler`
{% endhint %}

{% code title="Application.cfc" %}
```javascript
component{
    
    this.name = "My App";
    
    
    function onApplicationStart(){
        new wirebox.system.Injector();
        application.asyncManager = application.wirebox.getInstance( "wirebox.system.async.AsyncManager" );
        application.scheduler = application.asyncmanager.newScheduler( "appScheduler" );

          /**
           * --------------------------------------------------------------------------
           * Register Scheduled Tasks
           * --------------------------------------------------------------------------
           * You register tasks with the task() method and get back a ColdBoxScheduledTask object
           * that you can use to register your tasks configurations.
           */
          	
          application.scheduler.task( "Clear Unregistered Users" )
          	.call( () => application.wirebox.getInstance( "UsersService" ).clearRecentUsers() )
          	.everyDayAt( "09:00" );
          	
          application.scheduler.task( "Hearbeat" )
          	.call( () => runHeartBeat() )
          	.every( 5, "minutes" )
          	.onFailure( ( task, exception ) => {
          			sendBadHeartbeat( exception );
          	} );
          
          // Startup the scheduler
          application.scheduler.startup();
    
    }


    function onApplicationEnd( appScope ){
        // When the app is restart or dies make sure you cleanup
        appScope.scheduler.shutdown();
        appScope.wirebox.shutdown();
    }


}
```
{% endcode %}

### Configuration Methods

The following methods are used to impact the operation of all scheduled tasks managed by the scheduler:

| Method                    | Description                                       |
| ------------------------- | ------------------------------------------------- |
| `setTimezone( timezone )` | Set the timezone to use for all registered tasks  |
| `setExecutor( executor )` | Override the executor generated for the scheduler |

#### Timezone For All Tasks

By default, all tasks run under the system default timezone which usually is UTC.  However, if you would like to change to a different execution timezone, then you can use the `setTimeZone()` method and pass in a valid timezone string:&#x20;

```javascript
setTimezone( "America/Chicago" )
```

{% hint style="success" %}
You can find all valid time zone Id's here: [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html)
{% endhint %}

{% hint style="warning" %}
Remember that some timezones utilize daylight savings time. When daylight saving time changes occur, your scheduled task may run twice or even not run at all. For this reason, we recommend avoiding timezone scheduling when possible.
{% endhint %}

#### Custom Executor

By default the scheduler will register a `scheduled` executor with a default of 20 threads for you with a name of `{schedulerName}-scheduler.`  If you want to add in your own executor as per your configurations, then just call the `setExecutor()` method.

```javascript
setExecutor( 
    getAsyncManager().newScheduledExecutor( "mymymy", 50 ) 
);
```

{% hint style="info" %}
You can find how to work with executors in our [executors](executors.md) section.
{% endhint %}

### Scheduler Properties

Every scheduler has the following properties available to you in the `variables` scope

| Object         | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| `asyncManager` | Async manager reference                                       |
| `executor`     | Scheduled executor                                            |
| `started`      | A boolean flag indicating if the scheduler has started or not |
| `tasks`        | The collection of registered tasks                            |
| `timezone`     | Java based timezone object                                    |
| `util`         | ColdBox utility                                               |

### Scheduler Utility Methods

Every scheduler has several utility methods:

| Method                  | Description                                                                                                                                                                                                                                                                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `getRegisteredTasks()`  | Get an ordered array of all the tasks registered in the scheduler                                                                                                                                                                                                                                                                               |
| `getTaskRecord( name )` | <p>Get the task record structure by name:</p><p><code>{</code> </p><p> <code>name,</code> </p><p> <code>task,</code> </p><p> <code>future,</code> </p><p> <code>scheduledAt,</code> </p><p> <code>registeredAt,</code> </p><p> <code>error,</code> </p><p> <code>errorMessage,</code> </p><p> <code>stacktrace</code> </p><p><code>}</code></p> |
| `getTaskStats()`        | Builds out a struct report for all the registered tasks in this scheduler                                                                                                                                                                                                                                                                       |
| `hasTask( name )`       | Check if a scheduler has a task registered by name                                                                                                                                                                                                                                                                                              |
| `hasStarted()`          | Has the scheduler started already                                                                                                                                                                                                                                                                                                               |
| `removeTask( name )`    | Cancel a task and remove it from the scheduler                                                                                                                                                                                                                                                                                                  |
| `startup()`             | Startup the scheduler. This is called by ColdBox for you. No need to call it.                                                                                                                                                                                                                                                                   |
| `shutdown()`            | Shutdown the scheduler                                                                                                                                                                                                                                                                                                                          |
| `task( name )`          | Register a new task and return back to you the task so you can build it out.                                                                                                                                                                                                                                                                    |

## Scheduling Tasks

Ok, now that we have seen all the capabilities of the scheduler, let's dive deep into scheduling tasks with the `task( name )` method. &#x20;

### Registering Tasks&#x20;

Once you call on this method, the scheduler will create a `ColdBoxScheduledTask` object for you, configure it, wire it, register it and return it to you.&#x20;

```javascript
task( "my-task" )
```

You can find the API Docs for this object here: [https://s3.amazonaws.com/apidocs.ortussolutions.com/coldbox/6.4.0/coldbox/system/async/tasks/ScheduledTask.html](https://s3.amazonaws.com/apidocs.ortussolutions.com/coldbox/6.4.0/coldbox/system/async/tasks/ScheduledTask.html)

### Task Closure/Lambda/Object

You register the callable event via the `call()` method on the task object.  You can register  a closure/lambda or a invokable CFC.  If you register an object, then we will call on the object's `run()` method by default, but you can change it using the `method` argument and call any public/remote method.

```javascript
// Lambda Syntax
task( "my-task" )
    .call( () => runcleanup() )
    .everyHour();
    
// Closure Syntax
task( "my-task" )
    .call( function(){
        // task here
    } )
    .everyHourAt( 45 );
    
// Object with run() method
task( "my-task" )
    .call( wirebox.getInstance( "MyObject" ) )
    .everyDay()
    
// Object with a custom method
task( "my-task" )
    .call( wirebox.getInstance( "MyObject" ), "reapCache" )
    .everydayAt( "13:00" )
```

### Frequencies

There are many many frequency methods in scheduled tasks that will enable the tasks in specific intervals.  Every time you see that an argument receives a `timeUnit` the available options are:

* days
* hours
* minutes
* seconds
* **milliseconds (default)**
* microseconds
* nanoseconds

Ok, let's go over the frequency methods:

| Frequency Method                       | Description                                                                  |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| `every( period, timeunit )`            | Run the task every custom period of execution                                |
| `spacedDelay( spacedDelay, timeunit )` | Run the task every custom period of execution but with NO overlaps           |
| `everyMinute()`                        | Run the task every minute from the time it get's scheduled                   |
| `everyHour()`                          | Run the task every hour from the time it get's scheduled                     |
| `everyHourAt( minutes )`               | Set the period to be hourly at a specific minute mark and 00 seconds         |
| `everyDay()`                           | Run the task every day at midnight                                           |
| `everyDayAt( time )`                   | Run the task daily with a specific time in 24 hour format: HH:mm             |
| `everyWeek()`                          | Run the task every Sunday at midnight                                        |
| `everyWeekOn( day, time )`             | Run the task weekly on the given day of the week and time                    |
| `everyMonth()`                         | Run the task on the first day of every month at midnight                     |
| `everyMonthOn( day, time )`            | Run the task every month on a specific day and time                          |
| `onFirstBusinessDayOfTheMonth( time )` | Run the task on the first Monday of every month                              |
| `onLastBusinessDayOfTheMonth( time )`  | Run the task on the last business day of the month                           |
| `everyYear()`                          | Run the task on the first day of the year at midnight                        |
| `everyYearOn( month, day, time )`      | Set the period to be weekly at a specific time at a specific day of the week |
| `onWeekends( time )`                   | Run the task on Saturday and Sunday                                          |
| `onWeekdays( time )`                   | Run the task only on weekdays at a specific time.                            |
| `onMondays( time )`                    | Only on Mondays                                                              |
| `onTuesdays( time )`                   | Only on Tuesdays                                                             |
| `onWednesdays( time )`                 | Only on Wednesdays                                                           |
| `onThursdays( time )`                  | Only on Thursdays                                                            |
| `onFridays( time )`                    | Only on Fridays                                                              |
| `onSaturdays( time )`                  | Only on Saturdays                                                            |
| `onSundays( time )`                    | Only on Sundays                                                              |

{% hint style="success" %}
All `time` arguments are defaulted to midnight (00:00)
{% endhint %}

### Preventing Overlaps

![Tasks with a fixed frequency vs delayed frequency](../../.gitbook/assets/tasks-with-and-without-overlaps.png)

By default all tasks that have interval rates/periods that will execute on that interval schedule.  However, what happens if a task takes longer to execute than the period? Well, by default the task will execute even if the previous one has not executed.  If you want to prevent this behavior, then you can use the `withNoOverlaps()` method and ColdBox will register the tasks with a _fixed delay_. Meaning the intervals do not start counting until the last task has finished executing.

![Task With Fixed Delay](../../.gitbook/assets/tasks-with-no-overlaps.png)

```javascript
task( "test" )
	.call( () => getInstance( "CacheService" ).reap() )
	.everyMinute()
	.withNoOverlaps();
```

{% hint style="success" %}
Spaced delays are a feature of the Scheduled Executors. There is even a `spacedDelay( delay, timeUnit )` method in the Task object.
{% endhint %}

### Delaying First Execution

Every task can also have an initial delay of first execution by using the `delay()` method.

```javascript
/**
 * Set a delay in the running of the task that will be registered with this schedule
 *
 * @delay The delay that will be used before executing the task
 * @timeUnit The time unit to use, available units are: days, hours, microseconds, milliseconds, minutes, nanoseconds, and seconds. The default is milliseconds
 */
ScheduledTask function delay( numeric delay, timeUnit = "milliseconds" )
```

The `delay` is numeric and the `timeUnit` can be:

* days
* hours
* minutes
* seconds
* **milliseconds (default)**
* microseconds
* nanoseconds

```javascript
// Lambda Syntax
task( "my-task" )
    .call( () => wirebox.getInstance( "MyObject" ).runcleanup() )
    .delay( "5000" )
    .everyHour();
```

{% hint style="info" %}
Please note that the `delay` pushes the execution of the task into the future only for the first execution.
{% endhint %}

### One Off Tasks

A part from registering tasks that have specific intervals/frequencies you can also register tasks that can be executed **ONCE** **ONLY**.   These are great for warming up caches, registering yourself with control planes, setting up initial data collections and so much more.&#x20;

Basically, you don't register a frequency just the callable event.  Usually, you can also combine them with a delay of execution, if you need them to fire off after certain amount of time has passed.

```javascript
task( "build-up-cache" )
    .call( () => wirebox.getInstance( "MyObject" ).buildCache() )
    .delay( 1, "minutes" );
    
task( "notify-admin-server-is-up" )
    .call( () => wirebox.getInstance( "MyObject" ).notifyAppIsUp( getUtil().getServerIp() ) )
    .delay( 30, "seconds" );
    
task( "register-container" )
    .call( () => ... )
    .delay( 30, "seconds" );
```

### Life-Cycle Methods

We already saw that a scheduler has life-cycle methods, but a task can also have several useful life-cycle methods:

| Method                | Description                                                                                                                   |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `after( target )`     | <p>Store the closure to execute after the task executes<br></p><p><code>function( task, results )</code></p>                  |
| `before( target )`    | <p>Store the closure to execute before the task executes<br></p><p><code>function( task )</code></p>                          |
| `onFailure( target )` | <p>Store the closure to execute if there is a failure running the task<br></p><p><code>function( task, exception )</code></p> |
| `onSuccess( target )` | <p>Store the closure to execute if the task completes successfully<br></p><p><code>function( task, results )</code></p>       |

```javascript
task( "testharness-Heartbeat" )
	.call( function() {
			if ( randRange(1, 5) eq 1 ){
				 throw( message = "I am throwing up randomly!", type="RandomThrowup" );
			}
			  writeDump( var='====> I am in a test harness test schedule!', output="console" );
		} )
		.every( "5", "seconds" )
		.before( function( task ) {
			  writeDump( var='====> Running before the task!', output="console" );
		} )
		.after( function( task, results ){
			  writeDump( var='====> Running after the task!', output="console" );
		} )
		.onFailure( function( task, exception ){
			  writeDump( var='====> test schedule just failed!! #exception.message#', output="console" );
		} )
		.onSuccess( function( task, results ){
			  writeDump( var="====> Test scheduler success : Stats: #task.getStats().toString()#", output="console" );
		} );
```

### Timezone

By default, all tasks will ask the scheduler for the timezone to run in.  However, you can override it on a task-by-task basis using the `setTimezone( timezone )` method:

```javascript
setTimezone( "America/Chicago" )
```

{% hint style="success" %}
You can find all valid time zone Id's here: [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/ZoneId.html)
{% endhint %}

{% hint style="warning" %}
Remember that some timezones utilize daylight savings time. When daylight saving time changes occur, your scheduled task may run twice or even not run at all. For this reason, we recommend avoiding timezone scheduling when possible.
{% endhint %}

### Truth Test Constraints

There are many ways to constrain the execution of a task. However, you can register a `when()` closure that will be executed at runtime and boolean evaluated.  If `true`, then the task can run, else it is disabled.

```javascript
task( "my-task" )
    .call( () => wirebox.getInstance( "MyObject" ).cleanOldUsers() )
    .daily()
    .when( function(){
        // Can we run this task?
        return true;
    );
```

### Disabling/Pausing Tasks

Every task is runnable from registration according to the frequency you set.  However, you can manually disable a task using the `disable()` method:

```javascript
task( "my-task" )
    .call( () => getInstance( "securityService" ).cleanOldUsers() )
    .daily()
    .disable();
```

Once you are ready to enable the task, you can use the `enable()` method:

```javascript
myTask.enable()
```

### Task Stats

All tasks keep track of themselves and have lovely metrics.  You can use the `getStats()` method to get a a snapshot `structure` of the stats in time.  Here is what you get in the stats structure:

| Metric          | Description                                               |
| --------------- | --------------------------------------------------------- |
| `created`       | The timestamp of when the task was created in memory      |
| `inetHost`      | The hostname of the machine this task is registered with  |
| `lastRun`       | The last time the task ran                                |
| `lastResult`    | The last result the task callable produced                |
| `localIp`       | The ip address of the server this task is registered with |
| `neverRun`      | A boolean flag indicating if the task has NEVER been ran  |
| `nextRun`       | When the task will run next                               |
| `totalFailures` | How many times the task has failed execution              |
| `totalRuns`     | How many times the task has run                           |
| `totalSuccess`  | How many times the task has run and succeeded             |

```javascript
/**
 * Called after ANY task runs
 *
 * @task The task that got executed
 * @result The result (if any) that the task produced
 */
function afterAnyTask( required task, result ){
	log.info( "task #task.getName()# just ran. Metrics: #task.getStats().toString()# ");
}

```

### Task Helpers

We have created some useful methods that you can use when working with asynchronous tasks:

| Method            | Description                                                                                                                                              |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `err( var )`      | Send output to the error stream                                                                                                                          |
| `hasScheduler()`  | Verifies if the task is assigned a scheduler or not                                                                                                      |
| `isDisabled()`    | Verifies if the task has been disabled by bit                                                                                                            |
| `isConstrained()` | Verifies if the task has been constrained to run by weekends, weekdays, dayOfWeek, or dayOfMonth                                                         |
| `out( var )`      | Send output to the output stream                                                                                                                         |
| `start()`         | This kicks off the task into the scheduled executor manually. This method is called for you by the scheduler upon application startup or module loading. |

## Scheduled Executor Approach

Let's investigate now a second approach to task scheduling.  We have seen the `Scheduler` approach which is a self-contained object that can track multiple tasks for you and give you enhanced and fluent approaches to scheduling.  However, there are times, where you just want to use a `ScheduledExecutor` to send tasks into for either one-time executions, or also on specific frequencies and skip the `Scheduler.`

Like with anything in life, there are pros and cons.  The Scheduler approach will track all the scheduled future results of each task so you can see their progress, metrics and even cancel them.  With this approach, it is more of a set off and forget approach.

### Register a Scheduled Executor

Let's get down to business. The first step is to talk to the AsyncManager and register a scheduled executor. You can do this using two methods:

* `newExecutor( name, type, threads )` - Pass by type
* `newScheduledExecutor( name, threads )` - Shorthand

```javascript
// Create it with a default of 20 threads
application.asyncManager.newExecutor( "myTasks", "scheduled" );
// Create it with a default of 10 threads
application.asyncManager.newExecutor( "myTasks", "scheduled", 10 );
// Create it as a FIFO queue of 1 thread
application.asyncManager.newExecutor( "myTasks", "scheduled", 1 );


// Create it with a default of 20 threads
application.asyncManager.newScheduledExecutor( "myTasks" );
// Create it with a default of 10 threads
application.asyncManager.newScheduledExecutor( "myTasks", 10 );
// Create it as a FIFO queue of 1 threa,d
application.asyncManager.newScheduledExecutor( "myTasks", 1 );
```

Once you register the executor the Async Manager will track it's persistence and then you can request it's usage anywhere in your app via the `getExecutor( name )` method or inject it using the `executors` injection DSL.

```javascript
// Request it
var executor = application.asyncManager.getExecutor( "myTasks" );

// Inject it
property name="myTasks" inject="executor:myTasks";
```

### Scheduling

Now that we have a scheduler, we can use the `newTask()` method to get a `ScheduledTask` , configure it, and send it for execution.

```javascript
// Request it
var executor = application.asyncManager.getExecutor( "myTasks" );

var future = executor.newTask( "cache-reap" )
    .call( () => application.cacheFactory.reapAll() )
    .every( 5, "minutes" )
    .start();
```

As you can see, now we are in [Scheduling Tasks](scheduled-tasks.md#scheduling-tasks) mode, and all the docs on it apply.  Several things are different in this approach:

1. We talk to the executor via the `newTask()` method to get a new `ScheduledTask` object
2. We call the `start()` method manually, whenever we want to send the task into scheduling
3. We get a `ScheduledFuture` result object so we can track the results of the schedule.

{% embed url="https://s3.amazonaws.com/apidocs.ortussolutions.com/coldbox/6.4.0/coldbox/system/async/tasks/ScheduledFuture.html" %}
ScheduledFuture API Docs
{% endembed %}

### Work Queues

You can very easily create working queues in this approach by being able to send one-off tasks into the executors and forget about them.  Let's say we have an app that needs to do some image processing afte ran image has been uploaded.  We don't want to hold up (block) the calling thread with it, we upload, send the task for processing and return back their identifier for the operation.&#x20;

```javascript
... Upload File.

// Process Image in the Executor Work Queue
executor.newSchedule( task : function(){
        application.wirebox.getInstance( "ImageProcessor" ).processImage( fileName );
    }).start();
    
... Continue operation
```

{% hint style="info" %}
Remember you can set how many threads you want in a executor.  It doesn't even have to be a scheduled executor, but could be a cached one which can expand and contract according to work loads.
{% endhint %}
