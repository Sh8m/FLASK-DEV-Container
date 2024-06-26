# Flask Book API

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
