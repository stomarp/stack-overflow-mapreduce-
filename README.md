# stack-overflow-mapreduce-
Hadoop MapReduce analysis of Stack Overflow comments using Docker and Python
## Project Overview
Analyze user activity from a Stack Overflow comment dataset using Hadoop MapReduce with custom Python Mapper and Reducer scripts. The goal is to compute the number of comments made by each user efficiently at scale.

---

##  Objective
Gain hands-on experience with distributed processing using Hadoop. The project demonstrates how to:
- Preprocess real-world datasets
- Design scalable MapReduce jobs
- Run distributed jobs on a pseudo-distributed Hadoop cluster (Docker)
- Extract meaningful user-level insights

---

##  Dataset
**sample_comments.txt** – Contains tab-separated comment data from Stack Overflow.

Each line (record) contains several fields. Our mapper extracts the user ID (assumed to be field index 4).

---

##  Setup Instructions

### 1. Clone Project and Prepare Files
```
git clone <your-repo-url>
cd hadoop-project
```

Place your input file:
```
sample_comments.txt
```

### 2. Start Hadoop Cluster (Docker)
Ensure Docker is installed and running. Use the following containers:
- `hadoop_namenode`
- `hadoop_datanode`

Also start:
- `hadoop_resourcemanager`
- `hadoop_nodemanager`

You can use images from BDE2020 or a prepared docker-compose setup.

### 3. Upload Data to HDFS
```bash
docker exec -it hadoop_namenode hdfs dfs -mkdir /comments_input
docker cp sample_comments.txt hadoop_namenode:/sample_comments.txt
docker exec -it hadoop_namenode hdfs dfs -put /sample_comments.txt /comments_input
```

### 4. Run MapReduce Job
```bash
docker cp mapper.py hadoop_namenode:/mapper.py
docker cp reducer.py hadoop_namenode:/reducer.py

docker exec -it hadoop_namenode \
  hadoop jar /opt/hadoop-2.7.4/share/hadoop/tools/lib/hadoop-streaming-2.7.4.jar \
  -input /comments_input \
  -output /comments_output_py \
  -mapper "python3 mapper.py" \
  -reducer "python3 reducer.py" \
  -file /mapper.py \
  -file /reducer.py
```

### 5. Retrieve Output
```bash
docker exec -it hadoop_namenode hdfs dfs -ls /comments_output_py
docker exec -it hadoop_namenode hdfs dfs -cat /comments_output_py/part-00000 > result.txt
docker cp hadoop_namenode:/result.txt ./result.txt
```

---

##  Python Scripts

### mapper.py
```python
import sys

for line in sys.stdin:
    parts = line.strip().split('\t')
    if len(parts) >= 5:
        user_id = parts[4]
        if user_id:
            print(f"{user_id}\t1")
```

### reducer.py
```python
import sys

current_user = None
total = 0

for line in sys.stdin:
    user, count = line.strip().split('\t')
    count = int(count)

    if user == current_user:
        total += count
    else:
        if current_user:
            print(f"{current_user}\t{total}")
        current_user = user
        total = count

if current_user:
    print(f"{current_user}\t{total}")
```

---

##  Output Example
```
12345	27
98765	13
...
```

---

##  Notes
- Ensure Python 3 is installed inside the `hadoop_namenode` container.
- Input file must be encoded in UTF-8 to avoid decode errors.

---

## 🎓 Autor Name
Swati

---



