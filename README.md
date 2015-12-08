#CREATING AND DESTROYING OBJECTS
##1. Use STATIC FACTORY METHODS instead of constructors
**_ADVANTAGES_**

*Unlike constructors, they have names
*Unlike constructors, they are not requires to create a new object each time they're invoked
*Unlike constructors, they can return an object of nay subtype of their return type
*They reduce verbosity of creating parameterized type instances

**_DISADVANTAGES_**

*If providing only static factory methods, classes without public or protected constructors cannot be subclassed (encourage to use composition instead inheritance http://www.objectmentor.com/resources/articles/lsp.pdf)
*They are not readily distinguishable from other static methods (Some common names (each with a differnt pourpose) are: valueOf, of, getInstance, newInstance, getType and newType)

```java
> 	
	public static Boolean valueOf(boolean b){
		return b ? Boolean.TRUE :  Boolean.FALSE;
	}

```

##2. Use BUILDERS when faced with many constructors
Is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters.

Builder pattern simulates named optional parameters as in ADA and Python.


```java
>	
	public class NutritionFacts {
		private final int servingSize;
		private final int servings;
		private final int calories;
		private final int fat;
		private final int sodium;
		private final int carbohydrate;

		public static class Builder {
			//Required parameters
			private final int servingSize:
			private final int servings;

			/Optional parameters - initialized to defalult values
			private int calories		= 0;
			private int fat 			= 0;
			private int carbohydrate 	= 0;
			private int sodium 			= 0;

			public Builder (int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			}

			public Builder calories (int val) {
				calories = val;
				return this;				
			}

			public Builder fat (int val) {
				fat = val;
				return this;				
			}

			public Builder carbohydrate (int val) {
				carbohydrate = val;
				return this;				
			}

			public Builder sodium (int val) {
				sodium = val;
				return this;				
			}

			public NutritionFacts build(){
				return new NutritionFacts(this);
			}
		}

		private NutritionFacts(Builder builder){
			servingSize		= builder.servingSize;
			servings 		= builder.servings;
			calories		= builder.calories;
			fat 			= builder.fat;
			sodium 			= builder.sodium;
			carbohydrate	= builder.carbohydrate;
		}
	}
```
**_Calling the builder_**
```java
> 	
	NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();

```

##3. Enforce the singleton property with a private constructor or an enum type
There are different ways to create singletons:

**_Public fnial field_**

```java
> 	public class Elvis{
		public static final Elvis INSTANCE = new Elvis();
		private Elvis(){...}
		...
		public void singASong(){...}
	}
```

One problem is that a privileged client can invoke  the private constructor reflectively. Against this attack the constructor needs to be  modified to send  an exception iif it is asked to create a second instance.

**_Singleton with static factory_**

```java
> 	public class Elvis{
		private static final Elvis INSTANCE = new Elvis();
		private Elvis(){...}
		public static Elvis getInstance(){ return INSTANCE; }
		...
		public void singASong(){...}
	}
```

In this approach it can be change to a non singleton class without changing the class API.

**_Serialize a singleton_**

It is needed a _readResolve_ method and declare all the fields _transient_ in addtion to the _implements Serializable_ to mantain the singleton guarantee. 

```java
>	private Object readResolve(){
	//Return the one true Elvis and let the garbage collector take care of the Elvis impersonator
	return INSTANCE;
	}
```

**_Enum Singleton, the preferred approach (JAVA 1.5)_**

```java
> 	public enum Elvis(){
		INSTANCE;
		...
		public void singASong(){...}
	}
```

Equivalent to the public field, more concise, provides serialization machinery for free, and guarantee against multiple instantiation, even for reflection attacks and sophisticated serialization. _It is the best way to implement a singleton_.

##4. Enforce noninstantiablillity with a private constructor
For classes that group static methods and static fields. 
Used for example to: 
* Group related methods on primitive values or arrays.
* Group static methods, including factory methods, for objects tha implement a particular interface.
* Group methods on a final class instead of extending the class.

**_Include a private constructor_**
```java
>	public class UtilityClass{
		// Supress deafult constructor for noninstantiablillity (Add comment to clarify why the constructor is expressly provided)
		private UtilityClass(){
			throw new AssertionError();
		}
		...
	}
```

##5. Avoid creating unnecesary objects

**REUSE INMUTABLE OBJECTS**

**_Don't do this_**

```java
>	String s = new String("stringette");		
```

Every call creates a new String instance. The argument *"stringette"* is itself one. This call in a loop would create many of them.

**_Do this_**

```java
>	String s ="stringette";		
```

This one uses a single String instance rather than creating a new one.

**_Use static factory methods in preference to constructors (Item 1)_**

*Booelan.valueOf(String);* Is preferable to the constructor *Booelan(String)*.

**REUSE MUTABLE OBJECTS THAT WON'T BE MODIFIED**

**_Don't do this_**

```java
>	public class Person {
	private final Date birthDate;
	...
		public booelan isBabyBoomer(){
			//Unnecesary allocation of expensive object.
			Calendar gmtCal= Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946,Calendar.JANUARY,1,0,0,0);
			Date boomStart = gmtCal.getTime();
			gmtCal.set(1965,Calendar.JANUARY,1,0,0,0);
			Date boomEnd = gmtCal.getTime();
			return birthDate.compareTo(boomStart) >= 0 &&birthDate.compareTo(boomEnd)<0;
		}
	}
```

isBabyBoomer creates a new Calendar,TimeZone and two Date instances each time is invoked.

```java
>	public class Person {
	private final Date birthDate;
	...
	private static final Date BOOM_START;
	private static final Date BOOM_END;

	static {
			Calendar gmtCal= Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946,Calendar.JANUARY,1,0,0,0);
			BOOM_START = gmtCal.getTime();
			gmtCal.set(1965,Calendar.JANUARY,1,0,0,0);
			BOOM_END = gmtCal.getTime();
	}
		public booelan isBabyBoomer(){
			return birthDate.compareTo(BOOM_START) >= 0 &&birthDate.compareTo(BOOM_END)<0;
		}
	}
```

**_Prefer primitives to boxed primitives, and wtach out for unintentional autoboxing_**

```java
>	//Slow program. Where is the object creation?
	public static void main (String[] args){
		Long sum = 0L;
		for (long i = 0 ; i<= Integer.MAX_VALUE; i++){
			sum+=i;
		}
		System.out.println(sum);
	}
```	

*sum* is declared as *Long* instead of *long* that means that the programs constructs  unnecesary Long instances. 

**_Object polls are normally bad ideas_**

Unless objects in the pool are extremly heavyweight, like a database connections.

##6. Eliminate obsole object references
**_Can you spot the memory leak?_**
```java
>	public class Stack{
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITAIL_CAPACITY = 16;

		public Stack(){
			elements new Object [DEFAULT_INITAIL_CAPACITY];
		}

		public void push(Object e){
			ensureCapacity();
			elements[size++] = e;
		}

		public pop(){
			if (size == 0)
				throw new EmptyStacjException();
			return elements[--size];
		}

		private void ensureCapacity(){
			if(elements.length == size)
				elements = Array.copyOf(elements, 2 * size + 1);
		}
	}
``` 

If the stack grows and shrinks the objects popped will not be garbage collected. The stack mantains obsolete references (a reference that will never be dereferenced).

**_Null out references_**

```java
>	public pop(){
		if (size == 0)
			throw new EmptyStacjException();
		Object result = elements[--size];
		elements[size] = null; // Eliminate obsolete references.
		return result;
	}
```

Nulling out objects references should be the exception not the norm.  Do not overcompensate by nulling out every object.

Null out objects only in classes that manages its own memory.

**_Memory leaks in cache_**

 Using _WeakHashMap_ is useful when if desired lifetime of cache entries is determined by external references to the key, not the value.

 Clean oldest entries in cache is a common practice. To acommplish this behaviors, it can be used: background threads, automatically delete older after a new insertion or the _LinkedHashMap_ and its method _removeEldestEntry.

**_Memory leaks in listeners and callbacks_**

If clients register callbacks, but never deregister them explicity.

To solve it store only _weak references_ to them, for example storing them as keys in a _WeakHashMap_.

__Use a Heap Profiler from time to time to find unseen memory leaks__

##7. Avoid finalizers
 Finalizers are unpredictable, often dangerous and generally unnecesary. 

**_Never do anything time-critical in a finalizer._**
>There is no guarantee they'll be executed promptly. 

**_Never depend on a finalizer to update critical persistent state._**
>There is no guarantee they'll be executed at all. 

Uncaught exceptions inside a finalizer won't even print a warning.

Ther is a severe performance penalty for using finalizers.

**_Possible Solution_**

Provide an _explicit termiantion method_ like the _close_ on  _InputStream_, _OutputStream_, _java.sql.Connection_...

Explicit termination methods are typically used in combination with the _try-finally_ construct to ensure termination.
```java
>	Foo foo = new Foo(...);
	try {
	    // Do what must be done with foo
	    ...
	} finally {
	    foo.terminate();  // Explicit termination method
	}
```

**They are good for this two cases**

* As a safety net. Ask yourself if the extra protection is worth the extra cost.
* Use in native peers. Garbage collector doesn't know about this objects.

In this cases always remember to invoke super.finalize. 

#METHODS COMMON TO ALL OBJECTS
##8. Obey the general contract when overriding *equals*

**_Dont override if:_**

* Each instance of the class is inherently unique. I.e._Thread_
* You dont care wheter the class provides a "logical equallity" test. I.e. _java.util.Random_
* A superclass has already overriden _equals_, and the superclass behavior is appropriate for this class I.e. _Set_
* The class is private or package-private, and you are certain  that its _equals_ method will never be invoked

**_Override if:_**

A class has a notion of _logical equality_ that differs from mere object identity, and a superclass has not already overriden _equals_ to implement the desired behavior.

**_Equals implements an "equivalence relation"_**

* Reflexive: *x.equals(x)==true*
* Symmetric: *x.equals(y)==y.equals(x)*
* Transitive: *x.equals(y)==y.equals(z)==z.equals(x)*
* Consistent: *x.equals(y)==x.equals(y)==x.equals(y)==...*
* Non-nullity: *x.equals(null)->false*

**_The Recipe_**

* Use the == operator to check if the argument is a reference to this object (for performance)
* Use the _instanceof_ operator to check if the argument has the correct type
* Cast the argument to the correct type
* For each "significant" field in the class, check if that field of the argument matches the corresponding field of this object
* When you are finished writting your _equals_ method, ask yourself three questions: Is it Symmetric? Is it Transitive? Is it Consistent? (the other 2 usually take care of themselves)

**_Never Forget_**

* Always override _hashCode_ when you override _equals_
* Don't try to be too clever (simplicity is your friend)
* Don't substiture another type for _Object_ in the _equals_ declaration




