### Python Scripting for DevOps/SRE Engineers

Python scripting in DevOps/SRE involves writing scripts to automate tasks like configuration management, monitoring, and infrastructure orchestration. Here’s a roadmap to get started:

---

#### 1. **Understand the Basics of Scripting**
   - Scripts are typically short and focused on automating repetitive tasks.
   - Unlike development (where you might create full applications), scripting is about writing standalone utilities.

---

#### 2. **Key Areas for Python Scripting in DevOps/SRE**
   - **File and Directory Management**: Automate file creation, modification, backups, etc.
     ```python
     import os
     os.mkdir('test_dir')  # Create a directory
     with open('log.txt', 'w') as file:  # Create and write to a file
         file.write('Logging starts here.')
     ```
   - **Process Automation**:
     Use `subprocess` to execute shell commands.
     ```python
     import subprocess
     result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
     print(result.stdout)
     ```
   - **Infrastructure Management**:
     Use libraries like **boto3** (AWS), **google-cloud-sdk**, or **azure-mgmt** to manage cloud resources.
   - **API Automation**:
     Use the `requests` library to interact with REST APIs.
     ```python
     import requests
     response = requests.get('https://api.github.com')
     print(response.json())
     ```

---

#### 3. **Working with JSON in Python**
JSON is commonly used for configuration files and API responses.

- **Reading JSON from a File**:
  ```python
  import json
  
  with open('config.json', 'r') as file:
      data = json.load(file)
  print(data)
  ```
- **Parsing JSON from a String**:
  ```python
  json_string = '{"name": "Lijo", "role": "DevOps Engineer"}'
  data = json.loads(json_string)
  print(data['name'])  # Output: Lijo
  ```
- **Writing JSON to a File**:
  ```python
  data = {'name': 'Lijo', 'role': 'DevOps Engineer'}
  with open('output.json', 'w') as file:
      json.dump(data, file, indent=4)
  ```
- **Interacting with Nested JSON**:
  ```python
  nested_json = {
      "user": {"name": "Lijo", "details": {"role": "SRE", "skills": ["Python", "Kubernetes"]}}
  }
  print(nested_json['user']['details']['skills'][0])  # Output: Python
  ```

---

#### 4. **Common Libraries**
   - **os** and **shutil**: File operations.
   - **subprocess**: Run shell commands.
   - **json**: Handle JSON data.
   - **yaml**: Handle YAML (used in Kubernetes manifests).
     ```python
     import yaml

     with open('config.yaml') as file:
         data = yaml.safe_load(file)
     print(data)
     ```
   - **argparse**: Parse command-line arguments.
     ```python
     import argparse

     parser = argparse.ArgumentParser(description="Process some integers.")
     parser.add_argument('--name', type=str, help='Your name')
     args = parser.parse_args()
     print(f'Hello {args.name}!')
     ```

---

#### 5. **Integrating Python with DevOps Tools**
   - Use **Jenkins REST APIs** to trigger builds.
   - Write scripts for **Ansible Playbook** execution.
   - Work with **Prometheus APIs** for custom monitoring.

---

#### Next Steps:
1. Start with small automation projects like cleaning up old logs, checking system health, or automating backups.
2. Use virtual environments and good code structuring to maintain readability.
3. Leverage cloud SDKs and REST APIs to integrate with infrastructure.

Would you like specific use-case scripts (e.g., automating Kubernetes, interacting with AWS)?


Automating Kubernetes (K8s) using Python scripting is highly effective for managing deployments, scaling, and monitoring. Below are some key steps and concepts to get you started.

---

### **1. Install Required Tools**
- **Python Libraries**:
  - Use the **kubernetes** Python client library to interact with the Kubernetes API.
  ```bash
  pip install kubernetes
  ```

---

### **2. Authenticate to the Cluster**
- Make sure you have access to the Kubernetes cluster.
- The script will use the `~/.kube/config` file or in-cluster authentication.
  ```python
  from kubernetes import config

  # Load kubeconfig (used for local development)
  config.load_kube_config()

  # Or use in-cluster config if running within a pod
  # config.load_incluster_config()
  ```

---

### **3. Common Automation Tasks**

#### a. **List Pods in a Namespace**
```python
from kubernetes import client, config

# Load kubeconfig
config.load_kube_config()

v1 = client.CoreV1Api()

namespace = "default"
pods = v1.list_namespaced_pod(namespace)

for pod in pods.items:
    print(f"Pod Name: {pod.metadata.name}, Status: {pod.status.phase}")
```

---

#### b. **Create a Deployment**
```python
from kubernetes import client, config

config.load_kube_config()

apps_v1 = client.AppsV1Api()

deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="nginx-deployment"),
    spec=client.V1DeploymentSpec(
        replicas=3,
        selector={"matchLabels": {"app": "nginx"}},
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={"app": "nginx"}),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="nginx",
                        image="nginx:1.21",
                        ports=[client.V1ContainerPort(container_port=80)],
                    )
                ]
            ),
        ),
    ),
)

response = apps_v1.create_namespaced_deployment(
    namespace="default", body=deployment
)
print(f"Deployment created. Name: {response.metadata.name}")
```

---

#### c. **Scale a Deployment**
```python
from kubernetes import client, config

config.load_kube_config()

apps_v1 = client.AppsV1Api()
namespace = "default"
deployment_name = "nginx-deployment"

# Fetch current deployment
deployment = apps_v1.read_namespaced_deployment(deployment_name, namespace)

# Modify the replicas
deployment.spec.replicas = 5
apps_v1.replace_namespaced_deployment(deployment_name, namespace, deployment)

print(f"Scaled {deployment_name} to 5 replicas.")
```

---

#### d. **Delete Resources**
```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CoreV1Api()

namespace = "default"
pod_name = "example-pod"

v1.delete_namespaced_pod(name=pod_name, namespace=namespace)

print(f"Pod {pod_name} deleted.")
```

---

#### e. **Monitor Resource Usage**
Use **metrics-server** or Prometheus APIs to fetch resource usage.
```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CustomObjectsApi()

# Replace with the correct API group, version, and resource for metrics-server
metrics = v1.list_namespaced_custom_object(
    group="metrics.k8s.io", version="v1beta1", namespace="default", plural="pods"
)

for pod in metrics['items']:
    print(f"Pod: {pod['metadata']['name']}, CPU: {pod['containers'][0]['usage']['cpu']}")
```

---

### **4. Advanced Automation Tasks**
- **Rolling Updates**:
  Automate rolling updates to deployments for zero-downtime deployment.
- **Backup and Restore**:
  Use tools like **Velero** or interact with Persistent Volume Claims (PVCs).
- **Event Monitoring**:
  Monitor events for automated issue detection.
  ```python
  stream = client.Watch().stream(v1.list_event_for_all_namespaces)
  for event in stream:
      print(f"Event: {event['type']} - {event['object'].message}")
  ```

---

### **5. Best Practices**
- **Error Handling**:
  Add robust error-handling mechanisms.
  ```python
  try:
      v1.read_namespaced_pod("example-pod", "default")
  except client.exceptions.ApiException as e:
      print(f"Error: {e}")
  ```
- **Logging**:
  Use Python’s `logging` module for better observability.
- **Modular Code**:
  Split scripts into reusable functions.

---

### **Next Steps**
Would you like an example integrating Kubernetes scripting into CI/CD pipelines or monitoring?


Improving your Python scripting skills for DevOps/SRE engineering involves mastering a wide range of tasks, techniques, and tools. Here's a structured approach:

---

### **1. Practice with Different Scripting Tasks**

#### a. **File and Directory Management**
- Automate log rotation or archiving.
- Create scripts to search for files, process them, and move them based on conditions.
  ```python
  import os
  import shutil

  for file in os.listdir('/logs'):
      if file.endswith('.log'):
          shutil.move(f'/logs/{file}', '/archive/')
  ```

#### b. **System Monitoring**
- Write scripts to monitor disk space, memory, and CPU usage.
  ```python
  import psutil

  print(f"CPU Usage: {psutil.cpu_percent()}%")
  print(f"Memory Usage: {psutil.virtual_memory().percent}%")
  ```

#### c. **Network Tasks**
- Automate network checks like DNS resolution, ping, or port scanning.
  ```python
  import socket

  hostname = 'example.com'
  ip = socket.gethostbyname(hostname)
  print(f"{hostname} resolved to {ip}")
  ```

#### d. **API Interaction**
- Automate interactions with APIs for cloud, CI/CD, or monitoring tools.
  ```python
  import requests

  response = requests.get('https://api.github.com/repos/username/repo')
  print(response.json())
  ```

#### e. **Cloud Automation**
- Automate tasks using **boto3** (AWS), **azure-mgmt**, or Google Cloud libraries.
  ```python
  import boto3

  ec2 = boto3.resource('ec2')
  for instance in ec2.instances.all():
      print(f"Instance ID: {instance.id}, State: {instance.state['Name']}")
  ```

#### f. **Container Management**
- Automate Docker image building and container orchestration.
  ```python
  import docker

  client = docker.from_env()
  container = client.containers.run("nginx", detach=True)
  print(f"Started container: {container.id}")
  ```

#### g. **Orchestration and IaC**
- Write scripts for Kubernetes (as discussed earlier) or Terraform automation.

#### h. **CI/CD Pipelines**
- Trigger builds, deploy artifacts, or analyze test results via APIs.
  ```python
  import requests

  url = "http://jenkins.example.com/job/myjob/build"
  requests.post(url, auth=('user', 'pass'))
  ```

---

### **2. Enhance Skills through Structured Learning**

#### a. **Focus on Key Libraries**
- Learn **argparse** for scripting command-line utilities.
- Explore **paramiko** for SSH automation.
- Use **pexpect** to interact with interactive shell commands.
- Explore **pyyaml** for YAML file processing.

#### b. **Master Automation Tools**
- Experiment with Python scripts to trigger Ansible playbooks or Terraform commands.
- Automate Helm chart updates using Python.

#### c. **Integrate with DevOps Tools**
- Work with tools like Jenkins, Prometheus, Grafana, or Elastic Stack.
- Use APIs to monitor and manage CI/CD pipelines.

---

### **3. Build Real-World Projects**
- **Log Management**:
  Automate collection, processing, and analysis of log files.
- **Monitoring Dashboards**:
  Fetch data from Prometheus or other monitoring tools and generate reports.
- **Cluster Health Check**:
  Automate periodic health checks for Kubernetes clusters.
- **Infrastructure Automation**:
  Write scripts to provision resources dynamically based on usage.

---

### **4. Practice Problem-Solving**
- Use platforms like **LeetCode**, **HackerRank**, or **Advent of Code** to improve logical thinking.
- Solve scripting challenges specific to DevOps, e.g., automate backup, check health, or process alerts.

---

### **5. Improve Code Quality**
- Write modular, reusable functions.
- Follow best practices like using virtual environments, handling errors gracefully, and adding logging.
- Learn testing frameworks like **pytest** for script validation.

---

### **6. Contribute to Open-Source Projects**
- Find Python-based DevOps projects on GitHub and contribute to them.
- Collaborate with teams to learn advanced techniques.

---

### **7. Learn from Real-World Use Cases**
- Study automation workflows implemented by teams in your current or past roles.
- Analyze scripts from open-source projects to understand advanced patterns.

---

### **8. Develop a Daily Practice Habit**
- Dedicate 30–60 minutes daily to Python scripting.
- Pick one area (e.g., API interaction) and focus on building multiple scripts for it.

---

### **9. Read and Watch Resources**
- **Books**:
  - *Automate the Boring Stuff with Python* by Al Sweigart.
  - *Python for DevOps* by Noah Gift.
- **Courses**:
  - Take courses on platforms like Udemy, Pluralsight, or Coursera focused on Python for DevOps.
- **Documentation**:
  - Read official documentation for libraries like `kubernetes`, `boto3`, and `requests`.

---

### **10. Experiment and Fail**
- Write scripts even if you don’t know the complete solution; learn by debugging and improving.

Would you like specific practice tasks tailored to your goals or a project idea to work on?


