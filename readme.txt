
This is a blog post on instance variables that I have been working on.  It is in the draft stage; after getting it to a final version, I am planning on submitting it to Hacker News.  Doing research for it has helped me learn a great deal, and hopefully it can helpful to others.  


My Stance on Instance will Make You Dance, Enhance Your Brilliance, and Improve Your Romance	

If you're an idiot like me, instance variables in Ruby take some time to grasp.  They are designed to fill the gap in variable scope between local and global; however, they're also meant to fulfill the object-oriented goal of storing attributes for individual objects.  These two dimensions come from two different approaches of design, which I will call the top-down and bottom-up approaches.  The top-down approaches focuses on object-oriented principles like inheritance and individuation and the bottom-up approach focuses on making it easier to pass information between methods.	

Top-Down Approach

It is best to think of instance variables as the attributes, or properties, that are unique to each individual object within a larger type of object.  Each book has a title; each car has a color; each something has a something.  These properties together store an object's state, which basically is just the current values of its various attributes.  best also to create instance variables for the most important properties of the objects.  you can figure this out by thinking about what properties your methods will need to query to do their tasks.  

Instance variables are nil until the method in which they are assigned is called.  This means that if you assign an instance variable in one method and retrieve it in a second method, but then send the second method to an object instance before the first, the value of the variable will be nil.  This is different than the outcome of trying to retrieve an unassigned local variable—you would get an “undefined local variable or method error”  exception.  

Two important things are happening here: 

1. instance variables have to be assigned and retrieved within a method definition, as opposed to a class or global variable, which can be declared outside the scope of a method
2. you don't get an exception blowing up your program if your instance variable isn't assigned; you get nil, running the risk of stray nils in the program.  

For these reasons, it's a common practice to use the initialize method to assign instance variables.  This ensures that every instance of a class goes through the method call which first references the instance variables, and that you won't start out with nil instance variables (unless you design them that way).  

By assigning to instance variables in initialize, each object gets instantiated with its own unique state.  The attributes that make up an object's state stay attached to the object throughout any method calls; the instance variables of each object, therefore, are accessible across all method definitions within a class.  This is in contrast to local variables, which are only in scope within an individual method and disappear when returning from the method.

How to know what qualifies as an instance variable?  If it describes an attribute of the object and will be retrieved and updated without other manipulations, and if it will be used throughout many methods, it might make sense to make that variable an instance variable.  The best way to set up your instance variables is to wrap them in accessor methods or use Ruby's attr_accessor shorthand.  When your accessor method names match the instance variable names, you can retrieve and update the instance variables through the method invocations.  Another common pattern is to use attr_reader to define a simple wrapper method for reading the instance variable but manually creating a writer method if you need to do something more complicated than just setting the instance variable. 

Bottom Up Approach

Now that we understand what instance variables should be, let's try and understand what they can do.  There are 3 ways to store and access data through variables in methods:
1) create a local variable within the method
2) pass it to the method as a parameter
3) already have it accessible through the receiver's store of instance variables.
Local variables die when the method terminates.  Parameters are treated as inputs required by the method to reach its output.  But let's say you have a few methods that takes the same input, or perhaps a few methods that the will be sent to the object in a row that manipulate an input in stages.  Instead of passing the variable as a parameter to each method in turn, you can treat the variable as an instance variable and retrieve and assign to it directly within the body of any method.  

If you set up instance variables in methods other than initializer, you're not sure if the object if the instantiated object will have that available, unless you make sure each object goes through the same method call chain.  After all, an instance variable can only come into existence through the call of a method whose definition includes instance variable assignment.  If you want to have instance variables but the initial creation of the object doesn't really require anything special, you can assign empty data structures to instance variables in initialize.  If you don't have an initializer method but use another method as the main, or first, method that every instantiated object calls, then you should consider refactoring to make that an initializer method.  Otherwise, use memoization (||=) to ensure that if the instance variable had been created in a previous method, it isn't overwritten.

If you find yourself thinking about creating an instance variable in a method that isn't inside the definition of initialize, and find that the variable best applies to a subset of instances of the class that call a subset of methods, it might be time to think about refactoring and creating a new class (or subclassing the existing class).  Assign to the instance variable in the initialize method of the new class, and that way all instances of the new class have consistent instance variable usage.  The other benefit is that the methods defined in the new class have been refined and apply to only the objects that really need them.  

Final Thoughts

It's important to understand that these two approaches are not two fundamentally different definitions of instance variables.  Instead, the intrinsic properties of instance variables allow them to be  thought of as both attribute-holders and also stores-of-value that stay accessible across method calls.  What's actually happening under the hood is that each instance has an instance variable table created for it that keeps track of the variables for that object.  This table is meant to store the object's state, or in other words, its properties (color, name, etc.).   Thus the table can be thought of as an information storage container private to the object, fulfilling the object-oriented design purpose of inheriting properties and functionality from ancestors but allowing uniqueness at the object level.  

	Even though it is a design intention that instance variables store an object's state, in the end they are just variables and can store values other than attributes.  After all, a method call that includes instance variable assignment or retrieval just inspects the instance variable lookup table of the receiver for a match.  The inspection is unconcerned with whether the variable describes an attribute of the object or whether it holds any other arbitrary data.  Because the receiver of the method will always have its lookup table available, any instance variables for an object will be accessible to all instance methods defined in the same class.  

The extended scope of instance variables makes it easy to share their values across methods.  The benefit of this technique is ease of defining methods.  Parameter variables do not have to be passed from method to method, and it may make your coding process simpler by allowing you to access those variables in any method.  The disadvantage is that it breaks an important object-oriented idiom of only storing state in instance variables.  The other disadvantage is that it breaks encapsulation, which is a fancy way of saying that the properties of objects should be kept separate from each other and that the objects should communicate with each other through explicitly defined methods.  Keep these distinctions in mind when creating objects and writing methods for cleaner, more expressive code.  Get to it nerds!

Note: There is an extremely obvious case of this trade-off that we see all the time:  the sharing of instance variables between controllers and views.  In this case, there are two main things worth mentioning: 
1. Controller actions often assign records retrieved from the database to instance variables.  These are definitely not attributes of the instantiated controller class.
2. The variables became accessible by the view object that corresponds to the controller object. What da flipmode?  View object instances and Controller object instances are differenct objects, with different inheritance chains.  The accessibility clearly breaks with the core property of instance variables, namely that they are in scope only for the object to which they belong.  
According to The Rails 3 Way, what's basically happening is that Rails looks up the instance variable table for the controller object, loops through the variables, and creates new instance variables with the same name and values as those for the controller instance.  Many programmers have “railed” against (womp womp womp) this practice, but the advantage to the Rails community of easy access to data has thus far outweighed the disadvantage of breaking object-oriented design principles. 

Comments: If you have successfully used this post to somehow improve your romance, please email me immediately at manish.kakwani@gmail.com with step-by-step details, including pictures, of exactly what you did.  

Suggested References: I would recommend David Black's The Well Grounded Rubyist or Peter Cooper's Beginning Ruby: From Novice to Professional to anyone seeking further study.  In general, Mr. Cooper approaches instance variables in relation to variable scope, while Mr. Black approaches the topic in relation to object-oriented design.  If you're thinking, “I must be really smart because I've already read those books,” well, sorry to break it to you you handsome little devil, but nobody cares what you think.

Question—what are the best uses of setting instance variables outside of initializer?
