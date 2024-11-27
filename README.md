# React-Django To-Do App

A full-stack **To-Do List** application built with:
- **Frontend**: React
- **Backend**: Django REST Framework (Python)
- **Database**: PostgreSQL
- **Production Deployment**: Nginx, Gunicorn, PostgreSQL, and AWS EC2

This app allows users to create, update, delete, and manage their to-do lists.

---

## Features

- Full CRUD operations for To-Dos (Create, Read, Update, Delete)
- RESTful API built with Django REST Framework
- Responsive and interactive frontend built with React
- PostgreSQL database for efficient data management
- Production-ready setup with Nginx, Gunicorn, and load balancing on AWS EC2

---

## Architecture

1. **Frontend**: React for the user interface
2. **Backend**: Django REST Framework for the API
3. **Database**: PostgreSQL
4. **Web Server**: Nginx for serving static files and reverse proxy
5. **Application Server**: Gunicorn for handling requests to the Django application
6. **Load Balancer** (Optional): AWS Application Load Balancer for horizontal scaling

---

## Requirements

- Python 3.8+
- Node.js and npm
- PostgreSQL 12+
- Nginx (for production)
- AWS EC2 (for deployment)
- Gunicorn (for running Django in production)

---

## Local Development

### 1. Backend (Django)

#### 1.1 Setup

1. Clone the backend repository:
   ```bash
   git clone https://github.com/your-repo/todo_backend.git
   cd todo_backend
   ```

2. Create and activate a virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Setup the database (PostgreSQL):
   - Create a PostgreSQL database and update the `settings.py` file in the Django project:
     ```python
     DATABASES = {
         'default': {
             'ENGINE': 'django.db.backends.postgresql',
             'NAME': 'todo_db',
             'USER': 'youruser',
             'PASSWORD': 'yourpassword',
             'HOST': 'localhost',
             'PORT': '5432',
         }
     }
     ```

5. Run migrations:
   ```bash
   python manage.py migrate
   ```

6. Run the development server:
   ```bash
   python manage.py runserver
   ```

The backend should be running on [http://localhost:8000](http://localhost:8000).

#### 1.2 API Endpoints

- `GET /api/todos/`: Get the list of todos.
- `POST /api/todos/`: Create a new todo.
- `PUT /api/todos/<id>/`: Update a specific todo.
- `DELETE /api/todos/<id>/`: Delete a specific todo.

### 2. Frontend (React)

#### 2.1 Setup

1. Clone the frontend repository:
   ```bash
   git clone https://github.com/your-repo/todo_frontend.git
   cd todo_frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm start
   ```

The frontend should be running on [http://localhost:3000](http://localhost:3000).

### 3. Running Full Stack Locally

For local development, you can run the backend and frontend servers in parallel:
- **Backend**: Django running on [http://localhost:8000](http://localhost:8000)
- **Frontend**: React running on [http://localhost:3000](http://localhost:3000)

---

## Deployment on AWS EC2 with Nginx and Gunicorn

### 1. EC2 Setup

1. **Launch an EC2 instance**:
   - Choose an Ubuntu or Amazon Linux 2 instance.
   - Configure security groups to allow HTTP (port 80) and SSH (port 22).

2. **Connect to EC2 instance**:
   ```bash
   ssh -i your-key.pem ec2-user@your-ec2-ip
   ```

### 2. Backend Setup

1. **Install dependencies**:
   ```bash
   sudo apt update
   sudo apt install python3-pip python3-dev libpq-dev nginx curl git -y
   ```

2. **Clone the backend**:
   ```bash
   cd ~
   git clone https://github.com/your-repo/todo_backend.git
   cd todo_backend
   ```

3. **Set up a virtual environment and install Python dependencies**:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

4. **Configure PostgreSQL**:
   - Install PostgreSQL:
     ```bash
     sudo apt install postgresql postgresql-contrib
     ```
   - Create a database and user:
     ```bash
     sudo -u postgres psql
     CREATE DATABASE todo_db;
     CREATE USER youruser WITH PASSWORD 'yourpassword';
     GRANT ALL PRIVILEGES ON DATABASE todo_db TO youruser;
     \q
     ```

5. **Update `settings.py` with the PostgreSQL credentials**.

6. **Run migrations and collect static files**:
   ```bash
   python manage.py migrate
   python manage.py collectstatic
   ```

### 3. Frontend Setup

1. **Clone the frontend**:
   ```bash
   cd ~
   git clone https://github.com/your-repo/todo_frontend.git
   cd todo_frontend
   ```

2. **Install dependencies and build the frontend**:
   ```bash
   npm install
   npm run build
   ```

### 4. Gunicorn Setup

1. **Install Gunicorn**:
   ```bash
   pip install gunicorn
   ```

2. **Create a Gunicorn systemd service**:
   ```bash
   sudo nano /etc/systemd/system/gunicorn.service
   ```

   Add this configuration:

   ```ini
   [Unit]
   Description=gunicorn daemon for Django To-Do app
   After=network.target

   [Service]
   User=ec2-user
   WorkingDirectory=/home/ec2-user/todo_backend
   ExecStart=/home/ec2-user/todo_backend/venv/bin/gunicorn --workers 3 --bind unix:/home/ec2-user/todo_backend/gunicorn.sock todo_backend.wsgi:application

   [Install]
   WantedBy=multi-user.target
   ```

3. **Start Gunicorn**:
   ```bash
   sudo systemctl start gunicorn
   sudo systemctl enable gunicorn
   ```

### 5. Nginx Setup

1. **Remove the default Nginx configuration**:
   ```bash
   sudo rm /etc/nginx/sites-enabled/default
   ```

2. **Create a new Nginx config**:
   ```bash
   sudo nano /etc/nginx/sites-available/todo
   ```

   Add this configuration:

   ```nginx
   server {
       listen 80;
       server_name your-ec2-ip;

       location / {
           root /home/ec2-user/todo_frontend/build;
           try_files $uri /index.html;
       }

       location /api/ {
           proxy_pass http://unix:/home/ec2-user/todo_backend/gunicorn.sock;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }

       location /static/ {
           alias /home/ec2-user/todo_backend/static/;
       }
   }
   ```

3. **Enable the config and restart Nginx**:
   ```bash
   sudo ln -s /etc/nginx/sites-available/todo /etc/nginx/sites-enabled/
   sudo systemctl restart nginx
   ```

---

## Load Balancer (Optional)

1. **Create an Application Load Balancer** from AWS EC2 Console.
2. **Register your EC2 instances** in the target group.
3. **Point the DNS** to your load balancer’s DNS name in the Nginx configuration.

---

## License

This project is licensed under the MIT License.
