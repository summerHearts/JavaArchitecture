##Mac安装kafka 并生产消息和消费消息
- 1. 安装kafka

  ```
  $   brew install kafka
  ```
  - 1、 安装过程将依赖安装 zookeeper
  - 2、 软件位置
  
      ```
    /usr/local/Cellar/zookeeper
    /usr/local/Cellar/kafka
      ```
  - 3、 配置文件位置

        ```
     /usr/local/etc/kafka/zookeeper.properties
     /usr/local/etc/kafka/server.properties
        ```
备注：后续操作均需进入 /usr/local/Cellar/kafka/1.0.0/bin 目录下。

- 2、启动zookeeper

      ```
      zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
      ```
- 3、启动kafka服务

    ```
    kafka-server-start /usr/local/etc/kafka/server.properties &
    ```
- 4、 创建topic

     ```
     kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test1
     ```
- 5、查看创建的topic

  ```
  kafka-topics --list --zookeeper localhost:2181  
  ```
- 6、生产数据

  ```
  kafka-console-producer --broker-list localhost:9092 --topic test1
  ```
- 7、消费数据

  ```
  kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic test1 --from-beginning
  ```
备注：--from-beginning  将从第一个消息还是接收

配置文件加载

```
@Configuration
@ConfigurationProperties(prefix = "kafka")
@Setter
@Getter
public class KafkaConfig {
    private String producerPropertisePath;
    private String consumerPropertisePath;
    private String producerHandler;
    private Map<String,String> topicHandlerMap;
    private int threadNums=1000;
    @Autowired
    private ApplicationContext applicationContext;
    @Bean
    public KafkaProducerService kafkaProducerService(){
            ProducerHandler producerHandlerBean = producerHandler==null||producerHandler.isEmpty()?null:(ProducerHandler) applicationContext.getBean(producerHandler);
            return new KafkaProducerService(producerPropertisePath==null?"classpath:properties/producer.properties":producerPropertisePath, producerHandlerBean);
    }
    @Bean
    public KafkaConsumerService kafkaConsumerService(){
        Map<String,ConsumerHandler> topicHandlerMap2=new HashMap<String, ConsumerHandler>();
        if(topicHandlerMap!=null){
            for(String key:topicHandlerMap.keySet()){
                topicHandlerMap2.put(key,(ConsumerHandler) applicationContext.getBean(topicHandlerMap.get(key)));
            }
        }
        KafkaConsumerService kafkaConsumerService= new KafkaConsumerService(consumerPropertisePath==null?"classpath:properties/customer.properties":consumerPropertisePath,topicHandlerMap2,threadNums);
        kafkaConsumerService.run();
        return kafkaConsumerService;
    }
}
```
处理结果回调

```
@Component
public class ProducerHandlerImpl implements ProducerHandler<String> {
    @Override
    public void whenProducerFailed(String topic, String value, Exception exception) {
        System.out.println(">>>consumer>>>whenCommitOffsetFailed>>>topic:"+topic+">>>value:"+value+">>>>exception:"+exception);
    }

    @Override
    public void whenProduceSucceed(String topic, String value) {

    }
}

@Component
public class ConsumerHandlerImpl implements ConsumerHandler<String> {
    int i=0;
    @Override
    public void whenGetRecord(ConsumerRecord<String,String> record) throws Exception {
        System.out.println(new Date().toString()+">>>consumer>>>whenGetRecord>>>topic:"+record.topic()+">>>value:"+new String(record.value())+">>>>time:"+new Date(record.timestamp()).toString());
        i++;
        try{
            Thread.sleep(1000);//等待消费完成
        }catch (Exception e){

        }
//        if(i%5==0){
//            throw new Exception("处理错误");
//        }
    }

    @Override
    public Boolean whenRunFailed(ConsumerRecord<String,String> record, Exception exception) {
        System.out.println(">>>consumer>>>whenRunFailed>>>topic:"+record.topic()+">>>value:"+new String(record.value())+">>>>exception:"+exception);
        return false;
    }

    @Override
    public void whenCommitOffsetFailed(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
        System.out.println(">>>consumer>>>whenCommitOffsetFailed>>>topic:"+offsets.toString()+">>>>exception:"+exception);
    }
}
```

消费服务

```
@Component
public class KafkaConsumerService {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private Map<String, ConsumerHandler> topicHandlerMap;
    private int threadNums;
    private Boolean autoCommit;
    private ExecutorService executor;
    private Boolean serviceisRun = false;
    private Properties prop;
    public static ConcurrentHashMap<String, List<ConsumerThread>> consumerThreadMap = new ConcurrentHashMap<>();
    @Autowired
    private KafkaProducerService kafkaProducerService;

    /**
     * 消费者线程
     */
    public class ConsumerThread implements Runnable{
        private ConsumerHandler handler;
        private KafkaConsumer<String,byte[]> consumer;
        private Boolean threadIsRun = false;
        private String topic;
        private Boolean redo = false;
        private String name;

        public ConsumerThread(String topic, ConsumerHandler handler) {
            this.handler = handler;
            this.consumer =new KafkaConsumer<String, byte[]>(KafkaConsumerService.this.prop);
            this.topic = topic;
            consumer.subscribe(Arrays.asList(topic));
        }

        public ConsumerThread(String topic, ConsumerHandler handler,Properties prop) {
            this.handler = handler;
            this.consumer =new KafkaConsumer<String, byte[]>(prop);
            this.topic = topic;
            consumer.subscribe(Arrays.asList(topic));
        }

        public void run() {
            threadIsRun = true;
            while (serviceisRun&&threadIsRun) {
                ConsumerRecords<String, byte[]> records = consumer.poll(200);
                if(records.count()>0){
                    logger.info(">>>拉取到{}条信息",records.count());
                    if (!autoCommit) {
                        consumer.commitAsync();
                    }
                    //List<TopicPartition> errorTopic = new ArrayList<TopicPartition>();
                    for (final ConsumerRecord record : records) {
//                        TopicPartition topicPartition = new TopicPartition(record.topic(), record.partition());
//                        if(redo&&errorTopic.contains(topicPartition)){
//                            logger.debug(">>>kafka消息队列消费异常,暂停消费准备重试，topic:{},partition:{}", new Object[]{record.topic(),Integer.valueOf(record.partition())});
//                            continue;
//                        }
                        logger.info(">>>准备消费topic:{},partition:{},offset:{},value:{}", new Object[]{record.topic(), Integer.valueOf(record.partition()), Long.valueOf(record.offset()),record.value()});
                        try {
                            this.handler.whenGetRecord(record);
                        } catch (Exception e) {
                            logger.error(">>>kafka消息队列消费异常，topic:{},message:{},offset:{},partition:{}", new Object[]{record.topic(), record.value(), Long.valueOf(record.offset()), Integer.valueOf(record.partition())});
                            logger.error(">>>kafka异常信息", e);
                            Boolean redo =this.handler.whenRunFailed(record, e);
                            if(redo!=null&&redo){
                                logger.info(">>>kafka消息队列消费异常,消息放至队尾准备重试，topic:{},partition:{},offset:{}", new Object[]{record.topic(),Integer.valueOf(record.partition())}, Long.valueOf(record.offset()));
//                                consumer.seek(topicPartition,record.offset());
//                                errorTopic.add(topicPartition);
//                                redo = true;
                                kafkaProducerService.produce(record.topic(),record.key().toString(),record.value());
                            }
                        }
                        logger.info(">>>结束消费topic-partition-offset : {}-{}-{}", new Object[]{record.topic(), Integer.valueOf(record.partition()), Long.valueOf(record.offset())});
                    }
                }
            }
        }

        public void stop(){
            this.threadIsRun=false;
            logger.info("{},消费者停止",this.topic);
            consumerThreadMap.get(this.topic).remove(this);
        }

        public KafkaConsumer<String, byte[]> getConsumer() {
            return consumer;
        }

        public String getTopic() {
            return topic;
        }

        public Boolean isRun(){
            return this.threadIsRun;
        }
    }

    /**
     * 手动提交offset回调函数
     */
    class KafkaOffsetCommitCallback implements OffsetCommitCallback{

        private String topic;

        public KafkaOffsetCommitCallback(String topic){
            this.topic=topic;
        }

        @Override
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
            //处理异常
            if (null != exception) {
                // 如果异步提交发生了异常
                logger.error(">>>kafka提交offset异常 {}, 错误: {}", topic, exception);
                topicHandlerMap.get(topic).whenCommitOffsetFailed(offsets,exception);
            }
        }
    }

    /**
     * 初始化函数，当topicHandlerMap为空时，设定为生产模式
     * @param propertisePath
     * @param topicHandlerMap
     */
    public KafkaConsumerService(String propertisePath, Map<String, ConsumerHandler> topicHandlerMap,int threadNums){
        Assert.notNull(propertisePath,"consumer.properties路径为空");
        //InputStream in = Object.class.getResourceAsStream(propertisePath);
        ResourceLoader resourceLoader = new DefaultResourceLoader();
        Resource resource = resourceLoader.getResource(propertisePath);
        InputStream in=null;
        //InputStream in = Object.class.getResourceAsStream(propertisePath);
        try {
            in =resource.getInputStream();
        }catch (Exception e){
            throw new RuntimeException("读取异常:"+propertisePath);
        }
        Assert.notNull(in,propertisePath+"文件不存在");
        Properties prop = new Properties();
        try{
            prop.load(in);
            this.autoCommit = prop.getProperty("enable.auto.commit").equals("true")?true:false;
            this.threadNums = threadNums==0?100:threadNums;
            this.topicHandlerMap= topicHandlerMap;
            this.prop = prop;
        }catch (IOException e){
            e.printStackTrace();
            logger.error("创建消费者失败："+e.getMessage());
            Assert.state(false,"创建消费者失败："+e.getMessage());
        }
    }

    /**
     * 生产模式下生产并启动消费者
     * @param topic
     * @param consumerHandler
     * @return
     */
    public ConsumerThread createConsumerThread(String topic,ConsumerHandler consumerHandler){
        ConsumerThread consumerThread = new ConsumerThread(topic,consumerHandler);
        executor.submit(consumerThread);
        if(consumerThreadMap.get(topic)==null){
            consumerThreadMap.put(topic,new ArrayList<ConsumerThread>());
        }
        consumerThreadMap.get(topic).add(consumerThread);
        return consumerThread;
    }

    public ConsumerThread createConsumerThread(String topic,String groupId,ConsumerHandler consumerHandler){
        Properties _prop = (Properties)this.prop.clone();
        _prop.setProperty("group.id",groupId);
        ConsumerThread consumerThread = new ConsumerThread(topic,consumerHandler,_prop);
        executor.submit(consumerThread);
        if(consumerThreadMap.get(topic)==null){
            consumerThreadMap.put(topic,new ArrayList<ConsumerThread>());
        }
        consumerThreadMap.get(topic).add(consumerThread);
        return consumerThread;
    }
    /**
     * 启动消费者
     */
    public void run() {
        if(this.serviceisRun){
            logger.info("消费者已经启动");
            return;
        }
        this.logger.debug(">>>KafkaConsumerService.start！");
        this.serviceisRun = true;
        this.executor = Executors.newFixedThreadPool(threadNums);
        if(null == this.topicHandlerMap||this.topicHandlerMap.size()==0){
            return;
        }
        Iterator consumerMap = this.topicHandlerMap.keySet().iterator();
        List<String> topics=new ArrayList<String>();
        while(consumerMap.hasNext()) {
            String topic = (String)consumerMap.next();
            logger.info("订阅topic:{}",topic);
            ConsumerThread consumerThread = new ConsumerThread(topic,topicHandlerMap.get(topic));
            executor.submit(consumerThread);
            if(consumerThreadMap.get(topic)==null){
                consumerThreadMap.put(topic,Arrays.asList(consumerThread));
            }else{
                consumerThreadMap.get(topic).add(consumerThread);
            }
        }
    }

    /**
     * 关闭消费者
     */
    public void shutdown() {
        logger.info("开始停止消费者");
        this.serviceisRun=false;
         if (executor!= null) {
             executor.shutdown();
         }
         try {
             if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
                    logger.debug("线程池超时关闭，强制结束线程");
                 }
         } catch (InterruptedException ignored) {
             logger.error("线程意外终止");
             Thread.currentThread().interrupt();
         }

    }

    /**
     * 消费者是否启动
     * @return
     */
    public Boolean isRun() {
        return serviceisRun;
    }
}
```

生产消息服务

```
@Component
public class KafkaProducerService<T> {
    private Logger logger = LoggerFactory.getLogger(KafkaProducerService.class);
    private KafkaProducer producer;
    private ProducerHandler producerHandler;
    public KafkaProducerService(String propertisePath, ProducerHandler producerHandler){
        Assert.notNull(propertisePath,"producer.properties路径为空");
        ResourceLoader resourceLoader = new DefaultResourceLoader();
        Resource resource = resourceLoader.getResource(propertisePath);
        InputStream in=null;
        //InputStream in = Object.class.getResourceAsStream(propertisePath);
        try {
            in =resource.getInputStream();
       }catch (Exception e){
            throw new RuntimeException("读取异常:"+propertisePath+",异常:"+e.getMessage());
        }
        Assert.notNull(in,propertisePath+"文件不存在");
        Properties prop = new Properties();
        try{
            prop.load(in);
            this.producer = new KafkaProducer(prop);
            if(producerHandler!=null)this.producerHandler=producerHandler;
        }catch (IOException e){
            e.printStackTrace();
            logger.error("创建生成者失败："+e.getMessage());
            Assert.state(false,"创建生成者失败："+e.getMessage());
        }
    }

    /**
     * 发送消息异步
     * @param topic
     * @param value
     */
//    public void produce(String topic, T value) {
//        this.producer.send(new ProducerRecord(topic, value), new KafkaProducerService.ProduceCallback(topic, value, System.currentTimeMillis()));
//    }


    /**
     * 发送消息异步
     * @param topic
     * @param partition
     * @param key
     * @param value
     */

//    public void produce(String topic, Integer partition, String key, T value) {
//        this.producer.send(new ProducerRecord(topic, partition, key, value), new KafkaProducerService.ProduceCallback(topic, value, System.currentTimeMillis()));
//        logger.info("");
//    }
//
//    public void produce(String topic, Integer partition, T value) {
//        this.producer.send(new ProducerRecord(topic, partition, "", value), new KafkaProducerService.ProduceCallback(topic, value, System.currentTimeMillis()));
//    }

    public void produce(String topic, String key, T value) {
        this.producer.send(new ProducerRecord(topic, key, value), new KafkaProducerService.ProduceCallback(topic, value, System.currentTimeMillis()));
        logger.info("生产报文--->topic:{},key:{},msg:{}",topic, key, value);
    }

    /**
     * 发送消息同步(无回调)
     * @param topic
     * @param value
     */
//    public void produceSyn(String topic, T value) {
//        try {
//            this.producer.send(new ProducerRecord(topic, value)).get();
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//            throw new RuntimeException("报文发送异常:"+e.getMessage());
//        } catch (ExecutionException e) {
//            e.printStackTrace();
//            throw new RuntimeException("报文发送异常:"+e.getMessage());
//        }
//    }


    /**
     * 发送消息同步(无回调)
     * @param topic
     * @param partition
     * @param key
     * @param value
     */


    public void produceSyn(String topic,String key, T value) {
        try {
            this.producer.send(new ProducerRecord(topic, key, value)).get();
            logger.info("生产报文--->topic:{},key:{},msg:{}",topic, key, value);
        } catch (InterruptedException e) {
            e.printStackTrace();
            throw new RuntimeException("报文发送异常:"+e.getMessage());
        } catch (ExecutionException e) {
            e.printStackTrace();
            throw new RuntimeException("报文发送异常:"+e.getMessage());
        }
    }


    /**
     * 消息发送回调函数
     */
    class ProduceCallback implements Callback {
        private String topic;
        private T value;
        private long startTime;

        public ProduceCallback(String topic, T value, long startTime) {
            this.topic = topic;
            this.value = value;
            this.startTime = startTime;
        }

        public void onCompletion(RecordMetadata metadata, Exception exception) {
            if(metadata != null) {
                KafkaProducerService.this.logger.debug(">>>topic {}, message({}) sent to partition[{}],offset({}) in {} ms", new Object[]{this.topic, this.value, Integer.valueOf(metadata.partition()), Long.valueOf(metadata.offset()), Long.valueOf(System.currentTimeMillis() - this.startTime)});
                if(KafkaProducerService.this.producerHandler!=null)KafkaProducerService.this.producerHandler.whenProduceSucceed(this.topic,this.value);
            } else {
                exception.printStackTrace();
                if(KafkaProducerService.this.producerHandler!=null)KafkaProducerService.this.producerHandler.whenProducerFailed(this.topic,this.value,exception);
            }

        }
    }
```

序列化

```
public class StringDeserializerZip extends StringDeserializer {
    private String encoding = "UTF8";
    @Override
    public String deserialize(String topic, byte[] data) {
        try {
            return data == null?null: ZipStrUtil.unCompress(new String(data, this.encoding));
        } catch (UnsupportedEncodingException var4) {
            throw new SerializationException("Error when deserializing byte[] to string due to unsupported encoding " + this.encoding);
        } catch (IOException var5){
            throw new SerializationException("Error when deserializing :"+var5.getMessage());
        }
    }
}

public class StringSerializerZip extends StringSerializer {
    private String encoding = "UTF8";
    @Override
    public byte[] serialize(String topic, String data) {
        try {
            data = ZipStrUtil.compress(data);
            return data == null?null:data.getBytes(this.encoding);
        } catch (UnsupportedEncodingException var4) {
            throw new SerializationException("Error when serializing string to byte[] due to unsupported encoding " + this.encoding);
        }catch (IOException var5){
            throw new SerializationException("Error when deserializing :"+var5.getMessage());
        }
    }
}


public class ZipStrUtil {
    public static void main(String[] args) throws IOException {
        // 字符串超过一定的长度
        String str = "ABCdef123中文~!@#$%^&*()_+{};/1111111111111111111111111AAAAAAAAAAAJDLFJDLFJDLFJLDFFFFJEIIIIIIIIIIFJJJJJJJJJJJJALLLLLLLLLLLLLLLLLLLLLL" +
                "LLppppppppppppppppppppppppppppppppppppppppp===========================------------------------------iiiiiiiiiiiiiiiiiiiiiii";
        for(int i=0;i<10;i++){
            str+=str;
        }
        System.out.println("\n原始的字符串为------->" + str);
        float len0=str.length();
        System.out.println("原始的字符串长度为------->"+len0);

        String ys = compress(str);
        System.out.println("\n压缩后的字符串为----->" + ys);
        float len1=ys.length();
        System.out.println("压缩后的字符串长度为----->" + len1);

        String jy = unCompress(ys);
        System.out.println("\n解压缩后的字符串为--->" + jy);
        System.out.println("解压缩后的字符串长度为--->"+jy.length());

        System.out.println("\n压缩比例为"+len1/len0);

        //判断
        if(str.equals(jy)){
            System.out.println("先压缩再解压以后字符串和原来的是一模一样的");
        }
    }

    /**
     * 字符串的压缩
     *
     * @param str
     *            待压缩的字符串
     * @return    返回压缩后的字符串
     * @throws IOException
     */
    public static String compress(String str) throws IOException {
        if (null == str || str.length() <= 0) {
            return str;
        }
        // 创建一个新的 byte 数组输出流
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        // 使用默认缓冲区大小创建新的输出流
        GZIPOutputStream gzip = new GZIPOutputStream(out);
        // 将 b.length 个字节写入此输出流
        gzip.write(str.getBytes());
        gzip.close();
        // 使用指定的 charsetName，通过解码字节将缓冲区内容转换为字符串
        return out.toString("ISO-8859-1");
    }

    /**
     * 字符串的解压
     *
     * @param str
     *            对字符串解压
     * @return    返回解压缩后的字符串
     * @throws IOException
     */
    public static String unCompress(String str) throws IOException {
        if (null == str || str.length() <= 0) {
            return str;
        }
        // 创建一个新的 byte 数组输出流
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        // 创建一个 ByteArrayInputStream，使用 buf 作为其缓冲区数组
        ByteArrayInputStream in = new ByteArrayInputStream(str
                .getBytes("ISO-8859-1"));
        // 使用默认缓冲区大小创建新的输入流
        GZIPInputStream gzip = new GZIPInputStream(in);
        byte[] buffer = new byte[256];
        int n = 0;
        while ((n = gzip.read(buffer)) >= 0) {// 将未压缩数据读入字节数组
            // 将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此 byte数组输出流
            out.write(buffer, 0, n);
        }
        // 使用指定的 charsetName，通过解码字节将缓冲区内容转换为字符串
        return out.toString("UTF-8");
    }
}

```


