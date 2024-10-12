article:: true

- DOING complete me
  :LOGBOOK:
  CLOCK: [2024-10-12 Sat 15:59:51]
  :END:
- 之前学习过hibernate Validator，但当时做的笔记弄丢了，最近可能又要开始写Java，所以把这一部分学习一下，做个笔记。
- Hibernate Validator用于值校验，能够避免业务中过多地出现校验业务代码。Hibernate Validator支持对自定义类型，集合类型和内置Java类型进行校验，同时支持定义校验组，即让类型约束从属于特定组，只在进行该组的校验时才发挥作用。
- 为什么要使用Hibernate Validator：
	- 声明式，避免到处写丑陋的样板代码
	  logseq.order-list-type:: number
	- 更清晰地规范实体定义，校验注解本身就是一种注释
	  logseq.order-list-type:: number
	- 支持自定义校验逻辑
	  logseq.order-list-type:: number
- # 环境搭建和基本使用
- 通过下面的starter引入hibernate validator依赖：
- ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```
- Hibernate Validator利用**切面**完成自己的逻辑，它根据**类**上的`@Validated`注解去进行切入，并根据**参数**上的注解去进行校验。总的来说，要让校验起效，需要：
	- `spring-boot-starter-validation`依赖引入
	  logseq.order-list-type:: number
	- 类上标注`@Validated`（**要且必须要在类上标注！必须是`@Validated`而非`@Valid`**）
	  logseq.order-list-type:: number
	- 方法的参数上标注相应校验注解（其中，实体类参数使用`@Valid`或`@Validated`注解）
	  logseq.order-list-type:: number
	- 调用方法时不能从内部调用（切面嘛，懂的都懂）
	  logseq.order-list-type:: number
- 注意，Hibernate Validator通过切面工作，因此**它不仅能切入Controller，也能切入Service**，但仅此而已，某些时候还是需要手动进行校验。
- 下面是一个极简例子，涉及到控制器和实体类的校验，它已经提出了许多要注意的部分：
- ```java
  @RestController
  @Validated
  public class SomeController 
      // 定义带校验的实体类（注意实体类上不需要加任何额外注解，加注解也是没用的）
      @Data
      public static class SomeDto {
          @NotBlank // 对于实体类，注解加到字段上
          private String name;
  
          @NotBlank @Length(min=10) // 可以同时有多个校验 注解
          private String value;
  
          @Valid // 对实体类用 Valid 进行嵌套校验（注意Valid在字段为null的时候不检查！）
          private SomeDto next;
  
          private List<@Valid SomeDto> dtos; // 对集合进行校验（注意这里不会检查集合是否为null或空！同时也没检查集合元素是否为null！）
      }
      @PostMapping("/query")
      public SomeDto query(
              // 实体类参数用 @Valid 或 @Validated注解进行校验
              @Valid @RequestBody SomeDto dto,
              @NotBlank @Length(min=10) @RequestParam String param) {
          return dto;
      }
  }
  
  ```
- `@Valid`校验能进行嵌套校验，但**它会直接忽略掉null**，因此上面的`SomeDto`定义中，`next`字段为null，以及`dtos`字段为null或空集合，或集合中存在null，都是容忍的。
- Controller的参数校验似乎并非完全是通过切面完成的——即使Controller上未加`@Validated`注解，`@Valid`注解仍旧会生效，但是其它校验注解不会生效，因此**最佳实践是，总是在类上加`@Validated`，不要嫌麻烦**。
- # 关于Service的校验
- Hibernate Validator有一条规则：`A method overriding another method must not redefine the parameter constraint configuration`，它是说，**子类无法覆盖掉父类上的校验注解，即使父类上没有校验注解**。
- 上面的规则同时暗示了，Hibernate Validator的注解是能够**继承**的。一般而言，Spring项目中Service的接口和实现是分离的，如果要**校验Service**，根据上面的规则，我们应当：
	- 在接口上标注`@Validated`注解
	  logseq.order-list-type:: number
	- 在接口上的方法参数中添加相应校验注解
	  logseq.order-list-type:: number
	- **在实现上不需要添加任何注解**，或者保证实现上的注解和接口上的**完全相同**。
	  logseq.order-list-type:: number
- 初看感觉这个要求不太合理，但细想其实还好——按理来说，接口内部使用何种实现对接口的调用者是透明的，因此**值的约束必须是定义自接口层级上的，实现对值的约束只能更宽，不能更窄**，而我们无法判断约束的宽窄，所以就硬性要求它们保持一致。但其实作为业务的开发者来说，还是希望能够将注解只写在实现上。
- # 常用校验注解
- 这里列出可能会常用的注解，注意几乎所有注解都认为null是合法的。注解主要在`org.hibernate.validator.constraints`和`javax.validation.constraints`包下。
- |注解|作用|坑|
  |--|--|--|
  |Email|检查邮箱是否合法|null合法|
  |Past, Future, ...|时间是否是过去或未来|null合法|
  |Pattern|字符串必须满足正则|null合法|
  |Size|字符串长度或集合必须满足特定大小范围|null合法，大小区间前闭后闭|
  |Min, Max, Positive, Negative...|限制数字的最小值，最大值，正负性等|null合法，注意不要用Min和Max限制字符串长度，这个能启动，但**运行时会报错**|
  |Length|字符串长度必须在特定范围|null合法，前闭后闭|
  |Null|约束字段必须为null||
  |NotNull|约束字段必须不能为null||
  |NotEmpty|集合或字符串不能为null且非空||
  |NotBlank|字符串不能为null且必须包含非空字符||
- # 手动校验实体类
- 有时候可能会想要进行手动校验，**比如我们可能会想写mybatis拦截器，在插入和更新数据前进行校验**，手动校验可以利用Spring提供的 Bean`javax.validation.Validator`（这是它对JSR规范的实现），或者Bean `org.springframework.validation.SmartValidator`，前者返回"vioiations"，即实体对象对约束的违反，**后者是Spring的默认行为**，在底层利用前者，通过抛出异常的方式来代表违反。前后者都可以使用，下面的例子使用前者：
- ```java
  @Autowired
  private javax.validation.Validator validator;
  
  public void test() {
    SomeDto dto = new SomeDto();
    dto.setName("");
    dto.setValue("");
    Set<ConstraintViolation<SomeDto>> violations = validator.validate(dto);
    if (violations.isEmpty()) {
      // 校验成功，...
      return;
    }
    for (ConstraintViolation<SomeDto> violation : violations) {
      System.out.println(violation.getMessage());
    }
  }
  ```
-
- # 拦截校验异常
- # 自定义校验
- # 分组校验