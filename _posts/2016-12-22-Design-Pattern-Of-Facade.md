---
layout: post
title:  "Design Pattern of Facade"
date:   2016-12-22 17:58:17 +0800
categories: iOS
---

## 1.Summary

This article is about `Facade Design Pattern`, this pattern is usually used to handle requests which have a common feature or can be implemented by the same entrance.

First of all, let's see a picture of `Facade Design Pattern` which was drawn before through MindNode.app:

[![](/images/201612-facade-pattern/facade.png)](/images/201612-facade-pattern/facade.png){:target="_blank"}

## 2.The example in Java

The biggest feature of `Facade` pattern is that there is a key-door waiting to handle the requests. The main handler class is called `Facade.java` and it has a method called `public void handleRequest(EntryInfo entryInfo)`, we will use this method to play the role of doorman who will handle all the requests received.

Now, let see the codes of this method in `Facade.java`:

```
public void handleRequest(EntryInfo entryInfo) {
	//Facade Pattern Example
	try {
		entryMap.get(entryInfo.getCity()).enter(entryInfo);
	} catch (Exception e) {
		this.printError("Sorry, we can NOT handle " + entryInfo.getCity() + "’s cargos for now.");
	}
}
```

See? All the requests are handled by `entryMap.get(entryInfo.getCity()).enter(entryInfo)`, so, what happened in the `entryMap`?

The codes of entryMap's initialization:

```
private static Map<String,IEntry> entryMap = null;
	
static {
	entryMap = new HashMap<String,IEntry>();
	entryMap.put("Beijing", new EntryBeijingImpl());
	entryMap.put("Shanghai", new EntryShanghaiImpl());
	entryMap.put("Guangzhou", new EntryGuangzhouImpl());
	entryMap.put("Shenzhen", new EntryShenzhenImpl());
}
```

In this map, we put some key-values in it. The key is identified by a city's name, the value is a certain implementation of interface `IEntry`.

We have interface `IEntry` looks like this:

```
package org.acttos.pattern.facade.entry;

import org.acttos.pattern.facade.entry.domain.EntryInfo;

/**
 * @author <a href="mailto:acttosma@gmail.com">Acttos</a>
 * @version 1.0.0
 */
public interface IEntry {
	public void enter(EntryInfo entryInfo);
}
```

There is only method called `public void enter(EntryInfo entryInfo)` in this interface which needs to be implemented by the classes which do the real job.

We have `EntryBeijingImpl.java`,`EntryShanghaiImpl.java`,`EntryGuangzhouImpl.java` and `EntryShenzhenImpl.java` to handle the request (Here is `EntryInfo`).

One of the implementation of the interface `IEntry` can be on behalf of all theothers, here let's see `EntryBeijingImpl.java`:

```
public class EntryBeijingImpl implements IEntry {

	/* (non-Javadoc)
	 * @see org.acttos.pattern.facade.entry.IEntry#enter(org.acttos.pattern.facade.entry.domain.EntryInfo)
	 */
	@Override
	public void enter(EntryInfo entryInfo) {
		System.out.println("I am Beijing Entry handler, I received a cargo '" + entryInfo.getCargo() + "'.");
	}

}
```

We just print out the message with different cargos in all the EntryXxxxImpl.java files. In fact, we will do the different things in every implementation class of IEntry, that is the meaning of `Facade`:'Do different things through only one key-door'.

Then, we need to take a look at class `EntryInfo`:

```
public class EntryInfo {
	private String city;
	private String cargo;
	/**
	 * @return the city
	 */
	public String getCity() {
		return city;
	}
	/**
	 * @param city the city to set
	 */
	public void setCity(String city) {
		this.city = city;
	}
	/**
	 * @return the cargo
	 */
	public String getCargo() {
		return cargo;
	}
	/**
	 * @param cargo the cargo to set
	 */
	public void setCargo(String cargo) {
		this.cargo = cargo;
	}
	
}
```

Here `EntryInfo.java` is just a pojo which has some properties, we also need to design the common request class in real job.

Now, let's finish the main method in `Facade.java`:

```
public static void main(String[] args) {
	Facade facade = new Facade();
		
	EntryInfo entryInfo = new EntryInfo();
	entryInfo.setCity("Beijing");
	entryInfo.setCargo("Cargo for " + entryInfo.getCity());
	facade.handleRequest(entryInfo);
		
	entryInfo.setCity("Shanghai");
	entryInfo.setCargo("Cargo for " + entryInfo.getCity());
	facade.handleRequest(entryInfo);
		
	entryInfo.setCity("Guangzhou");
	entryInfo.setCargo("Cargo for " + entryInfo.getCity());
	facade.handleRequest(entryInfo);
		
	entryInfo.setCity("Shenzhen");
	entryInfo.setCargo("Cargo for " + entryInfo.getCity());
	facade.handleRequest(entryInfo);
		
	entryInfo.setCity("Nanjing");
	entryInfo.setCargo("Cargo for " + entryInfo.getCity());
	facade.handleRequest(entryInfo);
	
}
``` 

When we execute `Facade.java` as Java application, we will get the output mostly like this:

```
I am Beijing Entry handler, I received a cargo 'Cargo for Beijing'.
I am Shanghai Entry handler, I received a cargo 'Cargo for Shanghai'.
I am Guangzhou Entry handler, I received a cargo 'Cargo for Guangzhou'.
I am Shenzhen Entry handler, I received a cargo 'Cargo for Shenzhen'.
========== ERROR ==========
Sorry, we can NOT handle Nanjing’s cargos for now.
========== ERROR ==========
```

Maybe you could't understand the codes above, because these codes are just part of Java project `Design Pattern`, you can see the completed codes [[ Here ]](https://github.com/acttos/DesignPattern){:target="_blank"}

## 3.Thanks for visiting.
