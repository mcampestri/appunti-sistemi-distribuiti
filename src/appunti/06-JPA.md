# 06 - JPA


### Definizione componenti EJB
- Esempio Stateless SessionBean (locale a default)
    ```java
    @Stateless
    public class PayrollBean implements Payroll {
        public void setTaxDeductions() {
            ...
        }
    }
    ```
- Esempio Stateless SessionBean remoto
    ```java
    @Stateless
    @Remote
    public class PayrollBean implements Payroll {
        public void setTaxDeductions() {
            ...
        }
    }
    ```
- Esempio MessageDrivenBean
    ```java
    @MessageDriven
    public class PayrollMDB implements javax.jms.MessageListener {
        public void onMessage(Message message) {
            ...
        }
    }
    ```

---

### Dependency Injection
- Esempio Injection
    ```java
    //Injection a livello di classe
    @Resource(name="myDB", type=javax.sql.DataSource)
    @Stateless
    public class ShoppingCartBean implements ShoppingCart {
        // Injection a livello di metodo
        @Resource SessionContext ctx;
        public void setTaxDeductions() {
            ...
        }
    }
    ```
- Tipi di risorse iniettate:
    - ```@Resource<(JNDI name, type)>```: sorgenti dati, ambiente di esecuzione, topic/queue JMS, EJBContext
    - ```@EJB```: componenti EJB
    - ```@PersistenceContext / @PersistenceUnit```: sorgenti dati, ambiente di esecuzione, topic/queue JMS, EJBContext
    ```java
    @Stateless
    public class ShoppingCartBean implements ShoppingCart {
        
        public void setTaxDeductions() {
            ...
        }
    }
    ```
- Esempio injection a livello di classe
    ```java
    @Resources({
        @Resource(name="myMessageQueue", type=javax.jms.ConnectionFactory),
        @Resource(name="myMailSession", type=javax.mail.Session)
    })
    @Stateless
    public class ShoppingCartBean implements ShoppingCart {
        ...
    }
    ```

---

### Transazioni
- Conatiner managed transaction
    ```java
    @TransactionAttribute(MANDATORY)
    @Stateless
    public class PayrollBean implements Payroll {
        
        public void setTaxDeductions(int empId,int deductions) {
            ...
        }
        
        @TransactionAttribute(REQUIRED)
        public int getTaxDeductions(int empId) {
            ...
        }
    }
    ```
- Bean managed transaction
    ```java
    @TransactionManagement(BEAN)
    @Stateless
    public class PayrollBean implements Payroll {
        
        @Resource UserTransaction utx;
        @PersistenceContext EntityManager payrollMgr;
        
        public void setTaxDeductions(int empId, int deductions) {
            utx.begin();
            payrollMgr.find(Employee.class, empId).setDeductions(deductions);
            utx.commit();
        }
    }
    ```

---

### Sicurezza
- Esempio sicurezza
    ```java
    @Stateless
    public class PayrollBean implements Payroll {
        
        @RolesAllowed("HR_PayrollAdministrator")
        public void setSalary(int empId, double salary) {
            ...
        }
    }
    ```

---

### Intercettori
- Classe intercettata
    ```java
    @Interceptors(Profiler.class)
    public Objecty interceptedMethod(...) {
        ...
    }
    ```
- Classe intercettore
    ```java
    public class Profiler {
        @AroundInvoke
        public Object profile() throws Exception {
            ...
        }
    }
    ```