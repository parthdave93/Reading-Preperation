# Java Concepts

#### OOPS Concept

OPPS concepts are abstraction, encapsulation, inheritance, polymorphism.
Abstraction is defined as simple things to represent complexity, like mobile phones operating system. You know how to get things done using phone but you dont know how OS or hardware itself works that simply is an abstraction.
Encapsulation is defined as wrapping up the data in a single unit or entity known as class. Class itself represent encapsulation because it holds fields and methods in a single unit.
Inheritance is a best feature of programming where you're allowed to use other class functionality using inheriting them. Like a real life you inherit all property and DNA of your parents.
Polymorphism is defined as one entity have different meanings or you can use it under different context. polymorphism can be done using method overloading and method overriding. Method overloading means you will be adding extra parameter to the function already defined. like:
```Java
void functionOverload(int a){
    //.... code
}
void functionOverload(int a , int b){
    //.... code
}
```
Things to note here in method overloading means method will having same name and same return types with different range of paramters.

---

#
#### Access modifiers

Kotlin has a little different set of modifiers than java.
Java has private, protected, public,default(package-private).
Kotlin has private,protected, public , internal.
Default modifier for java is default i.e. package private means that any class that is inside that package can able to acceess the class defiend, while in kotlin it;s by default public any one can access it anywhere.
Quesions might rise are:

| Questions | links |
| ---- | ---- |
| 1 |  [Kotlin-Visibility-Modifiers] - for more details | 
| 2 | [Why not able to use protected modifier for top level class?] - or sometimes interviewer asks about what are the top level access modifiers by all means it is default, private, public. |

---

#
#### Interfaces

Interfaces are delightful structure for the developers to help multiple languages to support multilevel inheritence. Like in java if we need to have a class inherit two classes it's not possible due to ambiguous operation and so we have interface to rescue.
All fields in interface are ```public static final``` i.e. they are constants , and so it wont change.
Interviewer might ask you what will be the output of following code:
```Java
interface A {
    int a = 0;
    void increment();
}
class AImpl implements A {
    public void increment() {
        a++;
        print(a);
    }
}
```
There will be no output as above code will not get compiled. As we have stated before interface variables are by default public static final and you can not change final variable and so above code will show compilation error.

for default functions they might ask you this:
```Java
public class TestCase {
    public static void main(String args[]){
        AB ab = new AB();
        ab.samePrintMethod();
    }
}

interface A{
    default void samePrintMethod(){
        System.out.println("test class A");
    }
}

interface B {
    default void samePrintMethod(){
        System.out.println("test class B");
    }
}

class AB implements A,B{
}
```
Does above code will compile? and answer is no because there are two interfaces with the same default function which will result in compile time ambiguity as compiler will not be able to understand which method will needs to be called. but after updating AB class like below:
```Java
class AB implements A,B{
    @Override
    public void samePrintMethod() {
        System.out.println("AB class");
    }
}
```
So question might rise then how can you call individual interface default methods now? and answer is simple use super keyword.
```A.super.samePrintMethod();``` and ```B.super.samePrintMethod();``` for calling individual methods of same class.

Quesions might rise are:

| Questions | links |
| ---- | ---- |
| 1 | [Why java does not support multiple inheritence] - for more details |
| 2 | [default function in interface] - although java default functions where introduced in Java8 still it is worth to learn for mobile developers as there might time comes to introduce new function in interface without breaking old implemented code. |

---

#
#### Casting
Many times you have casted classes or objects or even cast has been done by compiler itself in order to give you best result. Casting is kind of telling machine that I know you're returning something but it's this type and I'm pretty sure of it so please change it to this type.
There's only one rule in casting is that each individual needs to share same hierarchy.
Basically there are two types of type casting
 - Primitive Casting: generally done by compiler or you can do it your self by casting int,float,double etc type of primitive types.
 - Derived Casting: casting from one derived type to another.

There's also few terminology that we need to understand :
Auto widening, explicit narrowing, boxing - unboxing, auto-up, explicit down. 
Auto widening is casting small type to big types like byte to short, short to int, int to long, long to float and float to double. In explicit narrowing we are doing oppositive of Auto widening. i.e. explicitly convert the data from double to float, float to long, long to int, int to short and short to byte.
Auto-Boxing, automatic convertion between primitive to wrapper class by means. Java compiler automatically box the int to an Integer, a double to a Double, and so on. If the conversion goes the other way, this is called unboxing.

| Questions | links |
| ---- | ---- |
| 1 | Auto Box - [Auto-boxing] |
| 2 | auto-up, explicit down - [AutoUp and down cast]|

Remember:
Casting does not change the actual object type. Only the reference type gets changed.
Upcasting is always safe and never fails.
Downcasting can risk throwing a ClassCastException, so the instanceof operator is used to check type before casting.

---

#
#### Collections

Collection defiend as group of objects represented as a single unit. There's two main interface or root interface in collection which is Map and Collection. 

Hierarchy of collection framework(Core interfaces)

| Parent | Child |
| --- | --- |
| Collection | Set, List, Queue, Dequeue |
| Map | Sorted Map |

Basic understandings:
 - Collection interface having basic method like add, addAll, remove, clear, contains etc.
 - Set: It does not allow duplication. Implementation provided by frameworks are HashSet and TreeSet.
 - List: It allows duplications and elements are ordered. basic examples are ArrayList and LinkedList.
 - Queue: It allows elements to enter/remove the stack in FIFO order i.e. First in first out except PriorityQueue.
- Dequeue: This on other hand allows elements to be entered/removed from both ends.
- Map: This is different from collection and so it is not extending collection interface but it has it's own. Which is used for key value pair. It does not allow duplicates. examples are HashMap and TreeMap.
- The difference between Set and Map interface is that in Set we 
have keys, whereas in Map, we have key, value pairs.

Note here collection framework has parent Iterator interface which is usually used by us for iterating over ArrayList or any other collection and updating at the same time. Also there is another thing to note here is iterator only moves forward.
 
---

List interface is a child interface of Collection interface which inhibits ordered collection with duplication.
ArrayList, Vector, Stack, LinkedList are implementation of List interface.

---

## ArrayList

ArrayList -> Abstract List -> List -> Collection -> Iterable

Java ArrayList uses dynamic array for storing elements. It's basically an array[] datatypes with some perks. 




### Questions and answers

| Question | Answer |
| --- | --- |
|What are the ways we can iterate over ArrayList? | There are many ways that you can iterate like for loop, forEach loop, forEachRemaining loop, using iterator|
| What is difference between forEach loop and forEachRemaining loop? | The basic difference lies in the word itself. Let's take an example, If I have arrayList having ```[1,2,3,4,5]``` and if I wrote a code like ```list.forEach(System.out::print)``` prints the array like 12345. If I use forEachRemaining it will still print same output, But but the difference is when we use iterator and call ```iterator.next()``` for 2,3 times that elements will be skipped for forEachRemaning loop because we already been iterated over it.|
| What retainAll method in ArrayList is used for? | It retains elements of other ArrayList and removes all other like union set(ref Mathematics). Take an example: If we have array ```a1 = [1,2,3,4,5]``` and ```a2 = [2,9]``` and if we call ```a1.retainAll(a2)``` the a1 will have union of both ArrayList which is ```a1 = [2]```|
| Default capacity of arrayList? | 10 |
| How much size ArrayList will increase if size is full? | Uptil Java6: ```(currentSize * 3/2) + 1```, In Java7 it will grow ```50%```. here is the code of determining the size of ArrayList ```int newCapacity = oldCapacity + (oldCapacity >> 1);``` from JDK itself [ArrayList Capacity Ref JDK7] check grow method,[ArrayList Capacity Ref JDK6] check ensureCapacity method. What >> operator or we can call it bitwise operator will do is it calculates for 50% of the given value so if we do this ```10>>1``` it's going to give us ```5``` in return.|
| What arrayList add and remove method internally does for array[]? | When we add an element to the ArrayList first it checks if size is full or not, if it's full then increase the size of ArrayList and move all elements to new location. If it's not full then it adds element at location just as it is. Although when we remove element from the ArrayList the gap has been created and it need to defragment the location so it shifts all elements to the right to left size by 1 (i.e. if we removed only one element.) [How arraylist internally works].|




### Reference Links
1. [basic understanding of arraylist code]
2. [ArrayList Capacity Ref JDK7]
3. [ArrayList Capacity Ref JDK6]
4. [How arraylist internally works]


---


## LinkedList

LinkedList implements List and Deque interface. Linked List is basically about the nodes. Node will be having 3 data to hold which is previous node, next node and element it self.(i.e. data/value).
This is quite different than array list because ArrayList works with the array[] manipulation in background while LinkedList works like dequeue in background.

Dequeue is like queue with two way traversal,which adds/removes elements from both the sides. 

And by doing so LinkedList has no mechanism of capacity like ArrayList.



### Questions and answers

| Question | Answer |
| --- | --- |
| Why get(index) method in LinkedList? | Yes there is a method to get element at index from LinkedList. The question is why? I mean arrayList works with array[] so it's fine, here we have nodes which is interconnected with the prev and next nodes so why? Answer lies in the method's code. Once think we think about getting middle or any other element like second last element from linkList we traverse through all the elements and when our count is exact the index we passed return the element correct? Well that's not the optimal solution. So JDK guys thought about the better way of getting element from LinkedList which is nothing but dividing LinkedList in two parts and traverse from first to mid if index < size of linked list mid point else traverse other-way. [LinkedList source code] check get(index) method.|
| How to find mid element in one pass? | Remember this is the algorithmic question not about method as question mentioned one pass. We will be needing two indexes, one will get incremented by 2 while other will be incremented by 1. Well the reason is when we increment index pointer by 2 it represents as double the index and hence if any element node of linked list will not contain the next element it will be the last node and so we will get mid element by slow pointer which has been incremented by 1.|
| Difference between singly and doubly linked list in Java | Singly linked list contains only next node while doubly link list contains both prev and next node. |

### Reference Links

[How linkedList internally works]


---


## HashMap

To learn about HashSet we need to learn about HashMap first, as HashSet uses HashMap internally. HashMap works using hashing mechanism. HashMap uses key, value pair to store the data. HashMap allows null as key.

When we call ```hashmap.put(key, value)``` the steps it does is like this:
1. Get the HashCode of key
2. find bucket for hashcode
1. Check if bucket is already having that hashcode?
   - If it exists ([Hashing Collision])
     - Try to find old node using key equals method if we found the Node then update it otherwise add new node with the key and value in that bucket.
   - If it does not exists, create a LinkedList and add element as first node.

[Hash Collision with example] - check for more details about Hash collision in hash map
And due to this reason it is very best to override equals method to find element in your HashMap.

### Reference Links:

1. [HashMap internals]
2. [Hash Collision with example]
3. [HashMap source code]

---



















[ArrayList Capacity Ref JDK7]:<http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/00cd9dc3c2b5/src/share/classes/java/util/ArrayList.java>
[ArrayList Capacity Ref JDK6]:<http://hg.openjdk.java.net/jdk6/jdk6/jdk/file/e0e25ac28560/src/share/classes/java/util/ArrayList.java>
[basic understanding of arraylist code]:<https://netjs.blogspot.com/2015/08/how-arraylist-works-internally-in-java.html>
[How arraylist internally works]:<https://codenuclear.com/how-arraylist-works-internally-java/>
[How linkedList internally works]:<https://netjs.blogspot.com/2015/08/how-linked-list-class-works-internally-java.html/>
[LinkedList source code]:<http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/java/util/LinkedList.java>
[HashMap internals]:<https://netjs.blogspot.com/2015/05/how-hashmap-internally-works-in-java.html>
[HashSet source code]:<http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/classes/java/util/HashSet.java>
[Hashing Collision]:<https://netjs.blogspot.com/2015/05/how-hashmap-internally-works-in-java.html>
[Hash Collision with example]:<https://www.geeksforgeeks.org/internal-working-of-hashmap-java/>
[HashMap source code]:<http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/java/util/HashMap.java>


[Kotlin-Visibility-Modifiers]: <https://medium.com/@parthdave93/modifiers-of-the-new-world-of-android-a42bba59b035>
[Why not able to use protected modifier for top level class?]: <https://stackoverflow.com/questions/3869556/why-can-a-class-not-be-defined-as-protected>
[Why java does not support multiple inheritence]: <https://www.geeksforgeeks.org/java-and-multiple-inheritance/>
[default function in interface]: <https://dzone.com/articles/interface-default-methods-java>
[Auto-boxing]: <https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html>
[AutoUp and down cast]:<https://www.codejava.net/java-core/the-java-language/what-is-upcasting-and-downcasting-in-java>
