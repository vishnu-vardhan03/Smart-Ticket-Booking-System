from flask import Flask, render_template, request, redirect, url_for, session
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.secret_key = "secret123"  # for session encryption
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
db = SQLAlchemy(app)

# User table
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    email = db.Column(db.String(100), unique=True)
    password = db.Column(db.String(100))

@app.before_request
def create_table():
    db.create_all()

@app.route('/')
def home():
    if "user_id" in session:
        user = User.query.get(session["user_id"])
        return render_template("booking.html", name=user.name)
    return render_template("index.html")

@app.route('/register', methods=["GET", "POST"])
def register():
    if request.method == "POST":
        name = request.form["name"]
        email = request.form["email"]
        password = request.form["password"]
        user = User(name=name, email=email, password=password)
        db.session.add(user)
        db.session.commit()
        return redirect(url_for("login"))
    return render_template("register.html")

@app.route('/login', methods=["GET", "POST"])
def login():
    if request.method == "POST":
        email = request.form["email"]
        password = request.form["password"]
        user = User.query.filter_by(email=email, password=password).first()
        if user:
            session["user_id"] = user.id
            return redirect(url_for("home"))
        else:
            return render_template("login.html", error="Invalid credentials")
    return render_template("login.html")

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for("home"))

@app.route('/book_ticket', methods=["POST"])
def book_ticket():
    if "user_id" not in session:
        return redirect(url_for("login"))
    user = User.query.get(session["user_id"])
    return redirect(url_for("payment", name=user.name))

@app.route('/payment')
def payment():
    name = request.args.get("name", "")
    return render_template("payment.html", name=name)

if __name__ == '__main__':
    app.run(debug=True)
