# Bug Tracking-System-Cloud-Computing

Creating a Bug Tracking System using Cloud Computing involves building a web application that tracks bugs or issues within a project, using cloud infrastructure to ensure scalability, availability, and storage. Below are the steps to create such a system, along with example code snippets in Python using Flask as the web framework, SQLAlchemy as the ORM, and AWS (or any cloud provider) for cloud services.

**1. Project Planning and Requirement Analysis**

**Objective:** Create a web-based system to log, track, and resolve bugs in a software development lifecycle, leveraging cloud services for scalability and availability.
**Features:**
User Authentication (Admin, Developer, Tester roles).
Create, view, update, and close bugs.
Assign bugs to developers.
Bug status tracking (Open, In-Progress, Closed).
Cloud storage for logs, backups, and bug reports.
**Tech Stack:**
**Web Framework:** Flask.
**Database:** PostgreSQL (or MySQL, SQLite).
**Cloud:** AWS (EC2 for hosting, S3 for storage, RDS for database).
**Frontend:** HTML/CSS, Bootstrap, JavaScript.

**2. Set Up Development Environment**

**Install Flask, SQLAlchemy, and PostgreSQL (or any preferred database):**

```bash
Copy code
pip install flask flask_sqlalchemy psycopg2-binary
If using AWS, install the AWS SDK (boto3) for cloud integration:
```

```bash
Copy code
pip install boto3
```

**3. Database Design**

**Design the database schema with the following tables:**

**Users:** To store user details (e.g., admin, developer, tester).
**Bugs:** To store bug information.
**Projects:** To categorize bugs based on projects.
**Bug Status:** To track the current status of bugs.

**4. Code Implementation**

**A. Initialize Flask App and Database**
```python
Copy code
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://username:password@localhost/bugtracking'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(100), unique=True, nullable=False)
    role = db.Column(db.String(50), nullable=False)  # e.g., admin, developer, tester

class Project(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), unique=True, nullable=False)

class Bug(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    description = db.Column(db.Text, nullable=False)
    status = db.Column(db.String(50), default='Open')
    project_id = db.Column(db.Integer, db.ForeignKey('project.id'), nullable=False)
    assigned_to = db.Column(db.Integer, db.ForeignKey('user.id'))
    created_by = db.Column(db.Integer, db.ForeignKey('user.id'))
    date_created = db.Column(db.DateTime, default=db.func.now())

if __name__ == '__main__':
    db.create_all()  # Create tables in the database
    app.run(debug=True)
**B. Bug Creation and Management**
**Route for Adding Bugs:**
```

```python
Copy code
from flask import request, jsonify

@app.route('/add_bug', methods=['POST'])
def add_bug():
    title = request.json['title']
    description = request.json['description']
    project_id = request.json['project_id']
    created_by = request.json['created_by']
    
    new_bug = Bug(title=title, description=description, project_id=project_id, created_by=created_by)
    db.session.add(new_bug)
    db.session.commit()
    
    return jsonify({'message': 'Bug created successfully'}), 201
Route for Updating Bug Status:
```

```python
Copy code
@app.route('/update_bug/<int:bug_id>', methods=['PUT'])
def update_bug(bug_id):
    bug = Bug.query.get_or_404(bug_id)
    bug.status = request.json['status']  # 'Open', 'In-Progress', 'Closed'
    bug.assigned_to = request.json.get('assigned_to', bug.assigned_to)
    
    db.session.commit()
    return jsonify({'message': 'Bug updated successfully'}), 200
Route for Viewing All Bugs:
```

```python
Copy code
@app.route('/get_bugs', methods=['GET'])
def get_bugs():
    bugs = Bug.query.all()
    return jsonify([{
        'id': bug.id,
        'title': bug.title,
        'description': bug.description,
        'status': bug.status,
        'project': bug.project_id,
        'assigned_to': bug.assigned_to,
        'created_by': bug.created_by,
        'date_created': bug.date_created
    } for bug in bugs])
```

**C. User Authentication**
Implement a simple user authentication system with user roles (Admin, Developer, Tester).

```python
Copy code
from flask import request, jsonify, session
from werkzeug.security import generate_password_hash, check_password_hash

# User Model Extended
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(100), unique=True, nullable=False)
    password = db.Column(db.String(200), nullable=False)
    role = db.Column(db.String(50), nullable=False)

@app.route('/register', methods=['POST'])
def register():
    username = request.json['username']
    password = generate_password_hash(request.json['password'])
    role = request.json['role']
    
    new_user = User(username=username, password=password, role=role)
    db.session.add(new_user)
    db.session.commit()
    
    return jsonify({'message': 'User registered successfully'}), 201

@app.route('/login', methods=['POST'])
def login():
    user = User.query.filter_by(username=request.json['username']).first()
    if user and check_password_hash(user.password, request.json['password']):
        session['user_id'] = user.id
        session['role'] = user.role
        return jsonify({'message': 'Login successful'}), 200
    return jsonify({'message': 'Invalid credentials'}), 401
```

**5. Cloud Integration**
For cloud computing, use AWS services like EC2 for hosting, RDS for database management, and S3 for bug attachments or logs.

**A. Deploy Flask App on AWS EC2**

**Step 1:** Launch an EC2 instance (e.g., t2.micro).
**Step 2:** SSH into your EC2 instance and install necessary software:

```bash
Copy code
sudo apt update
sudo apt install python3-pip
pip3 install flask flask_sqlalchemy psycopg2-binary
```

**Step 3:** Clone your project and run the Flask app on EC2.
**Step 4:** Set up AWS RDS for the PostgreSQL database and configure the Flask app to connect to it.

**B. Use AWS S3 for File Storage**
If you want to upload and store attachments (e.g., screenshots, logs) for each bug report, use AWS S3.

Code to Upload Files to S3:

```python
Copy code
import boto3
from flask import request

s3 = boto3.client('s3')
BUCKET_NAME = 'your-bucket-name'

@app.route('/upload_file/<int:bug_id>', methods=['POST'])
def upload_file(bug_id):
    file = request.files['file']
    s3.upload_fileobj(file, BUCKET_NAME, file.filename)
    return jsonify({'message': 'File uploaded successfully'}), 200
Code to Download Files from S3:
```

```python
Copy code
@app.route('/download_file/<string:filename>', methods=['GET'])
def download_file(filename):
    file_obj = s3.get_object(Bucket=BUCKET_NAME, Key=filename)
    return file_obj['Body'].read(), 200
```

**6. Frontend Development**
You can create a simple frontend for the bug tracking system using HTML/CSS, Bootstrap, and JavaScript.

Example of a bug creation form:

```html
Copy code
<form id="bugForm">
    <label for="title">Bug Title:</label>
    <input type="text" id="title" name="title" required>

    <label for="description">Bug Description:</label>
    <textarea id="description" name="description" required></textarea>

    <label for="project">Project:</label>
    <select id="project" name="project_id">
        <!-- Populate with project data -->
    </select>

    <button type="submit">Create Bug</button>
</form>

<script>
    document.getElementById('bugForm').addEventListener('submit', function(event) {
        event.preventDefault();
        
        const formData = new FormData(event.target);
        const data = Object.fromEntries(formData.entries());

        fetch('/add_bug', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data)
        }).then(response => response.json())
          .then(result => alert(result.message));
    });
</script>
```

**7. Testing and Validation**

**Test all API endpoints:** Ensure that the bug creation, update, and retrieval processes work correctly.
**User Authentication:** Validate the user registration and login functionality.
**Cloud Functionality:** Test uploading and downloading files from S3.

**8. Deployment**

**Use AWS Elastic Beanstalk:** For a more automated deployment, consider deploying your Flask application using AWS Elastic Beanstalk.
**Monitor and Scale:** Set up monitoring and alerts on your AWS resources to ensure the application runs smoothly.

**9. Enhancements**

**User Roles and Permissions:** Implement role-based access control for different user types.
**Search and Filter:** Add search and filter functionality to view bugs.
**Notifications:** Implement email notifications for bug assignment and status updates.

By following these steps and using the provided code snippets, you can create a functional Bug Tracking System leveraging cloud computing principles. Let me know if you need more details on any specific part!
