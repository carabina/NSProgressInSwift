# NSProgress in Swift

The NSProgress class is a new addition to Foundation in Mac OS X 10.9 and iOS 8. It provides a standard approach to keeping track of long running tasks within your application. The documentation though is a little light on at this point. This project is my attempt at a sample application to help demonstrate its usage.

Here is a good blog post from Ole Begemann on the subject http://oleb.net/blog/2014/03/nsprogress/

### Steps to build the app
- Built the interface in IB. Set the NSProgressIndicator to determinent mode by unchecking the checkbox in the Attributes panel. Set the maximum value to 1. We are going to update the progressIndicator by observing a fractionCompleted property of an NSProgress instance. As such we want our value to go from 0 to 1.0
- Wire up the indicators and buttons to the AppDelegate via outlets and actions.
- Create an NSProgress instance within the start action. We will call this instance the parentProgress as it will be used to capture the entire progress of the tasks we set in motion. We get to set a totalUnitCount here. Later, when you are scheduling tasks you will be able to give each task an estimated amount of time that it will take. This estimate will be a fraction of the totalUnitCount. I have chosen a value of 10 here for no particular reason.
- We add an observer to the fractionCompleted keypath of the parentProgress. This allows us to update the progressIndicator UI as the parentProgress is updated. Note that the observations will be called on the background thread so the actual UI update needs to be scheduled on the main thread
- Back in the startTask action method we use GCD to start our long running tasks off in a background thread. Before each task is started we call becomeCurrentWithPendingUnitCount() on the parentProgress instance. Note that this needs to happen on the same thread that the NSProgress instances for the tasks are created. Also note that the sum of pendingUnitCount figures for all of your tasks needs to add up to the totalUnitCount that you set for the parentProgress instance. Here I have allocated 4 units for the first task, 1 unit for the sleep inbetween the tasks and 5 units for task 2 (4 + 1 + 5 = 10 = parentProgress.totalUnitCount). These are my estimates for how long each of the tasks will take.
- Now it is time to write each of the tasks. In this example each task is simply a method of the AppDelegate. However the tasks can be anywhere in your code base or an NSProgress supported section of Cocoa. 
- At the start of both tasks we create new NSProgress instances. I have called them child1Progress and child2Progress. Because they are created on the same thread on which the parentProgress became current they are considered children of the parentProgressInstance. 
- Each task is simply a couple of loops of sleep calls. This allows us to break up the workload into a number of steps so that we can report the progress back to the UI. If you have a task that requires a lot of steps it might be a good idea to only report progress periodically rather than after every step. This is because the progress communication system requires a non-trivial amount of work to be performed. As a rule of thumb I find 100 steps to be very smooth and not overly taxing. Profile your app if you want to come up with a more informed figure.
- Within your loop you should be checking on the state of the childProgress.cancelled and childProgress.paused properties.
- As you work through your task increment the childProgress.completedUnitCount in order to report that progress back up to the parentProgress and UI. 


###TODO
- Ole Bergmann also links to this advice from Brent Simmons which looks like the right way to handle this sort of code to me. I haven't done it this way to date as I was struggling to figure out how to get NSProgress to work and what is included is the first success I managed. Should revisit and rewrite it following Brent's advice. http://inessential.com/2014/03/08/api_design_the_main_thread_and_queues
- How do you resume a paused NSProgress instance?
- The buttons should be enabled/disabled as appropriate.
- Cancelling the tasks should set the progressIndicator back to 0.

Idea: I should do a sample project using NSProgress written in Swift. This can go up on Github. Key points to demonstrate are
- The becomeCurrent calls need to occur on the same thread. 
- The totals and workUnitCounts of the different NSProgress items is difficult to grasp
- Show how to have multiple long running tasks combined into one progress indicator without them knowing about each other
- Implement cancel and pause

