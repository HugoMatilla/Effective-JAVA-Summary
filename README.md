#CREATING AND DESTROYING OBJECTS
##1. Use STATIC FACTORY METHODS instead of constructors
**_ADVANTAGES_**

* Unlike constructors, they have names
* Unlike constructors, they are not requires to create a new object each time they're invoked
* Unlike constructors, they can return an object of nay subtype of their return type
* They reduce verbosity of creating parameterized type instances

**_DISADVANTAGES_**

* If providing only static factory methods, classes without public or protected constructors cannot be subclassed (encourage to use composition instead inheritance http://www.objectmentor.com/resources/articles/lsp.pdf)
* They are not readily distinguishable from other static methods (Some common names (each with a differnt pourpose) are: valueOf, of, getInstance, newInstance, getType and newType)

```java

	public static Boolean valueOf(boolean b){
		return b ? Boolean.TRUE :  Boolean.FALSE;
	}

```

##2. Use BUILDERS when faced with many constructors
Is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters.

Builder pattern simulates named optional parameters as in ADA and Python.


```java

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
	
	public class Elvis{
		public static final Elvis INSTANCE = new Elvis();
		private Elvis(){...}
		...
		public void singASong(){...}
	}
```

One problem is that a privileged client can invoke  the private constructor reflectively. Against this attack the constructor needs to be  modified to send  an exception iif it is asked to create a second instance.

**_Singleton with static factory_**

```java
	
	public class Elvis{
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

	private Object readResolve(){
	//Return the one true Elvis and let the garbage collector take care of the Elvis impersonator
	return INSTANCE;
	}
```

**_Enum Singleton, the preferred approach (JAVA 1.5)_**

```java

	public enum Elvis(){
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

	public class UtilityClass{
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

	String s = new String("stringette");		
```

Every call creates a new String instance. The argument *"stringette"* is itself one. This call in a loop would create many of them.

**_Do this_**

```java

	String s ="stringette";		
```

This one uses a single String instance rather than creating a new one.

**_Use static factory methods in preference to constructors (Item 1)_**

*Booelan.valueOf(String);* Is preferable to the constructor *Booelan(String)*.

**REUSE MUTABLE OBJECTS THAT WON'T BE MODIFIED**

**_Don't do this_**

```java

	public class Person {
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

	public class Person {
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

	//Slow program. Where is the object creation?
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

	public class Stack{
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

	public pop(){
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

	Foo foo = new Foo(...);
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

1. Use the == operator to check if the argument is a reference to this object (for performance)
2. Use the _instanceof_ operator to check if the argument has the correct type
3. Cast the argument to the correct type
4. For each "significant" field in the class, check if that field of the argument matches the corresponding field of this object
5. When you are finished writting your _equals_ method, ask yourself three questions: Is it Symmetric? Is it Transitive? Is it Consistent? (the other 2 usually take care of themselves)


```java

	@Override
	public boolean equals (Object o){
		if(o == this)
			return true;

		if (!(o instanceof PhoneNumber))
			return false;

		PhoneNumber pn = (PhoneNumber)o;
		return pn.lineNumber == lineNumber
			&& pn.prefix == prefix
			&& pn.areaCode == areaCode;
	}
```

**_Never Forget_**

* Always override _hashCode_ when you override _equals_
* Don't try to be too clever (simplicity is your friend)
* Don't substiture another type for _Object_ in the _equals_ declaration


##9. Always override _hashCode_ when you override *equals*

**_Contract of hashCode_**

* Whenever _hashCode_ is invoked in the same object it should return the same itnteger.
* If two objects are equals according to the  _equals_, the should return the same integer calling _hashCode_.
* Is not required (but recommended) that two non _equals_ objects return distinct _hashCode_.

The second condition is the one that is more often violated.

**_The Recipe_**

1. Store constant value i.e. 17 in and integer called _result_.
2. For each field _f_ used in _equals_ do:
  * Compute _c_
    *	boolean: _(f ? 1 : 0)_
    *	byte, char, short or int: _(int) f_
    *	long: _(int) (f ^ (.f >>> 32))_
    *	float: _Float.floatToIntBits(f)_
    *	double: _Double.doubleToLongBits(f)_ and compute as a long
    *	object reference: if _equals_ of the reference use recutsivity, use recursivity for the _hashCode_
    *	array: each element as a separate field.
  * Combine: _result = 31 * result + c_
3. Return _result_
4. Ask yourself if equal instances have equal hash codes.

```java

	private volatile int hashCode; // Item 71 (Lazily initialized, cached hashCode)

	@Override public int hashCode(){
		int result = hashCode;
		if (result == 0){
			result = 17;
			result = 31 * result + areaCode;
			result = 31 * result + prefix;
			result = 31 * result + lineNumber;
			hashCode = result;
		}
		return result;
	}
```	
##10. Always override _toString_

Providing a good _toString_ implementation makes your class much more pleasant to read.

When practival, the _toString_ method return all of the  interesting information contained in the object.

It is possible to specify the format of return value in the documentation. 

Always provide programmatic access to all of the information contained in the value returned by _toString_ so the users of the object don't need to parse the output of the _toString_

##12. Consider implementing _Comparable_ 
_Comparable_ is an interface. It is not declared in _Object_

Sorting an array of objects that implement _Comparable_ is as simple as `Array.sort(a);`

The class will interoperate with many generic algorithms and collection implementations that depend on this interfce. You gain lot of power with small effort.

Follow this provisions (Reflexive, Transitive, Symmetric): 

1.	`if a > b then b < a`  `if a == b then b == a`  `if a < b then b > a`
2.	`if a > b and b > c then a > c`
3.	`if a ==  b and b == c then a == c`
4.	Strong suggestion: `a.equals(b) == a.compareTo(b)`

For integral primitives use `<` and `>`operators. 

For floating-point fields use _Float.compare_ or _Double.compare_

For arrays start with the most significant field and work your way down.

# CLASSES AND INTERFACES
##13. Minimiza the accesibility of classes and members

__Encapsulation__:

* A well designed module hides all of its implementation details. 
* Separates its API from its implementation.
* Decouples modules that comprise a system, allowing them to be isolated while: 
	* developed (can be developed in parallel)
	* tested (individual modules may prove succesful even if the system does not)
	* optimized and modified (no harm to other modules)
	* understood (dont need other modules to be understood)
	* used 

__Make each class or member as inaccesible as possible__ 

If a package-private top level class is used  by only one class make it a  private nested class of the class that uses it. (Item 22)

Is is acceptable to make a private member of a public class package-private in order to test it.

__Instance fields should never be public__ (Item 14) Class will not be thread-safe.

Static fields can be public if contain primitive values or references to inmutable objects. A final field containing a reference to a mutable object has all the disadvantages of a non final field.

Nonzero-length array is always mutable.
```java
	
	//Potential security hole!
	public static final Thing[] VALUES = {...}
```
Solution:
```java
	
	private static final Thing[] PRIVATE_VALUES ={...}
	public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```
Or:
```java
	
	private static final Thing[] PRIVATE_VALUES ={...}
	public static final Thing[] values(){
		return PRIVATE_VALUES.clone;
	}
```
##14 In public classes, use accessot methods, not public fields

Degenerate classes should not be public

```java

	class Point {
		public double x;
		public double y;
	}
```

* The don't benefit from _encapsulation_ (Item 13)

* Can't change representation without changing the API.

* Can't enforce invariants.

* Can't take auxiliary actions when a field is accessed.

Replace them with _accessor methods_ (getters) and _mutators_ (setters).

```java

	class Point {
		private double x;
		private double y;

		public Point(double x, double y){
			this.x = x;
			this.y = y;
		}

		public double getX() { return x; }
		public double getY() { return y; }

		public void setX(double x) { this.x = x; }
		public void setY(double y) { this.y = y; }
	}
```

If a class is accessed **outside its package**, provide **accesor methods**.

If a class is **package-private or is a private nested class**, its **ok to expose** its data fields.

In **public classes** it is a questionable option to **expose immutable fields**.



##15 Minimize Mutability
All the information of the instance is provided when it is created.
They are easier to design, implement and use. And they are less prone to errors and more secure

* Don't provide any methods that modify the object's state (no mutators)
* Ensure that the class can't be extended
* Make all fields final
* Make all fields private
* Ensure exclusive access to any mutable component

```java
	
	public final class Complex {
		private final double re;
		private final double im;

		public Complex (double re, double im) {
			this.re = re;
			this.im = im;
		}

		// Accessors with no corresponding mutators
		public double realPart() { return re;}
		public double imaganaryPart() { return im;}

		public Complex add(Complex c){
			return new Complex(re + c.re, im + c.im);
		}

		public Complex subtract(Complex c){
			return new Complex(re - c.re, im - c.im);
		}

		...

		@Override public boolean equals (Object o){...}
	}

```

The arithmetic operation __create and return a new instance__. (Functional approach)

Inmutable objects are simple. They only have one state for its lifetime.

Inmutable objects are thread-safe. Synchronization is not required. They can be shared freely and can reuse existing instances.

```java

	public static final Complex ZERO = new Complex(0,0)
	public static final Complex ONE = new Complex(1,0)
	public static final Complex I = new Complex(0,1)
```

Using static factories can create constants of frequently requested instances and serve them in future requests.

Internals of the inmutable objects can also be share.

They make great building blocks for other objects.

The disadvantes is that require a separate object for distinct values. In some cases it could reach to a performance problem.

__How to deny subclassing in imnutable objects__

1. Making it final

2. Make all of its contructors private or package-private and add a public static factory

```java

	public class Complex {
		private final double re;
		private final double im;

		private Complex (double re, double im){
			this.re = re;
			this.im = im;
		}

		public static Complex valueOf(double re, double im){
			return new Complex(re,im);
		}

		...
	}
```

This technique allows flexibily of multiple implementations, it's possible to tune  the performance and permit to create more factories with names that clarify its function.

__Summary__

Classes should be immutable unless there are good reasons to make them mutable.

If a  class can not be immutable, limit its mutability as much as possible.

Make every field final unles there is a good reason not to do it.

Some of the rules can be lightened to improve performance (caching, lazy initialization...).

##16 Favor composition over inheritance
Inheritance in this case is when a class extends another (_implementation inheritance_) Not interface inheritance.

**Inheritance violates encapsulation**

Fragility causes

1. A subclass depends on the implementation details of its superclass. If the superclass change the subclass may break.

2. The superclass can aquire new methods in new releases that might not be added in the subclass.

**Compostion**

Instead of extending, give your new class a private field that references an instance of the existing class.

Each instance method in the new class (_forwarding class_)invokes the corresponding method (_forwarding methods_) on the contained instance of the existing class and returns the results.


**Wrapper (Decorator Patterm)**
```java

	// Wrapper class - uses composition in place of inheritance
	public class InstrumentedSet<E> extends ForwardingSet<E> {
		private int addCount = 0;
		//It extends a class(inheritance),but it is a forwarding class that is actually a compositon of the Set
		(specifically a forwarding class), not the Set itself. 
		public InstrumentedSet (Set<E> s){ 
			super(s)
		}

		@Override
		public boolean add(E e){
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll (Collection< ? extends E> c){
			addCount += c.size();
			return super.addAll(c);
		}

		public int getAddCount() {
			return addCount;
		}
	}
```

```java

	// Reusable forwarding class
	public class ForwardingSet<E> implements Set<E> {
		private final Set<E> s; // Here is the composition. It uses the Set but not extends it.
		public ForwardingSet(Set<E> s) { this.s = s ; }

		// It implemets the Set, using the interface, and create the forwarding methods.
		public void clear() {s.clear();}
		public boolean conatains(Object o) { return s.contains(o)}
		...
		public boolean add(E e) { return s.add(e)}
		public boolean addAll (Collection< ? extends E> c){return s.addAll(c)}
		...
	}

```

##17. Design and document for inheritance or else prohibit it.

The class must document it _self-use_ of overridable methods.

Methods and constructors should document which _overridable_ methods or constructors (nonfinal, and public or protected ) invokes. The description begins with the phrase "This implementation."

To document  a class so that it can be safely subclassed, you must describe implementations details.

To allow programmers to write efficient subclasses without undue pain, a class may have to provide hooks into its internal working in the form of judiciously chosen protected methods.

__Test the class for subclassing__. The only way to test a class desgined for inheritance is to write subclasses.

Constructors must not invoke overridable methods. For _Serializable_ and  _Cloneable_ implementations neither _clone_ nor _readObject_ may  invoke overridable methods.

Prohibit subclassing in classes that are not designed and documented to be safely subclassed. 2 options:

* Declare the class final
* Make all constructors private or package-private and add public static factories in place of the constructors. (Item 15)

Consider use Item 16 if what you want is to increase the functionality of your class instead of subclassing.

##18. Prefer interfaces to abstract classes
Java permits only single Inheritance, this restriction on abstract classes severely contrains their use as type functions.

Intefaces is generally yhe best way to define a type that permits multiple implementations.

Existing classes can be easily retrofitted to implement a new interface.

Interfaces are ideal for defining mixins (a type that a class can implement in addition to its primary type to declare that it provides some optional bahaviour)

Interfaces allow the construction of nonhierarchical type frameworks.

Interfaces enable safe, powerful functionality enhancements (Wrapper class. Item 16)

Combine the virtues of interfaces and abstract classes, by providing an abstract **skeletal implementation** class to go with each **nontrivial interface** that you export.

```java

	//Concrete implementation built atop skeletal implementation
	static List<Integer> intArrayAsList(final int[] a) {
		if (a == null) throw new NullPointerException();


		// From the documentation
		//This class provides a skeletal implementation of the List interface to minimize the effort required to implement this interface backed by a "random access" data store (such as an array)
		//To implement an unmodifiable list, the programmer needs only to extend this class and provide implementations for the get(int) and size() methods.
		//To implement a modifiable list, the programmer must additionally override the set(int, E)

		return new AbstractList<Integer>(){
			public Integer get (int i){
				return a[i]; // Autoboxing (Item 5)
			}

			@Override
			public Intger set(int i, Integer val){
				int oldVal = a[i]; 
				a[i] = val;		// Auto-unboxing 
				return oldVal;	// Autoboxing 
			}

			public int size(){
				return a.length;
			}
		}
	}
```
Sekeltal implementations are designed for inheritance so follow Item 17 guidelines.

_simple implementation_ is like a skeletal implementation in that int implements the simplest possible working implementation.

Cons: It is far easier to evolve an abstract class than an interface. Once an interface is released and widely implemented, it is almost imposible to change.

##53. Prefer interfaces to reflection
_java.lang.reflection_ offers access to information about loaded classes.

Given a _Class_ object, you can obtain _Constructor_, _Method_ and _Field_ instances.

Allows one class to use another, even if the latter class did not exist when the former was compiled.

*	Lose of al benefits of compile-time type checking
*	Code to perform reflective access is clumsy and verbose
*	Performance suffers.

**As a rule, objects should not be accessed reflectively in normal applications at runtime**

Obtain many of the benfits of reflection incurring few of its costs by **creating instances reflectively and access them normally via their interface or superclass**.

```java

	// Reflective instantiation with interface access
	public static void main (String[] args){
		// Translate the class name into a class object
		Class<?> cl =  null;
		try{
			cl = Class.forName(args[0]);// Class is specified by the first command line argument
		}catch(ClassNotFoundException e){
			System.err.println("Class not found");
			System.exit(1);
		}

		//Instantiate the class
		Set<String> s = null;
		try{
			s = (Set<String>) cl.newInstance(); //  The class can be either a HashSet or a TreeSet
		} catch(IllegalAccessException e){
			System.err.println("Class not accessible");
			System.exit(1);
		}catch(InstantionationException e){
			System.err.println("Class not instantiable");
			System.exit(1);
		}

		//Excersice the Set
		// Print the remaining arguments. The order depends in the class. If it is a HashSet 
		// the order will be random, if it is a TreeSet it will be alphabetically
		s.addAll(Arrays.asList(args).subList(1,args.length));
		System.out.println(s);
	}
```

A legitimate use of reflection is to manage a class's dependencies on other classes, methods or fields that may be absent at runtime.

Reflection is powerfull and usefull in some sophisticated systems programming tasks. It has many disadvantages.
Use reflection, if possible, only to instantiate objects and access the objects using an interface or a superclass that is known at compile time.