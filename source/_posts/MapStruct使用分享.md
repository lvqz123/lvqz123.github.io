---
title: MapStruct使用分享
date: 2024-02-26
tags: mapStruct
categories: JAVA插件
---
## 简介

MapStruct是一个开源的基于Java的代码生成器，用于创建实现Java Bean之间转换的扩展映射器。使用MapStruct，我们只需要创建接口，而该库会通过注解在编译过程中自动创建具体的映射实现，大大减少了通常需要手工编写的样板代码的数量。

## 依赖

```xml
compile 'org.mapstruct:mapstruct:1.5.5.Final'
annotationProcessor "org.mapstruct:mapstruct-processor:1.5.5.Final"
```

## 常用注解

### 启动注解

#### @Mapper

@Mapper注解加到转换接口类上，证明该类是被mapstruct管理编译的转换类，并在编译期间生成对应的转换实现。

#### 注解参数：

| 参数名称       | 参数作用                                                     |
| -------------- | :----------------------------------------------------------- |
| componentModel | 指定生成的映射器遵循的组件模型，例如:spring 则是将该类转为spring的bean对象交给spring容器管理 |
| uses           | 引用其他converter或者自定义映射器，若遇到相同的入参、出参转换方法时会直接默认引用。 |
| imports        | 为生成的转换实现类引用对应类包                               |

### 业务注解

#### @Mapping

@Mapping接口是最常用的mapstruct注解，它的主要作用是指定将某个源字段转为某个目标字段，或者直接为某个目标字段提供转换初始化逻辑。

例如：

1. 我希望将A类的dog字段赋值给B类字段的haShiQi。由于变量名不同，组件无法自动转换，可以使用@Mapping注解指定转换。
2. 我希望转换到B类时，B类的dogName默认初始化为"旺财"，也可以通过@Mapping注解指定对应默认值。
3. 我希望转换到B类时，B类的createTime字段默认赋值为今天的0点，可以通过@Mapping注解指定对应的生成函数来处理这种转换逻辑。

#### 注解参数

| 参数名称        | 参数作用                                                     |
| --------------- | ------------------------------------------------------------ |
| source          | 转换源数据参数字段名称                                       |
| target          | 转换目标数据参数字段名称                                     |
| defaultValue    | 当源数据为null时，默认为目标字段数据赋值                     |
| expression      | 通过函数的方式为目标字段赋值，函数返回值为target字段值，一般与target参数同时存在 |
| qualifiedByName | 转换指定限定符，定位具体的映射方法                           |

表格中列出的仅仅是常用的几个参数，这几个参数几乎可以涵盖日常业务场景的80%-90%，其他的参数闲暇时间也可以自己研究一下~，例如：dateFormat、numberFormat可以处理对应的时间和数字为字符串。

#### @MappingTarget

该注解的作用是声明要转换的目标对象。第一次看到这个注解的时候还是有些不明白他的使用场景，因为一般我们的目的都是通过A类生成一个对应的B类。例如：

```java
ProjectDTO convertProjectDTO(ProjectBO projectBO)
```

那如果更新对象场景时应该怎么指定目标对象呢？

例如：

```java
void updateProjectDTO(ProjectBO projectBO,ProjectDTO projectDTO)
```

我的目的是将bo对象转换给dto，但是因为没有注解，组件也不知道这两个参数是哪两个互转。所以需要一个注解为他标识出来

```java
@Mapping(target = "projectName",source = "name")
void updateProjectDTO(ProjectBO projectBO,@MappingTarget ProjectDTO projectDTO)
```

#### @BeforeMapping

顾名思义，这个注解的作用是，在做对应方法映射之前预先处理的前置动作。举个例子：

```java
//我现在要将bo转换为dto，但是我在转换之前，希望先把bo的名字默认设置为"智联项目"
ProjectDTO convertProjectDTO(ProjectBO projectBO)

@BeforeMapping
default void setProjectName(ProjectBo projectBO,@MappingTarget ProjectDTO projectDTO){
	projectBO.setName("智联项目");
}
```

#### @AfterMapping

顾名思义，这个注解的作用是，在做对应方法映射之后后置处理的后置动作。举个例子：

```java
//我现在要将bo转换为dto，但是我在转换之后，希望把dto的项目预算通过bo的两个字段计算得到
ProjectDTO convertProjectDTO(ProjectBO projectBO)

@AfterMapping
default void setProjectMoney(ProjectBo projectBO,@MappingTarget ProjectDTO projectDTO){
	projectDTO.setAllMoney(bo.getA() + bo.getB());
}
```

### 扩展注解

#### @InheritInverseConfiguration

逆映射，即一个文件内，两个类之间互相转换映射时其实只要写好其中一个映射，另一个可以通过注解继承原有映射配置，即原有的target与source会互换。

```java
@Mapping(source = "describe", target = "des")
@Mapping(source = "apee", target = "apee2")
public abstract PersonVO transToViewObject(PersionDTO persionDTO);

@InheritInverseConfiguration
public abstract PersionDTO transToViewObject(PersonVO personVO);
```

#### @ValueMapping

用于转换枚举值的注解。可以理解为通过if判断枚举值，然后返回对应结果。

#### @MapMapping

用于转换map类型的注解。

