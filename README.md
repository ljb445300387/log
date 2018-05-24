# log.
netstat -anpt | grep redis | awk -F" " {'print $5'} | awk -F":" {'print $1'}| sort -rh | uniq -c

spring-boot如何配置监听两个不同的RabbitMQ
详细步骤：
1. application.properties 文件配置：
    u.rabbitmq.addresses=10.0.0.1:5672
    u.rabbitmq.username=username1
    u.rabbitmq.password=password1
    u.rabbitmq.vhost.provider=/provider

    b.rabbitmq.addresses=10.0.0.2:5672
    b.rabbitmq.username=username2
    b.rabbitmq.password=password1
    b.rabbitmq.vhost.provider=/provider 
u.rabbitmq：为第一个MQ的配置，后面简称UMQ 
b.rabbitmq：为第二个MQ的配置，后面简称BMQ

2. RabbitMqConfig中读取配置信息
@Value("${u.rabbitmq.username}")
private String uname;
@Value("${u.rabbitmq.password}")
private String upassword;
@Value("${u.rabbitmq.vhost.provider}")
private String uhost;
@Value("${u.rabbitmq.addresses}")
private String uaddress;

@Value("${b.rabbitmq.username}")
private String bname;
@Value("${b.rabbitmq.password}")
private String bpassword;
@Value("${b.rabbitmq.vhost.provider}")
private String bhost;
@Value("${b.rabbitmq.addresses}")
private String baddress;

3.RabbitMqConfig中配置链接
@Bean(name="uConnectFactory")
@Primary
public ConnectionFactory uConnectFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
    connectionFactory.setAddresses(uaddress);
    connectionFactory.setUsername(uname);
    connectionFactory.setPassword(upassword);
    connectionFactory.setVirtualHost(uhost);
    return connectionFactory;
}

@Bean(name="bConnectFactory")
public ConnectionFactory bConnectFactory() {
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
    connectionFactory.setAddresses(rpbaddress);
    connectionFactory.setUsername(rpbusername);
    connectionFactory.setPassword(rpbpassword);
    connectionFactory.setVirtualHost(rpbvhost);
    return connectionFactory;
}

@Primary标签指若不指定链接，默认选择链接，在配置了多个链接必须添加@Primary标签，否则有可能会找不到对应的rabbitMQ的连接

4.配置监听连接
@Bean(name="uRabbitListenerContainerFactory")
public SimpleRabbitListenerContainerFactory uRabbitListenerContainerFactory(
        @Qualifier("uConnectFactory") ConnectionFactory connectionFactory,
        RabbitProperties config) {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    RabbitProperties.Listener listenerConfig = config.getListener();
    factory.setAutoStartup(listenerConfig.isAutoStartup());
    if (listenerConfig.getAcknowledgeMode() != null) {
        factory.setAcknowledgeMode(listenerConfig.getAcknowledgeMode());
    }
    if (listenerConfig.getConcurrency() != null) {
        factory.setConcurrentConsumers(listenerConfig.getConcurrency());
    }
    if (listenerConfig.getMaxConcurrency() != null) {
        factory.setMaxConcurrentConsumers(listenerConfig.getMaxConcurrency());
    }
    if (listenerConfig.getPrefetch() != null) {
        factory.setPrefetchCount(listenerConfig.getPrefetch());
    }
    if (listenerConfig.getTransactionSize() != null) {
        factory.setTxSize(listenerConfig.getTransactionSize());
    }
    return factory;
}

@Bean(name="bRabbitListenerContainerFactory")
public SimpleRabbitListenerContainerFactory brabbitListenerContainerFactory(
        @Qualifier("bConnectionFactory") ConnectionFactory connectionFactory,
        RabbitProperties config) {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    RabbitProperties.Listener listenerConfig = config.getListener();
    factory.setAutoStartup(listenerConfig.isAutoStartup());
    if (listenerConfig.getAcknowledgeMode() != null) {
        factory.setAcknowledgeMode(listenerConfig.getAcknowledgeMode());
    }
    if (listenerConfig.getConcurrency() != null) {
        factory.setConcurrentConsumers(listenerConfig.getConcurrency());
    }
    if (listenerConfig.getMaxConcurrency() != null) {
        factory.setMaxConcurrentConsumers(listenerConfig.getMaxConcurrency());
    }
    if (listenerConfig.getPrefetch() != null) {
        factory.setPrefetchCount(listenerConfig.getPrefetch());
    }
    if (listenerConfig.getTransactionSize() != null) {
        factory.setTxSize(listenerConfig.getTransactionSize());
    }
    return factory;
} 
5.配置消息代理服务器
@Bean(name="uAmqpAdmin")
public AmqpAdmin amqpAdmin(@Qualifier("uConnectFactory") ConnectionFactory connectionFactory) {
    RabbitAdmin admin = new RabbitAdmin(connectionFactory);
    admin.setAutoStartup(false);
    return admin;
}

@Bean(name="bAmqpAdmin")
public AmqpAdmin rpdAmqpAdmin(@Qualifier("bConnectionFactory") ConnectionFactory connectionFactory) {
    RabbitAdmin admin = new RabbitAdmin(connectionFactory);
    admin.setAutoStartup(false);
    return admin;
} 
6.具体监听队列及使用配置
UMQ队列queue监听配置
@RabbitListener(queues = "queue", containerFactory = uRabbitListenerContainerFactory")
public void handleMessage(String message) throws Exception {
    具体操作。。。。
 }

RMQ队列queue监听配置
@RabbitListener(queues = "queue", containerFactory = rRabbitListenerContainerFactory")
    public void handleMessage(String message) throws Exception {
    具体操作。。。。
}
