# Flask Book API DEV Container Modul 169 BBZBL-IT

This project is a Flask-based REST API for managing a collection of books. It supports operations to create, read, update, and delete books stored in a SQLite database. The application is containerized using Docker.

## Table of Contents

1. [Requirements](#requirements)
2. [Setup and Running](#setup-and-running)
3. [API Endpoints](#api-endpoints)
4. [Troubleshooting](#troubleshooting)
5. [Explanation](#explanation)

## Requirements

- Docker
- curl (for testing endpoints)

## Setup and Running

### Step 1: Create the Application Files

1. **Create the Python script: `app.py`**

``` python
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import Column, Integer, String

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///books.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

class Book(db.Model):
    id = Column(Integer, primary_key=True)
    title = Column(String(80), nullable=False)
    author = Column(String(80), nullable=False)
    year = Column(Integer, nullable=False)

@app.before_first_request
def create_tables():
    db.create_all()

@app.route('/books', methods=['POST'])
def add_book():
    data = request.get_json()
    new_book = Book(title=data['title'], author=data['author'], year=data['year'])
    db.session.add(new_book)
    db.session.commit()
    return jsonify({'message': 'Book added successfully'}), 201

@app.route('/books', methods=['GET'])
def get_books():
    books = Book.query.all()
    result = []
    for book in books:
        book_data = {'id': book.id, 'title': book.title, 'author': book.author, 'year': book.year}
        result.append(book_data)
    return jsonify(result)

@app.route('/books/<int:id>', methods=['GET'])
def get_book(id):
    book = Book.query.get_or_404(id)
    return jsonify({'id': book.id, 'title': book.title, 'author': book.author, 'year': book.year})

@app.route('/books/<int:id>', methods=['PUT'])
def update_book(id):
    data = request.get_json()
    book = Book.query.get_or_404(id)
    book.title = data['title']
    book.author = data['author']
    book.year = data['year']
    db.session.commit()
    return jsonify({'message': 'Book updated successfully'})

@app.route('/books/<int:id>', methods=['DELETE'])
def delete_book(id):
    book = Book.query.get_or_404(id)
    db.session.delete(book)
    db.session.commit()
    return jsonify({'message': 'Book deleted successfully'})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
 ```


2. **Create the Dockerfile: `Dockerfile`**

```
# Use the official Python image from the Docker Hub
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Expose port 5000
EXPOSE 5000

# Run app.py when the container launches
CMD ["python", "app.py"]
```

3. **Create the requierements: `requirements.txt`**
```
flask
flask_sqlalchemy

```

### Step 2: Build and run the Docker Container


1. **Build the Docker image: `sh`**
Open a terminal, navigate to the directory containing your files, and run the following command:
```docker build -t flask-book-api .
```

2. **Run the Docker Container: `sh`**

```
docker run -p 5000:5000 flask-book-api
```

### Step 3: Test the API
Use a tool like Postman or curl to interact with the API. For example, to add a new book:

```
curl -X POST http://localhost:5000/books -H "Content-Type: application/json" -d '{"title": "1984", "author": "George Orwell", "year": 1949}'

```
To get all books

```

curl http://localhost:5000/books

```
### API Endpoints

    ** POST /books: Add a new book.
    ** GET /books: Get a list of all books.
    ** GET /books/int:id: Get a single book by ID.
    ** PUT /books/int:id: Update a book by ID.
    ** DELETE /books/int:id: Delete a book by ID.

### Troubleshooting

1. **List running containers: `sh`**
```
docker ps

```
### Check Docker Container Logs


1. **Get Container ID: `sh`**
```
docker ps

```

2. **Check the logs: `sh`**
```
docker logs <container_id>

```

### Ensure Flask App is Listening on All Interfaces

Ensure your Flask application is set to listen on all network interfaces. This is already done in the provided script with app.run(host='0.0.0.0', port=5000).

### Rebuild and Run the Container
Sometimes it helps to rebuild the Docker image and run the container again to ensure everything is correctly set up.

1. **Rebuild the Docker image: `sh`**

```docker build -t flask-book-api .
```

2. **Run the Docker Container: `sh`**

```
docker run -p 5000:5000 flask-book-api
```

### Access the Application
After ensuring the container is running and there are no errors in the logs, try accessing the application again.

1. **Access the Application: `sh`**
```
 curl http://localhost:5000/books
```

### Network Troubleshooting

1. **Check Docker network settings:**
Make sure Docker is set up to allow network traffic between your host and the container.

2. **Use Docker’s internal IP address:**
Sometimes using localhost might not work as expected. You can find the container's IP address and try accessing it directly.

```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id>

```
Then use the IP address to access the application:
```
curl http://<container_ip>:5000/books

```

## Explanation
### Flask Application

- Flask: A micro web framework for Python.
- Flask_SQLAlchemy: An extension for Flask that adds support for SQLAlchemy, an ORM (Object Relational Mapping) tool.

### Docker
- Docker: A platform for developing, shipping, and running applications inside containers.

### Dockerfile
- FROM python:3.9-slim: Use the official Python 3.9 slim image as the base image.
- WORKDIR /app: Set the working directory inside the container to /app.
- COPY . /app: Copy the current directory contents into the container at /app.
- RUN pip install --no-cache-dir -r requirements.txt: Install the dependencies listed in requirements.txt.
- EXPOSE 5000: Expose port 5000 to the host.
- CMD ["python", "app.py"]: Run app.py when the container launches.



### This Version of the docker Comes with an Initial Test book file


