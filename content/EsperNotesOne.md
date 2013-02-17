Title: Esper Notes One
Date: 2013-2-17 10:34
Category: Data
Slug: Esper Notes One
Author: Lu Wen
Tags: Esper, Big Data, Java

##Introduction##
* epser:tranditional db top-down, store the query, send the data
* **two basic funciton**: event pattern language; event stream queries(like SQL)
* dependency: antlr(parse), cglib(fast method call), jcl(logging)

##Event Representation##
* define: predefine, runtime(api and EPL)
* major types: pojo, map, object[], xml
* event properies: simple, indexed(name[1]), mapped(name("a")), nested
* escape: "\.", "``"
* []: index can only be constant integer, no expression
* Dynamic properties
* may don't know all the fieds in the compile time
* name?, name[1]?, name("aa")?, name?.nested
* always return "Object", null if there is none
* it's transitive, the nested properies will all be dynamic if the ancestor is
* operator: exists, cast, instanceof, typeof
*Fragment and Fragment type(**TODO**)

###pojo event###
* Configuration, if not standard java bean
* Index: Array or Iterable
* Constant and Enumeration values can be used
* class static methods and instance member funcitons can be used. Use the full name, or import package through configuration
```java
select * from MyEvent where enumProp=EnumClass.valueOf('ENUM_VALUE_1')
select myevent.computeSomething() as result from MyEvent as myevent
```
* parameterized types can be used
* setter methods can be used for update
* default to generate the bytecode, if fails, use reflection(slow)

###map event###
* at the run time, can add properites using 'updateMapEventType', not be updated or deleted; but can remove the whole event and then add back
* map super types: Add inheritance concept, support multiple inheritance
```java
epService.getEPAdministrator().getConfiguration().
    addEventType("AccountUpdate", accountUpdateDef,
    new String[] {"BaseUpdate"});
```
* nested map event
```java
Map<String, Object> updatedFieldDef = new HashMap<String, Object>();
epService.getEPAdministrator().getConfiguration().
    addEventType("UpdatedFieldType", updatedFieldDef);
Map<String, Object> accountUpdateDef = new HashMap<String, Object>();
accountUpdateDef.put("fields", "UpdatedFieldType");
// the latter can also be:  accountUpdateDef.put("fields", updatedFieldDef);
epService.getEPAdministrator().getConfiguration().
    addEventType("AccountUpdate", accountUpdateDef);
```
 * If the nested map event is an array, use `sale.put("items",  "OrderItem[]");`

###object array event###
* fastest
* advanced nested types, similar to map: 
```java
Object[] propertyTypesAccountUpdate = {long.class, "UpdatedFieldType"};
String[] propertyNames = {"userids", "salesPersons", "items"};
Object[] propertyTypes = {int[].class, SalesPerson[].class, "OrderItem[]");
```

###Map Events###

###Other concept###
* Updating and merging events, (update istream, on-merge or on-update)
* extract information from coarsed event: **TODO*8(5.19)
* `insert into` an event, the event must have setter methods, or relative construction methods, or a factory method through configuration

##Processing model##
* Use a UpdateListener for results
* the underlying objects for the result of join is `Java.util.map`
* By  default  the  engine  only  delivers  the  insert  stream  to  listeners  and  observers.  EPL supports optional istream, irstream and rstream keywords 
* event will be put into `old` array only it satisfies the where clause
* aggregation and groups
* we can define the visibility for service isolation and the esper time clock.

##Context and context partitions##
* A context takes a cloud of events and classifies them into one or more sets. These sets are called context partitions. An event processing operation that is associated with a context operates on each of these context partitions independently. 
* Main usage:
 * increase partitions for concurrency/lock optimization
 * use condition/pattern to dynamiclly identify specific circumstances
 * these can be shared by multiple statements
* context exists when there is statements referencing it
* context provides properties for use. Every statement will work on partitions independently.

###Keyed Segmented Context###
```java
create context SegmentedByCustomer partition by custId from BankTxn
create  context  SegmentedByCustomer  partition  by  custId  from  BankTxn,  account  from BankTxn 
```
* if context is from multiple streams, the keys must correspond to each other, like the same concept in different events(maybe different name)
* can use filters to control events dlivered to context 
* If using join, the join only occurs in the partition

###Hash Segmented Context###
```java
create context SegmentedByCustomerHash
    coalesce  by  consistent_hash_crc32(custId)  from  BankTxn  granularity  16
preallocate
```
* hash method: crc32, java hash code, user provide
* preallocate: use memory for time(if not preallocate, it needs dynamic check)
* performance concerns for the granularity

###Category Segmented Context###
```java
create context CategoryByTemp
  group temp < 65 as cold,
  group temp between 65 and 85 as normal,
  group temp > 85 as large
  from SensorEvent
```

###Non overlapping context###
* non-overlapping context that exists once or that repeats in a regular fashion as controlled by start and end conditions. The number of context partitions is always either one or zero: Context partitions do not overlap
* Once  the  start  condition  occurs,  the  engine  no  longer  observes  the  start  condition  and  begins observing  the  end  condition.  Once  the  end  condition  occurs,  the  engine  observes  the  start condition again
* condition can be an event filter, a pattern, a crontab or a time period.
```java
create context NineToFive start (0, 9, *, *, *) end (0, 17, *, *, *)
create context PowerOutage start PowerOutageEvent end pattern [PowerOnEvent -> timer:interval(5)]
```

###Overlapping context###
* This context initiates a new context partition when an initiating condition occurs, and terminates one  or  more  context  partitions  when  the  terminating  condition  occurs
```java
create context CtxTrainEnter
  initiated by TrainEnterEvent as te
  terminated after 5 minutes
```

* Context nesting
```java
create context NineToFiveSegmented
  context NineToFive start (0, 9, *, *, *) end (0, 17, *, *, *),
  context SegmentedByCustomer partition by custId from BankTxn
```
Only initial context meets, there will be the subcontext
* Partitioning Without Context Declaration
 * grouped data window
 * group aggration
 * pattern
 * match recognize
 * join and subquery
* output when terminate
```java
context CtxSample 
select context.startevent.id, context.endevent.id, count(*) from MyEvent
output snapshot when terminated
```
* context can be used to named window(**TODO**)
* **TODO**
  * Iterations on specific contexts
  * on demand query to do on specific contexts
