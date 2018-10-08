功能实现：
商城网站点击搜索结果某个商品进入详情页面时用redis实现服务端数据缓存，减轻数据库压力并且显示数据更快用户体验更好，
商品详情缓存业务逻辑：先从redis获取商品详情数据，如果没有则从数据库获取，获取后添加到redis同时返回数据。
代码：
//查询缓存
try {
    String json = jedisClient.get(REDIS_ITEM_PRE + ":" + itemId + ":BASE");
    if(StringUtils.isNotBlank(json)) {
        TbItem tbItem = JsonUtils.jsonToPojo(json, TbItem.class);
        return tbItem;
    }
} catch (Exception e) {
    e.printStackTrace();
}
//把结果添加到缓存
try {
    jedisClient.set(REDIS_ITEM_PRE + ":" + itemId + ":BASE", JsonUtils.objectToJson(list.get(0)));
    //设置过期时间
    jedisClient.expire(REDIS_ITEM_PRE + ":" + itemId + ":BASE", ITEM_CACHE_EXPIRE);
} catch (Exception e) {
    e.printStackTrace();
}

后台管理商品页面无需使用redis实现数据缓存，因为访问量不大，用户体验要求不高。

#后台添加新的商品后消息推送，search工程监听新增商品消息，把新的商品同步到索引库。
监听消息：
1.spring容器创建bean：
   	<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
   		<property name="brokerURL" value="tcp://192.168.25.161:61616" />
   	</bean>
   	<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
   	<bean id="connectionFactory"
   		class="org.springframework.jms.connection.SingleConnectionFactory">
   		<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
   		<property name="targetConnectionFactory" ref="targetConnectionFactory" />
   	</bean>
   	<!--这个是主题目的地，一对多的 -->
   	<bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
   		<constructor-arg value="itemAddTopic" />
   	</bean>
   	<!-- 监听商品添加消息，同步索引库 -->
   	<bean id="itemAddMessageListener" class="cn.e3mall.search.message.ItemAddMessageListener"/>
   	<bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
   		<property name="connectionFactory" ref="connectionFactory" />
   		<property name="destination" ref="topicDestination" />
   		<property name="messageListener" ref="itemAddMessageListener" />
   	</bean>
2.实现javax.jms.MessageListener接口：
    public class ItemAddMessageListener implements MessageListener
3.重写onMessage方法：
    @Override
    public void onMessage(Message message){
        TextMessage textMessage = (TextMessage) message;
        String text = textMessage.getText();
    }

发送消息：
1.spring容器创建bean：
	<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
	<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://192.168.25.161:61616" />
	</bean>
	<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
	<bean id="connectionFactory"
		class="org.springframework.jms.connection.SingleConnectionFactory">
		<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
		<property name="targetConnectionFactory" ref="targetConnectionFactory" />
	</bean>
	<!-- 配置生产者 -->
	<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
	<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
		<!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
		<property name="connectionFactory" ref="connectionFactory" />
	</bean>
	<!--这个是队列目的地，点对点的 -->
	<bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>spring-queue</value>
		</constructor-arg>
	</bean>
	<!--这个是主题目的地，一对多的 -->
	<bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
		<constructor-arg value="itemAddTopic" />
	</bean>
2.发送
    //发送商品添加消息
    jmsTemplate.send(topicDestination, new MessageCreator() {
        @Override
        public Message createMessage(Session session) throws JMSException {
            TextMessage textMessage = session.createTextMessage(itemId + "");
            return textMessage;
        }
    });