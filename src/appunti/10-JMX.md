# 10 - JMX


### Utilizzo MBean
- Registrazione MBean
    ```java
    ObjectName username = new ObjectName("example:name=user1");
    List serverList = MBeanServerFactory.findMBeanServer(null);
    MBeanServer server = (MBeanServer)serverList.iterator().next();
    
    // In alternativa, per creazione MBeanServer
    MBeanServer server = MBeanServerFactory.
    createMBeanServer();
    
    // Registrazione MBean
    server.registerMBean(new User(), username);
    ```
- Invocazione MBean
    ```java
    ObjectName username = new ObjectName("example:name=user1");
    Object result = server.invoke(
        username,       // nome MBean
        "printInfo",    // nome operazione
        null,           // parametri
        null);          // signature
    ```

---

### Meccanismo di notifica
- Notification Filter
    ```java
    public interface NotificationFilter {
        public boolean isNotificationEnabled(Notification notification);
    }
    ```
- Notification Broadcaster
    ```java
    public interface NotificationBroadcaster {
        
        public void addNotificationListener(
                NotificationListener listener,
                NotificationFilter filter,
                Object handback) throws IllegalArgumentException;
        
        ...
    }
    ```

---

### Tipi di MBean
- Standard MBean
    ```java
    public interface UserMBean{
        public long getId();
        public void setId(long id);
        public boolean isActive();
        public void setActive(boolean active);
        public String printInfo();
    }
    public class User
    implements UserMBean {
        ...
    }
    ```
- Dynamic MBean
    ```java
    public class DynamicUser extends NotificationBroadcasterSupport implements DynamicMbean {
        
        // Attributi
        final static String ID = "id";
        private long id = System.currentTimeMillis();
        public Object getAttribute(String attribute) throws AttributeNotFoundException, MBeanException, ReflectionException {
            if(attribute.equals(ID))
                return new Long(id);
            throw new AttributeNotFoundException("Missing attribute " + attribute);
        }
        
        // Operazioni
        final static String PRINT = "printInfo";
        public String printInfo() { return "Sono un MBean dinamico"; }
        public Object invoke(String actionName, Object[] params, String[] signature) throws ... {
            if(actionName.equals(PRINT))
                return printInfo();
            throw new UnsupportedOperationException("Unknown operation " + actionName);
        }
        
        public MBeanInfo getMBeanInfo() {
            final boolean READABLE = true; final boolean WRITABLE = true;
            final boolean IS_GETTERFORM = true;
            String classname = getClass().getName();
            String description = "Sono un MBean dinamico";
            MBeanAttributeInfo id = new MBeanAttributeInfo(ID, long.class.getName(), "id", READABLE, !WRITABLE, !IS_GETTERFORM);
            MBeanConstructorInfo defcon = new MBeanConstructorInfo("Default","Creates",null);
            MBeanOperationInfo print = new MBeanOperationInfo(PRINT, "Prints info", null, String.class.getName(), MBeanOperationInfo.INFO);
            return new MBeanInfo(
                classname,
                description,
                new MBeanAttributeInfo[] { id },
                new MBeanConstructorInfo[] { defcon },
                new MBeanOperationInfo[] { print },
                null);
        }
    }
    ```
- Model MBean
    ```java
    public interface ModelMBean extends DynamicMBean, PersistentMBean, ModelMBeanNotificationBroadcaster {
        
        public void setModelMBeanInfo(ModelMBeanInfo inModelMBeanInfo) throws ... ;
        public void setManagedResource(Object mr, String mr_type) throws ... ;
        
    }
    
    public class RequiredModelMBean implements ModelMBean {
        ...
    }
    ```

---

### Servizi standard livello Agente
- M-Let Service
    ```xml
    <MLET>
        CODE = com.mycompany.Foo
        ARCHIVE = "MyComponents.jar, acme.jar"
    </MLET>
    ```
- Timer Service
    ```java
        // Fa partire il servizio di timer
        List list = MBeanServerFactory.findMBeanServer(null);
        MBeanServer server = (MBeanServer)list.iterator().next();
        ObjectName timer = new ObjectName("service:name=timer");
        server.registerMBean(new Timer(), timer);
        server.invoke(timer, "start", null, null);
        
        // Configurazione di notification time
        Date date = new Date(System.currentTimeMillis()+Timer.ONE_SECOND*5);
        
        server.invoke(
            timer,                  // MBean
            ”addNotification”,      // metodo
            new Object[] {                  // args
                “timer.notification”,       // tipo
                “Schedule notification”,    // messaggio
                null,                       // user data
                date},                      // time
            new String[] { String.class.getName(),
            String.class.getName(),
            Object.class.getName(), // signature
            Date.class.getName()});
        
        // Registra il listener MBean
        server.addNotificationListener(timer, this, null, null);
    ```
- Relation
    ```java
    RoleInfo monInfo = new RoleInfo(
            "Monitor",
            "javax.management.monitor.GaugeMonitor",
            true, true, 0,
            ROLE_CARDINALITY_INFINITY,
            "Descrizione del monitor");
    
    RoleInfo obsInfo = new RoleInfo(
            "Observable",
            "examples.ThreadMonitor",
            true, true, 1, 1,
            "Descrizione del ruolo observable");
    
    // Oggetti relazione implementano l’interfaccia RelationType
    RelationTypeSupport relationType = new RelationTypeSupport(
            "ObservedMBean",
            new RoleInfo[] { observableInfo, monitorInfo });
    ```

---

### JMX Remote API

- Connector Service lato servitore
    ```java
    MBeanServer mbs = MBeanServerFactory.createMBeanServer();
    // URL (in formato JNDI) indica dove reperire uno stub RMI per il connettore
    JMXServiceURL url = new JMXServiceURL("service:jmx:rmi:///jndi/rmi://localhost: 9999/server");
    JMXConnectorServer cs = JMXConnectorServerFactory.
    newJMXConnector-Server(url, null, mbs);
    cs.start();
    ```
- Utilizzo lato cliente
    ```java
    JMXServiceURL url = new JMXServiceURL(service:jmx:rmi:///jndi/rmi://localhost:9999/server");
    JMXConnector jmxc = JMXConnectorFactory.connect(url, null);
    MBeanServerConnection mbsc = jmxc.getMBeanServerConnection;
    mbsc.createMBean(...);
    ```

---

### Esempio
