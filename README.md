# todo-flaskapp
#!/usr/bin/env python3
from flask import Flask, render_template_string, request, redirect, session, url_for
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.secret_key = "secret-key"

# In-memory storage (for demo)
users = {}  # {username: {"password": hash, "todos": []}}

# HTML templates (kept simple)
layout = """
<!doctype html>
<title>Flask Todo</title>
<h2 style="text-align:center;">📝 Flask To-Do List</h2>
{% if 'user' in session %}
<p style="text-align:center;">Welcome, {{session['user']}} | 
<a href="{{url_for('logout')}}">Logout</a></p>
{% endif %}
<hr>
<div style="width:400px;margin:auto;">{{content|safe}}</div>
"""

home_html = """
{% if 'user' in session %}
<h3>Your Tasks</h3>
<form method="post" action="{{url_for('add')}}">
    <input type="text" name="task" placeholder="New task" required>
    <button type="submit">Add</button>
</form>
<ul>
{% for i,todo in enumerate(todos) %}
<li>
    {% if todo.done %}
        <s>{{todo.task}}</s>
    {% else %}
        {{todo.task}}
    {% endif %}
    [<a href="{{url_for('toggle', index=i)}}">Toggle</a>]
    [<a href="{{url_for('delete', index=i)}}">Delete</a>]
</li>
{% endfor %}
</ul>
{% else %}
<p><a href="{{url_for('login')}}">Login</a> or <a href="{{url_for('register')}}">Register</a></p>
{% endif %}
"""

auth_html = """
<h3>{{title}}</h3>
<form method="post">
    <input name="username" placeholder="Username" required><br><br>
    <input type="password" name="password" placeholder="Password" required><br><br>
    <button type="submit">{{title}}</button>
</form>
<p><a href="{{url_for('login')}}">Login</a> | <a href="{{url_for('register')}}">Register</a></p>
"""

@app.route("/")
def home():
    if 'user' not in session:
        return render_template_string(layout, content=render_template_string(home_html))
    user = users[session['user']]
    todos = user["todos"]
    return render_template_string(layout, content=render_template_string(home_html, todos=todos))

@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "POST":
        username = request.form["username"]
        password = generate_password_hash(request.form["password"])
        if username in users:
            return "User already exists! <a href='/login'>Login</a>"
        users[username] = {"password": password, "todos": []}
        return redirect(url_for("login"))
    return render_template_string(layout, content=render_template_string(auth_html, title="Register"))

@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form["username"]
        password = request.form["password"]
        user = users.get(username)
        if not user or not check_password_hash(user["password"], password):
            return "Invalid credentials! <a href='/login'>Try again</a>"
        session["user"] = username
        return redirect(url_for("home"))
    return render_template_string(layout, content=render_template_string(auth_html, title="Login"))

@app.route("/logout")
def logout():
    session.pop("user", None)
    return redirect(url_for("home"))

@app.route("/add", methods=["POST"])
def add():
    if 'user' not in session:
        return redirect(url_for('login'))
    task = request.form["task"]
    users[session['user']]["todos"].append({"task": task, "done": False})
    return redirect(url_for("home"))

@app.route("/toggle/<int:index>")
def toggle(index):
    if 'user' not in session:
        return redirect(url_for('login'))
    todos = users[session['user']]["todos"]
    todos[index]["done"] = not todos[index]["done"]
    return redirect(url_for("home"))

@app.route("/delete/<int:index>")
def delete(index):
    if 'user' not in session:
        return redirect(url_for('login'))
    todos = users[session['user']]["todos"]
    todos.pop(index)
    return redirect(url_for("home"))

if __name__ == "__main__":
    app.run(debug=True)
