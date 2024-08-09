# node-js-todo-devops-project

Sure! I’ll guide you through creating a simple to-do application in Python and setting up a DevOps pipeline using Docker, Ansible, and deployment to an EC2 instance.

### Step 1: Create the To-Do Application in Python

#### 1.1. Set Up the Project Directory
Create a directory for your project:
```bash
mkdir todo-app
cd todo-app
```

#### 1.2. Create a Simple To-Do Application
We'll create a basic Flask application as our to-do app.

**Install Flask:**
```bash
pip install flask
```

**Create the main application file (`app.py`):**
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

todos = []

@app.route('/todos', methods=['GET'])
def get_todos():
    return jsonify(todos)

@app.route('/todos', methods=['POST'])
def add_todo():
    todo = request.json.get('todo')
    if todo:
        todos.append(todo)
        return jsonify({'message': 'Todo added successfully!'}), 201
    return jsonify({'message': 'Failed to add todo'}), 400

@app.route('/todos/<int:index>', methods=['DELETE'])
def delete_todo(index):
    if 0 <= index < len(todos):
        todos.pop(index)
        return jsonify({'message': 'Todo deleted successfully!'}), 200
    return jsonify({'message': 'Invalid index'}), 400

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Run the application locally:**
```bash
python app.py
```

Visit `http://127.0.0.1:5000/todos` in your browser to see the running app.

### Step 2: Dockerize the Application

#### 2.1. Create a Dockerfile
In the root of your project, create a `Dockerfile`:

```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install flask

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Run app.py when the container launches
CMD ["python", "app.py"]
```

#### 2.2. Build and Run the Docker Image
Build the Docker image:
```bash
docker build -t todo-app .
```

Run the Docker container:
```bash
docker run -d -p 5000:5000 todo-app
```

Your application should now be accessible at `http://localhost:5000/todos`.

### Step 3: Set Up Ansible for Deployment

#### 3.1. Create an Ansible Playbook
Create a directory for your Ansible files:
```bash
mkdir ansible
cd ansible
```

Create an inventory file (`hosts`):
```ini
[webserver]
ec2-instance ansible_host=<YOUR_EC2_PUBLIC_IP> ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/your-key.pem
```

Create the playbook (`deploy.yaml`):
```yaml
- hosts: webserver
  become: yes
  tasks:
    - name: Install Docker
      yum:
        name: docker
        state: present

    - name: Start Docker
      service:
        name: docker
        state: started
        enabled: true

    - name: Add the EC2 user to the docker group
      user:
        name: ec2-user
        groups: docker
        append: yes

    - name: Install Docker Compose
      get_url:
        url: "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)"
        dest: /usr/local/bin/docker-compose
        mode: '0755'

    - name: Pull Docker image
      command: docker pull your-dockerhub-username/todo-app

    - name: Run the container
      command: docker run -d -p 5000:5000 your-dockerhub-username/todo-app
```

### Step 4: Deploy the Application to EC2

#### 4.1. Set Up the EC2 Instance
Ensure your EC2 instance is running Amazon Linux or a compatible OS and that you can SSH into it using the key pair.

#### 4.2. Deploy Using Ansible
Run the Ansible playbook:
```bash
ansible-playbook -i hosts deploy.yaml
```

This will:
- Install Docker on the EC2 instance.
- Start the Docker service.
- Pull the Docker image for your app.
- Run the Docker container on the EC2 instance.

### Step 5: Access Your Application

After deployment, your to-do application will be running on your EC2 instance. You can access it using the EC2 instance's public IP and port 5000:

```
http://<YOUR_EC2_PUBLIC_IP>:5000/todos
```

### Summary
You've successfully built a simple to-do application in Python, containerized it using Docker, and set up a DevOps pipeline using Ansible to deploy the application to an AWS EC2 instance. 

Feel free to ask if you need further assistance with any specific step!
