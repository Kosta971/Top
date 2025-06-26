
from flask import Flask, render_template_string, request, redirect, session, url_for
import sqlite3

app = Flask(__name__)
app.secret_key = 'taskboost_secret_key'

# Инициализация базы данных
def init_db():
    conn = sqlite3.connect('taskboost.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    username TEXT UNIQUE,
                    password TEXT,
                    balance REAL DEFAULT 0
                )''')
    c.execute('''CREATE TABLE IF NOT EXISTS tasks (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    title TEXT,
                    description TEXT,
                    reward REAL,
                    owner TEXT,
                    done_by TEXT
                )''')
    conn.commit()
    conn.close()

@app.route('/')
def home():
    conn = sqlite3.connect('taskboost.db')
    c = conn.cursor()
    c.execute("SELECT * FROM tasks WHERE done_by IS NULL")
    tasks = c.fetchall()
    conn.close()
    return render_template_string(HOME_HTML, tasks=tasks, username=session.get('username'))

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        conn = sqlite3.connect('taskboost.db')
        c = conn.cursor()
        try:
            c.execute("INSERT INTO users (username, password) VALUES (?, ?)", (username, password))
            conn.commit()
        except:
            conn.close()
            return "Username already exists!"
        conn.close()
        return redirect(url_for('login'))
    return render_template_string(REGISTER_HTML)

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        conn = sqlite3.connect('taskboost.db')
        c = conn.cursor()
        c.execute("SELECT * FROM users WHERE username=? AND password=?", (username, password))
        user = c.fetchone()
        conn.close()
        if user:
            session['username'] = username
            return redirect(url_for('home'))
        return "Invalid credentials"
    return render_template_string(LOGIN_HTML)

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('home'))

@app.route('/create_task', methods=['GET', 'POST'])
def create_task():
    if 'username' not in session:
        return redirect(url_for('login'))
    if request.method == 'POST':
        title = request.form['title']
        description = request.form['description']
        reward = float(request.form['reward'])
        owner = session['username']
        conn = sqlite3.connect('taskboost.db')
        c = conn.cursor()
        c.execute("INSERT INTO tasks (title, description, reward, owner) VALUES (?, ?, ?, ?)",
                  (title, description, reward, owner))
        conn.commit()
        conn.close()
        return redirect(url_for('home'))
    return render_template_string(CREATE_TASK_HTML)

@app.route('/do_task/<int:task_id>')
def do_task(task_id):
    if 'username' not in session:
        return redirect(url_for('login'))
    conn = sqlite3.connect('taskboost.db')
    c = conn.cursor()
    c.execute("UPDATE tasks SET done_by=? WHERE id=?", (session['username'], task_id))
    c.execute("UPDATE users SET balance = balance + (SELECT reward FROM tasks WHERE id=?) WHERE username=?",
              (task_id, session['username']))
    conn.commit()
    conn.close()
    return redirect(url_for('home'))

@app.route('/profile')
def profile():
    if 'username' not in session:
        return redirect(url_for('login'))
    conn = sqlite3.connect('taskboost.db')
    c = conn.cursor()
    c.execute("SELECT balance FROM users WHERE username=?", (session['username'],))
    balance = c.fetchone()[0]
    conn.close()
    return render_template_string(PROFILE_HTML, username=session['username'], balance=balance)

HOME_HTML = '''
<!DOCTYPE html>
<html><head><title>TaskBoost</title></head><body>
<h1>Welcome to TaskBoost</h1>
{% if username %}
<p>Hello, {{username}} | <a href="/logout">Logout</a> | <a href="/create_task">Create Task</a> | <a href="/profile">My Profile</a></p>
{% else %}
<p><a href="/login">Login</a> or <a href="/register">Register</a></p>
{% endif %}
<h2>Available Tasks</h2>
<ul>
{% for task in tasks %}
    <li><b>{{task[1]}}</b> - {{task[2]}} - ${{task[3]}} 
    {% if username %}<a href="/do_task/{{task[0]}}">[Do Task]</a>{% endif %}</li>
{% endfor %}
</ul>
<hr><p>Support creator: send donation to card <b>5168 7520 2848 5707</b></p>
</body></html>
'''

REGISTER_HTML = '''
<h1>Register</h1>
<form method="post">
  Username: <input name="username"><br>
  Password: <input name="password" type="password"><br>
  <input type="submit" value="Register">
</form>
'''

LOGIN_HTML = '''
<h1>Login</h1>
<form method="post">
  Username: <input name="username"><br>
  Password: <input name="password" type="password"><br>
  <input type="submit" value="Login">
</form>
'''

CREATE_TASK_HTML = '''
<h1>Create a Task</h1>
<form method="post">
  Title: <input name="title"><br>
  Description: <input name="description"><br>
  Reward: <input name="reward"><br>
  <input type="submit" value="Create Task">
</form>
'''

PROFILE_HTML = '''
<h1>My Profile</h1>
<p>Username: {{username}}</p>
<p>Balance: ${{balance}}</p>
<a href="/">← Back</a>
'''

if __name__ == '__main__':
    init_db()
    app.run(debug=True)

@app.route('/admin')
def admin():
    if session.get('username') != 'admin':
        return redirect(url_for('home'))
    conn = sqlite3.connect('taskboost.db')
    c = conn.cursor()
    c.execute("SELECT * FROM users")
    users = c.fetchall()
    c.execute("SELECT * FROM tasks")
    tasks = c.fetchall()
    conn.close()
    return render_template_string(ADMIN_HTML, users=users, tasks=tasks)

@app.route('/withdraw', methods=['GET', 'POST'])
def withdraw():
    if 'username' not in session:
        return redirect(url_for('login'))
    if request.method == 'POST':
        amount = float(request.form['amount'])
        conn = sqlite3.connect('taskboost.db')
        c = conn.cursor()
        c.execute("SELECT balance FROM users WHERE username=?", (session['username'],))
        balance = c.fetchone()[0]
        if balance >= amount:
            c.execute("UPDATE users SET balance = balance - ? WHERE username=?", (amount, session['username']))
            conn.commit()
            message = f"Withdrawal request of ${amount:.2f} sent! 💸"
        else:
            message = "Not enough balance!"
        conn.close()
        return render_template_string(WITHDRAW_HTML, message=message)
    return render_template_string(WITHDRAW_HTML, message="")

WITHDRAW_HTML = '''
<h1>Withdraw Funds</h1>
<form method="post">
  Amount: <input name="amount" type="number" step="0.01" min="1"><br>
  <input type="submit" value="Request Withdrawal">
</form>
<p>{{message}}</p>
<a href="/">← Back</a>
'''

ADMIN_HTML = '''
<h1>Admin Panel</h1>
<h2>Users</h2>
<ul>
{% for user in users %}
  <li>{{user[1]}} - ${{user[3]}}</li>
{% endfor %}
</ul>
<h2>All Tasks</h2>
<ul>
{% for task in tasks %}
  <li>{{task[1]}} - ${{task[3]}} - Owner: {{task[4]}} - Done by: {{task[5]}}</li>
{% endfor %}
</ul>
<a href="/">← Back</a>
'''

PROFILE_HTML += '<br><a href="/withdraw">Withdraw Funds</a>'

