## Core Data 是什么？

Core Data是一个框架，它使得开发者能够将数据当作对象来操作，而不在乎数据在磁盘中的存储方式。

由Core Data提供的数据对象叫做托管对象（managed object）。而Core Data本身则位于应用程序和持久化存储区（persistent store）之间。

持久化存储区是个通用术语，指的是像SQLite数据库、XML文件（iOS不支持这种方式）捉着binary store(又叫atomic store，"atmic"是指，即便你只想修改少量数据，在保存的时候也依然需要把整个文件都写入磁盘)这种数据文件。还有一种In-Memory Store，并不持久存储的，也就是底层硬件重启之后，数据不会保留。但是开发者使用它管理数据的时候可以享受Core Data的所有优点，诸如变更管理和数据验证等，效率自然相当高。

为了把数据从托管对象映射到持久化存储区域，Core Data需要使用托管对象模型，可以通过对象图配置应用程序的数据结构。对象图里的对象是实体，实体用于制作自定义的托管对象。有了托管对象，就可以在OC中操作它们而无须写SQL代码了。

所有托管对象都必须位于托管对象上下文（managed object context）里面，而托管对象上下文位于高速易失性存储器中，也就是RAM中。原因之一就是在磁盘与RAM之间传输数据时会有开销。磁盘读写速度比RAM慢得多，所以不应该频繁访问它。而有了托管对象上下文之后，对于原来需要读取磁盘才能获取到的数据，现在只需访问这个上下文，就可以非常迅速地获取到了。托管对象上下文的另一个功能是记录开发者对托管对象所做的修改，以提供完整的撤销与重做支持。

Core Data 是一个模型层的技术。Core Data 帮助你建立代表程序状态的模型层。Core Data 也是一种持久化技术，它能将模型对象的状态持久化到磁盘，但它最重要的特点是：Core Data 不仅是一个加载、保存数据的框架，它还能和内存中的数据很好的共事。

还有一点要注意：Core Data 是完全独立于任何 UI 层级的框架。它是作为模型层框架被设计出来的。
下图描述了Core Data的几个核心概念：
![Core Data 概览](/assets/01-core data核心概念.png)

## 持久化存储协调器
持久化存储协调器（persistentstore coordinator）里面包含一个或多个持久化存储区，但是多个存储区中的数据之间不能建立“关系”。持久化存储区可以包含Bi-nary、XML或In-Memory等形式的持久化存储区。，Binary和XML格式的存储区是“原子的”（atomic），也就是说，即便你只想修改少量数据，在保存的时候也依然需要把整个文件都写入磁盘。与原子存储不同，SQLite数据库会在用户提交变更日志时进行增量更新，变更日志也叫做事务日志。由于采用了这种更新方式，所以SQLite数据库的内存占用量相对来说非常小。

要想创建持久化存储区，需生成NSPersis-tentStore类的实例；要想创建持久化存储协调器，需生成NSPersistentStoreCoordinator类的实例。

## 托管对象模型
托管对象模型，它位于持久化存储协调器和托管对象上下文之间。顾名思义，托管对象模型是描述数据结构的模型或图示（graph-ical representation），而托管对象正是以它为基础产生出来的。

要想创建托管对象模型，需生成NSManage-dObjectModel类的实例。

托管对象模型由一系列实体描述对象构成，这种对象就叫做实体。实体，用于创建托管对象。托管对象模型可以拥有一个或多个实体，每个应用程序的实体数量会有所差别。有了托管对象之后，我们就可以用Objective-C代码来操作其中的数据了。

#### 设计实体的步骤
而在设计实体时，你需要做的则是：
* 配置实体名称（entity name）。
* 配置属性，并为每个属性设定数据类型（属性必须有特定的数据类型）。
* 根据实体来配置NSManagedObject的子类（该项可选）。

如果想从实体中创建托管对象，那我们通常会根据实体来创建NSManagedObject的子类，但这并不是强制性的。采用NSManagedObject的子类确实有好处，比如可以在托管对象后面使用“点符号”（.）访问相关属性，这样可以令代码更易阅读。无论是从NSManagedObject类还是从NSManagedObject的子类创建实例，我们都可以通过托管对象这种形式来操作数据。用数据库领域的术语来说，托管对象的实例类似于数据库中某张数据表里的一行（row）。实体的名称与根据该实体创建出来的NSManagedObject子类的名称通常是一样的。根据实体来创建托管对象时，在实体中配置好的那些属性也会变成托管对象里的特性。

持久化存储区中的数据库与托管对象之间建立映射关系：
![](/assets/02-core data数据库和托管对象映射关系.png)

#### 实体命名
给实体所起的名字应该由一两个英文单词构成，它要能描述出实体所表示的数据。：为实体起的名字需要稍微通用一些，以便适应将来的变化，同时还必须足够具体，以便明确描述出它所表示的数据。

#### 属性命名
属性（attribute）是实体的特征（property）。属性的名称必须以小写字母开头，而且不应该与NSObject或NSManagedObject方法重名。Xcode不允许开发者违背这条规则，如果违背了，它会给出警告，比方说，把实体的属性名设为“description”就是非法的。

#### 实体的属性与托管对象的特性的对应关系
* 实体中的Date属性会成为类里的NSDate特性。
* 实体中的String属性会成为类里的NSString特性。
* 实体中的Decimal属性会成为类里的NS-DecimalNumber特性，而其他各种数值数据类型会成为类里的NSNumber特性。
* 实体中的Binary Data属性会成为类里的NSData特性。
* 实体中的Tranformable属性，会成为类里的id特性。

#### 托管对象的类文件
托管对象类的实现文件只是给各个特性都添加了@dynamic修饰符而已。Core Data用这种方式告诉大家：获取及设置特性值所需的方法都会动态地生成，不用开发者自己去实现。

## 托管对象上下文
托管对象上下文，其中包含多个托管对象。托管对象上下文负责管理其中对象的生命期（lifecycle），并且负责提供许多强大的功能，诸如faulting、变更追踪（change track-ing）、验证（validation）等。托管对象上下文也可以不止一个。

要想创建托管对象上下文，需生成NSMan-agedObjectContext类的实例。


## Core Data适用场合
如果应用程序要保存的设置数据太多，以致NSUserDefaults及“特性列表”（property list）这种简单的存储方案无法应付，那么就会出现内存占用量方面的问题。解决办法是直接使用数据库或通过Core Data来间接操作数据库。选用CoreData的好处是，不用再花时间编写数据库接口的代码了。此外，你还将享受性能方面的优势，而且可以使用诸如撤销及验证等强大的功能


## Core Data后端SQL的可见性
开启SQL Debug调试选项可以提供足够的信息，告诉你这些操作背后所发生的事情，从而令你知道上述那些问题的答案。这个调试选项会把系统自动生成的SQL查询语句打印出来，使开发者深刻认识到Core Data的工作原理。
1. 点击Product＞Scheme＞Edit Scheme...菜单项。
2. 点击Run Grocery Dude，并切换到Argu-ments分页。
3. 点击Arguments Passed On Launch区域中的“+”按钮，以新增参数。
4. 输入新参数-com.apple.Core-Data.SQLDebug 3，然后点击OK按钮。

## Core Data数据操作

### Core Data插入数据
新对象是由NSEntity-Description按照指定的名称并根据某个特定的实体而创建出来的。除了要指定对象所依据的实体之外，还需提供指向托管对象上下文的指针，创建好的托管对象将会放在那个上下文里面。

```objective-c
[NSEntityDescription insertNewObjectForEntityForName: inManagedObjectContext:];
```
### Core Data查询数据

```
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Item"];
     // 使用NSSortDescriptor为查询结果排序，查询结果按照名字字母升序排列
    NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES];
    [request setSortDescriptors:@[sortDescriptor]];
    
    // 记录查询过程中出现的错误
    NSError *error = nil;
    NSArray<Item *> *managedObjs = [self.coreDataHelper.context executeFetchRequest:request error:&error];
    
    // 查询结果过滤
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name beginswith 'M'"];
    [managedObjs filteredArrayUsingPredicate:predicate];
```

### Core Data查询数据模板

假如每次获取托管对象时都要手工编写谓词格式确实很累人，幸好Xcode的Data Model De-signer有预定义获取请求的功能。这些可复用的模板比谓词更容易配置，而且还能减少重复代码。只需根据应用程序的模型来操作一系列下拉列表框及文本框，即可配置好一份获取请求模板。但如果要自定义AND、OR这样的逻辑组合，那么这个模板就无法满足要求了，此时仍然需要通过代码来指定谓词。

```
    // 从托管对象模型中的请求模板获取数据查询请求对象
    NSFetchRequest *request = [[NSManagedObjectModel fetchRequestTemplateForName:@"Test"] copy];
    // 查询结果按照名字字母升序排列
    NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES];
    [request setSortDescriptors:@[sortDescriptor]];
    // 记录查询过程中出现的错误
    NSError *error = nil;
    NSArray *managedObjs = [self.coreDataHelper.context executeFetchRequest:request error:&error];
```




### Core Data删除数据
若想删除托管对象，只需在包含该对象的上下文中调用deleteObject或deleteObjects即可。请注意，此时对象并未永久删除，必须调用上下文的save：方法才能将其永久删去。
```objective-c
 [NSManagedObjectContext deleteObject:];
```




















