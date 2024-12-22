# confession-page
from flask import Flask, render_template, request, redirect
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///confessions.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)

# Define the database model
class Confession(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), nullable=False)
    message = db.Column(db.Text, nullable=False)

# Create the database
with app.app_context():
    db.create_all()

@app.route('/')
def home():
    confessions = Confession.query.all()
    return render_template('home.html', confessions=confessions)

@app.route('/add', methods=['POST'])
def add_confession():
    username = request.form['username']
    message = request.form['message']
    
    if username and message:
        new_confession = Confession(username=username, message=message)
        db.session.add(new_confession)
        db.session.commit()
        return redirect('/')
    return "Error: Missing username or message", 400

@app.route('/delete/<int:id>')
def delete_confession(id):
    confession = Confession.query.get_or_404(id)
    db.session.delete(confession)
    db.session.commit()
    return redirect('/')

if __name__ == '__main__':
    app.run(debug=True)
