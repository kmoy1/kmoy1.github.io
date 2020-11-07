# Secure Systems Design: Leveraging Modularity

### Modularity

Modularity, and abstraction, are some of the most beautiful concepts not just in computer science, but in life. A well-designed system is ideally split into modules, where modules interact with each other only through well-defined interfaces. Each module should have a **clear purpose or function**: developers should know what each module does in concept. In a factory, each worker is given a clear purpose or role. Similarly in code (which you can sorta think as a factory), whenever we make helper functions, we utilize modularity in that those helper functions do a specific task and that task only.

Not only does this help developers understand (and thus debug) their code conceptually, but modularity can also strengthen system security through isolation (remember TCB!!). We know where potential problems will show up in debugging, and we minimize the assumptions and interdependency made between components.

Consider partitioning a web server as a composition of two modules: One module for handling incoming network connections and reading URL requests, and another for handling translating this URL into a filename and reading it from the filesystem (and retrieving the IP address). The first module can be run with no privileges at all, and the second module would need to be run with enough privileges such that it can only find IP addresses of www.(*). Even if the second module is compromised, the attacker cannot affect the rest of the system.

These practices are known as *separation of privilege*, because our modules can have various levels of privilege needed to function. 