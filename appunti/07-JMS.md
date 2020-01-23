# 07 - JMS


### Esempi

- Publisher
    ```java
    // Ottiene oggetto InitialContext
    Context jndiContext = new InitialContext();
    
    // Trova l’oggetto ConnectionFactory via JNDI
    TopicConnectionFactory factory = (TopicConnectionFactory) jndiContext.lookup("MyTopicConnectionFactory");
    
    // Trova l’oggetto Destination via JNDI (Topic o Queue)
    Topic weatherTopic = (Topic) jndiContext.lookup("WeatherData");
    
    // Richiede la creazione di un oggetto Connection all’oggetto ConnectionFactory
    TopicConnection topicConnection = factory.createTopicConnection();
    
    // Crea un oggetto Session da Connection:
    //   - primo parametro controlla transazionalità
    //   - secondo specifica il tipo di ack
    TopicSession session = topicConnection.createTopicSession(false, session.CLIENT_ACKNOWLEDGE);
    
    // Richiede la creazione di un oggetto MessageProducer all’oggetto Session
    // TopicPublisher per Pub/Sub
    // QueueSender per Point-to-Point
    TopicPublisher publisher = session.createPublisher(weatherTopic);
    
    // Avvia la Connection
    topicConnection.start();
    
    // Creazione del messaggio
    TextMessage message = session.createMessage();
    message.setText("text:35 degrees");
    
    // Invio del messaggio
    publisher.publish(message);
    ```
- Subscriber
    ```java
    // Crea oggetto Subscriber da Session
    TopicSubscriber subscriber = session.createSubscriber(weatherTopic);
    
    // Crea oggetto MessageListener
    WeatherListener myListener = new WeatherListener();
    
    // Registra MessageListener per l’oggetto TopicSubscriber desiderato
    subscriber.setMessageListener(myListener);
    ```

---

### Configurazione

- Persistenza
    ```java
    // Metodo di MessageProducer
    producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
    
    // In alternativa, all'invio di un messaggio
    producer.send(message, DeliveryMode.NON_PERSISTENT, 3, 10000);
    ```
- Priorità
    ```java
    // Metodo di MessageProducer
    producer.setPriority(7);
    
    // In alternativa, all'invio di un messaggio
    producer.send(message, DeliveryMode.NON_PERSISTENT, 7, 10000);
    ```
- Time to live
    ```java
    // Metodo di MessageProducer
    producer.setTimeToLive( 60000
    
    // In alternativa, all'invio di un messaggio
    producer.send(message, DeliveryMode.NON_PERSISTENT, 3, 60000);
    ```