# BDD 行为驱动测试

这种常用于用户验收测试(UAT),呈现给项目利益相关者。
BDD也是敏捷的一种。

她使用一种叫DSL(领域特定语言)来描述，这个dsl就是一个自然语言。

不同的编程语言都实现了自己的bdd工具，
现在主流的都是ruby/js/java的bdd框架，go对应的是goConvey，
这个也被集成到chrome中了。

下一步就是重点去了解goConvey的用法。goconvey已经有了初步了解。
下一步就是了解通用的DSL对BDD的描述语法。

## wiki的introduce

在软件工程中，BDD是一种敏捷开发，让参与者更好的沟通。
参与者包括开发/QA/非技术人员/商业伙伴。

BDD主要是让这些参与者对"程序应该是什么样的"有统一的共识。

BDD来至TDD，原则上，BDD是从商业视角和技术视角来看待软件开发。

BDD的一个原则就是使用自然语言来描述BDD，就像之前学习的goconvey就是go来描述dsl，

## bdd历史

bdd来至tdd，bdd使用dsl更加简单，dsl会将自然语言转换成可执行的测试例子。

BDD关注的重点是：

1. 从哪儿开始执行
2. 要测试什么，不测试什么
3. 一次测试多少
4. 什么调用测试
5. 如何定义一个测试的失败

BDD的本质是重新考虑单元测试和验收测试的方法，以避免自然发生的问题。
eg：单元测试命名应该以动词开头，是一个完整的句子，按商业价值顺序编写;
验收测试应该用敏捷框架下的用户故事来写(作为xx，想要xx，结果xx)。
验收标准应该根据场景编写，要符合GTW规则。

GTW： Given [initial context], when [event occurs], then [ensure some outcomes]。
这是一种半结构化的方式取写测试用例，可以手工测试或用selenium进行浏览器自动测试。
下面会继续介绍GTW。

现在已经有很多人开发了BDD框架，给参与者使用。goconvey就是其中之一。
除了goconvey，还有很多有名的bdd框架，这里就不细说了。

## bdd原理

tdd是一种软件开发方法，对于每个软件单元：

- 先为每个软件单元定义一个测试集
- 让测试失败 - tdd 红灯
- 实现单元 - tdd绿灯
- 最后验证，让实现过的单元通过测试集

其实，tdd就是"红灯-绿灯-重构"的重复,这个定义很宽泛，
上到高级别的需求，下到底层的技术细节，用tdd都可以涵盖，
bdd就是在这种情况下发展的。

bdd规定，应该根据软件的行为来指定单元测试。
她借用敏捷里的"预期行为"，来满足商业上的需求，这样保证软件可以满足商业上的需求，
也就是保证了软件的商业价值，这就是BDD被称为"由外而内"的道理。

## 行为规格

基于bdd的原理，下一步就是如何指定"预期行为"。

bdd借用了敏捷里的用户故事中的"面向对象分析和设计"部分，使用了一种半正式的格式，
这种格式(猜到了吧，就是上面的GTW)，要使用程序的语言来实现对应的dsl。

基于这种格式，业务分析师和开发人员可以对需求达成共识。

下面介绍这种格式：

    Title  一个明确的标题
    Narrative 一个简短的介绍
      As a: 一个角色，可以从这个功能收益的人
      I want: 功能
      so that: 收益，或这个功能的价值
    Acceptance criteria 验收标准，对介绍中每个具体场景的描述，下面是结构：
      Given: 场景开始前的上下文初始化，可以有一个或多个
      when: 触发场景的事件
      then: 预期输出，可以有一个或多个

bdd对用户故事如何写，没有特别的格式要求，每个团队内部有有自己的一套标准。

下面是bdd大牛推荐的一个模版：

    Title: Returns and exchanges go to inventory.

    As a store owner,
    I want to add items back to inventory when they are returned or exchanged,
    so that I can track inventory.

    Scenario 1: Items returned for refund should be added to inventory.
    Given that a customer previously bought a black sweater from me
    and I have three black sweaters in inventory,
    when they return the black sweater for a refund,
    then I should have four black sweaters in inventory.

    Scenario 2: Exchanged items should be returned to inventory.
    Given that a customer previously bought a blue garment from me
    and I have two blue garments in inventory
    and three black garments in inventory,
    when they exchange the blue garment for a black garment,
    then I should have three blue garments in inventory
    and two black garments in inventory.

这个模版中使用固定的格式来描述，用类似用户故事的方式来表述。

理想情况下，用业务语言以声明式而非强制性来表达场景，
此时没有引用任何交互性ui元素。

在很多流行的bdd框架工具中，上面这种语法被称为"小黄瓜语言"。

## 用任何语言来表示规格

bdd从ddd(domain driven design领域驱动设计)从借用了一个概念：ubiquitous,
无处不在，也可以理解为自然的。

一个ubiquitous语言(自然语言)，是一种半正式语言，共享于软件开发团队，
包含了软件的开发者和非技术人员。这种语言可以作为一种共识来讨论软件中的问题，
这种语言在这个问题域可以被使用，也可以用于开发。这样，bdd才能称为
项目中所有人交流的通讯工具。

一般，商业利益相关着和开发者在沟通时，会存在很大的风险，bdd使用自然语言来
达成共识。这就是bdd坚持使用半正式规范的原因。

上面的例子中，bdd创建了一个用户故事，其中有利益相关者/业务效果/业务价值。
用例还描述了前提/触发/预期结果。

大多数bdd程序使用基于文本的dsl和规范方法。

## 有很多bdd工具都很专业

下面说下这些工具的原理

支持bdd的工具，一般都是一个测试框架，这点和支持tdd的工具类似。
tdd试图使用自由格式，bdd倾向于使用自然语言(就是半正式语言)

bdd中的自然语言，可以让业务需求分析员来写行为需求规格书，
通过这个和开发人员进行交流。支持bdd的工具，就是要将这个文档转换成
一个可执行的测试集合。

支持bdd的工具的执行过程一般如下：

- 工具读(加载)规格书
- 解析自然语言，将不同的场景解析为不同的条款
- 场景下的不同条款转换成测试的不同参数
- 框架执行每个场景的测试，参数就使用上一步的参数

## 用户故事和规格

支持bdd的工具，识别的是规格，而不是用户故事

栈的规范如下：

    Specification: Stack

    When a new stack is created
    Then it is empty

    When an element is added to the stack
    Then that element is at the top of the stack

    When a stack has N elements
    And element E is on top of the stack
    Then a pop operation returns E
    And the new size of the stack is N-1

上面这种规格对业务用户意义不大，这更多的针对组件的测试。
在bdd中，基于规格的测试是对基于用户故事测试的补充，且在底层操作。
规格测试，时常被看作是单元测试的代替。

## 3个朋友

3个朋友，也指规格工作室。是关于用例子讨论规格的会议，
会议的参与者包括产品所有者/QA/开发团队。
会议主要目的是触发会议，识别遗失的规格。是三方沟通的一个机会。

3个朋友指：

- 商业相关，角色是商业用户，只提问题，不提任何解决方案
- 开发，角色是解决问题的开发者
- 测试，角色是测试解决方案的测试者。帮助解决方案更加精准。
