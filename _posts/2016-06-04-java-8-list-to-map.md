---
id: 96
title: 'Java 8 Stream &#8211; From List to Map'
date: 2016-06-04T15:34:33+00:00
author: Mario Pio Gioiosa
layout: post
guid: http://reversecoding.net/?p=96
permalink: /java-8-list-to-map/
site_layout:
  - full-width
hefo_before:
  - "0"
hefo_after:
  - "0"
categories:
  - Java 8
tags:
  - Collectors
  - Java
  - Java 8
  - map
  - stream
  - toMap
---
An example to convert a _List<?>_ to a _Map<K,V>_ using Java 8 _[Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)_.

### Java 8 &#8211; Collectors.toMap()

Let&#8217;s define a Pojo class:

<pre class="lang:java decode:true">public class Person {

	private String email;
	private String name;
	private int age;

	public Person(String email, String name, int age) {
		this.email = email;
		this.name = name;
		this.age = age;
	}
	
	//Getters, setters 

        @Override
        public String toString() {
            return "Person [email=" + email + ", name=" + name + ", age=" + age + "]";
	}

}
</pre>

In the first example, we convert a _List<Person>_ in a _Map<String, Person>_ that has email as key and the object itself as value.

<pre class="lang:default decode:true">List&lt;Person&gt; people = Arrays.asList(
     new Person("mario@reversecoding.net", "Mario", 27),
     new Person("luigi@reversecoding.net", "Luigi", 30),
     new Person("steve@reversecoding.net", "Steve", 20)
 );
		
 Map&lt;String, Person&gt; mapEmailPerson = 
          people.stream()
                .collect(Collectors.toMap(Person::getEmail, Function.identity()));
		
 System.out.println(mapEmailPerson);

</pre>

The output will be:

<pre class="theme:dark-terminal striped:false lang:default decode:true">{steve@reversecoding.net=Person [email=steve@reversecoding.net, name=Steve, age=20], 
 mario@reversecoding.net=Person [email=mario@reversecoding.net, name=Mario, age=27], 
 luigi@reversecoding.net=Person [email=luigi@reversecoding.net, name=Luigi, age=30]}
</pre>

Or using lambda:

<pre class="lang:java decode:true" title="Using lambda">Map&lt;String, Person&gt; mapEmailPerson = 
         people.stream()				
               .collect(Collectors.toMap(person -&gt; person.getEmail(), person -&gt; person));
		
System.out.println(mapEmailPerson);</pre>

Output:

<pre class="theme:dark-terminal lang:default decode:true">{steve@reversecoding.net=Person [email=steve@reversecoding.net, name=Steve, age=20],
 mario@reversecoding.net=Person [email=mario@reversecoding.net, name=Mario, age=27], 
luigi@reversecoding.net=Person [email=luigi@reversecoding.net, name=Luigi, age=30]}
</pre>

### 

### Let&#8217;s break this out

First of all, we create a _Stream_ of _Person_ from the _List<Person>_ defined_._

Then we collect this stream in a _Map_. Java 8 helps us to define the needed _Collector_ by providing us the method: [Collectors.toMap()](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-).

Collectors.toMap() takes two functions &#8211; one for mapping the key and one for the value &#8211; and returns a `Collector` that accumulates elements into a `Map.`

Since we are working with _Stream_ of Person &#8211; our input it&#8217;s an object _Person_.

We have chosen the email as key, so that is a function that given the input &#8211; _Person_ &#8211; returns its email:

<pre class="lang:default decode:true">//Given a person, we get his email
person -&gt; person.getEmail()

//Or using method reference
Person::getEmail()

</pre>

and then the object itself as value, so it&#8217;s just an identity function:

<pre class="lang:default decode:true">//Given a person, we want as value the person itself
person -&gt; person

//Or 
Function.identity()</pre>

These are the parameters for _toMap._

### Another example

Given a _List<Person>_ we want to create a _Map<String, Integer>_ that contains the name as key and the age as value.

We just need to change the two parameters of the _Collectors.toMap_, by specifying:

<pre class="lang:default decode:true">//Name as key
person -&gt; person.getName()

//Age as value
person -&gt; person.getAge()</pre>

So the code will be:

<pre class="lang:java decode:true">List&lt;Person&gt; people = Arrays.asList(
    new Person("mario@reversecoding.net", "Mario", 27),
    new Person("luigi@reversecoding.net", "Luigi", 30),
    new Person("steve@reversecoding.net", "Steve", 20)
);

Map&lt;String, Integer&gt; mapNameAge = 
    people.stream()
          .collect(Collectors.toMap(person -&gt; person.getName(), person -&gt; person.getAge()));
		
System.out.println(mapNameAge);

</pre>

Output:

<pre class="lang:default decode:true ">{Steve=20, Luigi=30, Mario=27}</pre>

If you are a good observer you may have noticed that the order hasn&#8217;t been respected. That&#8217;s because the default implementation used by _toMap_ is the [_HashMap_](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html) that does not guarantee the order of the map.

&nbsp;

### Java 8 &#8211; Collectors.toMap with a LinkedHashMap

&nbsp;

If we want to preserve the order we should use a _[LinkedHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html) _instead of the HashMap.

Let&#8217;s try the previous example by passing a _LinkedHashMap_ to Collectors.toMap()

<pre class="lang:java decode:true ">Map&lt;String, Integer&gt; mapNameAge = 
	people.stream()
		  .collect(Collectors.toMap(
				  Person::getName, 
				  Person::getAge, 
				  (u,v) -&gt; { throw new IllegalStateException(String.format("Duplicate key %s", u)); },
				  LinkedHashMap::new
				  ));

System.out.println(mapNameAge);</pre>

Output:

<pre class="theme:dark-terminal lang:default decode:true">{Mario=27, Luigi=30, Steve=20}</pre>

We are using the definition of [toMap that takes four parameters](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html#toMap-java.util.function.Function-java.util.function.Function-java.util.function.BinaryOperator-java.util.function.Supplier-):

  * `keyMapper` &#8211; a mapping function to produce keys
  * `valueMapper` &#8211; a mapping function to produce values
  * `mergeFunction` &#8211; a merge function used to resolve collisions between values associated with the same key
  * `mapSupplier` &#8211; a function which returns a new, empty `Map` into which the results will be inserted

We&#8217;ve already discussed the first two parameters.

In case of a collision we just want to throw an exception, so as third parameter we define that. In the example, we used the same implementation of the static method _throwingMerger _defined in the java.util.stream.Collectors class.

The fourth parameter it&#8217;s the one in which we define a function that returns our _LinkedHashMap_.