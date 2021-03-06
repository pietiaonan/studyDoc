 ## 依赖关系
依赖关系(Dependency)是一种使用关系。通常，依赖关系体现在某个类的方法使用另一个类的对象作为参数。

依赖关系用带箭头的虚线表示，由依赖的一方指向被依赖的一方。

![依赖](..\\..\\img\pattern\dependency.png)




 ## 关联关系
* **双向关联**

  默认情况下，关联是双向的,双向关联用直线表示。

  ![依赖](..\\..\\img\pattern\biAsso.png)

* **单向关联**

类的关联关系也可以是单向的，单向关联用带箭头的实线表示。

![依赖](..\\..\\img\pattern\singleAsso.png)



* **自关联**

在系统中可能会存在一些类的属性对象类型为该类本身，这种特殊的关联关系称为自关联。

![依赖](..\\..\\img\pattern\selfAsso.png)

* **多重关联**

重数性关联关系又称为多重性关联关系(Multiplicity)，表示一个类的对象与另一个类的对象连接的个数。在UML中多重性关系可以直接在关联直线上增加一个数字表示与之对应的另一个类的对象的个数。

| 表示方式 | 多重性说明                                                 |
| -------- | :--------------------------------------------------------- |
| 1..1     | 表示另一个类的一个对象只与一个该类对象有关系               |
| 0..*     | 表示另一个类的一个对象与零个或多个该类对象有关系           |
| 1..*     | 表示另一个类的一个对象与一个或多个该类对象有关系           |
| 0..1     | 表示另一个类的一个对象没有或只与一个该类对象有关系         |
| m..n     | 表示另一个类的一个对象与最少m、最多n个该类对象有关系(m<=n) |

![依赖](..\\..\\img\pattern\multiAsso.png)


* **聚合**

聚合关系(Aggregation)表示一个整体与部分的关系。

在聚合关系中，成员类是整体类的一部分，即成员对象是整体对象的一部分，但是成员对象可以脱离整体对象独立存在。通常不直接实例化成员对象，通过构造函数或者Setter方法注入。

![依赖](..\\..\\img\pattern\aggreAsso.png)




* **组合**

组合关系(Composition)也表示类之间整体和部分的关系，但是组合关系中部分和整体具有统一的生存期。一旦整体对象不存在，部分对象也将不存在，部分对象与整体对象之间具有同生共死的关系。通常定义了成员对象后通过构造函数实例化。

在组合关系中，成员类是整体类的一部分，而且整体类可以控制成员类的生命周期，即成员类的存在依赖于整体类。

在UML中，组合关系用带实心菱形的直线表示

![依赖](..\\..\\img\pattern\comAsso.png)

## 泛化关系

泛化关系(Generalization)也就是继承关系，也称为“is-a-kind-of”关系，泛化关系用于描述父类与子类之间的关系，父类又称作基类或超类，子类又称作派生类。在UML中，泛化关系用带空心三角形的直线来表示。

![依赖](..\\..\\img\pattern\generalization.png)

## 接口实现关系

接口之间也可以有与类之间关系类似的继承关系和依赖关系，但是接口和类之间还存在一种实现关系(Realization)，在这种关系中，类实现了接口，类中的操作实现了接口中所声明的操作。在UML中，类与接口之间的实现关系用带空心三角形的虚线来表示

![依赖](..\\..\\img\pattern\realization.png)