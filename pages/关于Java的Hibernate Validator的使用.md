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
- 有时候可能会想要进行手动校验，**比如我们可能会想写mybatis拦截器，在插入和更新数据前进行校验**，手动校验可以利用Spring提供的 Bean`javax.validation.Validator`（这是它对JSR规范的实现），它返回"vioiations"，即实体对象对约束的违反。
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
- Hibernate Validator会抛出如下异常：
	- `org.springframework.web.bind.MethodArgumentNotValidException`，抛出在Controller的`@Valid`或`@Validated`注解的实体类参数（这简直就是历史遗留问题），默认响应码是400，消息是"Bad Request"
	  logseq.order-list-type:: number
	- `javax.validation.ConstraintViolationException`，其他情况，默认响应码是500，消息是"Internal Server Error"，这是符合道理的——**控制器的参数错误是用户的错误，服务层的参数错误是开发者的错误**。
	  logseq.order-list-type:: number
- 这两个异常都需要被拦截才能妥善把校验信息响应给前端……但这样真的好吗？全给到前端不是会让坏家伙有可乘之机吗？总之贴上（实际操作时应当像ruoyi那样，正常响应和错误响应形式保持一致）：
- ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
  
      // 处理 ConstraintViolationException
      @ExceptionHandler(ConstraintViolationException.class)
      public ResponseEntity<?> handleConstraintViolationException(ConstraintViolationException ex, WebRequest request) {
          Map<String, String> errors = new HashMap<>();
          ex.getConstraintViolations().forEach(violation ->
                  errors.put(violation.getPropertyPath().toString(), violation.getMessage())
          );
          return new ResponseEntity<>(errors, HttpStatus.INTERNAL_SERVER_ERROR);
      }
  
      // 处理 MethodArgumentNotValidException
      @ExceptionHandler(MethodArgumentNotValidException.class)
      public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
          Map<String, String> errors = new HashMap<>();
          String parameterName = ex.getParameter().getParameterName();
          ex.getBindingResult().getFieldErrors().forEach(error ->
                  errors.put(parameterName + "." + error.getField(), error.getDefaultMessage())
          );
          return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
      }
  
      // 处理其他异常
      @ExceptionHandler(Exception.class)
      public ResponseEntity<?> handleGlobalException(Exception ex, WebRequest request) {
          // ...
      }
  }
  ```
- # 自定义校验注解
- 有时候需要自定义校验，**比如要校验身份证**，这个肯定没有被提供，需要手写。这时候需要自定义Validator和注解。
- Validator不关心它被标注在字段上还是标注到参数上，**它直接拿到值然后去做校验，但同时也允许获取当前的字段路径等信息以构建错误消息**。
- Spring Boot会惰性地为**为每个不同的注解创建相应的Validator实例**，从而让每个Validator都负责同一个注解（类型相同，且所有字段值相同），避免任何并发问题，同时支持在创建Validator时注入依赖。
- 编写自定义校验注解需要：
	- 创建自定义注解，注解需要是Runtime的，需要能够标注到字段和参数上（**也可以让注解能标注到类上，这允许对整个类进行校验**），注解需要引用下一步中编写的自定义Validator，**注解必须包含groups，message, payload字段（可以直接抄现成的）**
	  logseq.order-list-type:: number
	- 创建自定义Validator，如果注解能够校验多种类型，则每个类型都需要一个Validator，Validator类要实现`ConstraintValidator<注解, T>`，**在构造函数中注入Spring Bean依赖，在`initialize`方法中注入注解，并在`isValid`中线程安全地进行校验**。
	  logseq.order-list-type:: number
	- 实现`isValid`时，第一个参数是字段值，第二个参数是当前上下文，`isValid`方法返回true或false，true表示校验通过，false表示不通过，此时hibernate validator会根据上下文去构造相应violation，此时可以自定义错误消息。
	  logseq.order-list-type:: number
- 下面编写一个身份证校验注解，其中演示了如何在校验过程中获取Bean使得能和系统其他部分进行交互，以及如何修改错误消息。
- ```java
  /**
   * 校验是否合法身份证，null 认为是合法
   */
  @Target({ FIELD, PARAMETER })
  @Retention(RUNTIME)
  @Documented
  @Constraint(validatedBy = IsIdCard.IsIdCardValidator.class)
  public @interface IsIdCard {
      String message() default "Invalid Id Card"; // 这个消息会是默认消息，它里面能使用{}插值注解上的字段
      Class<?>[] groups() default { };
      Class<? extends Payload>[] payload() default { };
      // 不需要，也不应该标注 @Component
      class IsIdCardValidator implements javax.validation.ConstraintValidator<IsIdCard, String> {
          private final SomeService someService;
          private IsIdCard anno;
          // 通过构造函数注入bean
          public IsIdCardValidator(SomeService someService) {
              this.someService = someService;
          }
          // 通过initialize方法注入注解
          @Override
          public void initialize(IsIdCard constraintAnnotation) {
              this.anno = constraintAnnotation;
          }
  
          // 实际校验操作
          @Override
          public boolean isValid(String value, ConstraintValidatorContext context) {
              // 修改掉原来的violation信息
              context.disableDefaultConstraintViolation(); // 移除掉原来的violation信息
              context.buildConstraintViolationWithTemplate("身份证不合法：" + value)
                      .addConstraintViolation();
  
              // 遵循规范，null认为合法
              if (value == null) {
                  return true;
              }
  
              return isValidIdCard(value);
          }
          private static boolean isValidIdCard(String idCard) {
              // ...
          }
      }
  }
  ```
- # 分组校验
- 所有校验注解都有`groups`参数，表示校验所属的校验组。在进行校验时，通过`@Validated`的groups参数指定只校验特定组的注解（注意它标识在参数上时表示只校验这些组的注解，标识在字段上时表示该校验属于这些组，这是两种不同的语义）。校验组使用**任意Class**进行标识，这些Class不需要任何实际操作。