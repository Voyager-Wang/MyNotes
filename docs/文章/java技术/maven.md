# Maven的依赖管理

## Maven中的依赖scope

scope定义了依赖的项目在编译、测试、打包三个不同阶段是否生效，下面列举了三个阶段中依赖是否生效的具体表现，以加深理解：

首先是编译阶段，这个阶段将用户的java文件编译成class文件，如果依赖在这个阶段不生效，那么当用户的主代码中使用了该依赖中的类时，会造成编译失败。具体现象是mvn compile失败，或者更直观的，在ide中提示找不到类，编译错误。

然后是测试阶段，如果依赖在测试阶段不生效，那么如果用户在测试代码中使用了该依赖中的类，会引起测试代码编译失败。

最后是打包阶段，如果依赖在打包阶段不生效，简单来说，打包后的文件中不会存在该依赖的jar包。这样打包后的文件，在后续运行阶段需要另外提供该依赖的jar包，否则当jvm尝试将该依赖中的class文件加载到内存中时，会报错找不到类。在不同的打包场景下，依赖是否生效的具体表现也是不同的，需要具体场景具体分析：

- 打war包：依赖是否生效意味着，该依赖的jar包是否出现在WEB-INF/lib目录中
- maven-assembly-plugin打jar包（包含依赖的包）：依赖是否生效意味着，是否将该依赖的jar包打进去（默认配置下的表现，实际上assembly-plugin配置自由度很高）
- maven-jar-plugin打普通的jar包（只包含主代码）：依赖是否生效意味着，如果指定了addClasspath，那么MANIFEST.MF中是否包含该jar包的路径

依赖在上述三个阶段是否生效有多种排列组合，maven定义了多个scope来描述这些场景，官方文档说明中共有6种scope：compile provided runtime test system import，常用和需要关注的有5种 compile provided system runtime test，而import比较特殊，使用的场景和其他几种完全不同，这里暂时不讨论。

这些scope和依赖在三个阶段是否生效的关系如下：

|                 | 编译 | 测试 | 打包 |
| --------------- | ---- | ---- | ---- |
| compile         | 是   | 是   | 是   |
| runtime         | 否   | 是   | 是   |
| provided/system | 是   | 是   | 否   |
| test            | 否   | 是   | 否   |

可以看到，依赖配置任何scope对于测试阶段都是生效的，相当于scope对测试其实是透明的，只要声明了依赖，对测试来说就是生效的。

那么这几种scope一般用在什么场景下呢？

compile是默认的配置，适用的场景是：如果依赖在编译阶段需要引入（用户写的主代码引用了依赖中的类），而且期望打包到lib中，那么就要声明为compile。

runtime适用的场景：编译阶段不需要，比如对于jdbc的具体实现mysql-jdbc，由于代码中直接使用的是jdbc的接口，所以编译阶段不需要依赖mysql-jdbc，如果同时我们希望将mysql-jdbc打包到lib中，以供运行阶段使用，则需要将该依赖声明为runtime。

provided适用的场景：依赖在编译阶段需要，但是我们不希望将该依赖打包到lib中，因为执行的容器已经提供了该jar包，如果我们重复提供可能会引起jar包冲突，比如打包flink任务的jar包时，flink相关的依赖，引擎已经提供了，就不需要也不应该打包进去。

system可以认为和provided一致，只是必需要额外指定一个本地路径，这个一般不推荐使用。

test适用的场景很单一，如果一个依赖只是和测试代码相关的，主代码不使用，那就就声明为test，该依赖在编译和打包时都会忽略。



## Maven中的依赖传递

首先要明确的是依赖传递解决的是什么问题，场景如下：

项目A依赖项目B，项目B依赖项目C，写作A->B，C->B

那么项目A对项目C的依赖关系是怎样的？这就是依赖传递要解决的问题。

依赖传递的复杂性主要体现在不同scope下的表现不同，如图所示：

https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html

![image-20210717183403288](https://tinkerer-pic.oss-cn-hangzhou.aliyuncs.com/picgo/image-20210717183403288.png)

上表中左侧的列是外层依赖（A->B）的scope，第一行是内层依赖（B->C）的scope，那么A->C的依赖结果为交叉点的结果。

结果说明：“-”表示不依赖，不依赖的结果是：A项目在编译、测试、打包时都不能访问C项目。



上表有几个典型的规律：

1.provided/test的依赖一定不会传递到外层

2.compile的依赖传递到外层时和外层的依赖scope相同

3.runtime的依赖传递到外层时和外层的依赖scope相同，除非外层是compile，会保持runtime



上述的规律还不足以明确地指导我们的使用，我们从具体的场景进行分析，分为两类问题：

- 问题一：如果我是A的开发，当我声明A->B的scope时，对A->C的依赖有什么影响？
- 问题二：如果我是B的开发，当我声明B->C的scope时，对A->C的依赖有什么影响？

问题一：

最常见的场景，我声明了A->B为compile，这已经是对B最强的依赖了，对于B内部的依赖C

- 如果是compile，会将compile传递出来，C在我的编译和打包阶段都会生效

- 如果是runtime，那我对C的依赖也是runtime，编译阶段我不能使用C了，毕竟B也不能用，我对B的内部的C权限也不能更高，这一点符合直觉；在打包时，C会打包到lib包中，这个需求是显然的，否则很可能B项目运行阶段会报错。

  那么我可以将C声明为provided吗？可以，只要我确信这个包在运行时会提供，这样带来的好处是可以避免jar包冲突，这样引入了一个技巧，为了避免C的jar包冲突，除了到每一个B依赖中排除掉依赖之外，也可以在A项目中，显示声明C为provided，这样更加简洁高效一些

- 如果是provided或者test，我应当清楚，我不会对C有依赖

第二个场景，我声明了A->B为provided，B不会打入我的jar包，而运行时会提供B的jar包，对于B内部的依赖C

- 如果是compile，此时我对C的依赖为provided，即我可以编译阶段使用C，但C也不会打入我的jar包。
- 如果是runtime，我对C的依赖变成了provided，在编译阶段看来，这似乎是一种强化，B不能直接使用C的类，我却可以了；在打包阶段看来，被弱化了，我不会将C打入自己的jar包，这一点与外层B的provided是一致的
- 如果是provided或者test，我应当清楚，我不会对C有依赖

第三个场景，我声明了A->B为runtime，B会打入我的jar包，对于B内部的依赖C，编译阶段我一定不能使用，毕竟B我都不能使用，这一点是符合直觉的；那么打包阶段呢，我会将compile和runtime的C打入jar包，忽略provided和test

第四个场景，我声明了A->B为test，同样B的provide和test不会被我访问到，但是B的compile和runtime依赖我可以在测试中自由使用，这里runtime似乎也升级了

问题二：

最常见的场景，我声明了B->C为compile，因为我对C的依赖是足够强的，我会期望外部对我的依赖scope也会透传到对C的依赖。

第二个场景，我声明了B->C为provided或test，这意味着外界对C完全不可见，不能编译，也不会打包自己的项目。

第三个场景，我声明了B->C为runtime，外部对我的依赖scope也会透传到对C的依赖，这个其实是不太安全的，这个时候我会发现我编译阶段不能依赖的C，外部反而可以了，这个时候我对投传下来的compile降级为runtime，其他的未做处理。

