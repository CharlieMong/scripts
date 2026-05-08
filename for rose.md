### Integration Steps
1. Install dependencies

```
pip install pyotp "qrcode[pil]"
```

2. Run the DB migration — add to the bottom of database.py or call from app.py:

```
from mfa import init_db_mfa
init_db_mfa(DATABASE)
```

3. Register the blueprint in app.py:

```
from mfa_routes import mfa_bp
app.register_blueprint(mfa_bp)
```

4. Hook into the login route in routes/auth_routes.py — after the password check succeeds, check if the user has MFA enabled before setting session["user_id"]:

```
from mfa import get_mfa_state, start_mfa_challenge
```

```
# inside the successful-login branch:
state = get_mfa_state(DATABASE, user["id"])
if state["enabled"]:
    start_mfa_challenge(user["id"])
    return redirect(url_for("mfa.verify", next=next_url))
else:
    session["user_id"] = user["id"]
    session.permanent = True
    return redirect(next_url)
```

5. Add setup/disable links to the user account page:

```
<a href="{{ url_for('mfa.setup') }}">Enable 2FA</a>
<!-- or if already enabled: -->
<a href="/mfa/disable">Disable 2FA</a>
```

The module is fully compatible with Google Authenticator, Authy, and any RFC 6238 TOTP app. It uses a ±30 s clock window to handle minor time skew.

### Just admin accounts

Three small changes to what was already built:

1. mfa_routes.py — guard the setup route to admins only

```
Replace @login_required with @admin_required on the setup route:

from auth import login_required, admin_required, current_user

@mfa_bp.route("/setup", methods=["GET", "POST"])
@admin_required   # was @login_required
def setup():
    ...
```
2. routes/auth_routes.py — only trigger the MFA challenge for admins

In the login route, add an is_admin check before routing to the TOTP step:

```
state = get_mfa_state(DATABASE, user["id"])
if user["is_admin"] and state["enabled"]:
    start_mfa_challenge(user["id"])
    return redirect(url_for("mfa.verify", next=next_url))
else:
    session["user_id"] = user["id"]
    session.permanent = True
    return redirect(next_url)
```

3. mfa_routes.py — guard the disable route the same way

```
@mfa_bp.route("/disable", methods=["POST"])
@admin_required   # was @login_required
def disable():
    ...
```
That's it. Non-admin users will never see the setup page (403 via admin_required), and even if an admin somehow had mfa_enabled=1 set on a non-admin account, the login route won't challenge them for a code.

If you want to go further and require MFA for all admins (force setup on first admin login before they can access anything), add this check inside admin_required in auth.py:

```
from mfa import get_mfa_state
from config import DATABASE

def admin_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        user = current_user()
        if not user:
            return redirect(url_for("auth.login", next=request.path))
        if not user["is_admin"]:
            abort(403)
        state = get_mfa_state(DATABASE, user["id"])
        if not state["enabled"]:
            flash("Admins must enable two-factor authentication.", "warning")
            return redirect(url_for("mfa.setup"))
        return f(*args, **kwargs)
    return decorated
```

This enforces MFA as a prerequisite for every admin route rather than just at login time.
