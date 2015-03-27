---
layout: post
title: Objective-C学习笔记
category: Objective-C
tags: [Objective-C]
---


方法调用
NSString* myString = [NSString string];
调用了NSString类的string方法，所有的Objective-C对象变量都是指针类型的。
（无论什么时候我们见到方括号，其实我们都是向一个对象或者一个类发送了一个消息。）

id类型
id类型已经预先被定义成一个指针类型了。所以我们不需要再加星号。
id myObject = [NSString string]; 
id的类型意味着myObject这个变量可以指向任意类型的变量。

嵌套调用
[NSString stringWithFormat:[prefs format]];

输入多参数
定义-(BOOL)writeToFile:(NSString *)path atomically:(BOOL)useAuxiliaryFile;  
调用BOOL result = [myData writeToFile:@"/tmp/log.txt" atomically:NO];  

创建对象
NSString* myString = [[NSString alloc] init];

内存管理
假如我们手动的通过alloc创建了一个对象，我们需要用完这个对象后release它。
// must release this when done  
 
NSString* string2 = [[NSString alloc] init];  
 
[string2 release];  
但我们不需要手动的去release一个autoreleased类型的对象，假如真的这样去做的话，我们的应用程序将会crash。
// string1 will be released automatically  
 
NSString* string1 = [NSString string]; 

Photo.h
#import  
 
@interface Photo : NSObject {  
 
NSString* caption;  
 
NSString* photographer;  
 
}  
 
- (NSString*) caption;  
 
- (NSString*) photographer;  
 
- (void) setCaption: (NSString*)input;  
 
- (void) setPhotographer: (NSString*)input;  
 
@end  
把Cocoa.h import进来。
@interface符号表明这是Photo类的声明。
冒号指定了父类。
不需要加get前缀。一个单独小横杆表明它是一个实例的方法。
Setters不需要返回任何值，所以我们把它的类型指定为void.

Photo.m
#import "Photo.h"  
 
@implementation Photo  
 
- (NSString*) caption {  
 
return caption;  
 
}  
 
- (NSString*) photographer {  
 
return photographer;  
 
}  
 
@end  
由@implementation再来加上类名


http://my.oschina.net/maomi/blog/72838
这个教程是最好的！就看这个就够了！

BOOL 在 Objective-C 中有兩種型態：YES 或 NO

編譯 hello world


hello.m

#import <stdio.h>

int main( int argc, const char *argv[] ) {
    printf( "hello world\n" );
    return 0;
}
輸出

hello world
在 Objective-C 中使用 #import 代替 #include

Objective-C 的預設副檔名是 .m


創建 classes

@interface

Fraction.h

#import <Foundation/NSObject.h>@interface Fraction: NSObject {
    int numerator;
    int denominator;
}

-(void) print;
-(void) setNumerator: (int) n;
-(void) setDenominator: (int) d;
-(int) numerator;
-(int) denominator;@end
NSObject：NeXTStep Object 的縮寫。因為它已經改名為 OpenStep，所以這在今天已經不是那麼有意義了。

繼承（inheritance）以 Class: Parent 表示，就像上面的 Fraction: NSObject。

夾在 @interface Class: Parent { .... } 中的稱為 instance variables。

沒有設定存取權限（protected, public, private）時，預設的存取權限為 protected。

Instance methods 跟在成員變數（即 instance variables）後。格式為：scope (returnType) methodName: (parameter1Type) parameter1Name;

scope 有class 或 instance 兩種。instance methods 以 - 開頭，class level methods 以 + 開頭。

Interface 以一個 @end 作為結束。


@implementation

Fraction.m

#import "Fraction.h"
#import <stdio.h>

@implementation Fraction
-(void) print {
    printf( "%i/%i", numerator, denominator );
}

-(void) setNumerator: (int) n {
    numerator = n;
}

-(void) setDenominator: (int) d {
    denominator = d;
}

-(int) denominator {
    return denominator;
}

-(int) numerator {
    return numerator;
}@end
Implementation 以 @implementation ClassName 開始，以 @end 結束。

Implement 定義好的 methods 的方式，跟在 interface 中宣告時很近似。


把它們湊在一起

main.m

#import <stdio.h>
#import "Fraction.h"

int main( int argc, const char *argv[] ) {
    // create a new instance
    Fraction *frac = [[Fraction alloc] init];

    // set the values
    [frac setNumerator: 1];
    [frac setDenominator: 3];

    // print it
    printf( "The fraction is: " );
    [frac print];
    printf( "\n" );

    // free memory
    [frac release];

    return 0;
}
output

The fraction is: 1/3
Fraction *frac = [[Fraction alloc] init];

這行程式碼中有很多重要的東西。

在 Objective-C 中呼叫 methods 的方法是 [object method]，就像 C++ 的 object->method()。

Objective-C 沒有 value 型別。所以沒有像 C++ 的 Fraction frac; frac.print(); 這類的東西。在 Objective-C 中完全使用指標來處理物件。

這行程式碼實際上做了兩件事： [Fraction alloc] 呼叫了 Fraction class 的 alloc method。這就像 malloc 記憶體，這個動作也做了一樣的事情。

[object init] 是一個建構子（constructor）呼叫，負責初始化物件中的所有變數。它呼叫了 [Fraction alloc] 傳回的 instance 上的 init method。這個動作非常普遍，所以通常以一行程式完成：Object *var = [[Object alloc] init];

[frac setNumerator: 1] 非常簡單。它呼叫了 frac 上的 setNumerator method 並傳入 1 為參數。

如同每個 C 的變體，Objective-C 也有一個用以釋放記憶體的方式： release。它繼承自 NSObject，這個 method 在之後會有詳盡的解說。


詳細說明...

多重參數


目前為止我還沒展示如何傳遞多個參數。這個語法乍看之下不是很直覺，不過它卻是來自一個十分受歡迎的 Smalltalk 版本。

Fraction.h

...
-(void) setNumerator: (int) n andDenominator: (int) d;
...
Fraction.m

...
-(void) setNumerator: (int) n andDenominator: (int) d {
    numerator = n;
    denominator = d;
}
...
main.m

#import <stdio.h>
#import "Fraction.h"

int main( int argc, const char *argv[] ) {
    // create a new instance
    Fraction *frac = [[Fraction alloc] init];
    Fraction *frac2 = [[Fraction alloc] init];

    // set the values
    [frac setNumerator: 1];
    [frac setDenominator: 3];

    // combined set
    [frac2 setNumerator: 1 andDenominator: 5];

    // print it
    printf( "The fraction is: " );
    [frac print];
    printf( "\n" );

    // print it
    printf( "Fraction 2 is: " );
    [frac2 print];
    printf( "\n" );

    // free memory
    [frac release];
    [frac2 release];

    return 0;
}
output

The fraction is: 1/3
Fraction 2 is: 1/5
這個 method 實際上叫做 setNumerator:andDenominator:

加入其他參數的方法就跟加入第二個時一樣，即 method:label1:label2:label3: ，而呼叫的方法是 [obj method: param1 label1: param2 label2: param3 label3: param4]

Labels 是非必要的，所以可以有一個像這樣的 method：method:::，簡單的省略 label 名稱，但以 : 區隔參數。並不建議這樣使用。


建構子（Constructors）

Fraction.h

...
-(Fraction*) initWithNumerator: (int) n denominator: (int) d;
...
Fraction.m

...
-(Fraction*) initWithNumerator: (int) n denominator: (int) d {
    self = [super init];

    if ( self ) {
        [self setNumerator: n andDenominator: d];
    }

    return self;
}
...
main.m

#import <stdio.h>
#import "Fraction.h"

int main( int argc, const char *argv[] ) {
    // create a new instance
    Fraction *frac = [[Fraction alloc] init];
    Fraction *frac2 = [[Fraction alloc] init];
    Fraction *frac3 = [[Fraction alloc] initWithNumerator: 3 denominator: 10];

    // set the values
    [frac setNumerator: 1];
    [frac setDenominator: 3];

    // combined set
    [frac2 setNumerator: 1 andDenominator: 5];

    // print it
    printf( "The fraction is: " );
    [frac print];
    printf( "\n" );

    printf( "Fraction 2 is: " );
    [frac2 print];
    printf( "\n" );

    printf( "Fraction 3 is: " );
    [frac3 print];
    printf( "\n" );

    // free memory
    [frac release];
    [frac2 release];
    [frac3 release];

    return 0;
}
output

The fraction is: 1/3
Fraction 2 is: 1/5
Fraction 3 is: 3/10
@interface 裡的宣告就如同正常的函式。

@implementation 使用了一個新的關鍵字：super

如同 Java，Objective-C 只有一個 parent class（父類別）。

使用 [super init] 來存取 Super constructor，這個動作需要適當的繼承設計。

你將這個動作回傳的 instance 指派給另一新個關鍵字：self。Self 很像 C++ 與 Java 的 this 指標。

if ( self ) 跟 ( self != nil ) 一樣，是為了確定 super constructor 成功傳回了一個新物件。nil 是 Objective-C 用來表達 C/C++ 中 NULL 的方式，可以引入 NSObject 來取得。

當你初始化變數以後，你用傳回 self 的方式來傳回自己的位址。

預設的建構子是 -(id) init。

技術上來說，Objective-C 中的建構子就是一個 "init" method，而不像 C++ 與 Java 有特殊的結構。


存取權限


預設的權限是 @protected

Java 實作的方式是在 methods 與變數前面加上 public/private/protected 修飾語，而 Objective-C 的作法則更像 C++ 對於 instance variable（譯注：C++ 術語一般稱為 data members）的方式。

Access.h

#import <Foundation/NSObject.h>@interface Access: NSObject {@public int publicVar;@private int privateVar;
    int privateVar2;@protected int protectedVar;
}@end
Access.m

#import "Access.h"

@implementation Access@end
main.m

#import "Access.h"
#import <stdio.h>

int main( int argc, const char *argv[] ) {
    Access *a = [[Access alloc] init];

    // works
    a->publicVar = 5;
    printf( "public var: %i\n", a->publicVar );

    // doesn't compile
    //a->privateVar = 10;
    //printf( "private var: %i\n", a->privateVar );

    [a release];
    return 0;
}
output

public var: 5
如同你所看到的，就像 C++ 中 private: [list of vars] public: [list of vars] 的格式，它只是改成了@private , @protected , 等等。


Class level access


當你想計算一個物件被 instance 幾次時，通常有 class level variables 以及 class level functions 是件方便的事。

ClassA.h

#import <Foundation/NSObject.h>

static int count;@interface ClassA: NSObject
+(int) initCount;
+(void) initialize;@end
ClassA.m

#import "ClassA.h"

@implementation ClassA
-(id) init {
    self = [super init];
    count++;
    return self;
}

+(int) initCount {
    return count;
}

+(void) initialize {
    count = 0;
}@end
main.m

#import "ClassA.h"
#import <stdio.h>

int main( int argc, const char *argv[] ) {
    ClassA *c1 = [[ClassA alloc] init];
    ClassA *c2 = [[ClassA alloc] init];

    // print count
    printf( "ClassA count: %i\n", [ClassA initCount] );
    
    ClassA *c3 = [[ClassA alloc] init];

    // print count again
    printf( "ClassA count: %i\n", [ClassA initCount] );

    [c1 release];
    [c2 release];
    [c3 release];
    
    return 0;
}
output

ClassA count: 2
ClassA count: 3
static int count = 0; 這是 class variable 宣告的方式。其實這種變數擺在這裡並不理想，比較好的解法是像 Java 實作 static class variables 的方法。然而，它確實能用。

+(int) initCount; 這是回傳 count 值的實際 method。請注意這細微的差別！這裡在 type 前面不用減號 - 而改用加號 +。加號 + 表示這是一個 class level function。（譯注：許多文件中，class level functions 被稱為 class functions 或 class method）

存取這個變數跟存取一般成員變數沒有兩樣，就像 ClassA 中的 count++ 用法。

+(void) initialize method is 在 Objective-C 開始執行你的程式時被呼叫，而且它也被每個 class 呼叫。這是初始化像我們的 count 這類 class level variables 的好地方。


繼承、多型（Inheritance, Polymorphism）以及其他物件導向功能

id 型別


Objective-C 有種叫做 id 的型別，它的運作有時候像是 void*，不過它卻嚴格規定只能用在物件。Objective-C 與 Java 跟 C++ 不一樣，你在呼叫一個物件的 method 時，並不需要知道這個物件的型別。當然這個 method 一定要存在，這稱為 Objective-C 的訊息傳遞。

main.m

#import <stdio.h>
#import "Fraction.h"
#import "Complex.h"

int main( int argc, const char *argv[] ) {
    // create a new instance
    Fraction *frac = [[Fraction alloc] initWithNumerator: 1 denominator: 10];
    Complex *comp = [[Complex alloc] initWithReal: 10 andImaginary: 15];
    id number;

    // print fraction
    number = frac;
    printf( "The fraction is: " );
    [number print];
    printf( "\n" );

    // print complex
    number = comp;
    printf( "The complex number is: " );
    [number print];
    printf( "\n" );

    // free memory
    [frac release];
    [comp release];

    return 0;
}
output

The fraction is: 1 / 10
The complex number is: 10.000000 + 15.000000i
這種動態連結有顯而易見的好處。你不需要知道你呼叫 method 的那個東西是什麼型別，如果這個物件對這個訊息有反應，那就會喚起這個 method。這也不會牽涉到一堆繁瑣的轉型動作，比如在 Java 裡呼叫一個整數物件的 .intValue() 就得先轉型，然後才能呼叫這個 method。


Protocols


Objective-C 裡的 Protocol 與 Java 的 interface 或是 C++ 的 purely virtual class 相同。

protocol 的宣告十分簡單，基本上就是 @protocol ProtocolName (methods you must implement) @end。

要遵從（conform）某個 protocol，將要遵從的 protocols 放在 <> 裡面，並以逗點分隔。如：@interface SomeClass <Protocol1, Protocol2, Protocol3>

protocol 要求實作的 methods 不需要放在 header 檔裡面的 methods 列表中。如你所見，Complex.h 檔案裡沒有 -(void) print 的宣告，卻還是要實作它，因為它（Complex class）遵從了這個 protocol。

Objective-C 的介面系統有一個獨一無二的觀念是如何指定一個型別。比起 C++ 或 Java 的指定方式，如：Printing *someVar = ( Printing * ) frac; 你可以使用 id 型別加上 protocol：id <Printing> var = frac;。這讓你可以動態地指定一個要求多個 protocol 的型別，卻從頭到尾只用了一個變數。如：<Printing, NSCopying> var = frac;

就像使用@selector 來測試物件的繼承關係，你可以使用 @protocol 來測試物件是否遵從介面。如果物件遵從這個介面，[object conformsToProtocol: @protocol( SomeProtocol )] 會回傳一個 YES 型態的 BOOL 物件。同樣地，對 class 而言也能如法炮製 [SomeClass conformsToProtocol: @protocol( SomeProtocol )]。


Protocols


Objective-C 裡的 Protocol 與 Java 的 interface 或是 C++ 的 purely virtual class 相同。

基於 "Programming in Objective-C," Copyright &copy; 2004 by Sams Publishing一書中的範例，並經過允許而刊載。

Printing.h

@protocol Printing
-(void) print;@end
Fraction.h

#import <Foundation/NSObject.h>
#import "Printing.h"@interface Fraction: NSObject <Printing, NSCopying> {
    int numerator;
    int denominator;
}

-(Fraction*) initWithNumerator: (int) n denominator: (int) d;
-(void) setNumerator: (int) d;
-(void) setDenominator: (int) d;
-(void) setNumerator: (int) n andDenominator: (int) d;
-(int) numerator;
-(int) denominator;@end
Fraction.m

#import "Fraction.h"
#import <stdio.h>

@implementation Fraction
-(Fraction*) initWithNumerator: (int) n denominator: (int) d {
    self = [super init];

    if ( self ) {
        [self setNumerator: n andDenominator: d];
    }

    return self;
}

-(void) print {
    printf( "%i/%i", numerator, denominator );
}

-(void) setNumerator: (int) n {
    numerator = n;
}

-(void) setDenominator: (int) d {
    denominator = d;
}

-(void) setNumerator: (int) n andDenominator: (int) d {
    numerator = n;
    denominator = d;
}

-(int) denominator {
    return denominator;
}

-(int) numerator {
    return numerator;
}

-(Fraction*) copyWithZone: (NSZone*) zone {
    return [[Fraction allocWithZone: zone] initWithNumerator: numerator
                                           denominator: denominator];
}@end
Complex.h

#import <Foundation/NSObject.h>
#import "Printing.h"@interface Complex: NSObject <Printing> {
    double real;
    double imaginary;
}

-(Complex*) initWithReal: (double) r andImaginary: (double) i;
-(void) setReal: (double) r;
-(void) setImaginary: (double) i;
-(void) setReal: (double) r andImaginary: (double) i;
-(double) real;
-(double) imaginary;@end
Complex.m

#import "Complex.h"
#import <stdio.h>

@implementation Complex
-(Complex*) initWithReal: (double) r andImaginary: (double) i {
    self = [super init];

    if ( self ) {
        [self setReal: r andImaginary: i];
    }

    return self;
}

-(void) setReal: (double) r {
    real = r;
}

-(void) setImaginary: (double) i {
    imaginary = i;
}

-(void) setReal: (double) r andImaginary: (double) i {
    real = r;
    imaginary = i;
}

-(double) real {
    return real;
}

-(double) imaginary {
    return imaginary;
}

-(void) print {
    printf( "%_f + %_fi", real, imaginary );
}@end
main.m

#import <stdio.h>
#import "Fraction.h"
#import "Complex.h"

int main( int argc, const char *argv[] ) {
    // create a new instance
    Fraction *frac = [[Fraction alloc] initWithNumerator: 3 denominator: 10];
    Complex *comp = [[Complex alloc] initWithReal: 5 andImaginary: 15];
    id <Printing> printable;
    id <NSCopying, Printing> copyPrintable;

    // print it
    printable = frac;
    printf( "The fraction is: " );
    [printable print];
    printf( "\n" );

    // print complex
    printable = comp;
    printf( "The complex number is: " );
    [printable print];
    printf( "\n" );

    // this compiles because Fraction comforms to both Printing and NSCopyable
    copyPrintable = frac;

    // this doesn't compile because Complex only conforms to Printing
    //copyPrintable = comp;

    // test conformance

    // true
    if ( [frac conformsToProtocol: @protocol( NSCopying )] == YES ) {
        printf( "Fraction conforms to NSCopying\n" );
    }

    // false
    if ( [comp conformsToProtocol: @protocol( NSCopying )] == YES ) {
        printf( "Complex conforms to NSCopying\n" );
    }

    // free memory
    [frac release];
    [comp release];

    return 0;
}
output

The fraction is: 3/10
The complex number is: 5.000000 + 15.000000i
Fraction conforms to NSCopying
protocol 的宣告十分簡單，基本上就是 @protocol ProtocolName (methods you must implement) @end。

要遵從（conform）某個 protocol，將要遵從的 protocols 放在 <> 裡面，並以逗點分隔。如：@interface SomeClass <Protocol1, Protocol2, Protocol3>

protocol 要求實作的 methods 不需要放在 header 檔裡面的 methods 列表中。如你所見，Complex.h 檔案裡沒有 -(void) print 的宣告，卻還是要實作它，因為它（Complex class）遵從了這個 protocol。

Objective-C 的介面系統有一個獨一無二的觀念是如何指定一個型別。比起 C++ 或 Java 的指定方式，如：Printing *someVar = ( Printing * ) frac; 你可以使用 id 型別加上 protocol：id <Printing> var = frac;。這讓你可以動態地指定一個要求多個 protocol 的型別，卻從頭到尾只用了一個變數。如：<Printing, NSCopying> var = frac;

就像使用@selector 來測試物件的繼承關係，你可以使用 @protocol 來測試物件是否遵從介面。如果物件遵從這個介面，[object conformsToProtocol: @protocol( SomeProtocol )] 會回傳一個 YES 型態的 BOOL 物件。同樣地，對 class 而言也能如法炮製 [SomeClass conformsToProtocol: @protocol( SomeProtocol )]。


Foundation framework classes

Foundation framework 地位如同 C++ 的 Standard Template Library。不過 Objective-C 是真正的動態識別語言（dynamic types），所以不需要像 C++ 那樣肥得可怕的樣版（templates）。這個 framework 包含了物件組、網路、執行緒，還有更多好東西。

NSArray


基於 "Programming in Objective-C," Copyright &copy; 2004 by Sams Publishing一書中的範例，並經過允許而刊載。

main.m

#import <Foundation/NSArray.h>
#import <Foundation/NSString.h>
#import <Foundation/NSAutoreleasePool.h>
#import <Foundation/NSEnumerator.h>
#import <stdio.h>

void print( NSArray *array ) {
    NSEnumerator *enumerator = [array objectEnumerator];
    id obj;

    while ( obj = [enumerator nextObject] ) {
        printf( "%s\n", [[obj description] cString] );
    }
}

int main( int argc, const char *argv[] ) {
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSArray *arr = [[NSArray alloc] initWithObjects:
                    @"Me", @"Myself", @"I", nil];
    NSMutableArray *mutable = [[NSMutableArray alloc] init];

    // enumerate over items
    printf( "----static array\n" );
    print( arr );

    // add stuff
    [mutable addObject: @"One"];
    [mutable addObject: @"Two"];
    [mutable addObjectsFromArray: arr];
    [mutable addObject: @"Three"];

    // print em
    printf( "----mutable array\n" );
    print( mutable );

    // sort then print
    printf( "----sorted mutable array\n" );
    [mutable sortUsingSelector: @selector( caseInsensitiveCompare: )];
    print( mutable );
    
    // free memory
    [arr release];
    [mutable release];
    [pool release];

    return 0;
}
output

----static array
Me
Myself
I
----mutable array
One
Two
Me
Myself
I
Three
----sorted mutable array
I
Me
Myself
One
Three
Two
陣列有兩種（通常是 Foundation classes 中最資料導向的部分），NSArray 跟 NSMutableArray，顧名思義，mutable（善變的）表示可以被改變，而 NSArray 則不行。這表示你可以製造一個 NSArray 但卻不能改變它的長度。

你可以用 Obj, Obj, Obj, ..., nil 為參數呼叫建構子來初始化一個陣列，其中 nil 表示結尾符號。

排序（sorting）展示如何用 selector 來排序一個物件，這個 selector 告訴陣列用 NSString 的忽略大小寫順序來排序。如果你的物件有好幾個排序方法，你可以使用這個 selector 來選擇你想用的方法。

在 print method 裡，我使用了 description method。它就像 Java 的 toString，會回傳物件的 NSString 表示法。

NSEnumerator 很像 Java 的列舉系統。while ( obj = [array objectEnumerator] ) 行得通的理由是 objectEnumerator 會回傳最後一個物件的 nil。在 C 裡 nil 通常代表 0，也就是 false。改用 ( ( obj = [array objectEnumerator] ) != nil ) 也許更好。

NSDictionary


基於 "Programming in Objective-C," Copyright &copy; 2004 by Sams Publishing一書中的範例，並經過允許而刊載。

main.m

#import <Foundation/NSString.h>
#import <Foundation/NSAutoreleasePool.h>
#import <Foundation/NSDictionary.h>
#import <Foundation/NSEnumerator.h>
#import <Foundation/Foundation.h<
#import <stdio.h>

void print( NSDictionary *map ) {
    NSEnumerator *enumerator = [map keyEnumerator];
    id key;

    while ( key = [enumerator nextObject] ) {
        printf( "%s => %s\n",
                [[key description] cString],
                [[[map objectForKey: key] description] cString] );
    }
}

int main( int argc, const char *argv[] ) {
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSDictionary *dictionary = [[NSDictionary alloc] initWithObjectsAndKeys:
        @"one", [NSNumber numberWithInt: 1],
        @"two", [NSNumber numberWithInt: 2],
        @"three", [NSNumber numberWithInt: 3],
        nil];
    NSMutableDictionary *mutable = [[NSMutableDictionary alloc] init];

    // print dictionary
    printf( "----static dictionary\n" );
    print( dictionary );

    // add objects
    [mutable setObject: @"Tom" forKey: @"tom@jones.com"];
    [mutable setObject: @"Bob" forKey: @"bob@dole.com" ];

    // print mutable dictionary
    printf( "----mutable dictionary\n" );
    print( mutable );

    // free memory 
    [dictionary release];
    [mutable release];
    [pool release];

    return 0;
}
output

----static dictionary
1 => one
2 => two
3 => three
----mutable dictionary
bob@dole.com => Bob
tom@jones.com => Tom

-EOF-
