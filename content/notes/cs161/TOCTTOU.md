# Secure Systems Design: TOCTTOU Attack

### Overview

A *Time-Of-Check To Time-Of-Use* attack is a type of *concurrency attack* that involves a resource and exploiting a conditional check. I didn't even attempt to give a comprehensive definition: there's simply no way to explain what this is besides a real world example. 

So let's say John wants to cop some retro Jordans from Foot Locker. But he's broke as hell so he has to go to the ATM first.  But when he returns, someone (me) already bought the shoes! And they're now sold out. Too bad. 

TOCTTOU involves a *race condition* where the state of the system (Jordans in stock at Foot Locker) changes between the moment when some condition was checked (John checking Foot Locker for the Jordans) by a process and the moment he tried to take action. John *assumed* that the Jordans would remain there between the time he went to get the sufficient money and the time he would buy the J's. His mistake was that he didn't *ensure* the Js would still be his by reserving the shoes for purchase. His assumption was broken by a *concurrent process*, me swooping in and buying the shoes. Notice the attack also involves some *shared resource*, in this case the J's. 

Similarly in system security, TOCTTOU also is prevalent, particularly in enforcing things like access control. Consider this code below:

<code>

def openFile(file):

​	if (isRegularFile(file) == false):

​		return

​	foo(file)

​	return open(file)

</code>

Looks innocent enough, right? The code is just trying to open a file iff it's a regular file. However, notice the code's making assumption that between the time of check <code>if (isRegularFile(file) == false) </code> and the time of use <code> return openfile(file) </code> that the file doesn't change. So if we have some *concurrent process* that changes the file to something malicious (and certainly not regular) *after* the regular file check but *before* the opening of the file- i.e. the <code>foo</code> function is corrupted to edit the file maliciously- then we bypass the check and have performed TOCTTOU. 

This happens quite frequently in UNIX with filesystem calls, because sequences of system calls do not execute *atomically*, or all together in order with no breaks. In general, TOCTTOU happens with mutable resources shared between two entities.