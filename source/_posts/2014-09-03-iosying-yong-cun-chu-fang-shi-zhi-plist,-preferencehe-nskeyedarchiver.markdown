---
layout: post
title: "iOS应用存储方式之Plist、Preference和NSKeyedArchiver"
date: 2014-09-03 21:57:05 +0800
comments: true
categories: iOS
---
### iOS应用数据存储方式

* XML属性列表（plist）归档

* Preference（偏好设置）

* NSkeyedArchiver归档(NSCoding)

* SQLite3

* Core Data

<!-- more -->

##### 应用沙盒

每个iOS应用都有自己的应用沙盒(文件系统目录),与其他文件系统隔离。应用必须待在自己的沙盒里，其他应用不能访问该沙盒。

* Documents: 保存应用程序运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录。例如，游戏应用可将游戏归档保存在该目录。
* tmp: 保存应用运行时所需的临时数据，使用完毕后再将相应的文件从该目录删除。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时不会同步该目录。
* Library/Caches: 保存应用运行时生成的需要持久化的数据，iTunes同步设备时不会备份该目录。一般存储体积大，不需要备份的非重要数据。
* Library/Preference: 保存应用所有的偏好设置，iOS的Settings(设置)应用会在该目录中查找应用的设置的设置信息。iTunes同步设备时会备份该目录。

##### 应用沙盒的常见获取方式

沙盒根目录：

	NSString *home = NSHomeDirectory();
	
**Documents:(2种方式)**

一. 利用沙盒根目录拼接"Documents"字符串
	
	NSString *home = NSHomeDirectory();
	NSString *documents = [home stringByAppendingPathComponent:@"Documents"];
	// 不建议，新版本操作系统可能会修改该目录名
	
二. 利用NSSearchPathForDirectoriesInDomain函数

	// NSUserDomainMask 代表从用户文件夹下找
	// YES 代表展开路径中的波浪字符“~”
	NSArray *array =  NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, NO);
	// 在iOS中，只有一个目录跟传入的参数匹配，所以这个集合里面只有一个元素
	NSString *documents = [array objectAtIndex:0];

**tmp: NSString \*tmp = NSTempraryDirectory()**

**Library/Caches:(跟Documents类似两种方法)**

**Library/Preference: 通过NSUserDefaults类获取该目下的设置信息**

##### XML属性列表（plist）归档

属性列表是一种XML格式的文件。扩展名为plist

如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，就可以使用writeToFile:actomaiclly:方法直接将对象写到属性列表中。例如：

	// 将数据封装成字典
	NSMutableDictionary *dict = [NSMutableDictionary dictionary];
	[dict setObject:@"母鸡" forKey:@"name"];
	[dict setObject:@"15013141314" forKey:@"phone"];
	[dict setObject:@"27" forKey:@"age"];
	// 将字典持久化到Documents/stu.plist文件中
	[dict writeToFile:path atomically:YES];


读取属性列表：

	// 读取Documents/stu.plist的内容，实例化NSDictionary
	NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:path];
	NSLog(@"name:%@", [dict objectForKey:@"name"]);
	NSLog(@"phone:%@", [dict objectForKey:@"phone"]);
	NSLog(@"age:%@", [dict objectForKey:@"age"]);
	
##### Preference（偏好设置）

* 很多iOS应用都支持偏好设置，比如保存用户名、密码、字体大小等设置，iOS提供了一套标准的解决方案来为应用加入偏好设置功能
* 每个应用都有个NSUserDefaults实例，通过它来存取偏好设置
* 比如，保存用户名、字体大小、是否自动登录

--

	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	[defaults setObject:@"itcast" forKey:@"username"];
	[defaults setFloat:18.0f forKey:@"text_size"];
	[defaults setBool:YES forKey:@"auto_login"];

* 读取上次保存的设置

--

	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	NSString *username = [defaults stringForKey:@"username"];
	float textSize = [defaults floatForKey:@"text_size"];
	BOOL autoLogin = [defaults boolForKey:@"auto_login"];

**注意**

UserDefaults设置数据时，不是立即写入，而是根据时间戳定时地把缓存中的的数据写入本地磁盘。所以调用了set方法之后数据有可能还没有写入磁盘应用程序就终止了。可以通过调用`synchornize`方法强制写入：

	[defaults synchornize];

##### NSkeyedArchiver

* 如果对象是NSString、NSDictionary、NSArray、NSData、NSNumber等类型，可以直接用NSKeyedArchiver进行归档和恢复。
* 不是所有的对象都可以直接使用这种方法进行归档，只有遵守了NSCoding协议的对象才可以。

NSCoding协议有两个方法:

**encodeWithCoder:**

每次归档对象时，都会调用这个方法。一般在这个方法里面指定如何归档对象中的实例变量，可以使用`encodeObject:forKey:`方法归档实例变量。

**initWithCoder**

每次从文件解码对象时，都会调用这个方法。一般在这个方法里面指定如何解码文件中的数据为对象的实例变量，可以使用`decodeObjectForKey:`方法解码实例变量。

示例：

	// 归档
	Person *person = [[Person alloc] init];
	person.name = @"Lily";
	person.age = 27;
	person.height = 1.83f;
	[NSKeyedArchiver archiveRootObject:person toFile:path];
	
	// 解码
	Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
	
对于Person.m文件:

	@implementation Person
	
	- (void)encodeWithCoder:(NSCoder *)encoder {
	    [encoder encodeObject:self.name forKey:@"name"];
	    [encoder encodeInt:self.age forKey:@"age"];
	    [encoder encodeFloat:self.height forKey:@"height"];
	}
	- (id)initWithCoder:(NSCoder *)decoder {
	    self.name = [decoder decodeObjectForKey:@"name"];
	    self.age = [decoder decodeIntForKey:@"age"];
	    self.height = [decoder decodeFloatForKey:@"height"];
	    return self;
	}
	@end

**NSKeyedArchiver归档对象注意:**

如果父类也遵守了NSCoding协议，应该在`encodeWithCoder:`方法加上一句`[super encodeWithCode:encode]`，确保继承的实例变量也能被编码，即也能被归档。应该在`initWithCoder:`方法加上:`self = [super initWithCoder:decoder]`确保集成的实例变量也能被解码。

###### 归档NSData

使用`archiveRootObject:toFile:`方法可以将一个对象直接写入到一个文件中，但有时候可能想将多个对象写入到同一个文件中，那么就使用NSData来进行归档对象。

NSData可以为一些数据提供临时的存储空间，以便以后写入文件，或者存放从磁盘文件读取的内容。可以使用`[NSMutableData data]`创建可变数据空间。

示例：
	
	// 归档
	// 新建一块可变数据区
	NSMutableData *data = [NSMutableData data];
	// 将数据区连接到一个NSKeyedArchiver对象
	NSKeyedArchiver *archiver = [[[NSKeyedArchiver alloc] initForWritingWithMutableData:data] autorelease];
	// 开始存档对象，存档的数据都会存储到NSMutableData中
	[archiver encodeObject:person1 forKey:@"person1"];
	[archiver encodeObject:person2 forKey:@"person2"];
	// 存档完毕(一定要调用这个方法)
	[archiver finishEncoding];
	// 将存档的数据写入文件
	[data writeToFile:path atomically:YES];
	
	// 解码
	// 从文件中读取数据
	NSData *data = [NSData dataWithContentsOfFile:path];
	// 根据数据，解析成一个NSKeyedUnarchiver对象
	NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
	Person *person1 = [unarchiver decodeObjectForKey:@"person1"];
	Person *person2 = [unarchiver decodeObjectForKey:@"person2"];
	// 恢复完毕
	[unarchiver finishDecoding];

##### 利用归档可以实现深复制

比如对一个Person对象进行深复制

	// 临时存储person1的数据
	NSData *data = [NSKeyedArchiver archivedDataWithRootObject:person1];
	// 解析data，生成一个新的Person对象
	Student *person2 = [NSKeyedUnarchiver unarchiveObjectWithData:data];
	// 分别打印内存地址
	NSLog(@"person1:0x%x", person1); // person1:0x7177a60
	NSLog(@"person2:0x%x", person2); // person2:0x7177cf0
