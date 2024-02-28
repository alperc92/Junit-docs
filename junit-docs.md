# JUnit 5 and Mockito in relation with Spring Boot

## Typical Test Class definition
```
public class <Class>Test {
    //Field properties, like in usual Class definition
    private <Object> <name>;
    ..


    //include all setups here, also for initiating the field properties
    @BeforeAll  //Replaces @BeforeClass in JUnit 4. PS! must be static!
    static void beforeAll(){ 
        //typically, define Mocks and Spy objects.
        
    } 

    //or if it needs to be setup one for each method
    @BeforeEach //replaces @Before in JUnit 4.
    public void setup() {        
        //typically, define Mocks and Spy objects.
    }

    @AfterEach //Replaces @After in JUnit 4.
    void tearDown() {
    // cleanup code here
    }


    @AfterAll //Replaces @AfterClass in Junit 4. PS! must be static!
    static void afterAll() {
    // one-time cleanup code here


    @Test
    void methodName_WhenCondition_ThenExpectedBehavior(){}      //Method naming 1: Camel Case

    @Test
    void givenPrecondition_whenAction_thenExpectedOutcome(){}   //Method naming 2: Given-When-Then
   }
}
```



## Object initiation methods for testing
### 1. `new <object>()` 

### 2. `@Autowired` for SpringBoot classes, like `@Configuration`, `@Component`, `@Service`
If you are going to test a Spring Boot class that is initiated as through SpringBoot DI container, you can inject the class using `@Autowired`

### 3. `spy(new <object>())` Mockito framework - A partial mocking

### 4. `mock(<object>.class)` Mockito framework


### **Example: Mocking and stubbing class object outcomes**
```
public class Generator{

   private Molecyle m;
   private Bolecyle b;

   private Dependency2 dependency2;
   public Generator(){
    this.m = new Molecyle();
    this.b = new Bolecyle();
   }

   public boolean generate(Molecyle m, Bolecycle b){
     m.bla().. //want to stub this.
     b.bla().. //want to stub this.
     
     if(m .. && b ..) return false; //want to stub this by mocking object m and b to whatever neeeded to bypass this if statements.

     ..

     return true; //want to test this outcome

   }

}

public class GeneratorTest{
    ..
    @BeforeEach
    void setup(){        
        generator = new Generator(dependency1, dependency2)
    }

    @Test
    void testGenerate(){
        //Arrange
        Molecyle m = mock(Molecyle.class);
        Bolecyle b = mock(Bolecyle.class);

        //arrange the mocks to bypass the if in the method generate by returning the needed values; and then test the rest until the return statement.
        when(m.bla()).thenReturn(..);
        when(b.bla()).thenReturn(..);

        //Act
        boolean result = generator.generate(m, b);

        //Assert
        assertThat(result, is(true));
    }

}

```




### **Example: Basic class method and injections**
```
public class Generator{

   private Dependency1 dependency1;

   private Dependency2 dependency2;

   @Autowired
   public Generator(Dependency1 dep1, Dependency2 dep2){
    ..
   }

   public void generate(){
     dependency1.doSomething();
     dependency2.doSomething();
     ..
   }

   public void manipulate() throws GeneratorException {
     String val1 = dependency1.getVal1();
     Boolean val2 = dependency1.getVal2();

     if(val2){
        dependency1.updateVal1(val1)
     }
     else
       throw new GeneratorException("getVal2 doesnt return true!");        
        
   }

}

public class GeneratorTest{
    ..

    @BeforeClass
    void beforeAll(){        
        ..
    }
    @BeforeEach
    void setup(){
        dependency1 = mock(Dependency1.class);
        dependency2 = mock(Dependency2.class);
        generator = new Generator(dependency1, dependency2)
    }

    @Test
    void testGenerate(){
        //Arrange
        ..     

        //Act
        generator.generate();

        //Assert
        verify(dependency1).doSomething(); //verify doSomething was called on dependency1
        verify(dependency2).doSomething(); //verify doSomething was called on dependency2
    }

    @Test
    void testManipulate(){
        //Arrange
        when(dependency1.getVal1()).thenReturn("just a String");
        when(dependency1.getVal2()).thenReturn(true);

        //Act
        generator.manipulate();

        //Assert
        verify(dependency1).updateVal1(anyString()); //verify updateVal1 to be called
    }

    @Test
    void testManipulateFalseGetVal2RaiseGeneratorException(){
        //Arrange
        when(dependency1.getVal1()).thenReturn("just a String");
        when(dependency1.getVal2()).thenReturn(false);

        //Act & Assert
        assertThrows(GeneratorException.class, () -> generator.manipulate());

        //Assert
        verify(dependency1, never()).updateVal1(anyString()); //verify that updateVal1 didnt get called.

        
    }
}

```
### **Example: Spy class method**
If you want to test a class that can be partially mocked, because you want to test some specific methods rather than the whole functionality because the constructor is doing some configuration like network calls and accessing a database, that parts can be stubbed.
```
public class Generator{

   public Generator(){
    ..
   }

   public boolean generate(){
     Network = createNetwork(..) //want to stub this call.


     .. // want to test the rest.
     
     
   }

   public Network createNetwork()..

}

public class GeneratorTest{
    ..
    @BeforeEach
    void setup(){
        generator = spy(new Generator())

        //stub the createNetwork method
        CreateNetwork network = mock(Network.class)
        doReturn(network).when(generator.createNetwork());
    }

    @Test
    void testGenerate(){
        //Act
        boolean result = generator.generate();

        //Assert
        assertTrue;

    }
}
```

### **Example: Argument Captors**
In methods, that can have different calls to other methods, it can be a nice feature to use argument captors, to check if some arguments given initially to a method doesnt get compromised.

Lets take an example:
```
public class Generator{

   private Service1 service1;

   private Service2 service2;

   @Autowired
   public Generator(Service1 ser1, Service2 ser2){
    ..
   }

   public boolean generate(Obj1 obj1, Obj2 obj2){

     ..
     service1.blabla(obj1);
     service2.blabla(obj2); 
     ..
     
   }

}

public class GeneratorTest{
    private Generator generator;
    private Service1 service1;
    private Service2 service2;

    @BeforeEach
    void setup(){
        service1 = mock(Service1.class);
        service2 = mock(Service2.class);
        generator = new Generator(service1, service2)
    }

    @Test
    void testGenerate(){
        //Arrange
        Object1 obj1 = new Object1();
        Object1 obj2 = new Object2();

        //Act
        boolean result = generator.generate(obj1, obj2);

        //Assert
        ArgumentCaptor<Object1> arg1 = ArgumentCaptor.forClass(Object1.class);
        ArgumentCaptor<Object2> arg2 = ArgumentCaptor.forClass(Object2.class);

        verify(service1, times(1)).blabla(arg1.capure());
        verify(service2, times(1)).blabla(arg2.capure());

        
        assertThat(arg1.getValue(), is(obj1));
        assertThat(arg2.getValue(), is(obj2));

    }
}
```


### NOTES:
1. When stubbing a method of a spy in Mockito, you should use `doReturn().when().methodName()` syntax instead of `when().thenReturn()` which is typically used with mocks. This is because `when().thenReturn()` calls the actual method on a spy, which might not be desirable when stubbing.


### Matcher (Hamrest library)
study it; it does have nice features.
1. assertThat
    - isInstanceOf
    - is 