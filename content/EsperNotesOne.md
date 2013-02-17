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
*Fragment and Fragment type(TODO)

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
* extract information from coarsed event: TODO(5.19)
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

