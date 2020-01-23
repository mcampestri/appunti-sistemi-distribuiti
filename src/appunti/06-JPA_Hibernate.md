# 06 - JPA e Hibernate


### Entity

- Entity ed ereditarietà
    ```java
    @Entity
    public abstract class Employee {
        @Id
        protected Integer employeeId;
    }
    
    // Superclasse contenente stato persistente ereditabile da sottoclassi Entity
    @MappedSuperclass
    public class Employee {
        @Id
        protected Integer employeeId;
    }
    
    @Entity
    public class FullTimeEmployee extends Employee {
        protected Integer salary;
    }
    
    @Entity
    public class PartTimeEmployee extends Employee {
        protected Float hourlyWage;
    }
    ```
- Eager loading
    ```java
    @OneToMany(cascade=ALL, mappedBy=“owner”, fetch=EAGER)
    ```
- Lazy loading
    ```java
    @OneToMany(cascade=ALL, mappedBy=“owner”, fetch=LAZY)
    ```

---

### EntityManager

- Container managed EntityManager
    ```java
    // EntityManager iniettato mediante dependency injection
    @PersistenceContext
    EntityManager em;

    public void enterOrder(int custID, Order newOrder) {
        Customer cust = em.find(Customer.class, custID);
        cust.getOrders().add(newOrder);
        newOrder.setCustomer(cust);
    }
    ```
- Application managed EntityManager
    ```java
    @PersistenceUnit
    EntityManagerFactory emf;
    EntityManager em = emf.createEntityManager();
    ```
- Esempio operazione persist()
    ```java
    public Order createNewOrder(Customer customer) {
        // Crea una nuova istanza dell’oggetto
        // Entity è in stato New/Transient
        Order order = new Order(customer);
        
        // Dopo l’invocazione del metodo persist(), lo stato viene cambiato in managed
        // Al prossimo flush o commit, le nuove istanze persistenti saranno inserite nella tabella del DB
        entityManager.persist(order);
        return order;
    }
    ```
- Esempio operazioni find() e remove()
    ```java
    public void removeOrder(Long orderId) {
        Order order = entityManager.find(Order.class, orderId);
        entityManager.remove(order);
        // Le istanze verranno eliminate dalla tabella al prossimo flush o commit
        // L’accesso a una Entity già rimossa restitisce risultati non definiti
    }
    ```
- Esempio operazione merge()
    ```java
    public OrderLine updateOrderLine(OrderLine orderLine) {
        // Il metodo merge restituisce una copia managed della data entity di tipo detached
        // Le modifiche fatte allo stato persistente dell’entity detached sono applicate a questa istanza managed
        return entityManager.merge(orderLine);
    }
    ```

---

### Query

- Query dinamiche (createQuery())
    ```java
    public List findWithName(String name) {
        return em.createQuery("SELECT c FROM Customer c WHERE c.name LIKE :custName")
            .setParameter("custName", name)
            .setMaxResults(10)
            .getResultList();
    }
    ```
- Query dinamiche (@NamedQuery)
    ```java
    @NamedQuery(name="findAllCustomersWithName", query="SELECT c FROM Customer c WHERE c.name LIKE :custName")
    
    @PersistenceContext
    public EntityManager em;
    
    customers = em.createNamedQuery("findAllCustomersWithName")
        .setParameter("custName", "Smith")
        .getResultList();
    ```

---

### Unità di Persistenza

- persistence.xml
    ```xml
    <persistence>
        <persistence-unit name="OrderManagement">
            <description>Questa unità gestisce ordini e clienti</description>
            <jta-data-source>jdbc/MyOrderDB</jta-data-source>
            <jar-file>MyOrderApp.jar</jar-file>
            <class>com.widgets.Order</class>
            <class>com.widgets.Customer</class>
        </persistence-unit>
    </persistence>
    ```

---

### Listener di Entity

- Classe ascoltata
    ```java
    @Entity
    @EntityListener(com.acme.AlertMonitor.class)
    public class AccountBean implements Account {
        
        @PrePersist
        public void validateCreate() {
            if(getBalance() < MIN_REQUIRED_BALANCE)
                throw new AccountException("Insufficient balance to open an account");
        }
        
        @PostLoad
        public void adjustPreferredStatus() {
            preferred = (getBalance() >= AccountManager.getPreferredStatusLevel());
        }
        
    }
    ```
- Classe listener
    ```java
    public class AlertMonitor {
        
        @PostPersist
        public void newAccountAlert(Account acct) {
            Alerts.sendMarketingInfo(acct.getAccountId(),
            acct.getBalance());
        }
        
    }
    ```

---

### Hibernate

- Hibernate Version Checking
    ```java
    @Entity
    @Table(name = "orders")
    public class Order { @Id private long id;
        
        @Version private
        int version;
        private String description;
        private String status;
        
        ...
    }
    ```
    ```sql
    UPDATE orders SET description=?, status=?, version=? WHERE id=? AND version=?
    ```
- Query By Example
    ```java
    Criteria crit = sess.createCriteria(Person.class);
    Person person = new Person();
    person.setName(“Shin”);
    
    Example exampleRestriction = Example.create(person);
    crit.add( exampleRestriction );
    
    List results = crit.list();
    ```