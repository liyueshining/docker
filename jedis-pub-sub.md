# Jedis实现Publish/Subscribe功能
Redis提供了publish/subscribe(发布/订阅)功能。可以对某个channel(频道)进行subscribe(订阅)，当这个channel上有publish(发布)的消息时，redis就会收到该channel上发布的消息，然后进行相应的处理。 
Java的redis客户端Jedis提供了publish/subscribe的接口。

## 演练
### 创建订阅类（Subscriber）
Jedis的抽象类JedisPubSub中定义了publish/subsribe的回调方法。
通过继承JedisPubSub类并重新实现这些回调方法，可以定制自己的业务逻辑。
```
import redis.clients.jedis.JedisPubSub;

public class Subscriber extends JedisPubSub
{
    public Subscriber() {
    }

    public void onMessage(String channel, String message) {
        System.out.println(String.format("receive redis published message, channel %s, message %s", channel, message));
    }

    public void onSubscribe(String channel, int subscribedChannels) {
        System.out.println(String.format("subscribe redis channel success, channel %s, subscribedChannels %d",
                channel, subscribedChannels));
    }

    public void onUnsubscribe(String channel, int subscribedChannels) {
        System.out.println(String.format("unsubscribe redis channel, channel %s, subscribedChannels %d",
                channel, subscribedChannels));
    }
}
```

### 创建订阅线程类（SubScriberThread）
Jedis的subscribe操作是阻塞的，因此需要单独起一个线程来进行subscribe操作
```
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class SubscribeThread implements Runnable {
    private final JedisPool jedisPool;
    private final Subscriber subscriber = new Subscriber();

    private final String channel = "mychannel";

    public SubscribeThread(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    @Override
    public void run() {
        System.out.println(String.format("subscribe redis, channel %s, thread will be blocked", channel));
        try(Jedis jedis = jedisPool.getResource()) {
            jedis.subscribe(subscriber, channel);
            System.out.println("message subscribe over");
        } catch (Exception e) {
            System.out.println(String.format("subsrcibe channel error, %s", e));
        }
    }
}
```

### 创建发布类（Publisher）
Publisher类接受用户的输入，并将输入发布到channel。当用户输入”quit”后，输入结束
```
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Publisher {
    private final JedisPool jedisPool;

    public Publisher(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    public void start() {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        while (true) {
            String line = null;
            try(Jedis jedis = jedisPool.getResource()) {
                line = reader.readLine();
                if (!"quit".equals(line)) {
                    jedis.publish("mychannel", line);
                } else {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 创建程序入口
```
import com.moon.springbootpractice.redis.Publisher;
import com.moon.springbootpractice.redis.SubscribeThread;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class JedisPubSubDemo{
    private static final String COMMA = ",";
    private static final String COLON = ":";

    public static void main( String[] args ) throws IOException{
        ExecutorService executorService = Executors.newCachedThreadPool();
        String redisIp = "172.16.0.93";
        int reidsPort = 6379;
        JedisPool jedisPool = new JedisPool(new JedisPoolConfig(), redisIp, reidsPort);
        System.out.println(String.format("redis pool is starting, redis ip %s, redis port %d", redisIp, reidsPort));

        executorService.execute(new SubscribeThread(jedisPool));

        Publisher publisher = new Publisher(jedisPool);
        publisher.start();

        System.out.println("idle  " + jedisPool.getNumIdle());
        System.out.println("active  " + jedisPool.getNumActive());
    }
}
```
在上面的代码中，我们首先生成了一个JedisPool的redis连接池，这是由于Jedis不是线程安全的，JedisPool是线程安全的。

## JedisCluster的使用
### 创建发布类
```
import redis.clients.jedis.JedisCluster;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class ClusterPublisher
{
    private final JedisCluster jedisCluster;

    public ClusterPublisher(JedisCluster jedisCluster){
        this.jedisCluster = jedisCluster;
    }

    public void start() throws IOException{
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

        while (true)
        {
            String line = null;
            line = reader.readLine();
            if (!"quit".equals(line)){
                jedisCluster.publish("mychannel", line);
            }else{
                break;
            }
        }
    }
}
```
JedisCluster的api中已经实现了关闭connection的功能，不需要再去关闭。

### 创建订阅线程
```
import redis.clients.jedis.JedisCluster;

public class ClusterSubThread implements Runnable
{
    private final JedisCluster jedisCluster;
    private final Subscriber subscriber = new Subscriber();

    private final String channel = "mychannel";

    public ClusterSubThread(JedisCluster jedisCluster){
        this.jedisCluster = jedisCluster;
    }

    @Override
    public void run(){
        System.out.println(String.format(
                "subscribe redis, channel %s, thread will be blocked",
                channel));

        jedisCluster.subscribe(subscriber, channel);
        System.out.println("message subscribe over");
    }
}
```

### 创建程序入口
```
import com.moon.springbootpractice.redis.ClusterPublisher;
import com.moon.springbootpractice.redis.ClusterSubThread;
import com.moon.springbootpractice.redis.Publisher;
import com.moon.springbootpractice.redis.SubscribeThread;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class JedisPubSubDemo
{
    private static final String COMMA = ",";
    private static final String COLON = ":";

    public static void main( String[] args ) throws IOException{
        ExecutorService executorService = Executors.newCachedThreadPool();
        
        String servers = "172.16.0.44:31001,172.16.0.44:31002,172.16.0.44:31003,172.16.0.102:31001,172.16.0.102:31002,172.16.0.102:31003";
        Set<HostAndPort> addrs = new HashSet<>();
        for (String server: servers.split(COMMA))
        {
            String[] addr = server.split(COLON);
            HostAndPort hostAndPort = new HostAndPort(addr[0], Integer.parseInt(addr[1]));
            addrs.add(hostAndPort);
        }

        JedisCluster jedisCluster = new JedisCluster(addrs, 1000, new JedisPoolConfig());
        executorService.execute(new ClusterSubThread(jedisCluster));
        ClusterPublisher publisher = new ClusterPublisher(jedisCluster);
        publisher.start();

        Map<String, JedisPool> nodeAndJedisPool = jedisCluster.getClusterNodes();
        int activeConnections = nodeAndJedisPool.values()
                .stream()
                .map(pool -> pool.getNumActive())
                .reduce((activeNum1, activeNum2) -> (activeNum1 + activeNum2))
                .get();

        int idleConnections = nodeAndJedisPool.values()
                .stream()
                .map(pool -> pool.getNumIdle())
                .reduce((idleNum1, idleNum2) -> (idleNum1 + idleNum2))
                .get();

        System.out.println("idle  " + idleConnections);
        System.out.println("active  " + activeConnections);
    }
}
```
