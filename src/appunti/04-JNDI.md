# 04 - JNDI


### Lookup
- Esempio lookup su RMI
    ```java
    Properties prop = new Properties();
    prop.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.rmi.registry.RegistryContextFactory");
    prop.put(Context.PROVIDER_URL, "rmi://lia.deis.unibo.it:5599");
    Context cxt = new InitialContext();
    IntRemota cxt.lookup("lia.deis.unibo.it:5599/ServerRemoto");
    ```
- Altro esempio di lookup
    ```java
    // Impostazione naming service provider e url del servizio
    Hashtable hashtableEnvironment = new Hashtable();
    hashtableEnvironment.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
    hashtableEnvironment.put(Context.PROVIDER_URL, "ldap://localhost:389/dc=etcee,dc=com");
    hashtableEnvironment.put(Context.SECURITY_PRINCIPAL, "name");
    hashtableEnvironment.put(Context.SECURITY_CREDENTIALS, "password");
    
    // Ottenimento initial context
    DirContext dircontext = new InitialDirContext(hashtableEnvironment);
    
    // Operazione di salvataggio su sistema di naming
    context.bind("name", object);
    
    // Operazione di lookup
    Object object = context.lookup("name")
    ```
---

### Tipi di Annotation personalizzati
- Definizione annotazione
    ```java
    public @interface GroupTODO {
      public enum Severity {CRITICAL, IMPORTANT, TRIVIAL};
      
      Severity severity() default Severity.IMPORTANT;
      String item;
      String assignedTo;
    }
    ```
- Utilizzo annotazione
    ```java
    @GroupTODO(severity=GroupTODO.Severity.CRITICAL, item="item name", assignedTo="nome persona")
    public void calculateInterest(float amount, float rate) {
      ...
    }
    ```

---

### Riferimenti fra Annotation
- Annotation 1
    ```java
    public @interface Trademark {
        String description();
        String owner();
    }
    ```
- Annotation 2
    ```java
    public @interface License {
        **Trademark[] trademarks();**
        String name();
        String notice();
        boolean redistributable();
    }
    ```
- Utilizzo Annotation 2
    ```java
    @License(
        name = "SWIMM";
        notice= "License notice";
        redistributable = true;
        trademarks = {
            @Trademark(description = "abc" , owner = "xyz"),
            @Trademark(description = "efg" , owner = "klm")
        }
    )
    public class ExampleClass { ... }
    ```

---

### Reflection
- Verifica della presenza di una annotazione in una classe
    ```java
    Class c = Sub.class;
    boolean inProgress = c.isAnnotationPresent(InProgress.class);
    ```
- Verifica della presenza di una annotazione in un metodo
    ```java
    Class c = ExampleClass.class;
    AnnotatedElement el = c.getMethod("calculateInterest", "float.class", "float.class");
    GroupTODO groupToDo = el.getAnnotation(GroupTODO.class);
    String assignedTo = groupToDo.assignedTo();
    out.println("TODO item assigned to: " + assignedTo);
    ```