# 09 - Spring


### Dependency Injection
- Dependency Injection a livello di costruttore
    ```java
    public class ConstructorInjection {
    
        private Dependency dep;
        
        public ConstructorInjection(Dependency dep) {
            this.dep = dep;
        }
        
    }
    ```
- Dependency Injection a livello di costruttore
    ```java
    public class SetterInjection {
    
        private Dependency dep;
        
        public void setMyDependency(Dependency dep) {
            this.dep = dep;
        }
        
    }
    ```
- XML Bean Factory
    ```java
    XmlBeanFactory factory = new XmlBeanFactory(new FileSystemResource("beans.xml"));
    SomeBeanInterface b = (SomeBeanInterface) factory.getBean("nameOftheBean");
    ```
- File di configurazione
    ```xml
    <!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">
        <beans>
            <!-- A livello di metodo -->
            <bean id="renderer" class="StandardOutMessageRenderer">
                <property name="messageProvider">
                    <ref local="provider"/>
                </property>
            </bean>
            
            <bean id="provider" class="HelloWorldMessageProvider"/>
            
            <!-- In alternativa a livello di costruttore -->
            <bean id="provider" class="ConfigurableMessageProvider">
                <constructor-arg>
                    <value> Questo è il messaggio configurabile</value>
                </constructor-arg>
            </bean>
        </beans>
        
        ...
    ```

---

### Aspect Oriented Programming
- Classe contenente Advice
    ```java
    public class MessageDecorator implements MethodInterceptor {
        
        public Object invoke(MethodInvocation invocation) throws Throwable {
            System.out.print("Hello");
            Object retVal = invocation.proceed();
            System.out.println("!");
            return retVal;
        }
        
    }
    ```
- Configurazione AOP
    ```java
    public static void main(String[] args) {
        MessageWriter target = new MessageWriter();
        ProxyFactory pf = new ProxyFactory();
        
        // Aggiunge advice alla coda della catena dell’advice
        pf.addAdvice(new MessageDecorator());
        
        // Configura l’oggetto dato come target
        pf.setTarget(target);
        
        // Crea un nuovo proxy in accordo con le configurazioni della factory
        MessageWriter proxy = (MessageWriter) pf.getProxy();
        proxy.writeMessage();
    }
    ```

---

### Intercettori

- Classe interceptor
```java
public class ServiceMethodInterceptor implements MethodInterceptor {
    
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        long startTime = System.currentTimeMillis();
        Object result = methodInvocation.proceed();
        long duration = System.currentTimeMillis() - startTime;
        
        Method method = methodInvocation.getMethod();
        String methodName = method.getDeclaringClass().getName() + "." + method.getName();
        System.out.println("Method '" + methodName + "' took " + duration + " milliseconds to run");
        return null;
    }
    
}
```
- Configurazione
```xml
<beans>
    <bean id="myService" class="com.test.MyService"/>
    <bean id="interceptor" class="com.test.ServiceMethodInterceptor"/>
    <bean id="interceptedService" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target">
        <ref bean="myService"/> </property>
        <property name="interceptorNames">
            <list>
                <value>interceptor</value>
            </list>
        </property>
    </bean>
</beans>
```

---

### Gestione ciclo di vita

- ApplicationContextAware
```java
public class Publisher implements ApplicationContextAware {
    
    private ApplicationContext ctx;
    
    // Questo metodo sarà automaticamente invocato da IoC container
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.ctx = applicationContext;
    }
    
    ...
}
```

---

### Gestione eventi

- Configurazione
```xml
<bean id="emailer" class="example.EmailBean">
    <property name="blackList">
        <list>
            <value>black@list.org</value>
            <value>white@list.org</value>
            <value>john@doe.org</value>
        </list>
    </property>
</bean>

<bean id="blackListListener" class="example.BlackListNotifier">
    <property name="notificationAddress" value="spam@list.org"/>
</bean>
```
- Classe che pubblica eventi
```java
public class EmailBean implements ApplicationContextAware {
    
    private List blackList;
    
    public void setBlackList(List blackList) {
        this.blackList = blackList;
    }
    
    public void setApplicationContext(ApplicationContext ctx) {
        this.ctx = ctx;
    }
    
    public void sendEmail(String address, String text) {
        if(blackList.contains(address)) {
            BlackListEvent evt = new BlackListEvent(address, text);
            ctx.publishEvent(evt);
        }
    }
}
```
- Classe che riceve eventi
```java
public class BlackListNotifier implements ApplicationListener {
    
    private String notificationAddress;
    
    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }
    
    public void onApplicationEvent(ApplicationEvent evt) {
        if(evt instanceof BlackListEvent) {
            // Invio dell’email di notifica all’indirizzo appropriato
        }
    }
}
```