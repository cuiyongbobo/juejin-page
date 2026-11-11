# Swift 面试 20 题结构图源码

采用类图风格；标注为 concept、usage、runtime 等的框是概念模型，不是真实 Swift 类。实线空心三角用于类继承，虚线空心三角用于协议遵循；关联线以标签为准，不代表固定内存布局。

## 第 1 题

Profile 的两个值分别保存自己的 name，b 改名不影响 a；引用 x、y 则关联同一个 Account 实例。State 框列出三个枚举 case，success 和 failure 的关联值随当前 case 保存。框内是示例值与成员，关联线表示赋值或引用，不代表固定内存地址。

```mermaid
classDiagram
    direction TB
    class ProfileA["a: Profile"] {
        <<struct>>
        name: String = 小明
    }
    class ProfileB["b: Profile"] {
        <<struct>>
        name: String = 小红
    }
    class X["x"] {
        <<reference>>
        指向 Account 实例
    }
    class Y["y"] {
        <<reference>>
        由 y = x 得到
    }
    class Account["Account 实例"] {
        <<class>>
        balance: Int = 100
        引用身份由 === 比较
    }
    class State["State"] {
        <<enum>>
        loading
        success(String)
        failure(Int)
    }
    ProfileA ..> ProfileB : b = a 后独立修改
    X --> Account : 强引用
    Y --> Account : 强引用
```

## 第 2 题

Optional 框展示两种 case；安全绑定与可选链框展示不同使用方式。as? 是产生可选结果的一种操作，转换失败得到 none。nickname 为空字符串时属于 some，而不是 none；?? 只在缺值时求默认值。

```mermaid
classDiagram
    direction TB
    class Optional["Optional of Wrapped"] {
        <<enum>>
        some(Wrapped)
        none
    }
    class Binding["可选绑定"] {
        <<usage>>
        if let 成功值限于分支
        guard let 成功值用于后续作用域
        guard 失败必须退出
    }
    class Access["可选链与默认值"] {
        <<usage>>
        nickname?.count 得到可选结果
        nickname ?? 游客 按需给默认值
    }
    class Cast["运行时类型转换"] {
        <<usage>>
        value as? String
        成功得到 some(String)
        失败得到 nil
    }
    Binding ..> Optional : 检查并绑定值
    Access ..> Optional : 按有值或缺值处理
    Cast --> Optional : 返回可选结果
```

## 第 3 题

Meter 框区分保存值、计算访问和延迟初始化；Vehicle 与 Car 的空心三角表示真正的类继承。Car 可以重写 class name()，不能重写 static category()。类型存储属性与实例 lazy 的初始化保证不同，图内单独列出，初始化安全不等于后续所有读写都安全。

```mermaid
classDiagram
    direction TB
    class Meter["Meter"] {
        <<struct>>
        value: Int 存储属性
        history: Array 实例 lazy var
        static let unit: String
        doubled.get() 根据 value 计算
    }
    class Vehicle["Vehicle"] {
        <<class>>
        static category() 不可重写
        class name() 可重写
    }
    class Car["Car"] {
        <<class>>
        override class name() 返回汽车
    }
    class Guarantee["初始化保证"] {
        <<concept>>
        实例 lazy 不保证并发只初始化一次
        类型存储属性只初始化一次
        共享可变状态另行同步
    }
    Vehicle <|-- Car : 继承
    Meter ..> Guarantee : 区分实例与类型属性
```

## 第 4 题

Vehicle 与 Car 表示真实继承关系，框内列出各自属性与初始化入口。Car 先赋 seats，再向上调用 Vehicle.init(name:)；Vehicle 的便利入口通过 self.init 横向委托。阶段框是初始化规则概览，不能在第一阶段尚未完成时读取完整 self 或调用实例方法。

```mermaid
classDiagram
    direction TB
    class Vehicle["Vehicle"] {
        <<class>>
        name: String
        init(name) 指定初始化器
        convenience init() 横向委托
    }
    class Car["Car"] {
        <<class>>
        seats: Int
        init(name,seats) 先设置 seats
        super.init(name) 初始化父类部分
    }
    class Phase["两阶段规则"] {
        <<concept>>
        阶段一 先子类再父类设置初值
        阶段二 先父类再子类完成定制
        阶段一结束后才能使用完整 self
    }
    class Age["Age"] {
        <<struct>>
        value: Int
        init?(value) 负数时返回 nil
    }
    Vehicle <|-- Car : 类继承
    Car ..> Vehicle : super.init 向上委托
    Car ..> Phase : 必须满足安全检查
```

## 第 5 题

Page 强持有 onTap 闭包，闭包默认强捕获 self 时构成强引用环。CaptureList 表示替代捕获策略，并不是额外分配的业务对象：[weak self] 不保活且会清零；[unowned self] 不保活但依赖生命周期保证。是否使用弱捕获由实际所有权关系决定。

```mermaid
classDiagram
    direction TB
    class Page["Page 实例"] {
        <<class>>
        onTap: Closure
        refresh()
        deinit()
    }
    class Closure["onTap 闭包"] {
        <<closure>>
        执行回调代码
        默认强捕获 self
    }
    class Capture["捕获列表策略"] {
        <<concept>>
        weak self 可自动变 nil
        unowned self 不自动清零
        两者均不增加强引用计数
    }
    Page --> Closure : 强持有
    Closure --> Page : 默认强捕获形成环
    Capture ..> Closure : 用合适捕获策略替换强回边
```

## 第 6 题

LiveCapture 和 SnapshotCapture 对应 readLive、readSnapshot：共享局部变量的前者读到 2，保存值快照的后者读到 1。参数规则框列出两个独立修饰维度。框名用于解释闭包与上下文，不表示 Swift 标准库存在这些同名类。

```mermaid
classDiagram
    direction TB
    class Local["局部变量 count"] {
        <<context>>
        初值为 1
        创建闭包后改为 2
    }
    class Live["readLive"] {
        <<closure>>
        默认捕获 count 变量
        call() 读到 2
    }
    class Snapshot["readSnapshot"] {
        <<closure>>
        捕获列表保存 count 值 1
        call() 仍读到 1
    }
    class Parameter["闭包参数规则"] {
        <<concept>>
        escaping 允许闭包活过函数返回
        autoclosure 将表达式变为无参闭包
        两者不自动创建异步任务
    }
    Live --> Local : 共享被捕获的变量
    Snapshot ..> Local : 创建时取值快照
    Parameter ..> Live : 生命周期规则独立于捕获方式
```

## 第 7 题

赋值后 a、b 是两个数组值，可关联同一份 Buffer；b.append(4) 在共享时先获得独立缓冲区再写，a 保持 [1,2,3]。Buffer 框是存储概念模型，具体地址可能被优化。Item 框说明：元素若是类引用，即使复制元素存储，也可能仍引用同一对象。

```mermaid
classDiagram
    direction TB
    class A["数组值 a"] {
        <<value>>
        元素为 1,2,3
        保持原始内容
    }
    class B["数组值 b"] {
        <<value>>
        由 b = a 得到
        append(4) 后为 1,2,3,4
    }
    class Buffer["共享 Buffer"] {
        <<storage>>
        保存元素或元素引用
        共享时写入需要分离
    }
    class Owned["b 的独立 Buffer"] {
        <<storage>>
        共享写入后由 b 使用
        独占写入通常无需再次复制
    }
    class Item["Item"] {
        <<class>>
        name: String
        改对象属性不是改数组元素引用
    }
    A --> Buffer : 赋值后的共享存储
    B --> Buffer : 写入前可共享
    B --> Owned : 共享时先分离再写入
    Buffer --> Item : 类元素时保存对象引用
    Owned --> Item : 不自动深复制对象
```

## 第 8 题

Input 和结果框使用页面相同的输入 1、x、3；map 保留 nil 的位置，compactMap 去掉失败项。Groups 框展示 flatMap 只拼接一层。这里列的是操作与结果类型，并不是三种实际的数组子类。

```mermaid
classDiagram
    direction TB
    class Input["字符串数组"] {
        <<value>>
        输入为 1、x、3
    }
    class Mapped["map 结果"] {
        <<value>>
        类型为 Array of Int?
        值为 some 1、nil、some 3
    }
    class Compacted["compactMap 结果"] {
        <<value>>
        类型为 Array of Int
        值为 1、3
    }
    class Groups["嵌套数组"] {
        <<value>>
        输入为 1,2 与 3 两组
        flatMap(identity) 得到 1,2,3
    }
    Input ..> Mapped : 每项转换保留位置
    Input ..> Compacted : 转换并去 nil
```

## 第 9 题

ThrowingFunction 是调用入口，调用策略框比较三种处理方式；Result 框列出两种枚举 case。defer 的规则单独放在作用域框中，它负责清理而非捕获错误，程序发生运行时终止时不能依赖它一定执行。

```mermaid
classDiagram
    direction TB
    class Throwing["可能抛错的函数"] {
        <<function>>
        throws 声明可能失败
        throw 传递具体错误
    }
    class Handling["调用策略"] {
        <<usage>>
        try 配合 catch 或继续抛出
        try? 失败变 nil
        try! 失败为运行时错误
    }
    class Result["Result"] {
        <<enum>>
        success(Success)
        failure(Failure)
    }
    class Scope["defer 所在作用域"] {
        <<concept>>
        执行到 defer 才注册
        正常结束或 return 或 throw 时执行
        同一作用域按后注册先执行
    }
    Handling ..> Throwing : 选择错误策略
    Result ..> Throwing : 可将成功或失败保存为值
    Scope ..> Throwing : 负责退出时的资源清理
```

## 第 10 题

四个框分别表示语言机制。Generic 的 T 保持输入输出关系；OpaqueResult 对应本例 make() 始终返回 User；Existential 可先保存 User 再换 Book；Any 不保证 Named 能力。它们之间没有继承关系，框内签名仅为规则示意。

```mermaid
classDiagram
    direction TB
    class Generic["泛型 T"] {
        <<concept>>
        输入与输出共享类型参数
        identity(T) 返回 T
        T 可附加协议约束
    }
    class Opaque["some Named 返回值"] {
        <<opaque>>
        实现方选择底层类型
        本例 make() 返回 User
        保留隐藏的类型身份
    }
    class Existential["any Named"] {
        <<existential>>
        当前值可为 User 或 Book
        暴露 Named 协议能力
        具体静态类型被擦除
    }
    class Any["Any"] {
        <<existential>>
        可保存任意类型的值
        不保证 Named 协议能力
        具体能力需检查或转换
    }
```

## 第 11 题

空心三角加虚线表示 User、Robot 遵循 Named；NamedExtension 通过依赖线为协议提供默认实现。show 只依赖 Named 要求：User 使用默认 greeting，Robot 使用自己的实现。协议与扩展的分离是示意，扩展不是可以实例化的父类。

```mermaid
classDiagram
    direction TB
    class Named["Named"] {
        <<protocol>>
        name: String get
        greeting() String
    }
    class Extension["Named extension"] {
        <<extension>>
        greeting() 默认返回问候
    }
    class User["User"] {
        <<struct>>
        name: String
        使用默认 greeting
    }
    class Robot["Robot"] {
        <<struct>>
        name: String
        greeting() 返回编号
    }
    class Show["show"] {
        <<generic>>
        T 受 Named 约束
        show(T) 调用 greeting
    }
    Named <|.. User : 遵循
    Named <|.. Robot : 遵循
    Extension ..> Named : 提供要求的默认实现
    Show ..> Named : 依赖协议要求
```

## 第 12 题

Greeter 只要求 hello；bye 位于扩展框。Person 虽然提供 hello、bye 两个同名成员，但协议视角 value 的 hello 通过遵循入口调用，bye 仍静态选扩展。具体类型 person.bye 则调用 Person.bye。虚线空心三角表示协议遵循，不把扩展画成父类。

```mermaid
classDiagram
    direction TB
    class Greeter["Greeter"] {
        <<protocol>>
        hello() String
    }
    class Extension["Greeter extension"] {
        <<extension>>
        hello() 默认 hello
        bye() 默认 bye
    }
    class Person["Person"] {
        <<struct>>
        hello() 自己的 hello
        bye() 自己的 bye
    }
    class Value["value: any Greeter"] {
        <<existential>>
        保存 Person 值
        hello() 得到自己的 hello
        bye() 得到默认 bye
    }
    Greeter <|.. Person : 遵循
    Extension ..> Greeter : 默认要求实现和扩展成员
    Value --> Person : 当前保存的具体值
    Value ..> Greeter : hello 经协议要求
    Value ..> Extension : bye 按静态信息选择
```

## 第 13 题

Dot 遵循原生 Drawable，并使用扩展提供的 draw；Button 遵循 @objc Clickable，却可以不实现 optional click。button.click?() 检查的是可选方法存在与否。本图两个空心三角都是协议遵循，NSObject 不是所有 Swift 协议遵循者必须具备的父类。

```mermaid
classDiagram
    direction TB
    class Drawable["Drawable"] {
        <<protocol>>
        draw() 必需要求
    }
    class Default["Drawable extension"] {
        <<extension>>
        draw() 默认绘制
    }
    class Dot["Dot"] {
        <<struct>>
        使用默认 draw
    }
    class Clickable["Clickable"] {
        <<objc_protocol>>
        optional click()
        仅类可遵循
    }
    class Button["Button"] {
        <<class>>
        本例继承 NSObject
        没有实现 click
    }
    Drawable <|.. Dot : 遵循
    Default ..> Drawable : 提供默认实现
    Clickable <|.. Button : 遵循但可省略 click
```

## 第 14 题

User 实际遵循的是 Encodable 与 Decodable，两者的组合名为 Codable，图中用两条遵循线表达。CodingKeys 将 JSON 的 display_name 映射到模型 name。编码器和解码器依赖模型各自的协议实现，并不是依靠反射直接给任意属性赋值。

```mermaid
classDiagram
    direction TB
    class Encodable["Encodable"] {
        <<protocol>>
        encode(to) throws
    }
    class Decodable["Decodable"] {
        <<protocol>>
        init(from) throws
    }
    class User["User"] {
        <<struct>>
        id: Int
        name: String
        nickname: String?
    }
    class Keys["User.CodingKeys"] {
        <<enum>>
        id 与 nickname 同名
        name 对应 display_name
    }
    class Decoder["JSONDecoder"] {
        <<class>>
        decode(User.Type, Data) throws
    }
    class Encoder["JSONEncoder"] {
        <<class>>
        encode(User) throws
    }
    Encodable <|.. User : 编码遵循
    Decodable <|.. User : 解码遵循
    User ..> Keys : 字段映射
    Decoder ..> Decodable : 调用解码实现
    Encoder ..> Encodable : 调用编码实现
```

## 第 15 题

实例框以原生 Swift 对象概念模型列出对象头和属性，metadata 线指向同类共享的类元数据；虚表线关联方法入口。本题代码继承 NSObject，具体头部与引用计数表示还受父类及运行时影响。Person.self 只是类型值，不是新实例。OC 元类框仅适用于 Apple 互操作模型，图省略字段偏移、父类布局和旁表，不能据此手算对象地址。

```mermaid
classDiagram
    direction TB
    class Object["Person 实例"] {
        <<runtime>>
        对象头 metadata 关联
        对象头 引用计数状态
        实例属性 age: Int
    }
    class Metadata["Person 类元数据"] {
        <<runtime>>
        父类与字段布局
        实例大小与对齐
        Swift 虚表
    }
    class VTable["Swift vtable"] {
        <<runtime>>
        可重写方法槽位
        记录实现入口
    }
    class Metaclass["OC 元类"] {
        <<objc_runtime>>
        OC 类方法等信息
        仅限支持 OC 互操作的环境
    }
    class TypeValue["Person.self"] {
        <<metatype_value>>
        类型为 Person.Type
        表示 Person 类型
    }
    Object --> Metadata : metadata
    Metadata --> VTable : 方法槽位
    Metadata --> Metaclass : Apple 互操作下的 isa
    TypeValue ..> Metadata : 语言层类型值关联
```

## 第 16 题

每个框列一条调用路径及其示例：animal.sound 通过实际 Dog 类虚表，speaker.speak 通过 Dog: Speaker 见证入口。静态框对应已知 final 目标；OC 框对应 @objc dynamic。方法实现可经适配入口继续转发，框之间不是类继承关系，也不表示固定性能排序。

```mermaid
classDiagram
    direction TB
    class Static["静态派发"] {
        <<dispatch>>
        本例 dog.speak 是 final
        编译期确定函数目标
        可直接调用并可能内联
    }
    class Virtual["虚表派发"] {
        <<dispatch>>
        animal 静态类型为 Animal
        运行时对象实际为 Dog
        查槽位得到 Dog.sound
    }
    class Witness["见证表派发"] {
        <<dispatch>>
        speaker 类型为 any Speaker
        查 Dog 对 Speaker 的遵循
        经见证入口调用 speak
    }
    class ObjC["OC 消息派发"] {
        <<dispatch>>
        本例 Demo.run 为 objc dynamic
        objc_msgSend 接收者与 selector
        缓存或方法查找得到 IMP
    }
```

## 第 17 题

容器三个分区分别指向值存储、具体类型信息与协议见证表。非类限定、单个普通方法协议的典型 64 位 ABI 可用三字缓冲区加两个指针解释；这不是所有 any 的固定布局。协议描述符描述要求，见证表实现要求；关联线只表示语义关联，不暗示表的物理相邻位置。

```mermaid
classDiagram
    direction TB
    class Container["any Speaker 容器"] {
        <<existential>>
        值缓冲区 通常三机器字
        具体类型 metadata 指针
        协议见证表指针
    }
    class Storage["具体值存储"] {
        <<storage>>
        小且对齐合适可内联
        否则关联外部分配
    }
    class Metadata["具体类型元数据"] {
        <<runtime>>
        描述当前具体类型
        关联值见证表
    }
    class VWT["Value Witness Table"] {
        <<runtime>>
        复制与移动
        销毁等生命周期操作
    }
    class PWT["Protocol Witness Table"] {
        <<runtime>>
        对应 Dog 遵循 Speaker
        speak 要求的实现入口
    }
    class Descriptor["Speaker 协议描述符"] {
        <<runtime>>
        协议身份
        协议要求描述
    }
    Container --> Storage : 值缓冲区
    Container --> Metadata : metadata
    Container --> PWT : witness 指针
    Metadata --> VWT : 值操作
    PWT ..> Descriptor : 遵循满足这些要求
```

## 第 18 题

Task 是工作及其状态，Executor 负责调度，Thread 是执行资源；三者不是同一个概念。异步函数挂起后可恢复，普通任务并不固定绑定某条线程。GCD 队列调度 block，异步提交本身也不会让提交者等待，不能把 GCD 整体等同于阻塞。

```mermaid
classDiagram
    direction TB
    class Task["Swift Task"] {
        <<runtime>>
        保存工作和取消等任务状态
        挂起时保存继续执行状态
        恢复时继续函数流程
    }
    class Executor["Executor"] {
        <<runtime>>
        调度可执行的任务片段
        遵循 actor 隔离规则
    }
    class Thread["执行线程"] {
        <<resource>>
        实际执行同步代码
        不与普通任务一一绑定
    }
    class Async["异步函数"] {
        <<function>>
        async 声明可挂起
        await 标记可能挂起点
    }
    class Queue["GCD 队列"] {
        <<runtime>>
        调度 block
        async 提交后不等待完成
        sync 调用同步等待完成
    }
    Task ..> Async : 任务内调用
    Executor ..> Task : 调度任务片段
    Executor ..> Thread : 使用执行资源
    Queue ..> Thread : 使用线程执行 block
```

## 第 19 题

实线关联表示结构化父子关系与作用域等待，虚线表示创建非结构化任务，不能沿该线推断自动取消传播。TaskHandle 框区分发出 cancel 请求、检查取消以及 await value 等待完成，避免把“取消状态”与“任务结束”混为同一时刻。

```mermaid
classDiagram
    direction TB
    class Parent["父任务作用域"] {
        <<structured>>
        async let 创建固定子任务
        TaskGroup 动态添加子任务
        退出前等待所有子任务结束
    }
    class A["子任务 A"] {
        <<task>>
        继承父子取消关系
        协作检查取消
    }
    class B["子任务 B"] {
        <<task>>
        受结构化作用域约束
        实际结束后才完成等待
    }
    class Unstructured["Task 创建的任务"] {
        <<unstructured>>
        可继承创建上下文隔离
        可活过创建作用域
        创建者取消不自动传播
    }
    class Handle["TaskHandle"] {
        <<usage>>
        cancel() 只请求取消
        checkCancellation() 可抛错
        value 等待任务实际结果
    }
    Parent --> A : 结构化子任务
    Parent --> B : 结构化子任务
    Parent ..> Unstructured : 创建而非结构化子关系
    Handle --> Unstructured : 管理取消与结果
```

## 第 20 题

Counter 用 actor 隔离 value，increment 没有 await，读改写在一个隔离片段内完成；ScreenModel 通过 MainActor 隔离 UI 状态；Snapshot 以不可变 Int 满足 Sendable。虚线空心三角表示 Snapshot 的显式 Sendable 遵循。actor 自身也满足 Sendable，这不让其内部隔离属性可被外部直接读写。

```mermaid
classDiagram
    direction TB
    class Counter["Counter"] {
        <<actor>>
        private value: Int
        increment() Int 无 await
    }
    class Screen["ScreenModel"] {
        <<MainActor>>
        private title: String
        apply(Snapshot) 更新状态
    }
    class Snapshot["Snapshot"] {
        <<struct>>
        let count: Int
    }
    class Sendable["Sendable"] {
        <<marker_protocol>>
        承诺值可安全跨隔离域使用
        不提供自动同步锁
    }
    class Reentry["actor 重入边界"] {
        <<concept>>
        await 真正挂起时可执行其他任务
        恢复时重新校验状态
        不保证整个异步方法原子性
    }
    Sendable <|.. Snapshot : 显式遵循
    Counter ..> Snapshot : 返回值用于快照
    Screen ..> Snapshot : 接收快照
    Counter ..> Reentry : 有挂起点时需考虑
```

