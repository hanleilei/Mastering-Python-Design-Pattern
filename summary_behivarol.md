# 行为型模式

行为型模式关注做事的过程，也就是算法及对象间的交互。提供了构思及规划计算过程的好办法。

Pattern	                Description
chain	                apply a chain of successive handlers to try and process the data
catalog	                general methods will call different specialized methods based on construction parameter
chaining_method	        continue callback next object method
command	                bundle a command and arguments to call later
iterator	            traverse a container and access the container's elements
mediator	            an object that knows how to connect other objects and act as a proxy
memento	                generate an opaque token that can be used to go back to a previous state
observer	            provide a callback for notification of events/changes to data
publish_subscribe	    a source syndicates events/data to 0+ registered listeners
registry	            keep track of all subclasses of a given class
specification	        business rules can be recombined by chaining the business rules together using boolean logic
state	                logic is organized into a discrete number of potential states and the next state that can be transitioned to
strategy	            selectable operations over the same data
template	            an object imposes a structure but takes pluggable components
visitor	                invoke a callback for all items of a collection

已经介绍过了责任链（chain），命令（command），观察者（observer），状态（state），策略（strategy），模版（template），访问者（visitor），似乎很多没有介绍过，但是对于Python语言，其中的迭代器（iterator）模式似乎很是多余，故在此忽略。

下面将扼要介绍以下catalog，mediator，memnto，publish_subscribe，registry和specification这些模式。

对于catalog模式：


对于 mediator（中介） 模式：


对于 memnto（备忘录） 模式：



对于 registry 模式：


对于 specification 模式：



1
