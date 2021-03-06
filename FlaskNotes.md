# Notes on Flask

(helpful when adding new functionality)

## Requirements

In addition to basic flask, this requires:
flask-login flask-sqlalchemy flask-migrate python-wtf Flask-Mail dateutil

I like to install from Debian packages when I can, so I'll get
security updates automatically. But python-wtf and Flask-Mail
aren't available on some versions of Debian, e.g.
[Bug 912069](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=912069):
watch [the Debian excuse page](https://qa.debian.org/excuses.php?package=flask-wtf) for updates.

I created a python3 venv:
python3 -m venv --system-site-packages python3-venv

Enabled it:
source python3-venv/bin/activate

Installed wtf:
pip3 install python-wtf
pip3 install Flask-WTF

Then inserted the relevant path into the .wsgi file apache will use:
sys.path.insert(0, '/path/to/python3-venv/lib/python3.5/site-packages/')

# Templates

Any variables that you want a template to be able to access with {{ var }}
should be passed as render_template(..., var=value) otherwise the template
won't see them (except certain global variables, maybe).

# Form submission:

To add a new form, reate a URL in routes.py, a class in forms.py,
and a template in templates/.

Validation happens in forms.py.
Any code that does something useful with the submitted data needs
to go in routes.py.
routes.py is also the place to clear the form field if you go back to
the same form after submitting.

# Database Upgrade and Downgrade Workflow

- Update models.db
- flask db migrate
- review the migration file to make sure it looks right
- flask db upgrade


## Migrations: adding/dropping tables in sqlite3

sqlite3 doesn't have any support for dropping tables.
Alembic has a way to do that with batch operations but it doesn't
do it by default. So if your migration has a line like:
    op.drop_column('user', 'bills_seen')
you need to edit it to use batch operations instead:
    with op.batch_alter_table('user') as batch_op:
        batch_op.drop_column('bills_seen')
None of the flask db tutorials I've seen mentioned this
even though most of them use sqlite.


## Fixing a migration problem

I made a migration, did flask db migrate but not flask db upgrade,
then decided I didn't want that migration after all and removed the
file. This makes alembic unhappy. But you can set the current version:
flask db stamp head

# Restart apache:

apachectl -k graceful


## Debugging with the Interactive Flask Shell

% export FLASK_APP=run_billtracker.py
% flask shell
>>> from billtracker.models import User
>>> allusers = User.query.all()
>>> u = User.query.filter_by(username='mary').first()
>>> u.bills
[Bill SB101 19, Bill HB55 20, ...]
>>> [b for b in u.bills if b.year == '20']
[Bill HB55 20, Bill SM9 20, Bill HB94 20, Bill SB17 20, Bill SB20 20]
>>> b =  Bill.query.filter_by(billno='HB55', year='20').first()
>>> b.users_tracking()
