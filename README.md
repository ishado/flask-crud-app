# Creating a CRUD Project with Python Flask and SQLite

Here's a step-by-step guide to building a basic CRUD (Create, Read, Update, Delete) application using Flask and SQLite.

## 1. Setup the Project

First, create a project directory and set up a virtual environment:

```bash
mkdir flask-crud-app
cd flask-crud-app
python -m venv venv
source venv/bin/activate  # On Windows use: venv\Scripts\activate
```

## 2. Install Required Packages

```bash
pip install flask
```

## 3. Project Structure

Create the following structure:
```
flask-crud-app/
│
├── app.py          # Main application file
├── database.db     # SQLite database (will be created automatically)
├── templates/
│   ├── base.html   # Base template
│   ├── index.html  # List all items
│   ├── add.html    # Add new item
│   └── edit.html   # Edit existing item
└── static/
    └── style.css   # Optional CSS
```

## 4. Database Setup (SQLite)

Flask can work with SQLite directly without additional drivers. Here's how to set it up:

### app.py

```python
from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)

# Database initialization
def get_db_connection():
    conn = sqlite3.connect('database.db')
    conn.row_factory = sqlite3.Row
    return conn

# Create table if not exists
def init_db():
    conn = get_db_connection()
    conn.execute('''
        CREATE TABLE IF NOT EXISTS items (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            description TEXT
        )
    ''')
    conn.commit()
    conn.close()

init_db()

@app.route('/')
def index():
    conn = get_db_connection()
    items = conn.execute('SELECT * FROM items').fetchall()
    conn.close()
    return render_template('index.html', items=items)

@app.route('/add', methods=('GET', 'POST'))
def add():
    if request.method == 'POST':
        name = request.form['name']
        description = request.form['description']
        
        conn = get_db_connection()
        conn.execute('INSERT INTO items (name, description) VALUES (?, ?)',
                     (name, description))
        conn.commit()
        conn.close()
        return redirect(url_for('index'))
    
    return render_template('add.html')

@app.route('/edit/<int:id>', methods=('GET', 'POST'))
def edit(id):
    conn = get_db_connection()
    item = conn.execute('SELECT * FROM items WHERE id = ?', (id,)).fetchone()
    
    if request.method == 'POST':
        name = request.form['name']
        description = request.form['description']
        
        conn.execute('UPDATE items SET name = ?, description = ? WHERE id = ?',
                     (name, description, id))
        conn.commit()
        conn.close()
        return redirect(url_for('index'))
    
    conn.close()
    return render_template('edit.html', item=item)

@app.route('/delete/<int:id>', methods=('POST',))
def delete(id):
    conn = get_db_connection()
    conn.execute('DELETE FROM items WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

## 5. Create Templates

### templates/base.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Flask CRUD App</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

### templates/index.html

```html
{% extends 'base.html' %}

{% block content %}
    <h1>Items</h1>
    <a href="{{ url_for('add') }}">Add New Item</a>
    <table>
        <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Description</th>
            <th>Actions</th>
        </tr>
        {% for item in items %}
        <tr>
            <td>{{ item['id'] }}</td>
            <td>{{ item['name'] }}</td>
            <td>{{ item['description'] }}</td>
            <td>
                <a href="{{ url_for('edit', id=item['id']) }}">Edit</a>
                <form action="{{ url_for('delete', id=item['id']) }}" method="POST" style="display: inline;">
                    <button type="submit">Delete</button>
                </form>
            </td>
        </tr>
        {% endfor %}
    </table>
{% endblock %}
```

### templates/add.html

```html
{% extends 'base.html' %}

{% block content %}
    <h1>Add New Item</h1>
    <form method="POST">
        <label for="name">Name:</label>
        <input type="text" name="name" required>
        <br>
        <label for="description">Description:</label>
        <textarea name="description"></textarea>
        <br>
        <button type="submit">Save</button>
    </form>
    <a href="{{ url_for('index') }}">Cancel</a>
{% endblock %}
```

### templates/edit.html

```html
{% extends 'base.html' %}

{% block content %}
    <h1>Edit Item</h1>
    <form method="POST">
        <label for="name">Name:</label>
        <input type="text" name="name" value="{{ item['name'] }}" required>
        <br>
        <label for="description">Description:</label>
        <textarea name="description">{{ item['description'] }}</textarea>
        <br>
        <button type="submit">Update</button>
    </form>
    <a href="{{ url_for('index') }}">Cancel</a>
{% endblock %}
```

## 6. Optional CSS (static/style.css)

```css
body {
    font-family: Arial, sans-serif;
    margin: 20px;
}

.container {
    max-width: 800px;
    margin: 0 auto;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}

th {
    background-color: #f2f2f2;
}

tr:nth-child(even) {
    background-color: #f9f9f9;
}

form {
    margin-bottom: 20px;
}

label {
    display: inline-block;
    width: 100px;
}

input, textarea {
    margin-bottom: 10px;
    width: 300px;
}

textarea {
    height: 100px;
}
```

## 7. Run the Application

```bash
python app.py
```

Visit `http://localhost:5000` in your browser to see the application in action.

## 8. Enhancements (Optional)

1. **Add input validation**: Use Flask-WTF for form validation
2. **Improve error handling**: Add proper error messages
3. **Add authentication**: Use Flask-Login for user authentication
4. **Use SQLAlchemy**: For a more robust ORM solution
5. **Add pagination**: For large datasets

This basic CRUD application demonstrates the fundamental operations with Flask and SQLite. You can expand it according to your project requirements.