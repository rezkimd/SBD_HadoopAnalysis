# SBD_HadoopAnalysis
Project untuk menganalisis waktu runtime efektif pada algoritma wordcount menggunakan hadoop dan python. Project ini digunakan untuk 

## Linux Installation

- Update system
```
sudo apt upgrade
```

- Install Java 8
```
sudo apt install openjdk-8-jdk
```

- Check java installation and version
```
java -version
```
![java version](./docs/java-version.png)

- Install openssh
```
sudo apt install openssh-server openssh-client
```

- Generate ssh key
```
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```

- Add ssh key to authorized key
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

- Set permission 
```
chmod 0600 ~/.ssh/authorized_keys
```

- Check ssh connection
```
ssh localhost
```

- Install hadoop
```
cd ~
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.5/hadoop-3.3.5.tar.gz
tar -xzvf hadoop-3.3.5.tar.gz
mv hadoop-3.3.5 hadoop
```

Hadoop excels when deployed in a fully distributed mode on a large cluster of networked servers. However, if you are new to Hadoop and want to explore basic commands or test applications, you can configure Hadoop on a single node.

This setup, also called pseudo-distributed mode, allows each Hadoop daemon to run as a single Java process. A Hadoop environment is configured by editing a set of configuration files:

- bashrc: Set environment variables used by Hadoop scripts
- hadoop-env.sh: Set environment variables used by Hadoop scripts
- core-site.xml: Set configuration parameters affecting Hadoop core, such as I/O settings that are common to HDFS and MapReduce
- hdfs-site.xml: Set configuration parameters specific to HDFS daemons, such as the location of data and name directories
- mapred-site.xml: Set configuration parameters specific to MapReduce daemons, such as the number of mapper and reducer slots to be used on each node
- yarn-site.xml: Set configuration parameters for the YARN daemons, such as the number of maximum applications to run simultaneously

- Edit bashrc
```
gedit ~/.bashrc
```

- Add the following lines to the end of the file
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=$HOME/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

![bashrc](./docs/bashrc.png)

- Refresh bashrc
```
source ~/.bashrc
```

- Edit hadoop-env.sh
```
gedit $HOME/hadoop/etc/hadoop/hadoop-env.sh
```

- Add this line
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

![env](./docs/hadoop-env.png)

- Edit core-site.xml
```
gedit $HOME/hadoop/etc/hadoop/core-site.xml
```

- Add the following lines
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

![core-site](./docs/core-site.png)

- Edit hdfs-site.xml
```
gedit $HOME/hadoop/etc/hadoop/hdfs-site.xml
```

- Add the following lines
```
<configuration>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>/home/juanjonathan67/data/nameNode</value>
    </property>

    <property>
            <name>dfs.datanode.data.dir</name>
            <value>/home/juanjonathan67/data/dataNode</value>
    </property>

    <property>
            <name>dfs.replication</name>
            <value>1</value>
    </property>
</configuration>
```

![hdfs-site](./docs/hdfs-site.png)

- Edit mapred-site.xml
```
gedit $HOME/hadoop/etc/hadoop/mapred-site.xml
```

- Add the following lines
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

![mapred-site](./docs/mapred-site.png)

- Edit yarn-site.xml
```
gedit $HOME/hadoop/etc/hadoop/yarn-site.xml
```

- Add the following lines
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

![yarn-site](./docs/yarn-site.png)

- Check hadoop version
```
hadoop version
```

- Format namenode
```
hdfs namenode -format
```

- Start hadoop
```
start-dfs.sh
start-yarn.sh
```

- Check hadoop status
```
jps
```

![jps](./docs/jps.png)

- Check hadoop web interface
```
http://localhost:8088
```

![web](./docs/web.png)

- Check hdfs web interface
```
http://localhost:9870
```

![hdfs](./docs/hdfs.png)

## Linux Word Count Example

- Word Count Java Program
```
cd ~
gedit WordCount.java
```

- Add the following lines
```
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.net.URI;
import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.Counter;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.StringUtils;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    static enum CountersEnum { INPUT_WORDS }

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    private boolean caseSensitive;
    private Set<String> patternsToSkip = new HashSet<String>();

    private Configuration conf;
    private BufferedReader fis;

    @Override
    public void setup(Context context) throws IOException,
        InterruptedException {
      conf = context.getConfiguration();
      caseSensitive = conf.getBoolean("wordcount.case.sensitive", true);
      if (conf.getBoolean("wordcount.skip.patterns", false)) {
        URI[] patternsURIs = Job.getInstance(conf).getCacheFiles();
        for (URI patternsURI : patternsURIs) {
          Path patternsPath = new Path(patternsURI.getPath());
          String patternsFileName = patternsPath.getName().toString();
          parseSkipFile(patternsFileName);
        }
      }
    }

    private void parseSkipFile(String fileName) {
      try {
        fis = new BufferedReader(new FileReader(fileName));
        String pattern = null;
        while ((pattern = fis.readLine()) != null) {
          patternsToSkip.add(pattern);
        }
      } catch (IOException ioe) {
        System.err.println("Caught exception while parsing the cached file '"
            + StringUtils.stringifyException(ioe));
      }
    }

    @Override
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      String line = (caseSensitive) ?
          value.toString() : value.toString().toLowerCase();
      for (String pattern : patternsToSkip) {
        line = line.replaceAll(pattern, "");
      }
      StringTokenizer itr = new StringTokenizer(line);
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
        Counter counter = context.getCounter(CountersEnum.class.getName(),
            CountersEnum.INPUT_WORDS.toString());
        counter.increment(1);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    GenericOptionsParser optionParser = new GenericOptionsParser(conf, args);
    String[] remainingArgs = optionParser.getRemainingArgs();
    if ((remainingArgs.length != 2) && (remainingArgs.length != 4)) {
      System.err.println("Usage: wordcount <in> <out> [-skip skipPatternFile]");
      System.exit(2);
    }
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount2.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);

    List<String> otherArgs = new ArrayList<String>();
    for (int i=0; i < remainingArgs.length; ++i) {
      if ("-skip".equals(remainingArgs[i])) {
        job.addCacheFile(new Path(remainingArgs[++i]).toUri());
        job.getConfiguration().setBoolean("wordcount.skip.patterns", true);
      } else {
        otherArgs.add(remainingArgs[i]);
      }
    }
    FileInputFormat.addInputPath(job, new Path(otherArgs.get(0)));
    FileOutputFormat.setOutputPath(job, new Path(otherArgs.get(1)));

    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

- Create input folder in the hadoop file system
```
hadoop fs -mkdir /input
```

- Create a text file with random words
```
gedit input.txt
```

- Add the following lines
```
Hello World Bye World
Hello Hadoop Goodbye Hadoop
```

- Copy the file to the hadoop file system
```
hadoop fs -put input.txt /input
```

- Export classpath
```
export HADOOP_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
```

- Compile the java program
```
mkdir WordCountCompiled
sudo chmod -R 777 WordCountCompiled
javac -classpath ${HADOOP_CLASSPATH} -d WordCountCompiled WordCount.java
```

- Create a jar
```
jar -cvf WordCount.jar -C WordCountCompiled/ .
```

- Run the jar to execute WordCount program
```
hadoop jar WordCount.jar WordCount /input /output
```

- Check the output
```
hadoop fs -cat /output/part-r-00000
```

Check localhost:8088 for the job and application status. Check localhost:9870 for the file system status.


## Windows Installation

References : 
1. https://youtu.be/knAS0w-jiUk
2. https://youtu.be/7O56u3LyPTY


- Install Java 8
```
https://www.oracle.com/java/technologies/downloads/#java8-windows
```

- Download & Extract Hadoop 3.3.5
```
https://hadoop.apache.org/
Download the binary one
```

- Add to Environment Variable
```
JAVA_HOME -> Java installation Path (jdk... path)
HADOOP_HOME -> Hadoop Extracted Bin Path
```

- Add to System Path Environment Variable
```
C:\Java\jdk....\bin
C:\hadoop\bin
C:\hadoop\sbin
```

- Check Installation
```
java -version
hadoop -version
```

- Configuring Hadoop


1. Go to "./etc/hadoop" and find "hadoop.env"
2. change the JAVA_HOME to your JAVA_HOME path 
3. Go back to hadoop folder and create a new folder called "data"
4. inside the data folder create 2 folder called "namenode" and "datanode"
5. now go back to "./etc/hadoop" and follow the hadoop_install_config.txt on Fix folder from this github

- Fixing Namenode Shutdown Problem

1. Copy the Fix folder from this github
2. delete the bin folder on the hadoop installation
3. copy the bin from this fix folder
4. Download the file below :
```
https://www.dll-files.com/msvcr120.dll.html
```
5. Copy the msvcr120.dll to "C:\Windows\System32"
6. Install msvc-170 from this link:
```
https://learn.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170
```


- Formatting Namenode (Administrator CMD)
```
hdfs namenode -format
```

- Launching Hadoop (go to hadoop sbin folder)

```
cd hadoop/sbin
start-dfs.cmd
start-yarn.cmd
```

- Check hadoop status
```
jps
```

- Check hadoop web interface
```
http://localhost:8088
```

- Check hdfs web interface
```
http://localhost:9870
```

## Word Count example on Windows

Always on Administrator CMD

- Create input folder in the hadoop file system
```
hdfs dfs -mkdir /input
```

- Put the file to the hadoop file system
```
hdfs dfs -put <source file path> /input
```

- Running word count

Find the path to the hadoop-mapreduce-examples-3.3.5,jar on the ./share/hadoop/mapreduce folder for example : C:\hadoop\share\hadoop\mapreduce\hadoop-mapreduce-examples-3.3.5.jar 

```
hadoop jar <hadoop-mapreduce-examples-3.3.5.jar path> wordcount <hadoop input path> <output hadoop folder path>
```
for example :
```
hadoop jar C:\hadoop\share\hadoop\mapreduce\hadoop-mapreduce-examples-3.3.5.jar  wordcount /input/test.txt /output/test
```
- Java heap error
```
Change the XMX line on the mapred-site.xml
```

- Getting the output to local
```
hdfs dfs -get <output hadoop part-r-00000 path> <local path>
```

for example
```
hdfs dfs -get /output/test/part-r-00000 C:\Downloads\test.txt

```