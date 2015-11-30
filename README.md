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
> 	public Elvis(){
		INSTANCE;
		...
		public void singASong(){...}
	}
```

Equivalent to the public field, more concise, provides serialization machinery for free, and guarantee against multiple instantiation, even for reflection attacks and sophisticated serialization. _It is the best way to implement a singleton_.

##4. Enforce noninstantiablillity with a private constructor
For classes that group static methods and static fields. 
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