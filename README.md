# Flask CRUD Application

A simple yet comprehensive CRUD (Create, Read, Update, Delete) application built with Python Flask and SQLite database. This project demonstrates fundamental web development concepts including database operations, form handling, and responsive web design.

## ✨ Features

- **Create** new items with name and description
- **Read** and display all items in a formatted table
- **Update** existing items with form validation
- **Delete** items with confirmation
- Clean, responsive user interface
- SQLite database for data persistence
- Modular template structure with Jinja2

## 🚀 Quick Start

### Prerequisites

- Python 3.7 or higher
- pip (Python package installer)

### Installation

1. **Clone or download the project**
   ```bash
   git clone <repository-url>
   cd flask-crud-app
   ```

2. **Create and activate virtual environment**
   ```bash
   # Create virtual environment
   python -m venv venv
   
   # Activate virtual environment
   # On Windows:
   venv\Scripts\activate
   # On macOS/Linux:
   source venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install flask
   ```

4. **Run the application**
   ```bash
   python app.py
   ```

5. **Access the application**
   Open your web browser and navigate to `http://localhost:5000`

## 📁 Project Structure

```
flask-crud-app/
│
├── app.py              # Main Flask application
├── database.db         # SQLite database (auto-created)
├── README.md          # Project documentation
│
├── templates/         # Jinja2 HTML templates
│   ├── base.html      # Base template with common layout
│   ├── index.html     # Homepage showing all items
│   ├── add.html       # Form to add new items
│   └── edit.html      # Form to edit existing items
│
├── static/            # Static assets
│   └── style.css      # Application styling
│
└── venv/              # Virtual environment (auto-created)
```

## 🛠️ Technical Implementation

### Database Schema

The application uses a simple SQLite table:

```sql
CREATE TABLE items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT
);
```

### Core Components

#### Flask Application (app.py)

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

### HTML Templates

#### Base Template (templates/base.html)

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

#### Main Page (templates/index.html)

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

#### Add Item Form (templates/add.html)

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

#### Edit Item Form (templates/edit.html)

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

### Styling (static/style.css)

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

## 🎯 Usage Guide

### Adding Items
1. Click "Add New Item" on the homepage
2. Fill in the name (required) and description (optional)
3. Click "Save" to create the item

### Viewing Items
- All items are displayed in a table on the homepage
- Each row shows ID, name, and description

### Editing Items
1. Click "Edit" next to any item
2. Modify the name or description
3. Click "Update" to save changes

### Deleting Items
1. Click "Delete" next to any item
2. The item will be permanently removed

## 🔧 Development

### Environment Setup

```bash
# Install development dependencies
pip install flask python-dotenv

# Set environment variables (optional)
export FLASK_ENV=development
export FLASK_DEBUG=1
```

### Database Management

The SQLite database is automatically created when you first run the application. To reset the database:

```bash
# Remove the database file
rm database.db

# Restart the application to recreate tables
python app.py
```

## 🚀 Deployment

### Production Considerations

1. **Disable Debug Mode**
   ```python
   app.run(debug=False)
   ```

2. **Use Production WSGI Server**
   ```bash
   pip install gunicorn
   gunicorn -w 4 app:app
   ```

3. **Environment Variables**
   ```bash
   export FLASK_ENV=production
   ```

## 📈 Future Enhancements

### Recommended Improvements

1. **Input Validation & Security**
   - Add Flask-WTF for CSRF protection
   - Implement server-side validation
   - Add input sanitization

2. **User Authentication**
   - Integrate Flask-Login
   - Add user registration/login
   - Implement role-based access

3. **Database Enhancements**
   - Migrate to Flask-SQLAlchemy ORM
   - Add database migrations
   - Implement connection pooling

4. **UI/UX Improvements**
   - Add Bootstrap or modern CSS framework
   - Implement responsive design
   - Add JavaScript for better interactivity

5. **API Development**
   - Create REST API endpoints
   - Add JSON response formats
   - Implement pagination

6. **Testing & Quality**
   - Add unit tests with pytest
   - Implement integration tests
   - Add code coverage reporting

### Advanced Features

- **Search and Filtering**: Add search functionality
- **Data Export**: Export items to CSV/JSON
- **File Uploads**: Allow image attachments
- **Real-time Updates**: WebSocket integration
- **Caching**: Redis or Memcached integration

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/new-feature`)
5. Create a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🆘 Troubleshooting

### Common Issues

**Database locked error:**
- Ensure no other applications are accessing the database
- Check file permissions in the project directory

**Template not found:**
- Verify the `templates/` directory structure
- Check template file names match the routes

**Static files not loading:**
- Confirm the `static/` directory exists
- Verify CSS file path in templates

**Port already in use:**
- Change the port: `app.run(port=5001)`
- Kill existing processes using the port

## 📚 Learning Resources

- [Flask Documentation](https://flask.palletsprojects.com/)
- [SQLite Tutorial](https://www.sqlitetutorial.net/)
- [Jinja2 Templates](https://jinja.palletsprojects.com/)
- [HTML Forms](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)

---

**Happy Coding! 🎉**