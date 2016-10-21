---
layout: post
title:  "Design Pattern of Factory"
date:   2016-10-18 17:41:49 +0800
categories: Design Pattern
---

## 1. Summary

When we development with some request description, we need to design a code struct to write codes with. The code struct design has some very famous design patterns when development. One of the most famous design pattern is called `Factory`.

In our developments, we usually need some domain classes to handle the producing of objects, and most of the time, the layer of handling logic and the layer of creating objects are divided, standalone and have no interactions at all. Both of the layers do not need to know what is happening in the other side and do not care how the other side works.

Design Pattern of `Factory` is a pattern producing objects in the development. It contains three different types of pattern: `Simple Factory Pattern`, `Factory Methods Pattern` and `Abstract Factory Pattern`.

## 2. The three design patterns of factory

### 2.1. Simple Factory Pattern

`Simple Factory Pattern` is the simplest pattern of 'Factory', it just creates products and return the objects, we can define a simple factory like this:

```
public class SimpleFactory {
	private static SimpleFactory instance;

	private SimpleFactory() {
		super();
	}

	public static SimpleFactory instance() {
		if (instance == null) {
			instance = new SimpleFactory();
		}

		return instance;
	}

	public static SimpleFactory getInstance() {
		if (instance == null) {
			instance = new SimpleFactory();
		}

		return instance;
	}

	public static Product create() {
		System.out.println("Factory is creating Product ...");
		Product product = new Product();

		return product;
	}
}
```

We have a `private` `static` object of type `SimpleFactory`, and it is not available to the outside of the class, we also set the constructor of `SimpleFactory` to be `private` too, so the caller can NOT invoke it directly, the only way we can get an instance if by calling `SimpleFactory.instance()` or `SimpleFactory.getInstance()`, then we can invoke `instance.create()` to produce objects.

Let's define a class `Producer` to use this `SimpleFactory`, it is just like this:

```
public class Producer {
	public static void main(String[] args) {
		Product p = SimpleFactory.create();
		System.out.println("A product called '" + p.getName() + "' has been created.");
	}
}
```

Here is the output of this class in my console:

```
Factory is creating Product ...
Product is created.
A product called 'Super Product' has been created.
```

We can see that, the `Producer` and the `Product` is divided, standalone and has no interactions at all. The `Producer` acquires an object `Product` through `SimpleFactory` and does not care how the product is produced. Apparently, we have reduced the coupling of producing an object.

The `Simple Factory Pattern` can produce single type of product very well, but when we need to add many other types of product, what are we going to do? Creating more SimpleFactory with a type of product? Adding more types of product in the single one `SimpleFactory`? Both are OK. This question brings another pattern of `Factory`, it is called `Factory Methods`:

### 2.2. Factory Methods Pattern

When we produce only one simple product, the pattern `Simple Factory` is well enough for us. when we need to add some more products, we need another pattern `Factory Methods Pattern`.

So, what is different between `Simple Factory` and `Factory Methods`? It is simple, `SimpleFactory` pattern is good at the condition of single one product, `Factory Methods` pattern is good at handling more than one products in the same type.

Though we need to support different sub-types of product, we need different factories to do the job. We hope all the factories have the same method for producer to invoke, so we can define factory methods with an interface in this way:

```
/**
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public interface IMethodFactory {
	public Product create();
}
```
We have an interface class with a method `create()` in it. All the factories implemented the interface must contain `create()`, so the producer can just invoke `create()` without worries.

Now, let's write down two classes which implemented the interface :

`MethodFactoryBeijingImpl` represents Beijing factory who only produces `BeijingProduct`：

```
/**
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class MethodFactoryBeijingImpl implements IMethodFactory {

	@Override
	public Product create() {
		System.out.println("Beijing factory is creating a product ...");

		Product product = new BeijingProduct();

		return product;
	}

}
```

`MethodFactoryShanghaiImpl` represents Shanghai factory who only produces `ShanghaiProduct`:

```
/**
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class MethodFactoryShanghaiImpl implements IMethodFactory {

	@Override
	public Product create() {
		System.out.println("Shanghai factory is creating a product ...");

		Product product = new ShanghaiProduct();

		return product;
	}

}
```
Both of `BeijingProduct` and `ShanghaiProduct` are sub-type of `Product`, they all are the same type.

We can see the Beijing factory and Shanghai factory have different implementations of `create()`, each factory creates product with specified type.

Now, let's check this codes out by creating a class `Producer`:

```
/**
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class Producer {

	/**
	 * We receive an implement of IMethodFactory to do the producing.
	 * @param factory the implement of IMethodFactory which does nothing but defining.
	 * @return a product created by the give factory.
	 */
	public static Product create(IMethodFactory factory) {
		Product product = factory.create();

		return product;
	}

	public static void main(String[] args) {
		/**
		 * These lines below are a directly invoking of the implements of the interface.
		 */
		System.out.println("These lines below are a directly invoking of the implements of the interface."
				+ "\n================================================================================\n");
		IMethodFactory beijingFactory = new MethodFactoryBeijingImpl();
		Product beijingP = beijingFactory.create();

		System.out.println("A product called '" + beijingP.getName() + "' has been created.\n");

		IMethodFactory shanghaiFactory = new MethodFactoryShanghaiImpl();
		Product shanghaiP = shanghaiFactory.create();

		System.out.println("A product called '" + shanghaiP.getName() + "' has been created.\n");

		/**
		 * These lines below create a product with the given implement of IMethodFactory.
		 */
		System.out.println("These lines below create a product with the given implement of IMethodFactory."
				+ "\n================================================================================\n");
//		Product product = Producer.create(beijingFactory);//Uncomment this line to verify the beijingFactory
		Product product = Producer.create(shanghaiFactory);

		System.out.println("A product called '" + product.getName() + "' has been created.\n");

	}
}
```

Here is the output in my console, you can type the codes yourself and check the result:

```
These lines below are a directly invoking of the implements of the interface.
================================================================================

Beijing factory is creating a product ...
Product is created.
BeijingProduct is created at Beijing.
A product called 'BeijingProduct' has been created.

Shanghai factory is creating a product ...
Product is created.
ShanghaiProduct is created at Shanghai.
A product called 'ShanghaiProduct' has been created.

These lines below create a product with the given implement of IMethodFactory.
================================================================================

Shanghai factory is creating a product ...
Product is created.
ShanghaiProduct is created at Shanghai.
A product called 'ShanghaiProduct' has been created.
```

The `Factory Methods Pattern` is designed to handle multiple products of a single type with multiple factories, each sub-factory produces a specified product. But, what are we going to do when we need produce more than one type of product? such as `InternalProduct` and `AbroadProduct`? they are with different type, how to handle this?

### 2.3. Abstract Factory Pattern

The `Factory Methods Pattern` is for producing multiple sub-types products with one single type. when we want to produce multiple types of product, we have `Abstract Factory Pattern`.

In `Abstract Factory Pattern`, the factory can produce more than one type of product, so we define a `abstract class` called `AbstractFactory`:

```
/**
 * The creator of product, the 'createProduct..' methods count is the same as the type of product
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
 public abstract class AbstractFactory {
 	/**
 	 * Define a method to produce ProductA.
 	 * @return an instance of AbstractProductA.
 	 */
 	public abstract AbstractProductA createProductA();
 	/**
 	 * Define a method to produce ProductB.
 	 * @return an instance of AbstractProductB.
 	 */
 	public abstract AbstractProductB createProductB();
 }
```

In this class, we have two methods `createProductA()` and `createProductB` which is made to produce `ProductA` and `ProductB.

In `Abstract Factory Pattern`, we can produce different types of product. `ProductA` and `ProductB` are different types of product. so we defined two methods to do the job.

Now, let's implement this class `AbstractFactory, here we go:


`AbroadProductFactory` implements `AbstractFactory`:

```
/**
 * Only produces abroad products
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class AbroadProductFactory extends AbstractFactory {

	@Override
	public AbstractProductA createProductA() {
		return new AbroadProductA();
	}

	@Override
	public AbstractProductB createProductB() {
		return new AbroadProductB();
	}

}
```

> Here we define class `AbroadProductFactory` for abroad products, we can just define a class `ProductAFactory` only produce `ProductA`, that is under `Abstract Factory Pattern` too. Here we produce two types of product to make it a little complex to help us to understand the pattern.

`InternalProductFactory` implements `AbstractFactory`:

```
/**
 * Only produces internal products
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class InternalProductFactory extends AbstractFactory {

	@Override
	public AbstractProductA createProductA() {
		return new InternalProductA();
	}

	@Override
	public AbstractProductB createProductB() {
		return new InternalProductB();
	}

}
```

We have two factories for the producing, now, let's check the result in a class called `Producer`:

```
/**
 * Here is a demo class shows how to use 'Design Pattern Abstract Factory'.
 * Commonly, we define two products and two types, and one Abstract Factory
 * with the same count methods of products.
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public class Producer {
	@SuppressWarnings("unused")
	public static void main(String[] args) {
		/**
		 * Here we define two factories for 'Abroad' and 'Internal' products
		 */
		AbstractFactory abroadProductFactory = new AbroadProductFactory();
		AbstractFactory internalProductFactory = new InternalProductFactory();

		/**
		 * Here we define two 'ProductA' with 'Abroad' and 'Internal'
		 */
		AbstractProductA abstractAbroadProductA = abroadProductFactory.createProductA();
		AbstractProductA abstractInternalProductA = internalProductFactory.createProductA();

		/**
		 * Here we define two 'ProductB' with 'Abroad' and 'Internal'
		 */
		AbstractProductB abstractAbroadProductB = abroadProductFactory.createProductB();
		AbstractProductB abstractInternalProductB = internalProductFactory.createProductB();

		/**
		 * Here we can do anything we want.
		 */
		System.out.println(abroadProductFactory);
		System.out.println(internalProductFactory);

		System.out.println(abstractAbroadProductA);
		System.out.println(abstractInternalProductA);

		System.out.println(abstractAbroadProductB);
		System.out.println(abstractInternalProductB);
	}
}
```

In the class' main method, we just print out the class name and the addresses in memory, we can do much more with the products in specified requirements.

Here is the output in my console:

```
org.acttos.pattern.factory.abstrakt.AbroadProductFactory@7852e922
org.acttos.pattern.factory.abstrakt.InternalProductFactory@4e25154f
org.acttos.pattern.factory.AbroadProductA@70dea4e
org.acttos.pattern.factory.InternalProductA@5c647e05
org.acttos.pattern.factory.AbroadProductB@33909752
org.acttos.pattern.factory.InternalProductB@55f96302
```

You can practice with the codes above to understand `Abstract Factory Pattern` deeply.

PS: The codes above are just apart of the whole project, you can imagine the last part by yourself or you can visit [My Git](https://github.com/acttos/){:target="_blank"} to clone the whole project (Design Pattern) to verify.

## 3. Conclusion

| Pattern Name |Scenarios|
|:-------------:|:---------:|
|Simple Factory|Produces simple products, does not support multiple types of products, when need to add a new type of product, you need to add new factory class.|
|Factory Methods|Produces products in a single type. Supports sub-type of products with sub-class of super-class of product, does not support to add new types of product.|
|Abstract Factory|Produces multiple types of product, each type of product needs a method in the factory class, does not support to add new product.|

Design Pattern is very useful to manage the code struct in our projects, a well formatted code struct can bring happiness while coding, less communications, more comfortable feelings.

I was plan to write codes about `Design Pattern` in [My Git](https://github.com/acttos/){:target="_blank"}, welcome to be a part of [this project](https://github.com/acttos/DesignPattern){:target="_blank"} with me.

## 4.Thanks for visiting.
