<font size="10" face="Verdana" color="red">Spring-Boot-Validation</font>

### 什么是Validation
在我们的RESTful Service中帮助我们完成期望的数据校验,通过配置Validation可以很轻松的完成对数据的约束
在我们接收到不想要的数据时:

- 返回错误的状态码和错误的提示信息
- 在响应的结果中去除敏感信息
- .....

#### 错误的状态码
返回的响应码推荐使用400->bad request.

#### 引导使用正确的REST资源
通过提供的报错信息来引导合理使用restful服务的资源,并且完成CRUD方法的调用.

> 通过Validation我们可以使用一种相同的模板方法来完成异常控制

#### 在SpringBoot中使用Validation

##### 错误的相应类型
如果你是用的是application/xml的处理类型,SpringBoot将默认返回状态码为415的Media Type.

##### 无效的JSON内容
如果你发送了一个无效的JSON内容,你会的到状态码为400的Bad Request.

##### 数据缺失的JSON
如果你的请求JSON中缺失数据,springboot将返回201状态码给你.

### 自定义Validation
接下来将使用Hibernate自带的Validatior来自定义一套Bean的Validator API.

在SpringBoot的项目中使用Hibernate的Validator是非常容易的

#### 在Bean上通过Validations 注解实现
使用@Size注解来指定数据的长度和报错是提示的内容
```java
@Entity
public class Student {
  @Id
  @GeneratedValue
  private Long id;
  
  @NotNull
  @Size(min=2, message="Name should have atleast 2 characters")
  private String name;
  
  @NotNull
  @Size(min=7, message="Passport should have atleast 2 characters")
  private String passportNumber;
```
 这里将可以使用到的注解放在下面一遍查看[也可以看我的另一篇博客](https://blog.csdn.net/qq_19663899/article/details/79386043)
 ```
空检查

@Null       验证对象是否为null

@NotNull    验证对象是否不为null, 无法查检长度为0的字符串

@NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.

@NotEmpty 检查约束元素是否为NULL或者是EMPTY.

 

Booelan检查

@AssertTrue     验证 Boolean 对象是否为 true  

@AssertFalse    验证 Boolean 对象是否为 false  

 

长度检查

@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  

@Length(min=, max=) Validates that the annotated string is between min and max included.

 

日期检查

@Past           验证 Date 和 Calendar 对象是否在当前时间之前  

@Future     验证 Date 和 Calendar 对象是否在当前时间之后  

@Pattern    验证 String 对象是否符合正则表达式的规则

 

数值检查，建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，但可以转换为Stirng为"",Integer为null

@Min            验证 Number 和 String 对象是否大等于指定的值  

@Max            验证 Number 和 String 对象是否小等于指定的值  

@DecimalMax 被标注的值必须不大于约束中指定的最大值. 这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度

@DecimalMin 被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度

@Digits     验证 Number 和 String 的构成是否合法  

@Digits(integer=,fraction=) 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度。

 

@Range(min=, max=) Checks whether the annotated value lies between (inclusive) the specified minimum and maximum.

@Range(min=10000,max=50000,message="range.bean.wage")
private BigDecimal wage;

 

@Valid 递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验.(是否进行递归验证)

@CreditCardNumber信用卡验证

@Email  验证是否是邮件地址，如果为null,不进行验证，算通过验证。

@ScriptAssert(lang= ,script=, alias=)

@URL(protocol=,host=, port=,regexp=, flags=)
 ```

#### 在请求的时候进行Valid配置
**post请求**对于post在使用@Valid的时候需要在注解后跟随@RequestBoday,如果使用url传参,使用@Valid的时候也需要使用@RequestParam

```java
public ResponseEntity<Object> createStudent(@Valid @RequestBody Student student) {
```
**get请求**同理,直接使用@Valid+@RequestParam就可以了
这是对接口发出请求,我们可以看到服务返回了404的Bad Request
```java
@GetMapping("/type/{type}/{year}")
	public Object selectRootByType(@NotNull(message = "结构层级不能为空！") @PathVariable("type") ExamTypeEnum type,
			@NotNull(message = "年份不能为空！") @PathVariable("year") Integer year) 
```
```
  {
    "name": "",
    "passportNumber": "A12345678"
  }
```
 但是并没有返回给我们相关的提示信息.
 - 消费者知道这是一个Bad Request
 - 但是他们不知道为什么错了?那个元素出错了?为了解决这个错误该如何解决呢?

#### 自定义Validation Response
首先需要创建一个简单的错误相应Bean
```java
public class ErrorDetails {
  private Date timestamp;
  private String message;
  private String details;

  public ErrorDetails(Date timestamp, String message, String details) {
    super();
    this.timestamp = timestamp;
    this.message = message;
    this.details = details;
  }
```

 ###### 通过继承ResponseEntityExceptionHandler来自定义异常处理类,需要在类上添加注解@ControllerAdvice
 ```java
 @ControllerAdvice
@RestController
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
 ```

###### 通过ErrorDetails来封装错误信息并且将错误信息通过json的形式返回给前段进行展示
```java
@ControllerAdvice
@RestController
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
  @ExceptionHandler(StudentNotFoundException)
  public final ResponseEntity<ErrorDetails> handleUserNotFoundException(StudentNotFoundException ex, WebRequest request) {
    ErrorDetails errorDetails = new ErrorDetails(new Date(), ex.getMessage(),
        request.getDescription(false));
    return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);
  }
```
##### 如果请求的时候提供一个错误的参数,你讲收到后台发来的提示
```
{
    "name": "",
    "passportNumber": "A12345678"
  }
```
响应结果
```
{
    "timestamp": "2018-12-13T14:02:24.066+0000",
    "message": "Validation Failed",
    "details": "org.springframework.validation.BeanPropertyBindingResult: 1 errors\nField error in object 'student' on field 'name': rejected value []; codes [Size.student.name,Size.name,Size.java.lang.String,Size]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [student.name,name]; arguments []; default message [name],2147483647,2]; default message [用户名长度不能小于2]"
}
```
### 自定义Validators
JSR 303验证提供可很多默认的验证模式,但是有的时候我们还是需要根据自己的需求自定义验证器
> javax 提供了一个validation包用来帮助我们完成参数校验

如果想要实现自己的Validator就必须要实现ConstraintValidator,实现这个类可以帮助我们在解析参数时通过@Valid标注的方法参数进行验证
```java
public class InRangeValidator implements ConstraintValidator<InRange, Integer> {

    private int min;
    private int max;

    /**
     * 用来完成将注解中的内容初始化
    */
    @Override
    public void initialize(InRange constraintAnnotation) {
        this.min = constraintAnnotation.min();
        this.max = constraintAnnotation.max();
    }

    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext context) {
        return value == null || (value >= min && value <= max);
    }
}
```
实现自己的注解也是很容易,只需要添加Constraint注解来实现对注解的约束,同时定义默认值和默认的错误消息
```java
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = { InRangeValidator.class })
public @interface InRange {
    String message() default "Value is out of range";
 
    Class<?>[] groups() default {};
 
    Class<? extends Payload>[] payload() default {};
 
    int min() default Integer.MIN_VALUE;
 
    int max() default Integer.MAX_VALUE;
}
```

### 高级使用
@Validator一系列注解不仅仅给我们提供了默认的信息提示以及相应的参数比较.同时还在注解中定义了groups的概念,通过引入自定义interface可以创建一些显示的声明从而帮助我们完成更好的验证

#### 设置分组
在注解参数Group中添加interface的方法名来区分组别
```java
  @PostMapping
    public Object insert(@RequestBody @Validated(value = {ValidateGroups.ValidateSave.class}) PositionInfoDto.Save save) {
        return testService.insert(save);
    }
```