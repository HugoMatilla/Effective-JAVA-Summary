#1. Use STATIC FACTORY METHODS instead of constructors
*ADVANTAGES*
*Unlike constructors, they have names
*Unlike constructors, they are not requires to create a new object each time they're invoked
*Unlike constructors, they can return an object of nay subtype of their return type
*They reduce verbosity of creating parameterized type instances
*DISADVANTAGES*
*If providing only static factory methods, classes without public or protected constructors cannot be subclassed (encourage to use composition instead inheritance http://www.objectmentor.com/resources/articles/lsp.pdf)
*They are not readily distinguishable from other static methods (Some common names (each with a differnt pourpose) are: valueOf, of, getInstance, newInstance, getType and newType)

```
> 	
	public static Boolean valueOf(boolean b){
		return b ? Boolean.TRUE :  Boolean.FALSE;
	}

```

#2. Use BUILDERS when faced with many constructors
Is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters.

Builder pattern simulates named optional parameters as in ADA and Python.


```
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
*Calling the builder*
```
> 	
	NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();

```

