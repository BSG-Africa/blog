**Cyclic Reference detector**

This blog covers a contribution I did for an open source library [shazamcrest](https://github.com/shazam/shazamcrest). Shazamcrest is a useful tool used for assertion of unit tests. Assertions on complete beans are made simpler by serialising the actual and expected beans to json, and then compare the two resulting jsons. The difference between the two json are shown using the comparison functionalities of IDEs like Eclipse or IntelliJ. The library works very well until you try to use it on objects that has cyclic reference on any level, which resulsts in StackOverflowException.
<!--more-->

[This issue was resolved by another developer](http://www.briandupreez.net/2014/07/tdd-hamcrest-shazamcrest.html), but to get around this, one has to know all the classes that would cause the StackOverflowException. This was an issue because some objects could have many levels, and finding all those objects with cyclic reference would be cumbersome. [My contribution](https://github.com/shazam/shazamcrest/blob/master/src/main/java/com/shazam/shazamcrest/CyclicReferenceDetector.java) was to detect all objects that would cause the StackOverflowException on the fly so that a developer does not have to care about the structure of the DTO.

Below is the description of the algorithm/pseudo code I used to find all objects that will cause StackOverflowException.

```
function find_objects_with_cyclic_reference (object)
	if object already in a list of objects in path
		add object to a list of objects with circular reference
		return;
	else
		add object to a list of objects in path
		for every field value in object
		find_objects_with_cyclic_reference(field value)
```

The graph below shows how the algorithm works. A circle represents an object, a circle filled in black colour shows the object being processed, circle filled in red represent an object than can cause StackOverflowException.

<center>
  <img title="Graphical representation of how an object with cyclic reference is detected." src="{{ site.baseurl }}/images/wizard.jpg"/>
</center>

From the graph above, one can see that this is a [`Depth-first search`](https://en.wikipedia.org/wiki/Depth-first_search) algorithm, and we are searching for an object that we have already traversed.

This algorithm works for almost all objects and the pull request has now been merged into version `0.10` of Shazamcrest.
