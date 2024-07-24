[TOC]

# 一. Flink

`Apache Flink`是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。Flink设计在所有常见的集群环境中运行，以内存速度和任何规模执行计算。

## 1. 流数据

流数据是一组顺序、大量、快速、连续到达的数据序列,一般情况下,数据流可被视为一个随时间延续而无限增长的动态数据集合。应用于网络监控、传感器网络、航空航天、气象测控和金融服务等领域。

流数据有四个特点：

- 数据实时到达
- 数据到达次序独立，不受应用系统所控制
- 数据规模宏大且不能预知其最大值
- 数据一经处理，除非特意保存，否则不能再次取出处理，或者再次提取数据代价昂贵

流处理与批处理的区别

|          |                   批处理                   |                          流处理                          |
| :------: | :----------------------------------------: | :------------------------------------------------------: |
| 数据范围 | 对数据集中的所有或大部分数据进行查询或处理 | 对滚动时间窗口内的数据或仅对最近的数据记录进行查询或处理 |
| 数据大小 |                 大批量数据                 |            单条记录或包含几条记录的微批量数据            |
|   性能   |           几分钟至几个小时的延迟           |                  大约几秒或几毫秒的延迟                  |
|   分析   |                  复杂分析                  |              简单的响应函数、聚合和滚动指标              |

## 2. 流数据分类

数据可以分为**无界数据流（Unbounded Streams)或有界数据流（Bounded Streams)**

- 无界流数据有一个开始，但没有定义的结束。他们不会在生成时终止并提供数据，必须持续处理无界流，即必须在摄取事件后理解处理事件，无法等待所有输入数据到达，因为输入是无界的，并且在任何时间点都不会完成。处理无界数据通常要求以特定顺序（例如事件发生的顺序）摄取事件，以便能够推断结果完整性。
- 有界流数据具有定义的开始和结束，可以执行任何计算之前通过摄取所有数据来处理有   界流数据。处理有界流数据不需要有序摄取，因为可以始终对有界数据集进行排序，**有界流的处理也成为批处理**。

![](D:\workspace\myNotes\assets\无界数据流与有界数据流.jpg)

注意，我们一般所说的<font color=red size=5>**数据流**</font>是指数据集，而<font color=red size=5>**流数据**</font>则是指数据流中的数据

Flink以流处理的方式来进行流、批统一，既可以兼容高效率的流式计算，也可以兼容高吞吐的批量计算

流处理最需要关注的是低延迟和`Exactly-once`（精确处理，既不丢数据，也不重复处理数据）保证，批处理更关注高吞吐、高效处理

## 3. Flink中的编程模型

Flink中，编程模型的抽象层级主要分为以下4种，越往下抽象都越低，编程越复杂，灵活度越高

![Flink编程模型](D:\workspace\myNotes\assets\Flink编程模型.jpg)

在上述4层中，一般用于开发的时第三层，即`DataStream/DataSet API`，用户可以使用`DataSteam API`处理无界流数据，使用`DataSet API`处理有界流数据。同时这两个`API`提供了各种各样的借口来处理数据，例如常见的`map`、`filter`、`flatmap`等等，而且支持`python`、`scala`、`java`等编程语言。

## 4. 流式计算梳理

在处理流式数据时，通常有两种不同的思路

1. 来一条数据处理一条数据

   > 例如利用`Kafka`建立一个消费者应用。这个应用会一直挂起来，`Kafka`中来一条消息，就会处理一条消息，但当数据越来越多时，处理能力不足，就可能产生消息积累，从而影响消息处理的及时性

2. 将流式数据堪称一个一个小的批量数据

   > 例如在`Spark Streaming`在处理流数据时，是把流式数据划分成一个小的批量数据，称为窗口。

# 二. Flink安装部署

## 1. Flink的部署方式

Flink的部署方式非常多，支持`Local`、`Standalone`、`Yarn`、`Mesos`、`Docker`、`Kubernetes`、`AWS`等

`Local`：不单独部署运行环境，在代码中可以直接调试

`Standalone`：独立运行环境，在这种模式下，Flink将自己完全管理运行资源，这种方式，其资源利用率时比较低的

`Yarn`：以`Hadoop`提供的`Yarn`作为资源管理服务，可以更高效的使用集群的机器资源

## 2. 实验环境与前置软件

前置软件：`JDK8`、`Hadoop`、`Zookeeper`（Flink内置了`zookeeper`，但在优化部署时通常采用外置`zookeeper`）

# 三. Flink运行架构

## 1. JobManager和TaskManager

Flink中的节点可以分为`JobManager`和`TaskManager`

<font color=red size=6>JobManager</font>处理器也称为`Master`，用于<font color=red size=6>协调分布式任务执行</font>。它们用来调度`task`进行具体的任务，<font color=red size=6>TaskManager</font>处理器也称为`Worker`，<font color=red size=6>用于实际执行任务</font>

![JobManager与TaskManager](D:\workspace\myNotes\assets\JobManager与TaskManager.jpg)

当`JobManager`在接受到任务时，整体执行的流程（`yarn`管理模式）如下：

![JobManager执行流程](D:\workspace\myNotes\assets\JobManager执行流程.jpg)

Flink客户端会往`JobManager`提交任务，`JobManager`会往`ResouceManager`申请资源，当资源足够时，再将任务分配给集群中的`TaskManager`去执行

## 2. 并发度与Slots

每一个`TaskManager`都是一个独立的`JVM`进程，它可以在独立的线程上执行一个或多个任务`task`。为了控制一个`TaskManager`能接受多少个`task`，`TaskManager`上就会划分出多少个`slot`来进行控制，每个`slot`表示的是`TaskManager`上拥有资源的一个固定大小的子集。`flink-conf.yaml`配置文件中的`taskmanager.numberOfTaskSlots`属性就是配置了`TaskManager`上有多少个`slot`，默认值是1。

<font color=red size=4>**`Task Slot`**</font>是一个静态的概念，代表的是`TaskManager`<font color=red size=4>**具有的并发执行能力**</font>。另外还有一个概念是<font color=red size=4>**并行度`parallelism`**</font>，它是一个动态的概念，表示的是<font color=red size=4>**运行程序时实际需要使用的并发能力**</font>，可以通过Flink程序控制

如果集群提供的`slot`资源不够，那么程序就无法正常执行下去，会表现为任务阻塞或者超时异常

程序运行的`parallelism`管理有三个地方可以配置：

- 优先级最低的是在`flink-conf.yaml`文件中的`parallelism.default`这个属性，其默认值是1
- 优先级较高的是在提交任务时可以指定任务整体的并行度要求，这个并行度可以在提交任务的管理页面和命令行中添加
- 优先级最高的是在程序中指定的并行度，几乎每一个分布式操作都可以定制单独的并行度

![Flink集群架构图](D:\workspace\myNotes\assets\Flink集群架构图.png)

- **客户端**

对于Flink，可以通过执行一个`Java`/`Scala`程序，或者通过`./bin/flink run`指令启动一个客户端，客户端将把`sataflow`提交给`JobManager`。客户端的主要作用其实就是构建一个`Dataflow graph`或者也称为`JobGraph`，然后提交给客户端，而这个`JobGraph`如果在客户端本地构建，这就是`Per-job`模式，如果是提交到`JobManager`由Flink集群来构建，这就是`Application`模式。然后将提交完成后，客户端可以选择立即结束，这就是`detached`模式；也可以选择继续执行，来不断跟踪`JobManager`反馈的任务执行情况，这就是默认的`attached`模式

- **`JobManager`**

  `JobManager`会首先接受到客户端提交的应用程序，这个应用程序整体会包含几个部分：作业图`JobGraph`，数据流图`Logic Dataflow Graph`以及打包了所有类似库以及资源的`jar`包，这些资源都将分发给所有的`TaskManager`去真正执行任务。

  `JobGraph`相当于是一个设计图，之前`Yarn`的`Per-job`模式，往集群提交的就是这个`JobGraph`，`JobManager`会把`JobGraph`转换成一个物理层面的数据流图，这个图被叫做执行图`ExecutionGraph`，这其中包含了所有可以并发执行的任务，相当于是一个执行计划。接下来`JobGraph`回想资源管理器（例如`Yarn`中的`ResourceManager`请求执行任务必要的资源），这些资源会表现为`TaskManager`上的`slot`插槽。一旦获得了足够多的资源，就会将执行图分发到真正运行任务的`TaskManager`上，而在运行过程中，`JobManager`还会负责所有需要中央协调的操作（例如反馈任务执行结果，协调检查点备份，协调故障恢复等）。

  `JobManager`整体上由三个功能模块组成：

  - `ResourceManager`

    `ResourceManager`在Flink集群中负责申请、提供和注销集群资源，并且管理`task slots`。Flink中提供了非常多的`ResourceManager`实现（比如`Yarn`，`Mesos`，`K8s`和`standalone`模式）。在`standalone`模式下，`ResourceManager`只负责在`TaskManager`之间协调`slot`的分配，而`TaskManager`的启动只能由`TaskManager`自己管理

  - `Dispatcher`

    `Dispatcher`模块提供了一系列的`REST`接口来提交任务，Flink的控制台也是由这个模块来提供，并且对于每一个执行的任务，`Dispather`会启动一个系的`JobMaster`来对任务进行协调

  - `JobMaster`

    一个`JobMaster`负责管理一个单独的`JobGraph`，Flink集群中，同一时间可以运行多个任务，每个任务都由一个对应的`JobMaster`来管理

  一个集群中最少有一个`JobManager`，而在高可用部署时，也可以有多个发`JobManager`。这些`JobManager`会选出一个作为`Leader`，而其他的节点就处于`StandBy`备用状态

- **`TaskManager`**

`TaskManager`也称为`Worker`，每个`TaskManager`上可以有一个或多个`Slot`，<font color=red>`Slot`就是程序运行的最小单元</font>，可以在`flink.conf.yaml`文件中通过`taskmanager.numberOfTaskSlots`属性进行配置

![TaskManager](D:\workspace\myNotes\assets\TaskManager.png)

每一个`TaskManager`都是一个单独的`JVM`进程，而每个`Slot`就会以这个进程中的一个线程来执行，这些`Slot`在同一个任务中时共享的，一个`Slot`就足以贯穿应用的整个处理流程。Flink集群只需关注一个任务的内的最大并行数，提供足够的`Slot`即可

# 四. Flink DataStream API

```java
//  设置任务并行度
environment.setParallelism(1);
//  设置任务最大并发量
environment.setMaxParallelism(2);
/**
 * 设置运行模式
 * RuntimeExecutionMode 为枚举类型，其包含
 *  STREAMING-流式模式
 *  BATCH-批量模式
 *  AUTOMATIC-根据数据类型自动选择
 */
environment.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
```

Flink提供的`API`非常丰富，总体来说可以分为`DataStream API`、`DataSet API`、`Table API & SQL`三部分

- `DataStream API`时Flink主要进行流计算的模块
- `DataSet API`是Flink中主要进行批量计算的模块
- `Table API & SQL`主要是对Flink数据集提供类似于关系型数据库的数据查询过滤等功能

## 1. Flink程序的基础运行模型

`DataStream`在Flink的应用程序中被认为是一个不可更改的数据集，这个数据集可以是无界的，也可以是有界的，Flink对他们的处理方式是一定的，这也就是所谓的流批统一。一个`DataStream`和`Java`基础中的集合很相似，都可以迭代处理，但`DataStream`中的数据在创建之后就不能再进行增删改的操作

一个Flink的基础运行模型如下

![Flink基础运行模型](D:\workspace\myNotes\assets\Flink基础运行模型.png)

一个Flink的客户端应用主要分为五个阶段：

- 获取一个执行环境`Environment`
- 通过`Source`，定义数据的来源
- 对数据定义一系列的操作，`Transfomations`
- 通过`Sink`，定义程序处理的结果要输出到哪里
- 提交并启动任务

## 2. Environment运行环境

`StreamExecutionEnvironment`是所有Flink中流式计算程序的基础，有三种创建环境的方式

```java
//	处理流数据的运行环境
StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
StreamExecutionEnvironment.createLocalEnvironment();
StreamExecutionEnvironment.createRemoteEnvironment(String host, int port, String jarFiles);
//	处理批量数据的运行环境
ExecutionEnvironment executionEnvironment = ExecutionEnvirnment.getExecutionEnvironment();
```

通常情况只需使用`getExecutionEnvionment()`这一种方式就可以，其会根据运行环境自动创建正确的`StreamExecutionEnvironment`对象，无须区分应用实在本地执行还是在`Flink Cluster`上执行

`Environment`可以进行诸多的设置，例如：

```java
//  设置任务并行度
environment.setParallelism(int num);
//  设置任务最大并发量
environment.setMaxParallelism(int num);
/**
 * 设置运行模式
 * RuntimeExecutionMode 为枚举类型，其包含
 *  STREAMING-流式模式
 *  BATCH-批量模式
 *  AUTOMATIC-根据数据类型自动选择
 */
environment.setRuntimeMode(RuntimeExecutionMode mode);
```

`StreamExcutionEnvironment`对象可以通过`setRuntimeMode()`方法设置`RuntimeExecutionMode`枚举类型的运行模式，该类型有三个可选的枚举值：

- `STREAMING`：流式模式

流式模式下，所有的`task`都会在应用执行时完成部署，后续所有的任务都会连续不断的进行

- `BATCH`：批量模式

Flink早期的处理模式，所有的任务都会周期性的部署，`shuffle`的过程也会造成阻塞，相当于是拿一批数据处理结束之后，再接收并处理下一批任务

- `AUTOMIC`：自动模式

Flink将会根据数据集类型自动选择处理模式，有界流下选择`BATCH`模式，无界流下选择`STREAMING`模式

通常，执行模式不建议在代码中指定，其还可以通过`flink-conf.yaml`文件中通过`execution.runtime-mode`进行整体设置，或在提交任务时指定，这样可以让应用更加灵活。

## 3. Source数据源

Source表示Flink应用程序的数据输入，Flink中提供了十分丰富的Source实现，目前主流的数据源都可以对接

### 3.1 基于file的数据源

1. `readTextFile(path)`

   按行读取文件中的内容，并将结果以`String`的形式返回

   ```java
   StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
   environment.setRuntimeMode(RuntimeExecutionMode.AUTOMATIC);
   DataStreamSource<String> stream = environment.readTextFile("D:/test.txt");
   stream.print();
   environment.execute();
   ```

   打印结果中每一行前的数字表示这一行是由哪一个线程打出来的

2. `readfile((FileInputFormat inputFormat, String filePath))`

   ```java
   DataStreamSource<String> stream = environment.readFile(new TextInputFormat(new Path("D://test.txt")), "D://test.txt");
   ```

   `TextInputFormat`是一个接口，`OUT`泛型代表返回的数据类型。`TextInputFormat`的返回类型是`String`，`PojoCsvInputFormat`就可以指定从csv文件中读取一个`POJO`类型的对象

### 3.2 基于Socket的数据源

对接一个Socket通道以读取数据

```java
DataStreamSource<String> stream = environment.socketTextStream("localhost", 11111);
stream.print();
environment.execute("stream word count");
```

### 3.3 基于集合的数据源

1. `fromCollection`从集合获取数据

   ```java
   StreamExecutionEnvironment environment = StreamExecutionEnvirionment.getEnvironment();
   List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
   DataStreamSource<Integer> stream = environment.fromCollection(list);
   stream.print();
   environment.execute("stream");
   ```

2. `fromElements`从指定元素结合中获取数据

   ```java
   DataStreamSource<Integer> stream = environment.fromElements(1, 2, 3, 4,5);
   ```

### 3.4 从Kafka读取数据

Flink提供了针对Kafka的Source支持，引入Kafka的连接器，需要引入maven依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.12</artifactId>
    <version>1.12.3</version>
</dependency>
```

注：`fink-connector-kafka_2.12`中数字代表Scala版本，`<version>`标签中代表Kafka版本

然后使用`FlinkKafkaConsume`创建Source

```java
StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();

// 配置Kafka连接属性
Properties properties = new Properties();
properties.setProperty("bootstrap servers", "work01:9092, work02:9092, work03: 9093");
properties.setProperty("group.id", "test");

FlinkKafkaConsumer<String> mysource = new FlinkKafkaConsumer<String>("flinktopic", new SimpleStringSchema(), properties);
DataStream<String> stream = environment.addSource(mysource);
stream.print();
environment.execute("Kafka Consume");
```

Flink提供了非常多常用组件的Connector，例如Hadoop/HBase/ES/JDBC等，具体可参考官网的Connector模块

> 对于组件 RocketMQ，Flink官方并没有提供 RocketMQ 的 Connector，但是 RocketMQ 社区做了一个 Flink 的 Connector ，可参见 Git 仓库：https://github.com/apache/rocketmq-externals

### 3.5 自定义Source

用户程序可以基于 Flink 提供的 `SourceFunction`，配置自定义的 Source 数据源。

```java
public class UDFSource {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setParallelism(1);
        DataStreamSource<Stock> orderDataStreamSource = environment.addSource(new MyOrderSource());
        orderDataStreamSource.print();
        environment.execute("UDFSource");
    }

    public static class MyOrderSource implements SourceFunction<Stock> {
        private boolean running = true;

        @Override
        public void run(SourceContext<Stock> sourceContext) throws Exception {
            final Random  random = new Random();
            while (running) {
                Stock stock = new Stock();
                stock.setId("stock" + System.currentTimeMillis() % 700);
                stock.setPrice(random.nextDouble() * 100);
                stock.setStockName("UDFStock");
                stock.setTimestamp(System.currentTimeMillis());

                sourceContext.collect(stock);
                Thread.sleep(1000);
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }
}
```

## 4. Sink输出

Sink 是 Flink 的输出组件，负责将 DataStream 中的数据输出到文件、Socket、外部系统等

但是需要添加单独的 Maven 依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-files</artifactId>
    <version>${flink.version}</version>
</dependency>
```

### 4.1 输出到控制台

DataStream 可以 通过`print()`和`printToErr()`将结果输出到标准控制台，在 Flink 中可以在 TaskManager 的控制台中查看

### 4.2 输出到文件

对于 DataStream，有两个方法 `writeAsText` 和 `writeAsCsv`，可以将结果直接输出到文本文件中。但是在当前版本下这两个方法已被标记为过时。当前推荐使用 `SteamingFileSink`

```java
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.connector.file.sink.FileSink;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.OutputFileConfig;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;

import java.io.FileReader;
import java.net.URL;

public class FileSinkDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.enableCheckpointing(100);
        URL resource = FileReader.class.getResource("/test.txt");
        String filePath = resource.getFile();
        DataStreamSource<String> stream = environment.readTextFile(filePath);

        OutputFileConfig outputFileConfig = OutputFileConfig
                .builder()
                .withPartPrefix("prefix")
                .withPartSuffix(".txt")
                .build();
        StreamingFileSink<String> stringStreamingFileSink = StreamingFileSink
                .forRowFormat(new Path("D:/workspace/FlinkTest"), new SimpleStringEncoder<String>("UTF-8"))
                .withOutputFileConfig(outputFileConfig)
                .build();
        // addSink(SinkFuncation funcation)方法传入参数为 SinkFunction 对象
        stream.addSink(stringStreamingFileSink);

        /**
        FileSink<String> fileSink = FileSink
                .forRowFormat(new Path("D:/workspace/FlinkTest"), new SimpleStringEncoder<String>("UTF-8"))
                .withOutputFileConfig(outputFileConfig)
                .build();
        // sinkTo(FileSink fileSink)方法传入参数为 FileSink 对象
        stream.sinkTo(fileSink);
         */

        environment.execute("FileSink");
    }
}
```

通常情况下，流式数据很少会要求输出到文件中，更多的场景是会直接输出到其他下游组件中，如 Kafka、ES等

### 4.3 输出到Socket

```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.serialization.SerializationSchema;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

import java.nio.charset.StandardCharsets;

public class SocketSinkDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        ParameterTool parameterTool = ParameterTool.fromArgs(args);
        String host = parameterTool.get("host");
        int port = parameterTool.getInt("port");

        DataStreamSource<String> source = environment.socketTextStream(host, port);

        DataStream<Tuple2<String, Integer>> wordcounts = source.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String s, Collector<Tuple2<String, Integer>> collector) throws Exception {
                String[] words = s.split(" ");
                for (String word: words) {
                    collector.collect(new Tuple2<String, Integer>(word, 1));
                }
            }
        })
                .keyBy(s -> s.f0)
                .sum(1);
        wordcounts.print();

        // 输出到 Socket
        wordcounts.writeToSocket(host, port, new SerializationSchema<Tuple2<String, Integer>>() {
            @Override
            public byte[] serialize(Tuple2<String, Integer> element) {
                return (element.f0 + "-" + element.f1).getBytes(StandardCharsets.UTF_8);
            }
        });
        environment.execute("SocketSinkDemo");
    }
}
```

### 4.4 输出到Kafka

Flink 提供了 Kafka 的 Connector 模块，即提供了 FlinkKafkaConsumer 作为 Source 消费信息，也提供了 FlinkKafkaProducer 作为 Sink 生产消息

```java
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;
import org.apache.flink.streaming.connectors.kafka.partitioner.FlinkFixedPartitioner;
import org.apache.flink.streaming.util.serialization.SimpleStringSchema;

import java.util.Properties;

public class KafkaSinkDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "test");

        FlinkKafkaConsumer<String> mySource = new FlinkKafkaConsumer<String>("flinktopic", new SimpleStringSchema(), properties);
        DataStream<String> stream = environment.addSource(mySource);
        stream.print();

        //  转存到另一个topic
        properties = new Properties();
        properties.setProperty("bootstrap.servers", "worker01:9092, worker02:9092, worker03:9092");
        FlinkKafkaProducer<String> myProducer = new FlinkKafkaProducer<String>("flinktopic02", new SimpleStringSchema(), properties, new FlinkFixedPartitioner<>(), FlinkKafkaProducer.Semantic.EXACTLY_ONCE, 5);
        stream.addSink(myProducer);

        environment.execute("KafkaConsumer");
    }
}
```

该 Demo 从一个 topic 接收数据，处理完成后，转发到另一个 topic ，这是一个典型的流式计算场景

### 4.5 自定义Sink

与 Source 类似，可以通过不带生命周期的 SinkFunction 以及带生命周期的 RickSInkFunction 来定义自己的 Sink 实现。

如下展示了把一个消息存入到 MySQL 的实例

```java
import com.Sinotruk.flink.basic.beans.Stock;
import com.Sinotruk.flink.basic.source.UDFSource;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

public class CustomSinkDemo {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStreamSource<Stock> source = environment.addSource(new UDFSource.MyOrderSource());
        source.addSink(new MyJDBCSink());

        environment.execute("CustomSinkDemo");
    }

    // 也可以继承 SinkFunction，但由于要开启/关闭数据库的连接，因此此处继承 RichSinkFunction
    public static class MyJDBCSink extends RichSinkFunction<Stock> {
        Connection connection = null;
        PreparedStatement insertPstmt = null;
        PreparedStatement updatePstmt = null;

        @Override
        public void open(Configuration parameters) throws Exception {
            connection = DriverManager.getConnection("jdbc:mysql//localhost:3306/testdb", "root", "123456");
            insertPstmt = connection.prepareStatement("insert into flink_order(id, price, stockname) values (?, ?, ?)");
            updatePstmt = connection.prepareStatement("update flink_order set price=?, stockname=? where id=?");
        }

        @Override
        public void close() throws Exception {
            insertPstmt.close();
            updatePstmt.close();
            connection.close();
        }

        @Override
        public void invoke(Stock value, Context context) throws Exception {
            System.out.println("更新记录: " + value);
            updatePstmt.setDouble(1, value.getPrice());
            updatePstmt.setString(2, value.getStockName());
            updatePstmt.setString(3, value.getId());
            updatePstmt.execute();

            if (updatePstmt.getUpdateCount() == 0) {
                insertPstmt.setString(1, value.getId());
                insertPstmt.setDouble(2, value.getPrice());
                insertPstmt.setString(3, value.getStockName());
                insertPstmt.execute();
            }
        }
    }
}
```

另外，Flink 提供了一个 JDBC 的 Sink 工具包（不包含 JDBC 驱动）

```xml
<dependency>
	<groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-jdbc_2.12</artifactId>
    <version>${version}</version>
</dependency>
```

## 5. Transformation数据转换

对 DataStream 进行数据变换的操作（也称为算子），具体可见官方文档(https://nightlies.apache.org/flink/flink-docs-release-1.14/docs/dev/datastream/operators/overview/)

### 5.1 Map

DataStream -> DataStream

由一个元素生成另一个元素

```java
DataStream<Integer> dataStream = //...;
/**
  * 数据源传输进来的数据是 MapFunction 中第一个 Integer 参数
  * MapFunction 中第二个 Integer 参数表示输出数据的类型
  */
dataStream.map(new MapFunction<Integer, Integer>() {
    @Override
    public Integer map(Integer value) throws Exception {
        return 2 * value;
    }
});
```

### 5.2 FlatMap

DataStream -> DataStream

由一个元素生成零个、一个或多个元素，能够将复杂数据扁平化

```java
dataStream.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String value, Collector<String> out) throws Exception {
        for (String word: value.split(" ")) {
            out.collect(word);
        }
    }
});
```

### 5.3 Filter

DataStream -> DataStream

过滤，将满足条件的数据保留，不满足条件的数据剔除。为每个元素计算布尔值，并保留该函数返回 True 的元素，过滤掉返回 False 的元素。

```java
dataStream.filter(new FilterFunction(Integer) {
    @Override
    public boolean filter(Integer value) throws Exception {
        return value != 0;
    }
});
```

### 5.4 KeyBy

DataStream -> KeyedStream

将数据按照 key 分组，并按照给定的计算方法将 key 相同的的 value 聚合成一个新的 value

```java
dataStream.keyBy(value -> value.getSomeKey());
dataStream.keyBy(value -> value.f0;
```

> <font color=red size=5>**注意**</font>：以下类型不能作为 key
>
> 1.  POJO 类型没有复写 `hasCode()`方法并实现`Object.hasCode()`接口
> 2. 包含任何数据类的数组

KeyedStream 不能像 DataStream 一样执行诸如 Map/FlatMap 等算子操作，一般都会在 KeyBy 后执行 Reduce 算子操作将 KeyedStream 重新转换为 DataStream

### 5.5 Reduce

KeyedStream -> DataStream

在 KeyedStream 上将数据“滚动”减少，将当前元素与上一轮“减少”的元素结合成新的元素

```java
dataStream.reduce(new ReduceFunction(Integer)() {
    @Override
    public Integer reduce(Integer value1, Integer value2) throws Exception {
        return value1 + value2;
    }
})
```

> 例如当前经过 KeyBy 算子操作后，获得的数据为 `A:[1, 2, 1, 3, 5]`
>
> 则上述 Reduce 操作的过程为 `[1, 2, 1, 3, 5] -> [3, 1, 3, 5] -> [4, 3, 5] -> [7, 5] -> 12`

### 5.6 Aggregations

KeyedStream -> DataStream

在 KeyedStream 上进行诸多统计操作，诸如：

```java
keyedStream.sum(0);
keyedStream.sum("key");
keyedStream.min(0);
keyedStream.min("key");
keyedStream.max(0);
keyedStream.max("key");
keyedStream.minBy(0);
keyedStream.minBy("key");
keyedStream.maxBy(0);
keyedStream.maxBy("key");
```

`min()`与`minBy()`的不同之处在于`min()`返回一个最小值，而`minBy()`返回域中含有最小值的元素

### 5.7 Connect

DataStream, DataStream -> ConnectedStream

连接两个保持他们类型的数据流，两个数据流被 Connect 之后，只是被放在同一个流中，内部依然保持独立：数据和形式不发生任何的变化，两个流相互独立，通常只作为一个中间状态，然后进行后续的统计

```java
DataStream<Integer> someStream = ...;
DataStream<String> otherStream = ...;

ConnectedStreams<Integer, String> connectedStream = someStream.connect(otherStream);
```

### 5.8 CoMap/CoFlatMap

ConnectedStream -> DataStream

与之前的 Map/FlatMap 类似，只不过其作用在 ConnectedStream 上

```java
connectedStreams.map(new CoMapFunction<Integer, String, boolean>() {
	@Override
	public Boolean map1(Integer value) {
		return true;
	}
	
	@Override
	public boolean map2(String value) {
		return false;
	}
});

connectedStreams.flatMap(new CoFlatMapFunction<Integer, String, String>() {
	@Override
	public void flatMap1(Integer value, Collector<String> out) {
		out.collect(value.toString());
	}
	
	@Override
	public void flatMap2(String value, Collector<String> out) {
        for (String word: value.split(" ")) {
            out.collect(word);
        }
	}
});
```

### 5.9 Union

DataStream, DataStream -> DataStream

将两个 DataStream 的数据集合到一起，产生一个包含全部元素的新 DataStream，但是 Union 操作是不会去重的

```java
DataStream<Integer> stream = environment.fromElements(2, 4, 6, 8);
DataStream<Integer> stream2 = environment.fromElements(1, 3, 5, 7);
DataStream<Integer> union = stream.union(stream2);	// 1,2,3,4,5,6,7,8
```

### 5.10 Function与RichFunction

Function 是一个顶级的处理函数接口，之前用到的各种Source/Sink/Transform都是这两个接口的子实现类，Function 代表一个普通的函数接口，只对数据进行计算，Function接口本身并没有提供任何方法

RichFunction 是 Function 的一个直接子接口，包含了对任务的生命周期的管理：例如 `open()`方法，在 Slot 任务执行前触发，可以做一次性的初始化工作；`close()`方法则是在 Slot 任务执行之后触发，可以做一次性的收尾工作；`getRuntimeContext()`方法可以拿到方法执行的上下文及任务执行时的信息，例如当前子任务的ID、当前任务的状态后端等等

```java
import com.Sinotruk.flink.basic.beans.Stock;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.RichMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.nio.charset.StandardCharsets;

public class RichFunctionDemo {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setParallelism(4);

        final String filePath = RichFunctionDemo.class.getResource("/stock.txt").getFile();
        final DataStreamSource<String> dataStream = environment.readTextFile(filePath, StandardCharsets.UTF_8.name());

        // 将每一行转换为一个 Stock 对象
        final SingleOutputStreamOperator<Stock> stockStream = dataStream.map(new MapFunction<String, Stock>() {
            @Override
            public Stock map(String s) throws Exception {
                String[] split = s.split(",");
                return new Stock(split[0], Double.parseDouble(split[1]), split[2], Long.parseLong(split[3]));
            }
        });

        final SingleOutputStreamOperator<Tuple2<String, Integer>> resultStream = stockStream.map(new MyRichFunction());
        resultStream.print("resultStream");

        environment.execute("RichFunctionDemo");
    }

    public static class MyFunction implements MapFunction<Stock, Tuple2<String, Integer>> {
        @Override
        public Tuple2<String, Integer> map(Stock stock) throws Exception {
            return new Tuple2<>(stock.getId(), stock.getId().length());
        }
    }

    // 实现自定义富函数类
    public static class MyRichFunction extends RichMapFunction<Stock, Tuple2<String, Integer>> {
        @Override
        public Tuple2<String, Integer> map(Stock stock) throws Exception {
            // getRunTimeContext() 获取当前子任务执行的上下文，每个 TaskManager 都会有一个 RuntimeContext
            return new Tuple2<>(stock.getId(), getRuntimeContext().getIndexOfThisSubtask());
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            // 执行初始化操作，一般是定义状态，或者建立数据库链接，每个 slot 会执行一次
            System.out.println("open");
        }

        @Override
        public void close() throws Exception {
            // 一般是关闭连接和清空状态的收尾工作，每个 slot 会执行一次
            System.out.println("close");
        }
    }
}
```

## 6. Window 窗口计算

在 Flink 的流式计算中，数据都是以 DataStream 的形式来表示，而对数据的计算，基本都是**<font color=red>先分流后合流</font>**的过程，而 Windows 开窗函数可以理解为是一种更高级的分流方法：Window 将一个无限的流式数据 DataStream 拆分成有限大小的 “Bucket”桶，通过对桶中数据的计算最终完成整个流式数据的计算。这也是处理流式数据的一种常见的方法，在 Kafka/Stream/Spark Streaming 等这些流式框架中都有

Window 计算是流式计算中非常常用的数据计算方式之一，通过按照**<font color=red size=5>固定长度或长度</font>**将数据流切分成不同的窗口，然后对数据进行相应的聚合运算，从而得到一定时间范围内的统计结果

在 Flink DataStream API 中内建了大多数窗口算子，在每个窗口算子中包含了 Windows Assigner、Windows Trigger(窗口触发器)、Evictor(数据剔除器)、Lateness(时延设定)、Output Tag(输出标签)、Windows Function 等组成部分，其中 **<font color=red size=5>Windows Assigner 和 Windows Function 是所有窗口算子必须指定的属性</font>**，其余的属性都是根据实际情况来选择指定

```java
stream.keyBy(...) // 是 Keyed 类型数据集
    .window(...) // 指定窗口分配器类型
    [.trigger(...)] // 指定触发器类型
    [.evictor(...)] // 指定 evictor 或者不指定（可选）
    [.allowedLateness(...)] // 指定是否延迟处理数据（可选）
    [.sideOutputLateData(...)] // 指定 Output Lag （可选）
    .reduce/aggregate/apply() // 指定窗口计算函数
    [.getSideOutput(...)] // 根据 Tag 输出数据（可选）
```

- Windows Assigner: 指定窗口的类型，定义如何将数据流分配到一个或多个窗口
- Windows Trigger: 指定窗口触发的时机，定义窗口满足什么样的条件触发计算
- Evictor: 用于数据剔除
- Lateness: 标记是否处理迟到数据，当迟到数据到达窗口中是否触发计算
- Output Tag: 标记输出标签，然后在通过 `getSideOutput()` 将窗口中的数据根据标签输出
- Windows Function: 定义窗口上数据处理的逻辑，例如对数据进行 sum 操作

### 6.1 Windows 生命周期

一个 window 会指定一个包含数据的范围，从第一个属于它的数据到达之后就被创建出来，等所有的数据都被处理完成后就会被彻底移除。这个移除的时刻是由直嘀咕的窗口结束时间加上后续设定的 allowedLatensess时长决定的。

例如每分钟创建一个 window，正常从每分钟的0秒开始创建一个 window，然后到这一分钟的60秒就会结束这个 window，但是 Flink 允许设定一个延迟时间，比如5秒，那么这个 window 就会在下一分钟的5秒才会移除，这是为了防止网络传输延时造成的数据丢失

在 Flink 中，需要通过一个 WindowAssigner 对象来指定数据开窗的方式。例如，对于 DataStream，它的开窗方式如下

```java
stream.windowAll(TumblingEventTimeWindows.of(Time.seconds(60)));
public <W extends Window> AllWindowedStream<T, W> widowsAll(WindowAssigner<? super T, W> assigner)
```

### 6.2 Windows Assigner 

1. Keyed 和 Non-Keyed 窗口

   在运用窗口计算时，Flink 会根据上游数据集是否为 KeyedStream 类型（将数据集按照 Key 分区），对应的 Windows Assigner 也会有所不同

   - Keyed Window

     针对 KeyedStream 进行开窗。KeyedStream会将原始的无界流切分成多个逻辑撒上的KeyedStream，在KeyedStream上的开窗函数`window`，可以指定并行度，由多个任务并行执行计算任务，所有拥有相同 Key 的数据将会被分配到同一个并行任务中

     上游数据集如果是 KeyedStream 类型，则调用 DataStream API 的 `window()`方法指定 Windows Assigner，数据会根据 Key 在不同的 Task 实例中并行分别计算，最后得出针对每个 Key 统计的结果

   - Non-Keyed Window

     针对 DataStream 进行开窗，这种开窗是将所有的流式数据生成一个 window，这时这个 window 就不能进行并行计算，只能以并行度1来进行计算，由一个单独的任务进行计算。这种开窗6方式显然是不利于利用集群的整体资源的，所以通常用的比较少

     如果是 Non-Keyed 类型，则调用 `windowsAll()`方法来指定 Windows Assigner ，所有的数据都会在窗口算子中路由到一个 Task去计算并得到全局统计结果

2. 类型

   在 Flink 流式计算中，通过 Windows Assigner 将接入数据分配到不同的窗口，根据 Windows Assigner 数据分配方式的不同将 Windows 分为4大类，分别是：滚动窗口(Tumbling Window)、滑动窗口(Sliding Windows)、会话窗口(Session Windows)和全局窗口(Global Windows)

   - **滚动窗口**

     滚动窗口需要指定一个固定的窗口大小 window size，根据固定时间或大小进行切分，且窗口和窗口之间的元素不会重叠，如下图所示

     ![Tumbling Window](D:\workspace\myNotes\assets\Tumbling Window.png)

     这种类型的窗口的最大特点是比较简单，但可能会导致某些由前后关系的数据计算结果不正确，因此更加适用于固定大小和周期统计某一指标的这种类型的窗口计算

     ```java
     DataStream<T> input = ...;
     
     // tumbling event-time windows
     input
         .keyBy(<key selector>)
         .window(TumblingEventTimeWindows.of(Time.seconds(5)))
         .<windowed transformation>(<window function>);
     
     // tumbling processing-time windows
     input
         .keyBy(<key selector>)
         .window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
         .<windowed transformation>(<window function>);
     
     // daily tumbling event-time windows offset by -8 hours.
     input
         .keyBy(<key selector>)
         .window(TumblingEventTimeWindows.of(Time.days(1), Time.hours(-8)))
         .<windowed transformation>(<window function>);
     ```

     默认窗口时间的时区是 UTC-0，因此 UTC-0 以外的其他地区均需通过设定时间偏移量调整时区，<font color=red>**在国内需要指定 `Time.hours(-8)`的偏移量**</font>

   - **滑动窗口**

     滑动窗口其特点是在滚动窗口基础之上增加了窗口滑动时间(Slide Time)，且允许窗口数据发生重叠

     当 Windows size 固定之后，窗口并不像滚动窗口按照 Windows Size 向前移动，而是根据设定的 Slide Time 向前滑动

     ![Sliding Windows](D:\workspace\myNotes\assets\Sliding Windows.png)

     窗口之间的数据重叠大小根据 Window size 和 Slide time 决定，当 Slide time 小于 Window size ，便会发生窗口重叠；Slide size 大于 Windows size 就会出现窗口不连续，数据可能不会在任何一个窗口内计算；Slide size 和 Windows size 相等时，Sliding Windows 其实就是 Tumbling Windows

     ```java
     DataStream<T> input = ...;
     
     // sliding event-time windows
     input
         .keyBy(<key selector>)
         .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
         .<windowed transformation>(<window function>);
     
     // sliding processing-time windows
     input
         .keyBy(<key selector>)
         .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
         .<windowed transformation>(<window function>);
     
     // sliding processing-time windows offset by -8 hours
     input
         .keyBy(<key selector>)
         .window(SlidingProcessingTimeWindows.of(Time.hours(12), Time.hours(1), Time.hours(-8)))
         .<windowed transformation>(<window function>);
     ```

   - **会话窗口**

     会话窗口主要是将某段时间内活跃度较高的数据聚合成一个窗口进行计算，窗口触发的条件是 Session Gap，是指在规定时间内如果没有数据活跃接入，则认为窗口结束，然后触发窗口计算结果。需要注意的是，如果数据一直不间断的进入窗口，也会导致窗口始终不触发的情况

     与滑动窗口、滚动窗口不同的是，Session Windows 不需要由固定的 window size 和 slide time，只需要自定义 session gap 来规定不活跃数据的时间上限即可

     如图所示，通过 session gap 来判断数据是否属于同一数据集，从而将数据切分成不同的窗口进行计算

     ![Session Windows](D:\workspace\myNotes\assets\Session Windows.png)

     ```java
     DataStream<T> input = ...;
     
     // event-time session windows with static gap
     input
         .keyBy(<key selector>)
         .window(EventTimeSessionWindows.withGap(Time.minutes(10)))
         .<windowed transformation>(<window function>);
         
     // event-time session windows with dynamic gap
     input
         .keyBy(<key selector>)
         .window(EventTimeSessionWindows.withDynamicGap((element) -> {
             // determine and return session gap
         }))
         .<windowed transformation>(<window function>);
     
     // processing-time session windows with static gap
     input
         .keyBy(<key selector>)
         .window(ProcessingTimeSessionWindows.withGap(Time.minutes(10)))
         .<windowed transformation>(<window function>);
         
     // processing-time session windows with dynamic gap
     input
         .keyBy(<key selector>)
         .window(ProcessingTimeSessionWindows.withDynamicGap((element) -> {
             // determine and return session gap
         }))
         .<windowed transformation>(<window function>);
     ```

     由于 session windows 没有固定的起点和终点，因此对他们的评估不同于翻滚窗口和滑动窗口。<font color=blue size=5>在内部，会话窗口算了</font>**<font color=red size=6>为每个接受到的记录创建一个新的窗口</font>**，<font color=blue size=5>如果窗口之间的距离比定义的间隙更近，则将窗口合并在一起</font>。为了能够合并，会话窗口算子需要一个合并触发器和一个合并窗口函数，比如`ReduceFunction()`/`ArrgegateFunction()`/`ProcessWindowFunction()`

     `AggregateFunction` 中存在一个 `merger()` 方法，该方法仅会被 SessionWindow 调用

   - 全局窗口

     全局窗口(Global Windows)将所有相同的 key 的数据分配到单个窗口中计算结果，窗口没有起始和结束时间，窗口需要借助于 Triger 来触发计算，如果不对 Global Windows 指定 Triger，窗口时不会触发计算的。因此<font color=red size=6>**使用 Global Windows 需要**</font>非常谨慎，用户需要非常明确自己在整个窗口中统计出的结果是什么，并<font color=red size=6>**指定对应的触发器，同时还需要指定相应的数据清理机制，否则数据将一直留在内存中**</font>。
   
     ![Global Windows](D:\workspace\myNotes\assets\Global Windows.png)

### 6.3 Trigger 窗口触发器

数据接入窗口后，窗口是否触发 WindowFunction 计算，取决于窗口是否满足触发条件，每种类型的窗口都有对应的窗口触发机制，保障一次接入窗口的数据都能够按照规定的触发逻辑进行统计计算。Flink 在内部定义了窗口触发器来控制窗口的触发机制，分别有 EventTimeTrigger、ProcessTimeTrigger 以及 CountTrigger 等，每种触发器都对应不同的 Window Assigner

- EventTimeTrigger: 通过对比 Watermark 和 Windows EndTime 确定是否触发窗口，如果 Watermark 的时间大于 Windows EndTime 则触发计算，否则窗口继续等待
- ProcessTimeTrigger: 通过对比 ProcessTime 和 Window EndTime 确定是否触发窗口，如果窗口 ProcessTime 大于 Windows EndTime则触发计算，否则窗口继续等待
- ContinuousEventTimeTrigger: 根据间隔时间周期性触发窗口或者 Window 的结束时间小于当前 EventTime 触发窗口计算
- ContinuousProcessTimeTrigger: 根据间隔时间周期性触发窗口或者 Window 的结束时间小于当前 EventTime 触发窗口计算
- CountTrigger: 根据接入数据量是否超过设定的阈值确定是否触发窗口计算
- DeltaTrigger: 根据接入数据计算出来的 Delta 指标是否超过指定的 Threshold，判断是否触发窗口计算
- PurgingTrigger: 可以将任意触发器作为参数转换为 Purge 类型触发器，计算完成后数据将被清理

如果 Trigger 无法满足实际需求，也可以通过继承并实现 Trigger 自定义触发器，Flink Trigger 接口中共有如下方法可以腹泻，然后在 DataStream API 中调用 trigger 方法传入自定义 Trigger

- `onElement()`: 针对每一个接入窗口的数据元素进行触发操作
- `onEventTime()`: 根据接入窗口的 EventTime 进行触发操作
- `onProcessTime()`: 根据接入窗口的 Proces sTime 进行触发操作
- `OnMerger()`: 对多个窗口进行 Merge 操作，同时进行状态的合并
- `Clear()`: 执行窗口及状态数据的清除

在自定义触发器时，判断窗口触发方法返回的结果有如下类型，分别是：

- CONTINUE: 代表当前不触发计算，继续等待
- FIRE: 触发计算，但是数据继续保留
- PURGE: 窗口内部数据清除，但不触发计算
- FIRE_AND_PURGE: 触发计算，并清除对应的数据

用户在指定出发逻辑满足时可以通过将以上状态返回给 Flink，由 Flink 在窗口计算过程中，根据返回的状态选择是否触发对当前窗口的数据进行计算

<font color=red size=6>**Global Window 的默认触发器时 NeverTrigger，如果使用 Global Window 窗口，则必须自定义触发器，否则数据接入 Window 后将永远不会触发计算，窗口中的数据量会越来越大，最终导致系统内存溢出等问题**</font>

### 6.4 Evictors 数据剔除器

Evictors 主要是对进入 WindowsFunction 前后的数据进行剔除处理，Flink 内部实现了 CountEvictor、DeltaEvictor、TimeEvictor 三种 Evictors。在 Flink 中，Evictors 通过调用 DataStream API 中 `evictor()`方法使用，且默认的 Evictors 都是在 WindowsFunction 计算之前对数据进行剔除处理

- CountEvictor: 保持在窗口中具有固定数量的记录，将超过指定大小的数据在窗口计算前剔除
- DeltaEvictor: 通过定义 DeltaFunction 和指定 threshold，并计算 Windows 中的元素与最新元素之间的 Delta 大小，如果超过 threshold 则将当前数据元素剔除
- TimeEvictor: 通过指定时间间隔，将当前窗口中最新元素的时间减去 Interval，然后将小于该结果的数据全部删除，其本质是将具有最新时间的数据选择出来，删除过时的数据

### 6.5 开窗聚合算子

对流式数据进行开窗的目的，是为了对窗口中的数据进行统计计算，这些统计方法和基础的 DataStream 统计十分相似

窗口函数定义了要对窗口中收集的数据做的计算操作，根据处理的方式可以分为两类：增量聚合函数和全窗口函数

1. 增量聚合函数(Incremental aggregation functions)

    窗口将数据收集起来，最基本的数据操作就是聚合，每来一个新的数据，就在之前的结果上聚合一次，这就是“增量聚合”

    典型的增量聚合函数有两个：`ReduceFunction`和`AggregateFunction`

    - `ReduceFunction`规约函数

        `ReduceFunction`可以解决大多数规约聚合问题，但是缺陷在于其为规约函数，只是为了精简结果，其输入、输出以及中间运算结果类型均相同，无法满足复杂应用场景的应用

    - `AggregateFunction`聚合函数

        使用更加灵活，需要传入三个参数，为别是输入、累加器以及输出

2. 

#### 6.5.1 Window Apply

WindowedStream/AllWindowedStream -> DataStream

给窗口内的所有数据提供一个整体的处理函数，可以称为全窗口聚合函数

```java
windowedStream.apply(new WindowFunction<Tuple2<String,Integer>, Integer, Tuple, Window>() {
    public void apply (Tuple tuple,
            Window window,
            Iterable<Tuple2<String, Integer>> values,
            Collector<Integer> out) throws Exception {
        int sum = 0;
        for (value t: values) {
            sum += t.f1;
        }
        out.collect (new Integer(sum));
    }
});

// applying an AllWindowFunction on non-keyed window stream
allWindowedStream.apply (new AllWindowFunction<Tuple2<String,Integer>, Integer, Window>() {
    public void apply (Window window,
            Iterable<Tuple2<String, Integer>> values,
            Collector<Integer> out) throws Exception {
        int sum = 0;
        for (value t: values) {
            sum += t.f1;
        }
        out.collect (new Integer(sum));
    }
});
```

#### 6.5.2 Window Reduce

WindowedStream -> DataStream

与 Reduce 算子类似，将数据进行两两计算

#### 6.5.3 Window Apply 与 Window Aggregate 聚合方式的比较

```java
import com.Sinotruk.flink.basic.beans.Stock;
import org.apache.commons.collections.IteratorUtils;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.WindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingProcessingTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

public class WindowFunctionDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setParallelism(1);

//        String filePath = WindowFunctionDemo.class.getResource("/stock.txt").getFile();
//        DataStreamSource<String> dataStream = environment.readTextFile(filePath, "UTF-8");

        DataStreamSource<String> dataStream = environment.socketTextStream("localhost", 7777);

        final SingleOutputStreamOperator<Stock> stockStream = dataStream.map(new MapFunction<String, Stock>() {
            @Override
            public Stock map(String s) throws Exception {
                String[] splits = s.split(",");
                return new Stock(splits[0], Double.parseDouble(splits[1]), splits[2], Long.parseLong(splits[3]));
            }
        });

        stockStream.keyBy(new KeySelector<Stock, String>() {
            @Override
            public String getKey(Stock stock) throws Exception {
                return stock.getId();
            }
        })
                // 开窗分组
                .window(TumblingProcessingTimeWindows.of(Time.seconds(1)))
//                .aggregate(new MyAvgFunction())
                .apply(new WindowFunction<Stock, Object, String, TimeWindow>() {
                    // 四个参数分别表示：
                    // 当前数据的 key
                    // 当前窗口类型
                    // 当前窗口内所有数据的迭代器
                    // 输出结果收集器
                    @Override
                    public void apply(String s, TimeWindow timeWindow, Iterable<Stock> iterable, Collector<Object> collector) throws Exception {
                        int count = IteratorUtils.toList(iterable.iterator()).size();
                        collector.collect(count);
                    }
                }).print("stockStream");

        environment.execute("WindowAssignerDemo");
    }

    // AggregateFunction 三个泛型输入分别表示 传入数据类型、累加器类型、输出数据类型
    public static class MyAvgFunction implements AggregateFunction<Stock, Tuple2<Double, Integer>, Double> {
        // 创建一个累加器，初始值
        @Override
        public Tuple2<Double, Integer> createAccumulator() {
            return new Tuple2<>(0.0, 0);
        }

        // 在累加器上添加一个元素
        @Override
        public Tuple2<Double, Integer> add(Stock stock, Tuple2<Double, Integer> doubleIntegerTuple2) {
            return new Tuple2<>(doubleIntegerTuple2.f0 + stock.getPrice(), doubleIntegerTuple2.f1 + 1);
        }

        // 返回最终的结果
        @Override
        public Double getResult(Tuple2<Double, Integer> doubleIntegerTuple2) {
            return doubleIntegerTuple2.f0 / doubleIntegerTuple2.f1;
        }

        // 将两个累加器合并到一起，主要用于合并分区
        @Override
        public Tuple2<Double, Integer> merge(Tuple2<Double, Integer> doubleIntegerTuple2, Tuple2<Double, Integer> acc1) {
            return new Tuple2<>(doubleIntegerTuple2.f0 + acc1.f0, doubleIntegerTuple2.f1 + acc1.f1);
        }
    }
}
```

apply 聚合方式传入参数为 WindowFunction，它会持续收集窗口中的数据，待窗口中的数据全部收集完成之后再进行统一处理，类似于一个微“批量处理”

aggregate 聚合方式则是“来一条，处理一条”，其中间结果通过一个累加器来保存，待全部数据收集之后，直接从累加器中取计算结果，因此也叫做“流式聚合”

aggregate 聚合方式的效率比较高，apply 则能够拿到全部的窗口信息，使用相对更加灵活

### 6.6 第一个窗口的时间

窗口在定义时候，可以说窗口固化了窗口，是<font color=red size=5>所有类型的窗口，第一个窗口的开始时间都是**1970-01-01 08:00:00**</font>

## 7. CEP 编程模型

Flink CEP 即 Flink Complex Event Processing，是基于 DataStream 流式数据提供的一套复杂事件处理编程模型，可以理解为是基于无界流的一套正则匹配系统，即对于无界流中的各种数据（称为事件），提供一种组合匹配的功能

![CEP](D:\workspace\myNotes\assets\CEP.png)

使用 CEP 模型要引入 maven 依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-cep_2.11</artifactId>
    <version>${flink.version}</version>
    <type>pom</type>
</dependency>
```

CEP 的基本流程：

```java
DataStream<Event> datastream = ...; // 1.获取原始数据流
Pattern<Event, ?> pattern = ...; // 2.定义匹配器
PatternStream<Event> patternStream = CEP.pattern(input, pattern); // 3.获取匹配流
DataStream<Result> resultStream = patternStream.process(
	new PatternProcessFunction<Event, Result>() {
        @Override
        public void processMatch(Map<String, List<Event>> pattern,
                                 Context ctx, Collector<Result> out) throws Exception {
            ...; // 4.对匹配的数据进行处理
        }
    });
```

### 7.1 定义匹配器

定义匹配器的基本方式都是通过 Pattern 类，以流式编程的方式定义一个完整的匹配器

```java
DataStream<Event> input = ...;

Pattern<Event, ?> pattern = Pattern.<Event>begin("start").where(
        new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getId() == 42;
            }
        }
    ).next("middle").subtype(SubEvent.class).where(
        new SimpleCondition<SubEvent>() {
            @Override
            public boolean filter(SubEvent subEvent) {
                return subEvent.getVolume() >= 10.0;
            }
        }
    ).followedBy("end").where(
         new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getName().equals("end");
            }
         }
    );

PatternStream<Event> patternStream = CEP.pattern(input, pattern);

DataStream<Alert> result = patternStream.process(
    new PatternProcessFunction<Event, Alert>() {
        @Override
        public void processMatch(
                Map<String, List<Event>> pattern,
                Context ctx,
                Collector<Alert> out) throws Exception {
            out.collect(createAlertFrom(pattern));
        }
    });
```

Flink CEP 支持以下形式的连续策略：

1. `next()`，指定**严格连续**，期望所有匹配的事件严格的一个接一个出现，中间没有任何不匹配的事件
2. `followedBy()`，指定**松散连续**，忽略匹配的事件之间的不匹配的事件
3. `followedByAny()`，指定**不确定的松散连续**，更进一步的松散连续，允许忽略掉一些匹配事件的附加匹配
4. `notNext()`，指定**严格不连续**，期望后面不能直接连着一个特定事件
5. `notFollowedBy()`，指定**松散不连续**，如果不想一个特定事件发生在两个事件之间的任何地方

<font color=red>注意</font>：模式序列不能以`notFollowedBy()`结尾

也可以为时间定义一个有效时间约束，可以通过`pattern.within()`方法。例如`next.within(Time.seconds(10))`，就将该模式限定在10秒之内发生

> 一个模式只能有一个时间限制，如果限制了多个时间在不同的单个模式上，会使用最小的那个时间限制

# 五. Flink 时间语义

对于流式数据处理，时间是非常重要的。顺序是通过时间来表示的，尤其对于开窗计算，时间顺序不同会导致窗口无法正确的收集数。但是，数据在网络传输的过程中，会产生各种中断或者延迟，因此可能会导致后发的消息经过网络传输后反而先到达 Flink 进行计算，或者某些连续的数据由于网络不稳定导致终端数据顺序错乱。因此，一定要定义不同的时间语义来管理消息的顺序。

## 1. Flink 的自然时间语义

在 Flink 中定义了三种基本的时间语义：

1. Event Time: 事件真实发生的事件
2. Ingestion Time: 事件进入 Flink 的时间，也就是 Data Source 读入的时间
3. Process Time: 事件进入 Processor 真正开始计算的时间

![Flink Time](D:\workspace\myNotes\assets\Flink Time.png)

在三种语义中，通常情况下，计算关注最多的是 Event Time，但是 Flink 是无法直接知道 Event 的发生时间的，Ingestion Time 没有太多的业务价值，通常不会太过关心，Process Time 是 Flink 能够自行知道的时间，在 Event Time 不确定的情况下，Flink 只能根据 Process Time来进行计算

## 2. 设置 Event Time

在大部分业务场景中，业务更加关注事件的发生时间 Event Time。比如对一个系统的日志进行一些时间敏感的流式操作，我们更应该关注 log 日志中分析出来的事件事件 Event Time，而不太会关注 Flink 是什么时候开始计算的，也就是 Process Time

如果需要使用 Event Time，需要在 `StreamExecutionEnvironment` 中进行设置，具体可以自行指定

```java
final StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
environment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

1.09版本之前默认为 Process Time，因此需要如上手动设置 Event Time。之后版本该方法已过时，默认为 Event Time，一般无需再手动设置。如果需要使用 Process Time，大部分场景都提供了显式的 API 调用

事件时间是在 Flink 计算之前的，是 Flink 所不知道的，所以需要使用事件时间语义，通常事件时间都是作为事件中的一个字段传递进来。如下所示，将 Stock 中自己的  timestamps 字段作为 Event Time

```java
final WatermarkStrategy<Stock> stockWatermarkStrategy = WatermarkStrategy.<Stock>forBoundedOutOfOrderness(Duration.ZERO)
    .withTimestampAssigner(new SerializableTimestampAssigner<Stock>() {
        @Override
        public long extractTimestamp(Stock element, long recordTimestamp) {
            return element.getTimestamp;
        }
    });
stockstream.assignTimestampsAndWatermarks(stockWatermarkStrategy);
```

在`Watermark`的定义过程中，`forBoundedOutOfOrderness()`就是 Flink 针对乱序数据提供的一种实现方法，另外还有一个`forMonotonousTimestamps()`方法是 Flink 针对单调有序的数据提供的一种实现方法

`withTimestampAssigner()`方法是给数据指定 Event Time 的一种方法，这个方法是可选的。在没有指定 Event Time 的情况下，会自动使用 Processing Time 来计算。例如：如果使用 Flink 提供的 Kafka Connector，那 Flink 会去识别 Kafka 各个分区的消息投递时间，自动完成 Event Time 的设置

## 3. Flink 如何处理乱序数据

在进行 Window 开窗操作时，乱序的问题容易出现，比如：现有1-6这样的6个事件发送到 Flink

$[1] -> [2] -> [3] -> [4] -> [5] -> [6]$

括号中的每个数字表示事件发生的时间（单位假定为秒）Event Time。在正常不发生乱序的情况下，我们按照每5秒开启一个滚动窗口，Flink 是可以正确处理数据的，其会预先开启一个$ [0,5) $​​的 bucket，用来接收0秒到5秒的事件，依次将事件放到 bucket，当发现第五秒的消息$[5]$到达之后，就将 bucket 关闭，不再接收新的数据，准备后续的窗口聚合操作。

但是一旦数据发生乱序，在传输过程导致出现类似如下情况

$[1] -> [2] -> [5] -> [3] -> [4] -> [6]$

这样在开窗过程中，Flink 依然按照$[0, 5)$开启窗口，但是当$[5]$数据到来时， Bucket根据时间设定，会误以为已接收完全部数据，关闭 Bucket，进行下一步的窗口计算，但是当后续$[3][4]$​到来时，导致没有 Bucket 存放该数据

Flink 中有一系列完整的机制来处理数据乱序问题

1. WaterMark 水位线机制。窗口可以设置一个短暂的等待时间，等后面的数据到达之后，再关闭窗口
2. allowLateness 延迟窗口关闭时间。再窗口关闭后设置一个延迟时间，延迟时间内到达的数据会在后续窗口计算过程中重新进行一次窗口聚合
3. sideOutputStream 侧输出流。窗口完成聚合计算后，就不再接收数据，迟到的数据只能选择收集在另外一个侧输出流中，用户自己决定要如何处理此类数据，这是一种最后的兜底方案 

## 4. WaterMark 水位线

### 4.1 水位线机制

Watermark 是 Flink 处理乱序数据的第一道闸门，也是最为重要的一个机制

#### 4.1.1 Watermark 工作机制

Watermark 本质上就是一个时间戳，表示数据的事件时间 Event Time 推进到哪一个时间点，从数据形式上，Watermark 是只增不减的，代理着时间在按正常时间顺序往下推进，Watermark  必须与事件时间相关联，这样 Watermark 才有业务含义，Watermark 会随着数据流一起传输，可以将其看成一个特殊的数据

![Watermark](D:\workspace\myNotes\assets\Watermark.png)

1. Watermark 只增不减，如上图中，5、4、3三个数据发生了乱序，Watermark 只会记录最高位的5，直到后续6数据到达之后才会继续向上堆高

2. Flink 对数据流进行开窗后，会**<font color=red>根据事件时间 Event Time 来判断数据属于哪一个窗口，但是窗口何时关闭，是通过 Watermark 判断的</font>**

   对于一个 KeyedStream，进行5秒的滚动开窗 Tumbling Window后，Flink 会依次划分出多个 window（这些 window 的本质是一个个的 Bucket），每个 window 都是左开右闭的，就会划分出诸如$[0,5)[5,10)$这样一个个窗口，这些窗口会根据 Watermark 水位线来判断是否需要关闭。上图中，$[0,5)$这个窗口会等到5号 Watermark 出现时关闭，然后进行后续的窗口聚合计算

3. 如果事件时间的顺序是一致的，那么正常划分窗口时没问题的，一旦事件时间发生乱序，就不可避免的会造成数据丢失

   如上图中，当事件3与事件4到达时，$[0,5)$​这个窗口已经关闭，无法正确接收数据，如果不做处理，那么事件3与事件4在流式计算过程中会丢失

#### 4.1.2 Watermark 如何处理乱序问题

Watermark 处理乱序问题的方式比较简单，就是与真实的事件时间 Event Time 之间保存一个延迟

![Watermark delay](D:\workspace\myNotes\assets\Watermark delay.png)

如上所示，如果让 Watermark 与 Event Time 之间保持一个1秒的延迟，那么当5号事件过来时，Watermark还只是4，$[0,5)$这个窗口就不会关闭，会继续等待收集新的事件，事件3与事件4就能正常被窗口收集，当事件6到达后，Watermark 被推高到了5，这时$[0,5)$这个窗口才会关闭，停止收集数据，开始后续窗口聚合计算

但是 <font color=red>**Watermark 的延迟时间一般不宜设置过长，因为其会影响事件的响应速度**</font>。另外，由于无法精确的预测事件的乱序程度，所以 Watermark 机制并不能完全处理乱序问题，还需要又后续的兜底方案，即侧输出流

#### 4.1.3 如何分配 Watermark

```java
final WatermarkStrategy<Stock> stockWatermarkStrategy = WatermarkStrategy.<Stock>forBoundedOutOfOrderness(Duration.ZERO);
```

对于乱序的数据流，`forBoundedOutOfOrderness()`方法传入的时间参数表示延迟时间，如果事件本身就是严格有序递增的，就不会有乱序的问题，也就无需设置延迟时间，因此 WatermarkStrategy 针对有序数据流提供的 `forMonotonousTimestamps()`方法就不需要传入任何参数

```java
final WatermarkStrategy<Stock> stockWatermarkStrategy = WatermarkStrategy.forMonotonousTimestamps();
```

Watermark 的推高都是通过事件来推动的，如果一个数据流长期没有事件，就会造成 Watermark 长期得不到推高，很多 window 窗口，就会进行无用的数据等待，WatermarkStrategy 提供了一个处理空闲数据流的方式，来定时推高 Watermark

```java
final WatermarkStrategy<Stock> stockWatermarkStrategy = WatermarkStrategy.withIdleness(Duration.ofSeconds(10));
```

### 4.2 定制 Watermark 生成策略

Flink 内置的针对有序数据流和无序数据流的两个 Watermark 机制，已经能够应对大部分的自定义计算过程。但是在对接一些特定数据源时，可以将 Watermark 的分配机制整合到 Source 数据源中。例如，如果使用 Flink 提供的 Kafka Connector，就不需要定制 WatermarkStrategy，因为 Flink 提供的消费者端已经实现了一套 WatermarkStrategy

在 WatermarkStrategy类内部，有一个 WatermarkGenerator 接口的属性，负责生成 Watermark，如果需要定制 Watermark 实现类，可以通过实现 WatermarkGenerator 接口的方式来定制

### 4.3 Watermark 传播机制

如果将环境运行并行度设置为1，那么只要有一个超过 Watermark 的数据进来，就会关闭一个计算窗口。但是当并行度设置为大于1的值（比如4）时会出现，提交一个超过 Watermark 的数据，并不会触发上一个计算窗口的关闭动作，而需要等到积累4个或者4个以上的超过 Watermark 的数据时，才会触发上一个计算窗口的关闭动作。这其中就涉及到了 Watermark 在 Slot 之间的传递机制

在定制 Watermark 生成策略时，通过 WatermarkOutput 的 `emitWatermark()` 向下游发射 Watermark。而 Flink 中，Watermark 会在各个计算流程之间传递，并在处理过程中进行整合。例如进行某一个计算任务，他的上游任务有N个并行度，那就会有N个 Slot 进行并行计算，由于每个 Slot 的处理时间及网络传输时间不一样，也就会产生N个不同的 Watermark。当前任务就需要将所有的上游 Watermark 都保留下来，然后选取最靠后的 Watermark 作为上游计算的整体 Watermark

![Watermark传播机制](D:\workspace\myNotes\assets\Watermark传播机制.png)

这种传播机制，对于 SocketStream 数据源，有序需要阻塞线程，所以只能以一个线程（也就是并行度1）来读取数据，Flink 只能通过读取三个或以上的数据，将这些数据尽量平均的分配给各个线程（并行度），这样才能够保证向下游 slot 传递 Watermark

对接 Kafka 这样的数据时，问题就不会太过明显，因为 Kafka 本身就实现了多线程的数据读取

## 5. allowLateness 允许等待时间

对于 WindowedStream 和 AllWindowedStream，可以通过 allowLateness 设置一个等待时间，作为 Watermark 后的补充

默认情况下，等待时间时被设置为0，当事件的 Event Time 晚于 Watermark 后，这个事件就会被抛弃，也就是说窗口不再接收这些数据

Flink 对于这些迟到的数据，允许进行一些补偿处理，当手动设置了等待时间，例如设置为等待5秒，Flink 依然会在 Watermark 时间到了之后关闭窗口，进行后续的窗口集合计算，但是，在5秒内有新的事件进来时，Flink 会重新进行一次聚合计算，将这些信赖的事件包含进来

![allowLateness等待机制](D:\workspace\myNotes\assets\allowLateness等待机制.png)

等待时间内的数据处理是比较消耗性能的，因此等待时间一般不宜设置过长。另外，在 TumblingWindow 下，每个数据肯定都是有所属窗口的

## 6. sideOutputStream 侧输出流

对于乱序数据，Flink 已经做了两次宽大处理，一次是 Watermark，对于短期迟到的数据，Watermark 机制可以让窗口等待迟到数据到达后再关闭窗口；另一次是延迟时间 allowLateness，对于超过 Watermark 等待时间的迟到数据，延迟时间机制可以在迟到数据到达窗口后再重新进行一次窗口聚合计算

但是，上述机制依然不能保证所有数据都能够完全被窗口收录，对于那些超过最长等待时间的事件，Flink 提供的思路是不再提供统一的处理，而是将此类事件单独放到另一个侧输出流中，由用户决定如何处理这些数据，是将数据丢弃还是进行一些计算补偿行为，都将由用户程序来决定

侧输出流的作用不只是在处理乱序数据，它是完全交由用户自行完成的一个补偿机制，从一个主要的 DataStream 数据流中，可以产生任意数量的侧输出结果流，并且这些结果流的数据类型也不需要完全与主要的数据里的数据类型一致，不同的侧输出流，他们的类型也不必要完全相同。<font color=red>侧数据流完全由用户自行把控</font>

# 六. Flink Table API与Flink SQL

## 1. Table API和SQL是什么

Flink 为流式/批量处理应用程序提供了不同级别的抽象

![Flink客户端API体系](D:\workspace\myNotes\assets\Flink客户端API体系.png)

四层API依次向上支撑

- Flink API 最底层的抽象就是有状态实时流处理 Stateful Stream Processing，是最底层的 Low-Level API。实际上就是基于 ProcessFunction 提供的一整套 API。这是最灵活、功能最全面的一层客户端 API，允许应用程序定制复杂的计算过程，但是这一层大部分常用功能都已经封装在上层 Core API 中，大部分的应用都不会需要使用到这一层  API

- Core APIs 主要是 DataStream API 以及针对批处理的 DataSet API。是最为常用的一套 API，其中又以 DataStream API为主，本质是基于一系列 Process Function 的封装，可以极大的简化客户端应用程序的开发

- Table API 主要是以表（Table）为中心的声明式编程API，它允许应用程序像操作关系型数据库一样对数据进行一些 select/join/groupby 等典型的逻辑操作，并且也可以通过用户自定义函数进行功能扩展，而不用确切地指定程序指定的代码。Table API的表达能力不如 Core API 灵活，大部分情况下，应将 Table API 和 DataStream API 混合使用

- SQL 是 Flink API 中最顶层的抽象，功能类似于 Table API，只是程序实现的是直接的 SQL 语句支持，本质上还是基于 Table API 的一层抽象

  Table API 和 Flink SQL 是一套给 Java 和 Scala 语言提供的快速查询数据的 API，是集成在一起的一阵套 API，通过 Table API，用户可以像操作数据库中的表一样查询流式数据（这里注意，Table API 主要是针对数据查询操作），而“表”中数据的本质还是对流式数据的抽象，SQL 则是直接在“表”上提供 SQL 语句支持

## 2. 如何使用Table API

引入 maven 依赖

```xml
<!-- 语言包 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-api-java-bridge_2.11</artifactId>
    <version>${flink.version}</version>
    <scope>provided</scope>
</dependency>
<!-- Planner -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_2.11</artifactId>
    <version>${flink.version}</version>
    <scope>provided</scope>
</dependency>
<!-- 自定义函数 -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-common</artifactId>
    <version>${flink.version}</version>
    <scope>provided</scope>
</dependency>
```

## 3. 基础编程框架

Flink 中对于批处理和流处理的Table API 和 SQL 程序都遵循一个相同的模式，如下所示结构

```java
import org.apache.flink.table.api.*;
import org.apache.flink.connector.datagen.table.DataGenOptions;

// Create a TableEnvironment for batch or streaming execution.
// See the "Create a TableEnvironment" section for details.
TableEnvironment tableEnv = TableEnvironment.create(/*…*/);

// Create a source table
tableEnv.createTemporaryTable("SourceTable", TableDescriptor.forConnector("datagen")
    .schema(Schema.newBuilder()
      .column("f0", DataTypes.STRING())
      .build())
    .option(DataGenOptions.ROWS_PER_SECOND, 100)
    .build())

// Create a sink table (using SQL DDL)
tableEnv.executeSql("CREATE TEMPORARY TABLE SinkTable WITH ('connector' = 'blackhole') LIKE SourceTable");

// Create a Table object from a Table API query
Table table2 = tableEnv.from("SourceTable");

// Create a Table object from a SQL query
Table table3 = tableEnv.sqlQuery("SELECT * FROM SourceTable");

// Emit a Table API result Table to a TableSink, same for SQL result
TableResult tableResult = table2.executeInsert("SinkTable");
```

基本步骤可以概括为：

1. 创建 Table Environment
2. 将流数据转换成动态表 Table
3. 在动态表上计算一个连续查询，生成一个新的动态表
4. 生成的动态表再次转换回流数据

## 4. 扩展编程框架

### 4.1 临时表与永久表

注册动态表时，额可以选择注册为临时表或者是永久表

临时表只能在当前任务中访问，任务相关的所有 Flink 的会话 Session 和集群 Cluster 都能够访问表中的数
据，但是任务结束后，这个表就会删除

永久表时在 Flink 集群的整个运行过程中都存在的表，所有任务都可以像访问数据库一样访问这些表，直到这个表被显式的删除

表注册完成之后，可以将 Table 对象中的数据直接插入到表中

```java
// 创建临时表 从orders数据流创建一个名为 Order 的临时表
tableEnvironment.createTemporatyView("Order", orders);
// 创建永久表
Table orders = tableEnvironment.from("Orders");
orders.executeInsert("OutOrders");
// 老版本的 insertInto 方法已经过期，不建议使用
```

Flink 的永久表需要一个 catalog 来维护表的元数据，一旦永久表被创建，任何连接到这个 catalog 的 Flink 会话都可见并且持续存在，知道这个表被明确删除，也就是说，永久表是在 Flink 的绘画之间共享的

临时表通常保存于内存中，并且只在创建它的 Flink 会话中存在，这些表对于其他会话是不可见的，他们也不需要与 catalog 绑定，临时表是不共享的

在 Table 对象中也能对表做一些结构化管理的工作，例如对表中列进行增删改查等操作，但是通常不建议这么做，因为 Flink 针对的是流式数据计算，它的表保存的应该只是计算过程中的临时数据，频繁的表结构变动只是增加计算过程的复杂性

当一个会话中由两个重名的临时表和永久表时，将会只有临时表生效，如果临时表没有删除，那么永久表就iu无法访问

### 4.2 AppendStream 和 RetractStream

两个方法都是将 Table 转换成 DataStream，但是 groupby 语句不支持 toAppendStream

### 4.3 内置函数与自定义函数

Flink 提供了丰富的 SQL 内置函数，这些函数可以在 Table API 中调用，也可以在 SQL 中直接调用 ，调用方式与在关系型数据库中调用方式类似，详情可参见官方文档：https://nightlies.apache.org/flink/flink-docs-release-1.14/zh/docs/dev/table/functions/systemfunctions/

自定义函数扩展了查询的表达能力，使用自定义函数需要注意一下两点：

1. 大多数情况下，用户自定义的函数需要先注册，然后才能在查询中使用。注册的方法有两种

   ```java
   // 注册一个临时函数
   tableEnvironment.createTemporaryFunction(String path, Class<? extends UserDefinedFunction> functionClass);
   // 注册一个临时的系统函数
   tableEnvironment.createTemporarySystemFunction(String name, Class<? extends UserDefinedFunction> functionClass);
   ```

   两者的区别在于，用户函数只在当前 Catalog 和 Database 中生效，系统函数能由独立于 Catalog 和 Database 的全局名称进行标识，使用系统函数可以继承 Flink 的一些内置函数，比如 trim，max等

2. 自定义函数需要按照函数类型继承一个 Flink 中指定的函数基类。Flink 中有以下几种函数基类

   - 标量函数`org.apacha.flink.table.functions.ScalarFunction`

     标量函数可以将0个或者多个标量值，映射成一个新的标量值，例如获取昂前事件、字符串转大写、加减法、多个字符串拼接，都属于标量函数。下面定义一个hash方法

     ```java
     public static class HashCode extends ScalarFunction {
     	private int factor = 13;
         public HashCode(int factor) {
             this.factor = factor;
         }
         
         public int eval(String s) {
             return s.hashCode() * factor;
         }
     }
     ```

     

   - 表函数`org.apache.flink.table.functions.TableFunction`

     表函数以0个或者多个标量为输入，可以返回任意数量的行作为输出，而不是单个值。如下拆分函数

     ```java
     public class Split extends TableFunction<String> {
         private String separator = ",";
         public Split(String separator) {
             this.separator = separator;
         }
         
         public void eval(String str) {
             for (String s: str.split(" ")) {
                 collect(s);
             }
         }
     }
     ```

   - 聚合函数`org.apache.flink.table.functions.AggregateFunction`

     聚合函数可以把一个表中一列的数据，聚合成一个标量值。定义聚合函数时，首先需要顶一个累加器 Accumulator，用来保存聚合中间结果的数据结构，可以通过 `createAccumulator()`方法构建空累加器，然后通过`accumulate()`方法来对每一个输入行进行累加值更新，最后调用`getValue()`方法来计算并返回最终结果。如下时计算字符串出现次数的 count 方法

     ```java
     public static class CountFunction extends AggregateFunction<String, CountFunction.MyAccumulator> {
         public static class MyAccumulator {
             public long count = 0L;
         }
         
         public MyAccumulator createAccumulator() {
             return new MyAccumulator();
         }
         
         public void accumulate(MyAccumulator accumulator, Integer i) {
             if (i != null)
                 accumulator.count += i;
         }
         
         public String getValue(MyAccumulator accumulator) {
             return "Result: " + accumulator.count;
         }
     }
     ```

### 4.4 基于 Connector 进行数据流转

Flink 中的流数据，大部分都是映射的一个外部的数据源，所以，通常创建表时，也需要通过 connector 映射外部的数据源。基于 Connector 注册表的通用方式如下：

```java
tableEnvironment
    .connect(...) // 定义表的数据来源，和外部系统建立连接
    .withFormat(...) // 定义数据格式化方法
    .withSchema(...) // 定义表结构
    .createTemporaryTable("MyTable"); // 创建临时表
```

例如针对文本数据

```java
tableEnvironment
    .connect(new FileSystem().path("YOUR_Path.sensor.txt")) // 定义到文件系统的连接
    .withFormat(new Csv()) // 定义以csv格式进行数据格式化
    .withSchema(new Schema()
               .field("id", DataTypes.STRING())
               .field("timestamp", DataTypes.BIGINT())
               .field("temperature", DataTypes.DOUBLE())) // 定义表结构
    .createTemporaryTable("sensorTable");
```

针对 Kafka 数据

```java
tableEnvironment
    .connect(new Kafka()
            .version("0.11")
            .topic("sinkTest")
            .property("zookeeper.connect", "locahost: 2181")
            .property("bootstrap.servers", "localhost: 9092"))
    .withFormat(new Csv())
    .withSchema(new Schema()
               .field("id", DataTypes.STRING())
               .field("temp", Datatypes.DOUBLE()))
    .createTemporaryTable("kafkaOutPUtTable")
```

### 4.5 Flink Table API & SQL 时间语义

Flink Table API & SQL 的时间语义通常并不会对一个表进行开窗处理，就是将时间语义作为 Table 中的一个字段引入进来，由应用程序决定要怎么使用时间
将 DataStream 转化成 Table 时，可以用 .rowtime 后缀在定义 Schema 时定义，这种方式需要在 DataStream 上定义好时间戳和 Watermark，使用 .rowtime 修饰的，可以是一个已有的字段，也可以是一个不存在的字段，如果字段不存在，则会在 Schema 的结尾追加一个新的字段，之后就可以像使用一个普通 Timestamp 类型的字段一样使用这个字段，不管在那种情况下，事件时间字段的值都是 DataStream 中定义的事件时间

```java
// Option 1:
// 基于 stream 中的事件产生时间戳和 Watermark
DataStream<Tuple2<String, String>> stream = inputStream.assignTimestampAndWatermarks(...);

// 声明一个额外的逻辑字段作为事件时间属性
Table table = tEnvironment.fromDataStream(stream, $("user_name"), $("data"), $("user_action_time").rowtime());

// option 2:
// 从第一个字段获取事件时间，并且产生 Watermark
DataStream<Tuple3<Long, String, String>> stream = inputStream.assignTimestampsAndWatermarks(...);

// 第一个字段已经用作事件时间抽取，无需再用新字段来表示事件时间
Table table = tEnvironment.fromDataStream(stream, $("user_action_time").rowtime(), $("user_name"), $("data"));
```

### 4.6 查看 SQL 执行计划

查看 SQL 执行计划的 API

```java
final String explaination = tableEnvironment.explainSql(sql);
System.out.println(explaination);
```

在 `explainSql()` 方法中，可以传入一组可选的 `ExplainDetail` 参数，以展示更多的执行计划的细节，是一个枚举值

```java
/** ExplainDetail defines the types of details for explain result. */
@org.apache.flink.annotation.PublicEvolving
public enum ExplainDetail {
    ESTIMATED_COST, CHANGELOG_MODE, JSON_EXECUTION_PLAN;

    private ExplainDetail() { /* compiled code */ }
}
```

Flink 的 Table API & SQL 提供了一组高级的抽象API，主要是简化对流式数据的检索过程。但是在生产环境中，目前不建议机型深度使用。

# 七. Flink 状态机制

由一个任务来维护，并且要参与到数据计算过程中的数据称为**<font color=red>状态</font>**，这一类的计算任务称为**<font color=red>有状态的任务</font>**，比如 reduce/sum/min/minby/max/maxby等操作，都是有状态的算子；只依赖于输入数据的计算任务，则称为**<font color=red>无状态的任务</font>**，多个任务叠加在一起，就组成了一个客户端应用

状态，可以理解为是一个本地变量，可以被一个客户端应用中的所有计算任务访问。在分布式流式计算场景下，任务是并行计算的，所以状态需要分开保存，集群计算结束后又需要合并读取，在算子并行度发生变化时要维护状态的一致性，同时要考虑状态数据要尽量高效的存储与访问等等。Flink 的状态机制提供了对这类状态数据的统一管理，开发人员可以专注于开发业务逻辑，而不用时刻考虑状态的复杂管理机制

对于状态，由两种管理机制，一种是 managed state，就是 Flink 管理的状态机制，对状态管理问题提供了统一的管理机制，另一种是 raw state，就是用户自己管理的状态机制。只需要 Flink 提供一个本地变量空间，由应用程序自己去管理这一部分的状态。Flink 的状态管理机制非常强大，所以在大部分开发场景中，使用 Flink 提供的状态管理机制已经足够

Flink 管理的状态都是跟特定计算任务关联在一起，主要有两种状态：一种是 Operate State 算子状态，一种是 keyed state 键控状态

## 1. Operate State 算子状态

算子状态的作用范围限定为当前任务计算任务内，这种状态是跟一个特定的计算任务绑定的，算子状态的作用范围只限定在算子任务内，由同一并行任务所处理的所有数据都可以访问到相同的状态，并且这个算子状态不能由其他子任务访问

这一类算子需要按任务分开保存，当任务并行度发生变化时，需要支持在并行运算实例之间重新分配状态

如下是一个简单的求和算子保存了一个状态

```java
public class SumOperateState {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        environment.setParallelism(3);

        final DataStreamSource<Integer> stream = environment.fromElements(1, 2, 3, 4, 5, 6);
        final SingleOutputStreamOperator<Integer> stream2 = stream.map(new MySumMapper("mysummapper"));
        stream2.print();

        environment.execute("SumOperateState");
    }

    public static class MySumMapper implements MapFunction<Integer, Integer>, CheckpointedFunction {
        private int sum;
        private String stateKey;
        private ListState<Integer> checkpointedState;

        public MySumMapper(String stateKey) {
            this.stateKey = stateKey;
        }

        @Override
        public Integer map(Integer integer) throws Exception {
            return sum += integer;
        }

        @Override
        public void snapshotState(FunctionSnapshotContext functionSnapshotContext) throws Exception {
            checkpointedState.clear();
            checkpointedState.add(sum);
        }

        @Override
        public void initializeState(FunctionInitializationContext functionInitializationContext) throws Exception {
            ListStateDescriptor<Integer> descriptor = new ListStateDescriptor<Integer>(stateKey, TypeInformation.of(new TypeHint<Integer>() {}));
            checkpointedState = functionInitializationContext.getOperatorStateStore().getListState(descriptor);
            if (functionInitializationContext.isRestored()) {
                for (Integer subSum: checkpointedState.get()) {
                    sum += subSum;
                }
            }
        }
    }
}
```

Flink 中的算子状态操作需要给算子继承一个 `CheckpointedFunction` 接口，这个接口又两个方法，`snapshotState()`方法会在算子执行过程中调用，进行状态保存；`initializaState()`方法是在任务启动时初始化的状态

这样，算子在执行过程中就可以将中间结果保存到 checkpointedState 状态中，当算子异常终止时，下一次启动又可以从这个 checkpointedState 状态中加载之前的计算结果

**关于不同的状态类型**

在获取状态时，`context.getOperatorStateStore()`这个方法有几个重载方法：`getListState()`、`getUnionListState()`、`getBroadcastState()`

`getListState()`与`getUnionListState()`两个方法都是处理 ListState，也即是不同的任务点，状态也不相同，只是两个状态的底层状态分配机制不同。ListState 是将不同的子状态分配好之后，分给不同的算子实例去处理，而 UnionListState 则是将所有的子状态都分配给所有的算子实例，由算子实例自行调节每个实例获取哪些状态。FlinkKafkaConsumer 使用的便是 UnionListState。`getBoradcastState()`是处理广播状态，也就是所有节点的状态都是相同的

其他算子，包括 function，source，sink都可以自行添加状态管理，这其中需要理解的就是 checkpointedState 的形式

![checkpointedState](D:\workspace\myNotes\assets\checkpointedState.png)

因为 Flink 的计算任务都是并行执行的，那么在计算过程中，每一个并行的实例都会有一个自己的状态，所以在 snapshotState 保存状态时，是将每个并行实例内的状态进行保存，那整个任务整体就会保存成一个集合。所以实例中保存的其实是每个子任务内计算得到的 sum 和

当任务重新启动时，Flink 可能还需要对子任务的状态进行重新分配，因为任务的并行度有可能进行了调整。所以实例中 initializeState 方法加载状态时，也是将各个子状态的 sum 加到一起，才是一个完整的求和计算

## 2. keyed State 键控状态

算子状态针对的是普通算子，在任何 DataStream 和 DataSet 中都可以使用。keyed state 键控状态是针对 keyby 产生的 KeyedStream。KeyedStream 的计算任务都跟当前分配的 key 直接关联，key 是在计算任务运行时分配的。这一类状态，无法在任务启动过程中完成状态的分配，需要在任务执行过程中，根据 key 的不同而进行不同的分配，Flink 针对 keyed Stream，会在内部根据每一个 key 维护一个键控状态，在具体运算过程中，根据 key 的分配情况，将状态分配给不同的计算任务

针对键控状态，Flink 提供了一些列 Rich 开头的富计算因子抽象类，这些抽象类提供了更丰富的计算任务生命周期管理，用户程序通过继承这些抽象类，就可以获取到与当前分配的 key 相关的状态

下面实现一个自定义的 word count 的 KeyedStream 算子

```java
public class WCKeyedState {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment environment = StreamExecutionEnvironment.getExecutionEnvironment();
        final DataStreamSource<Tuple2<String, Integer>> stream = environment.fromCollection(Arrays.asList(Tuple2.of("a", 1), Tuple2.of("a", 5),
                Tuple2.of("b", 6), Tuple2.of("b", 2), Tuple2.of("b", 3), Tuple2.of("b", 8),
                Tuple2.of("c", 8), Tuple2.of("c", 4), Tuple2.of("c", 6), Tuple2.of("c", 4)));

        final KeyedStream<Tuple2<String, Integer>, String> keyedStream = stream.keyBy((value) -> value.f0);
        keyedStream.flatMap(new WCFlatMapFunction("WCKeyedState")).print();

        environment.execute("WCKeyedState");
    }

    public static class WCFlatMapFunction extends RichFlatMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>> {
        private String stateDesc;
        ValueState<Tuple2<String, Integer>> valueState;

        public WCFlatMapFunction(String stateDesc) {
            this.stateDesc = stateDesc;
        }

        @Override
        public void flatMap(Tuple2<String, Integer> stringIntegerTuple2, Collector<Tuple2<String, Integer>> collector) throws Exception {
            Tuple2<String, Integer> wordCountList = valueState.value();
            if (null == wordCountList) {
                wordCountList = stringIntegerTuple2;
            } else {
                wordCountList.f1 += stringIntegerTuple2.f1;
            }
            valueState.update(wordCountList);
            collector.collect(wordCountList);
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            ValueStateDescriptor<Tuple2<String, Integer>> descriptor = new ValueStateDescriptor<Tuple2<String, Integer>>(stateDesc, TypeInformation.of(new TypeHint<Tuple2<String, Integer>>() {}));
            // 设置状态存活时间
            StateTtlConfig ttlConfig = StateTtlConfig
                    .newBuilder(Time.seconds(1))
                    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
                    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
                    .build();
            descriptor.enableTimeToLive(ttlConfig);
            valueState = this.getRuntimeContext().getState(descriptor);
        }
    }
}
```

## 3. Checkpointing 检查点

Flink 中的每个算子都可以是有状态的，这些状态话的方法和算子可以使 Flink 的计算过程更为精确，在开发过程中，应尽量使用带状态的算子。对于这些状态，除了可以通过算子状态和键控状态进行扩展外，Flink 也停工了一种自动的兜底机制-CheckPointing 检查点

CheckPointing 检查点是一种由 Flink 自动执行的一种状态备份机制，其目的是能够从故障中回复，快照中包含了每个数据源 Source 的指针（例如，到文件或 Kafka 分区的偏移量）以及每个有状态算子的状态副本

默认情况下，检查点机制是禁用的，需要在应用中通过`StreamExecutionEnvironment`进行配置，基础的配置方式是通过`StreamExecutionEnvironment.enableCheckpointing()`方法开启，开启时需要传入一个参数，表示多长时间执行依次快照。另外有其他选项，可参见如下示例

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// 每1000毫秒开始一次 checkpoint
env.enableCheckpointing(1000);
// 高级选项
// 设置模式为精确一次（这是默认值）
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
// 确认 checkpoints 之间的时间会进行500毫秒
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);
// Checkpont 必须要在1分钟内完成，否则就会被抛弃
env.getCheckpointConfig().setCheckpointTimeout(6000);
// 同一时间只允许一个 checkpoint 进行
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);
// 开启在 job 中止后仍然保留的 externalized checkpoints
env.getCheckpointConfig().enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION)
```

## 4. Flink的容错重启机制

当某一个 task 发生故障时，Flink 需要重启出错的 Task 以及其他受到影响的 Task，以使得作业恢复到正常执行的状态，Flink 通过重启策略和故障恢复策略来控制 Task 重启，重启策略决定是否可以重启以及重启的间隔，故障恢复策略决定哪些 Task 需要重启

重启策略可以通过配置文件 flink-conf.yaml 中通过 restart-strategy 属性进行配置，同样也可以在应用程序中覆盖配置文件中的配置，如果没有启动 checkpoint，那就采用“不重启”策略；如果启用了 checkpoint 并且没有配置重启策略，那么就采用固定延时重启策略，这种情况下最大尝试重启次数是`Integer.MAX_VALUE`，基本就可以认为是会不停的尝试重启

restart-strategy 属性可选的配置有一下几种

- none 或者 off 或者 disable：不重启。checkpointing 关闭后的默认值
- fixeddelay，fixed-delay：固定延迟重启策略。checkpointing 启用时的默认值
- failurerate, failure-rate：失败率重启策略

这些配置项可以在应用程序中定制

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironmen();
env.setRestartStrategy(RestarStrategies.fixedDelayRestart(
	3, // 尝试重启次数
	Time.of(10, TimuUnit.SECONDS) // 延时
))
```

**fixeddelay** 策略，还可以定制两个参数，`restart-strategy.fixed-delay.attempts` 重试次数以及 `restart-strategy.fixed-delay.delay` 延迟时间。第一个参数表示重启任务的尝试次数，第二个参数表示重启失败后，再次尝试重启的时间间隔，可以配置为“1 min”，“20 s”诸如此类。

```yaml
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s
```

或者在应用程序中

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironmen();
env.setRestartStrategy(RestarStrategies.fixedDelayRestart(
	3, // 尝试重启此时
	Time.of(10, TimuUnit.SECONDS) // 延时
))
```

**Failure Rate** 策略表示当故障率（每个时间假根发生故障的次数）超过设定的限制时，作业就会最终失败。在连续的两次重启尝试之间，重启策略会等待一段固定长度的时间

在这种策略下，可以定义三个详细的参数

- `restart-strategy.failure-rate.max-failures-per-interval`：任务失败前，在固定时间间隔内的最大重启尝试次数
- `restart-strategy.failure-rate.failure-rate-interval`：检测失败率的窗口间隔
- `restart-strategy.failure-rate.delay`：两次重启尝试之间的间隔时间

```yaml
restart-strategy: failure-rate
restart-strategy.failure-rate.max-failure-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 10 s
```

或者在应用程序中配置

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.failureRateRestart(
	3, // 每个时间间隔的最大故障次数
	Time.of(5, TimeUnit.MINUTES), // 测量故障率的时间间隔
	Time.of(10, TimeUnit.SECONDS) // 延时
));
```

## 5. State Backend 状态存储方式与位置

通过算子状态、键控状态以及检查点，可以对计算过程中的中间状态进行保存，保存下来的状态即可以在计算中使用，也可以在计算程序异常终止后恢复计算状态时使用。

Flink 提供了多种 State Backend 状态后端，用来管理状态数据具体的存储方式与位置。Flink 默认提供了三种状态后端：jobmanager/filesystem/rocksdb。设置方式可以在file-conf.yaml中，通过 state.backend 属性进行配置，也可以在程序中通过`StreamExecutionEnvironment`配置

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(...);
```

- `jobmanager`

  `jobmanager`在后台由一个`MemoryStateBackend`类实现，是基于内存实现的，状态信息会保存到`TaskManager`的JVM堆内存中，而检查点信息则会直接保存到`JobManager`的内存中，这些检查点信息虽然都是基于内存工作，但是也依然会持久化到文件系统中

  由于检查点保存在`JobManager`中，会加大`TaskManager`和`JobManager`之间的网络请求，并且也会加大`JobManager`的负担，所以**<font color=red>这种方式通常只用于实验场景或者小状态的本地计算场景</font>**

- `filesystem`

  `filesystem`在后台由一个`FsStateBackend`类实现，是基于内存和文件系统进行状态保存。但是检查点信息是由`TaskManager`进行保存，保存的文件地址是可以自行配置的。由于`TaskManager`上执行的任务是动态分配的，所以通常这个保存地址需要配置成所有`TaskManager`都能访问到的地方（比如`HDFS`）。`TaskManager`上由于会有多个并行任务，他们的文件存储地址也会用数字进行版本区分

  `filesystem`的状态访问很快速，适合需要大的堆内存的场景，但是`filesystem`是受限于内存和GC的，所以它支持的状态数据大小优先

- `rocksdb`

  `rocksdb`在后台由一个`RocksDBStateBackend`类来实现，`RocksDB`是一个访问快速的 key-value 本地缓存，可以理解为是一个本地的 Redis。能够基于文件系统提供高效的访问，是一个常用的流式计算持久化工具。使用`RocketDB`后，状态数据不再受限于内存，转而受限于硬盘

  `RocketDBStateBackend`适合支持非常大的状态信息存储，但是`RocksDB`毕竟是基于文件系统，执行速度会比`filesystem`慢，大概是`filesystem`的10倍，但是在大多数场景下已经足够使用

  > 如果在应用中使用`rocksdb`，需要引入 maven 依赖
  >
  > ```xml
  > <dependency>
  >     <groupId>org.apache.flink</groupId>
  >     <artifactId>flink-statebackend-rocksdb_2.12</artifactId>
  >     <version>${flink.version}</version>
  > </dependency>
  > ```
  >
  > 然后使用`StreamExecutionEnvironment`设置
  >
  > ```java
  > env.setStateBackend(new RocksDBStateBackend("key"));
  > ```

## 6. 总结

在流式计算场景下，应用程序通常是无法预知数据会何时到来，只能一直运行随时等待数据接入，一旦应用程序突然出错终止，就很容易导致数据丢失。所以在流式计算场景下，需要对程序的健壮性做更多考量。Flink 提供了一系列的状态机制来加强程序的健壮性，但是在重要的生产环境中，对程序健壮性做再多考量都是不过分的，因此通常还要加上一些基于运维的监控机制（例如监控 Flink 的进程，监控 yarn 中的任务状态等），来保证流式计算程序的安全

# 八、其他

## 1. 广播变量

在Flink 集群生产环境中，全局静态变量 public static 不能被 taskManager 访问，可以采用其他方式，即广播变量

广播变量允许所有的并行算子以及常规的输入来使用该数据集。这对于辅助数据集或依赖数据的参数化非常有用。然后，操作员可以将数据集作为集合进行访问

- 广播变量示例一（广播变量作为配置）：

```java
/**
 * Desc
 * 需求:
 * 使用Flink的BroadcastState来完成
 * 事件流和配置流(需要广播为State)的关联,并实现配置的动态更新!
 */
public class BroadcastStateConfigUpdate {
    public static void main(String[] args) throws Exception{
        //1.env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //2.source
        //-1.构建实时的自定义随机数据事件流-数据源源不断产生,量会很大
        //<userID, eventTime, eventType, productID>
        DataStreamSource<Tuple4<String, String, String, Integer>> eventDS = env.addSource(new MySource());
 
        //-2.构建配置流-从MySQL定期查询最新的,数据量较小
        //<用户id,<姓名,年龄>>
        DataStreamSource<Map<String, Tuple2<String, Integer>>> configDS = env.addSource(new MySQLSource());
 
        //3.transformation
        //-1.定义状态描述器-准备将配置流作为状态广播
        MapStateDescriptor<Void, Map<String, Tuple2<String, Integer>>> descriptor =
                new MapStateDescriptor<>("config", Types.VOID, Types.MAP(Types.STRING, Types.TUPLE(Types.STRING, Types.INT)));
        //-2.将配置流根据状态描述器广播出去,变成广播状态流
        BroadcastStream<Map<String, Tuple2<String, Integer>>> broadcastDS = configDS.broadcast(descriptor);
 
        //-3.将事件流和广播流进行连接
        BroadcastConnectedStream<Tuple4<String, String, String, Integer>, Map<String, Tuple2<String, Integer>>> connectDS =eventDS.connect(broadcastDS);
        //-4.处理连接后的流-根据配置流补全事件流中的用户的信息
        SingleOutputStreamOperator<Tuple6<String, String, String, Integer, String, Integer>> result = connectDS
                //BroadcastProcessFunction<IN1, IN2, OUT>
                .process(new BroadcastProcessFunction<
                //<userID, eventTime, eventType, productID> //事件流
                Tuple4<String, String, String, Integer>,
                //<用户id,<姓名,年龄>> //广播流
                Map<String, Tuple2<String, Integer>>,
                //<用户id，eventTime，eventType，productID，姓名，年龄> //需要收集的数据
                Tuple6<String, String, String, Integer, String, Integer>>() {
 
            //处理事件流中的元素
            @Override
            public void processElement(Tuple4<String, String, String, Integer> value, ReadOnlyContext ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                //取出事件流中的userId
                String userId = value.f0;
                //根据状态描述器获取广播状态
                ReadOnlyBroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                if (broadcastState != null) {
                    //取出广播状态中的map<用户id,<姓名,年龄>>
                    Map<String, Tuple2<String, Integer>> map = broadcastState.get(null);
                    if (map != null) {
                        //通过userId取map中的<姓名,年龄>
                        Tuple2<String, Integer> tuple2 = map.get(userId);
                        //取出tuple2中的姓名和年龄
                        String userName = tuple2.f0;
                        Integer userAge = tuple2.f1;
                        out.collect(Tuple6.of(userId, value.f1, value.f2, value.f3, userName, userAge));
                    }
                }
            }
 
            //处理广播流中的元素
            @Override
            public void processBroadcastElement(Map<String, Tuple2<String, Integer>> value, Context ctx, Collector<Tuple6<String, String, String, Integer, String, Integer>> out) throws Exception {
                //value就是MySQLSource中每隔一段时间获取到的最新的map数据
                //先根据状态描述器获取历史的广播状态
                BroadcastState<Void, Map<String, Tuple2<String, Integer>>> broadcastState = ctx.getBroadcastState(descriptor);
                //再清空历史状态数据
                broadcastState.clear();
                //最后将最新的广播流数据放到state中（更新状态数据）
                broadcastState.put(null, value);
            }
        });
        //4.sink
        result.print();
        //5.execute
        env.execute();
    }
 
    /**
     * <userID, eventTime, eventType, productID>
     */
    public static class MySource implements SourceFunction<Tuple4<String, String, String, Integer>>{
        private boolean isRunning = true;
        @Override
        public void run(SourceContext<Tuple4<String, String, String, Integer>> ctx) throws Exception {
            Random random = new Random();
            SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            while (isRunning){
                int id = random.nextInt(4) + 1;
                String user_id = "user_" + id;
                String eventTime = df.format(new Date());
                String eventType = "type_" + random.nextInt(3);
                int productId = random.nextInt(4);
                ctx.collect(Tuple4.of(user_id,eventTime,eventType,productId));
                Thread.sleep(500);
            }
        }
 
        @Override
        public void cancel() {
            isRunning = false;
        }
    }
    /**
     * <用户id,<姓名,年龄>>
     */
    public static class MySQLSource extends RichSourceFunction<Map<String, Tuple2<String, Integer>>> {
        private boolean flag = true;
        private Connection conn = null;
        private PreparedStatement ps = null;
        private ResultSet rs = null;
 
        @Override
        public void open(Configuration parameters) throws Exception {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata", "root", "root");
            String sql = "select `userID`, `userName`, `userAge` from `user_info`";
            ps = conn.prepareStatement(sql);
        }
        @Override
        public void run(SourceContext<Map<String, Tuple2<String, Integer>>> ctx) throws Exception {
            while (flag){
                Map<String, Tuple2<String, Integer>> map = new HashMap<>();
                ResultSet rs = ps.executeQuery();
                while (rs.next()){
                    String userID = rs.getString("userID");
                    String userName = rs.getString("userName");
                    int userAge = rs.getInt("userAge");
                    //Map<String, Tuple2<String, Integer>>
                    map.put(userID,Tuple2.of(userName,userAge));
                }
                ctx.collect(map);
                Thread.sleep(5000);//每隔5s更新一下用户的配置信息!
            }
        }
        @Override
        public void cancel() {
            flag = false;
        }
        @Override
        public void close() throws Exception {
            if (conn != null) conn.close();
            if (ps != null) ps.close();
            if (rs != null) rs.close();
        }
    }
}
```

- 广播变量示例二（广播变量作用于算子）

```java
// 1. The DataSet to be broadcast
DataSet<Integer> toBroadcast = env.fromElements(1, 2, 3);

DataSet<String> data = env.fromElements("a", "b");

data.map(new RichMapFunction<String, String>() {
    @Override
    public void open(Configuration parameters) throws Exception {
      // 3. Access the broadcast DataSet as a Collection
      Collection<Integer> broadcastSet = getRuntimeContext().getBroadcastVariable("broadcastSetName");
    }


    @Override
    public String map(String value) throws Exception {
        ...
    }
}).withBroadcastSet(toBroadcast, "broadcastSetName"); // 2. Broadcast the DataSet
```

