# 08 - CORBA


### Definizione componenti CORBA
- Esempio componente
    ```c_pp
    // IDL 3
    interface rate_control
    {
        void start ();
        void stop ();
    };
    component RateGen supports rate_control {};
    ```
- CCM Home
    ```c_pp
    // IDL 3
    home RateGenHome manages RateGen
    {
        factory create_pulser (in rateHz r);
    };
    ```
- Facet
    ```c_pp
    // IDL 3
    interface position
    {
        long get_pos ();
    };
  
    component GPS
    {
        provides position MyLocation;
        ...
    };
    ```
- Receptacle
    ```c_pp
    // IDL 3
    component NavDisplay
    {
        uses position GPSLocation;
        ...
    };
    ```
- Sorgente di eventi
    ```c_pp
    // IDL 3
    component RateGen
    {
        publishes tick Pulse;
        emits tick Trigger ;
        ...
    };
    ```
- Sorgente di eventi
    ```c_pp
    // IDL 3
    component RateGen
    {
        publishes tick Pulse;
        emits tick Trigger ;
        ...
    };
    ```
- Sink di eventi
    ```c_pp
    // IDL 3
    component NavDisplay
    {
        consumes tick Refresh;
        ...
    };
    ```
- Attributi
    ```c_pp
    // IDL 3
    typedef unsigned long rateHz;
    component RateGen supports rate_control
    {
        attribute rateHz Rate;
        ...
    };
    ```

---

### Navigation e Introspection componenti
- Interfacce di Navigation
    ```c_pp
    CORBA::ORB_var orb = CORBA::ORB_init (argc, argv);
    
    // Get the NameService reference
    CORBA::Object_var o = ns->resolve_str (“myHelloHome");
    HelloHome_var hh = HelloHome::_narrow (o.in ());
    HelloWorld_var hw = hh->create ();
    
    // Get all facets and receptacles
    Components::FacetDescriptions_var fd = hw->get_all_facets ();
    Components::ReceptacleDescriptions_var rd = hw->get_all_receptacles ();
    
    // Get a named facet with a name “Farewell”
    CORBA::Object_var fobj = hw->provide (“Farewell”);
    
    // Can invoke sayGoodbye() operation on Farewell after narrowing to the Goodbye interface
    ...
    ```