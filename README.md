# Kafka Installation & KRaft Setup on Java 11

---

````markdown
# ⚙️ Kafka Installation & KRaft Setup on Java 11 (Step-by-Step Guide)

Apache Kafka is a powerful distributed streaming platform used for building real-time data pipelines and event-driven systems.  
With the introduction of **KRaft mode (Kafka Raft Metadata mode)**, Kafka can now run **without ZooKeeper**, making deployment simpler and more efficient.

This guide walks you through a **single-node Kafka installation** using **KRaft mode on Java 11**.

---

## 🧩 Overview: What is KRaft Mode?

KRaft (Kafka Raft) mode replaces **ZooKeeper** with a built-in consensus mechanism using the **Raft algorithm**.  
It simplifies Kafka’s architecture by merging the controller and broker roles, enabling faster startup and easier configuration.

---


## 🚀 Step 1: Install Java 11

Kafka requires Java to run. Install **OpenJDK 11** using the following commands:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
````

Ensure Java 11 is active:

```bash
sudo update-alternatives --config java
```

✅ Expected output:

```
openjdk version "11.x.x"
```

---

## 📦 Step 2: Download & Extract Kafka

Download and extract Kafka (version 3.1.0 in this guide):

```bash
wget https://downloads.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
tar -xzf kafka_2.13-3.1.0.tgz
mv kafka_2.13-3.1.0 ~/kafka
cd ~/kafka
```


---

## ⚙️ Step 3: Configure Kafka for KRaft Mode

Open the Kafka configuration file:

```bash
nano config/kraft/server.properties
```

Ensure the following properties are configured:

```properties
############################# Server Basics #############################
node.id=1

############################# Listeners #############################
listeners=PLAINTEXT://localhost:9092,CONTROLLER://localhost:9093
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
inter.broker.listener.name=PLAINTEXT

############################# Log Directories #############################
log.dirs=/home/<USER>/kafka/kraft-combined-logs
metadata.log.dir=/home/<USER>/kafka/kraft-metadata-logs

# For temporary testing (cleared on shutdown)
log.dirs=/tmp/kraft-combined-logs

############################# KRaft Mode #############################
process.roles=broker,controller
controller.quorum.voters=1@localhost:9093
controller.listener.names=CONTROLLER
```

> 💡 Replace `<USER>` with your actual system username.
> Save and exit (`Ctrl + X`, then `Y`).

---

## 💾 Step 4: Format the Log Directory with `kafka-storage.sh`

Kafka requires a **Cluster ID** to initialize metadata storage.

Generate a unique Cluster ID:

```bash
bin/kafka-storage.sh random-uuid
```

Example output:

```
c0a8e8e4-0f2e-4d33-9b2a-7c2e3cf2c1f5
```

Format the Kafka storage directories:

```bash
bin/kafka-storage.sh format -t <CLUSTER_ID> -c config/kraft/server.properties
```

> ⚠️ Replace `<CLUSTER_ID>` with the one you generated.

---

## 🧩 Step 5: Start Kafka in KRaft Mode

Start Kafka using the formatted configuration:

```bash
bin/kafka-server-start.sh config/kraft/server.properties
```

If configured correctly, Kafka will start successfully **without ZooKeeper**, operating fully in **KRaft mode**.

---

## ⚡ Step 6: Verify Kafka Setup

### ✅ Create a Topic

```bash
bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
```

### ✅ List Topics

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### ✅ Start a Producer

```bash
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
```

### ✅ Start a Consumer

```bash
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

Now, type a message in the producer terminal — it will instantly appear in the consumer terminal! 💬

To verify running processes:

```bash
jps
```

Example output:

```
8804 Kafka
10076 ConsoleProducer
10413 ConsoleConsumer
10745 Jps
```

---

### ✍️ Author

**Neeraj Kumar Kannoujiya**
