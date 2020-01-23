# 03 - Annotations


### Categorie di Annotation
- Marker annotation
    ```java
    @AnnotationName
    ```
- Single value annotation
    ```java
    @AnnotationName("value")
    ```
- Full annotation
    ```java
    @AnnotationName(value1="value", value2="2")
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