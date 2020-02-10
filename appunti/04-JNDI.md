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
