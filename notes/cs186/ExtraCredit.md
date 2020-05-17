# Database Implementation: XJoin

Authors: Kevin Moy and Nick Kisel 

XJoin is an advanced and pipelineable join algorithm, and was one of the subjects of our technical report. This document details the implementation of the algorithm into MOOCBase. 

```
Q = {}
function PMJ(R,S,B,F):
	while (R not empty and S not empty):
		create sorted run R', S' from R, S
		RS = simplejoin(R',S')
		flush(RS)
		Q.add(RS)
	while (|Q| > 1):
		Load Q' into memory, 
```



## Testing

We implemented testing of the PMJ and XJoin algorithms in our joins testing file.