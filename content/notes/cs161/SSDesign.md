# Secure Systems: Trusted Computing Base

There exist important patterns and rules for building secure systems. How can we choose an architecture that reduces likelihood of flaws, and thus attackers breaking in? Additionally, how can we *mitigate* damage if an attacker *can* find a way in? 

### Trusted Computing Base

A **trusted component** of our system is a component we rely on to operate correctly in order to preserve full security. If it fucks up or misbehaves in any way, security is compromised. On the other hand, a **trustworthy component** of a system is one we *expect* to operate correctly. For example, in Unix systems root is the trusted component, and the root users are trustworthy components. 

The **trusted computing base** (TCB) of a system is one that **must** operate to expectation, every component of it, for security. Anything outside the TCB isn't relied on: those can misbehave as much as they want without affecting security. For example, if we want to ensure security of login via SSH, our TCB includes:

- SSH daemon (background process): this handles authentication+authorization for users trying to login. 
- Operating system: Deals directly with operation of all processes (and thus SSH daemon).
- CPU: Executes the SSH daemon instructions.

Ideally, the system is built such that unprivileged applications, like a web browser or some other program, cannot affect the system security if malicious or malfunctioning.

### TCB Design Principles

Ideally, a TCB should be *unpassable*, in that you cannot by pass the TCB. For example, if our TCB is a firewall protecting a network, no alternate tricky routes should be found in that don't go through the firewall. 

A TCB should also be *tamper resistant*. Other parts of the system outside the TCB should not be able to modify or affect the TCB code or state. 

Finally, the TCB should be *verifiable of correctness*. Thus the **TCB should be as simple as possible** to ensure that this can be done. Thus we want to move as much system code as possible *outside the TCB*.

### TCB Design Benefits

TCBs allow us to isolate the security-critical part of the system from the part(s) that are not. This is important because security is, in general, difficult. The more code and pieces that a system contains, the more potential attack points. Thus if we only need to focus on the TCB to ensure security. 

For example, let's say we were working on a database, and we we're focusing on the process of saving records. Once a record is saved, we also want to ensure it cannot be (permanently) deleted or corrupted by an adversary. Thus we need to build a database that only allows append operations- NOT removal or editing. How might we design this system?

Let's try one way: we first have to note every individual contributing to that database. Let's say for each of these individuals, these records were stored on some special directory on their personal computers, then flushed to the database. The TCB would consist of every operating system of these individuals, software contributing to storing and flushing the records, and the individuals with direct (root) access to the database. That's a huge TCB. If a database admin took a bribe it's over. 

Maybe another way then. Maybe change the database to **physical** records on paper, which we keep in a locked room. Records will be considered added when a special printer prints it out and stores it (into an adjacent box or something). The TCB in this case is the security of the room and the printer. Unfortunately, nobody does this anymore except if you're Amish or something. Imagine looking through 40 million sheets of paper for a record, even if it is indexed somehow. We want to keep things electronic, but still equally as secure.

So we'll instead set up a special computer that has the sole job of flushing records to this electronic database. The computer accepts connections from anyone, and when the record is sent, the computer flushes it. We also ensure our database only implements write-only-once and read capabilities. We might also configure our network such that other computers besides the special flusher cannot connect to that database in any way. Now, the TCB consists of the flusher computer, the administrators of this computer, the firewall stopping connections to the database, the database room security, etc. This isn't as "TCB secure" as the paper printing method, but it's also realistic and certainly not unmanageable. 

Overall, maximize **simplicity and security** in a system TCB to realistic expectations.

 