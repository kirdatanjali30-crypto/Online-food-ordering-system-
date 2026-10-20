from flask import Flask, render_template_string, request, redirect, session
import sqlite3

app = Flask(__name__)
app.secret_key = "fitmeal_secret_key"

def get_db():
    conn = sqlite3.connect("fitmeal.db")
    conn.row_factory = sqlite3.Row
    return conn

def create_tables():
    conn = get_db()
    conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE,
            password TEXT
        )
    """)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS foods (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            category TEXT,
            price INTEGER,
            calories INTEGER,
            protein INTEGER
        )
    """)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS orders (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT,
            food_name TEXT,
            price INTEGER
        )
    """)

    count = conn.execute(
        "SELECT COUNT(*) FROM foods"
    ).fetchone()[0]

    if count == 0:
        foods = [
            ("Jowar Bhakri Meal", "Homemade", 80, 350, 8),
            ("Dal Rice", "Homemade", 90, 400, 12),
            ("Sprouts Salad", "Diet", 60, 180, 10),
            ("Paneer Bowl", "Fitness", 120, 320, 20),
            ("Vegetable Khichdi", "Diet", 70, 280, 9),
            ("Homemade Chapati Meal", "Homemade", 100, 380, 12),
            ("Boiled Egg Meal", "Fitness", 90, 250, 18),
            ("Healthy Poha", "Breakfast", 50, 220, 6)
        ]

        conn.executemany("""
            INSERT INTO foods
            (name, category, price, calories, protein)
            VALUES (?, ?, ?, ?, ?)
        """, foods)

    conn.commit()
    conn.close()

HOME = """
<!DOCTYPE html>
<html>
<head>
<title>FitMeal</title>
<style>
body {
    font-family: Arial;
    background: #f1f8f1;
    margin: 0;
    color: #26352b;
}
header {
    background: #167447;
    color: white;
    padding: 22px;
    text-align: center;
}
nav {
    background: #105b36;
    padding: 14px;
    text-align: center;
}
nav a {
    color: white;
    margin: 8px;
    text-decoration: none;
}
.hero {
    text-align: center;
    padding: 35px 12px;
}
button {
    background: #167447;
    color: white;
    padding: 10px 18px;
    border: none;
    border-radius: 6px;
}
.container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
}
.card {
    background: white;
    width: 220px;
    margin: 12px;
    padding: 18px;
    border-radius: 12px;
    box-shadow: 0 2px 8px #cccccc;
}
.price {
    color: #167447;
    font-weight: bold;
}
footer {
    background: #105b36;
    color: white;
    text-align: center;
    padding: 18px;
    margin-top: 30px;
}
</style>
</head>
<body>

<header>
<h1>🍱 FitMeal</h1>
<p>Healthy Food, Homemade Love!</p>
</header>

<nav>
<a href="/">Home</a>
<a href="/menu">Food Menu</a>
<a href="/register">Register</a>
<a href="/login">Login</a>
<a href="/subscription">Tiffin Plans</a>
<a href="/orders">My Orders</a>
<a href="/admin">Admin</a>
</nav>

<div class="hero">
<h2>Welcome to FitMeal!</h2>
<p>Fresh homemade food and healthy meals at your fingertips.</p>
<a href="/menu"><button>Explore Food</button></a>
</div>

<h2 style="text-align:center">Our Special Categories</h2>

<div class="container">
<div class="card">
<h3>🏠 Homemade Food</h3>
<p>Bhakri, Chapati, Dal Rice and Bhaji.</p>
</div>
<div class="card">
<h3>🥗 Diet Food</h3>
<p>Balanced meals and healthy salads.</p>
</div>
<div class="card">
<h3>💪 Fitness Food</h3>
<p>Protein-rich meal options.</p>
</div>
<div class="card">
<h3>📅 Tiffin Plans</h3>
<p>Choose a 7-day or 30-day plan.</p>
</div>
</div>

<footer>
<p>FitMeal | Healthy & Homemade Food</p>
</footer>

</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HOME)

@app.route("/menu")
def menu():
    category = request.args.get("category", "")
    conn = get_db()

    if category:
        foods = conn.execute(
            "SELECT * FROM foods WHERE category = ?",
            (category,)
        ).fetchall()
    else:
        foods = conn.execute(
            "SELECT * FROM foods"
        ).fetchall()

    conn.close()

    return render_template_string("""
    <h1>🍱 FitMeal Food Menu</h1>
    <p>
    <a href="/menu">All</a> |
    <a href="/menu?category=Homemade">Homemade</a> |
    <a href="/menu?category=Diet">Diet</a> |
    <a href="/menu?category=Fitness">Fitness</a> |
    <a href="/menu?category=Breakfast">Breakfast</a>
    </p>
    <style>
    body {font-family:Arial;background:#f1f8f1;padding:20px}
    .card {background:white;padding:15px;margin:12px;
    border-radius:10px;max-width:350px}
    a {color:#167447}
    button {background:#167447;color:white;padding:9px;border:0}
    </style>
    {% for f in foods %}
    <div class="card">
    <h3>{{ f['name'] }}</h3>
    <p>Category: {{ f['category'] }}</p>
    <p>Calories: {{ f['calories'] }} kcal (approx.)</p>
    <p>Protein: {{ f['protein'] }} g (approx.)</p>
    <p>Price: ₹{{ f['price'] }}</p>
    <form action="/order/{{ f['id'] }}" method="post">
    <button type="submit">Order Now</button>
    </form>
    </div>
    {% endfor %}
    <a href="/">Back Home</a>
    """, foods=foods)

@app.route("/register", methods=["GET", "POST"])
def register():
    message = ""

    if request.method == "POST":
        username = request.form["username"].strip()
        password = request.form["password"]

        if not username or not password:
            message = "Enter username and password."
        else:
            conn = get_db()
            try:
                conn.execute(
                    "INSERT INTO users (username, password) VALUES (?, ?)",
                    (username, password)
                )
                conn.commit()
                message = "Registration successful! Please login."
            except sqlite3.IntegrityError:
                message = "Username already exists."
            finally:
                conn.close()

    return render_template_string("""
    <h1>Customer Registration</h1>
    <p>{{ message }}</p>
    <form method="post">
    <input name="username" placeholder="Username" required>
    <br><br>
    <input name="password" type="password"
    placeholder="Password" required>
    <br><br>
    <button>Register</button>
    </form>
    <a href="/login">Login</a> |
    <a href="/">Home</a>
    """, message=message)

@app.route("/login", methods=["GET", "POST"])
def login():
    message = ""

    if request.method == "POST":
        username = request.form["username"].strip()
        password = request.form["password"]

        conn = get_db()
        user = conn.execute(
            "SELECT * FROM users WHERE username = ? AND password = ?",
            (username, password)
        ).fetchone()
        conn.close()

        if user:
            session["username"] = username
            return redirect("/")
        message = "Invalid username or password."

    return render_template_string("""
    <h1>Customer Login</h1>
    <p>{{ message }}</p>
    <form method="post">
    <input name="username" placeholder="Username" required>
    <br><br>
    <input name="password" type="password"
    placeholder="Password" required>
    <br><br>
    <button>Login</button>
    </form>
    <a href="/register">Register</a> |
    <a href="/">Home</a>
    """, message=message)

@app.route("/order/<int:food_id>", methods=["POST"])
def order(food_id):
    if "username" not in session:
        return redirect("/login")

    conn = get_db()
    food = conn.execute(
        "SELECT * FROM foods WHERE id = ?", (food_id,)
    ).fetchone()

    if food:
        conn.execute("""
            INSERT INTO orders (username, food_name, price)
            VALUES (?, ?, ?)
        """, (
            session["username"],
            food["name"],
            food["price"]
        ))
        conn.commit()

    conn.close()
    return redirect("/orders")

@app.route("/orders")
def orders():
    if "username" not in session:
        return redirect("/login")

    conn = get_db()
    rows = conn.execute(
        "SELECT * FROM orders WHERE username = ? ORDER BY id DESC",
        (session["username"],)
    ).fetchall()
    conn.close()

    return render_template_string("""
    <h1>My Orders</h1>
    {% for o in rows %}
    <div style="padding:12px;background:#e8f5e9;margin:10px">
    <p>Order #{{ o['id'] }}</p>
    <p>{{ o['food_name'] }} - ₹{{ o['price'] }}</p>
    </div>
    {% else %}
    <p>No orders yet.</p>
    {% endfor %}
    <a href="/menu">Order Food</a> |
    <a href="/logout">Logout</a>
    """, rows=rows)

@app.route("/logout")
def logout():
    session.clear()
    return redirect("/")

@app.route("/subscription", methods=["GET", "POST"])
def subscription():
    message = ""

    if request.method == "POST":
        if "username" not in session:
            return redirect("/login")

        plan = request.form["plan"]
        message = (
            "Your " + plan +
            " plan selection was received. Contact the admin to confirm."
        )

    return render_template_string("""
    <h1>Monthly Tiffin Subscription</h1>
    <p>Choose your homemade meal plan.</p>
    <form method="post">
    <select name="plan">
    <option value="7-Day">7-Day Plan</option>
    <option value="30-Day">30-Day Plan</option>
    </select>
    <button>Choose Plan</button>
    </form>
    <p>{{ message }}</p>
    <a href="/">Home</a>
    """, message=message)

@app.route("/admin")
def admin():
    return """
    <h1>FitMeal Admin Dashboard</h1>
    <p>Food Menu and Order Management</p>
    <a href="/menu">View Food Menu</a><br><br>
    <a href="/orders">View Current User Orders</a><br><br>
    <a href="/">Home</a>
    """

create_tables()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=False)
