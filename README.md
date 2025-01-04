# **Streamline Orchestrator using Kafka, Kubernetes and Docker**

## **Overview**
This project focuses on building a scalable data processing pipeline using **Kubernetes**, **Kafka**, and **Neo4j**. The pipeline processes NYC taxicab data, demonstrating real-time data production, ingestion, and analytics using Neo4j's Graph Data Science (GDS) library.

## **Prerequisites**
- Kubernetes (with `kubectl` configured)
- Minikube
- Helm
- Python 3.x with the following libraries:
  - `kafka-python`
  - `pyarrow`
  - `pandas`

## **File Descriptions**

| File                        | Description                                                                                   |
|-----------------------------|-----------------------------------------------------------------------------------------------|
| `zookeeper-setup.yaml`      | Kubernetes configuration to deploy Zookeeper.                                                |
| `kafka-setup.yaml`          | Kubernetes configuration to deploy Kafka.                                                    |
| `neo4j-service.yaml`        | Kubernetes configuration to deploy the Neo4j service.                                        |
| `data_producer.py`          | Python script to generate and send test data to Kafka.                                       |
| `grader.md`                 | Grading instructions and commands used in the grading environment.                           |
| `neo4j-values.yaml`         | Helm values file to deploy Neo4j Enterprise with Graph Data Science (GDS) plugin.            |

## **Setup and Execution**

### **Step 1: Start Minikube**
1. Start Minikube with sufficient resources: <br>
   ```bash
   minikube start --memory=10240 --cpus=8
   ```

### **Step 2: Deploy Zookeeper**
1. Deploy Zookeeper:
   ```bash
   kubectl apply -f zookeeper-setup.yaml
   ```

2. Verify the deployment:
   ```bash
   kubectl get pods
   ```

3. Restart Zookeeper if needed:
   ```bash
   kubectl rollout restart deployment/zookeeper-deployment
   ```

### **Step 3: Deploy Kafka**
1. Deploy Kafka:
   ```bash
   kubectl apply -f kafka-setup.yaml
   ```

2. Verify the deployment:
   ```bash
   kubectl get pods
   ```

3. Restart Kafka if needed:
   ```bash
   kubectl rollout restart deployment/kafka-deployment
   ```

### **Step 4: Deploy Neo4j with GDS Plugin**
1. Add the Neo4j Helm repository:
   ```bash
   helm repo add neo4j https://neo4j.github.io/neo4j-helm/
   helm repo update
   ```

2. Install Neo4j with the custom values file:
   ```bash
   helm install neo4j-standalone -f neo4j-values.yaml neo4j/neo4j
   ```

3. Verify Neo4j pod status:
   ```bash
   kubectl get pods
   kubectl logs statefulset/neo4j-standalone
   ```

4. Confirm plugin installation:
   ```bash
   kubectl exec -it neo4j-standalone-0 -- ls -l /var/lib/neo4j/plugins/
   ```

5. Access Neo4j Browser: <br>
  Set up port-forwarding:
    ```bash
    kubectl port-forward svc/neo4j-service 7474:7474 7687:7687
    ```
    - Open your browser and navigate to [http://localhost:7474](http://localhost:7474).
    - Log in with:
      - **Username**: `neo4j`
      - **Password**: `project1phase2`

### **Step 5: Test the Data Producer**
1. Ensure the Kafka topic `nyc_taxicab_data` exists. Create it if not:
   ```bash
   kubectl exec -it kafka-deployment-7cf77dcbf8-22llp -- kafka-topics.sh --create --topic nyc_taxicab_data --partitions 1 --replication-factor 1 --bootstrap-server localhost:9092
   ```

2. Run the `data_producer.py` script:
   ```bash
   python3 data_producer.py
   ```

3. Verify data in Kafka using a consumer:
   ```bash
   kafka-console-consumer --topic nyc_taxicab_data --from-beginning --bootstrap-server localhost:9092
   ```

---

### **Step 6: Analyze Data in Neo4j**
1. Access Cypher Shell:
   ```bash
   kubectl exec -it neo4j-standalone-0 -- bash
   cypher-shell -u neo4j -p project1phase2
   ```

2. Perform PageRank:
   ```cypher
   CALL gds.pageRank.stream('graph_name')
   YIELD nodeId, score
   RETURN gds.util.asNode(nodeId).name AS name, score
   ORDER BY score DESC;
   ```

3. Perform Breadth-First Search (BFS):
   ```cypher
   CALL gds.bfs.stream('graph_name', { startNode: 'node_id' })
   YIELD nodeId, cost
   RETURN gds.util.asNode(nodeId).name AS name, cost;
   ```

## **Verification Commands**
- Check Pods:
  ```bash
  kubectl get pods
  ```
- Check Services:
  ```bash
  kubectl get services
  ```
- Port Forward:
  ```bash
  kubectl port-forward svc/neo4j-service 7474:7474 7687:7687
  ```

## **Cleanup**
To uninstall all deployments:
```bash
kubectl delete -f zookeeper-setup.yaml
kubectl delete -f kafka-setup.yaml
kubectl delete -f neo4j-service.yaml
helm uninstall neo4j-standalone
kubectl delete pvc data-neo4j-standalone
```