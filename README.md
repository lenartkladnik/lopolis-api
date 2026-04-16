# lopolis-api

This is an API wrapper for the lopolispro.si school management platform

## Example usage
```python
import lopolis

session = lopolis.Session(username, password)

session.api # This contains most of the existing API routes

# Get the timetable (Urnik) as a dict
print(session.api.get_timetable())

# Make a custom request
session.request("/") # https://starsi.lopolispro.si/
session.request("/inbox/messages", subdomain="sporocila") # https://sporocila.lopolispro.si/inbox/messages

# When you get logged out you don't have to
# create a new Session object, instead just
# call Session.refresh, which will generate
# a new session cookie
session.refresh()
```

## Prerequisites
- python (no extra packages)
- curl (installed and on PATH)
- LopolisPro account username and password

## Documentation

### Session class

Constructor
```python
Session(username: str, password: str)
```

Parameters:

    username (str): Your Lopolis Pro username
    password (str): Your Lopolis Pro password

Raises:

    LoginError: If authentication fails (invalid credentials, network issues, or OTP verification failure)

Example:
```python
try:
    session = Session("my_username", "my_password")
except LoginError as e:
    print(f"Login failed: {e}")
```

### Methods
```python
request(
    path: str,
    subdomain: str = "starsi",
    data: Any = [],
    method: str = "GET",
    headers: dict = {},
    text: bool = False
) -> urllib.response.addinfourl | str
```

> Internal method for making HTTP requests. Most users should use the session.api methods instead.

Parameters:
```python
path (str): API endpoint (eg.: "/api/chat_unread_count" or "evaluations"). Leading/trailing slashes are optional.
subdomain (str, default: "starsi"): Subdomain for the request (only alphabetic characters are preserved)
data (Any, default: []): Request payload (converted to JSON automatically)
method (str, default: "GET"): HTTP method ("GET", "POST", etc.)
headers (dict, default: {}): Additional headers
text (bool, default: False): If True, returns response as string, otherwise returns response object
```
Returns:
```
if text=True: Response body as a string
if text=False: urllib.response.addinfourl object
```
Example:
```python
response = session.request("/api/banners", text=True)
```

---

```python
refresh() -> None
```

Refreshes the session cookie. Call this if your session has expired.

Example:
```python
session.refresh()
```

---

API Class

Provides high-level methods to fetch and modify lopolis data. Access via ```Session.api```

Methods
```python
get_chat_unread_count() -> Any
```

Fetches the number of unread chat messages.

Returns:
```
{'ok': <True or False>, 'value': <unread_chat_messages_number: int>}
```

Example:
```python
response = session.api.get_chat_unread_count()
if response["ok"]:
  print(f"Unread messages: {response['value']}")
```

---

```python
get_evaluations() -> Any
```

Retrieves all passed and upcoming exams.

Returns:

    JSON object with ok and value fields

Example:
```python
response = session.api.get_evaluations()
if response["ok"]:
  for evaluation in response["value"]["futureEvaluations"]:
    print(evaluation["teacher"], evaluation["title"])
```

---

```python
get_banners() -> Any
```

Unknown.

Returns:

    JSON object with ok and value fields

Example:
```python
banners = session.api.get_banners()
```

---

```python
get_absences() -> Any
```

Retrieves and absence information.

Returns:
    
    JSON object (without ok and value fields)

Example:
```python
response = session.api.get_absences()
print(response["summary"]["excusedHours"])
```

---

```python
get_timetable(date: datetime = datetime.today()) -> Any
```

Fetches the school timetable for week that the specified date falls into.

Parameters:

    date (datetime, default: today's date): The date to use for retrieving the timetable

Returns:

    JSON object with ok and value fields

Example:
```python
from datetime import datetime, timedelta

# Get today's timetable
today_timetable = session.api.get_timetable()

# Get timetable for a specific date
next_week = datetime.today() + timedelta(days=7)
future_timetable = session.api.get_timetable(date=next_week)
```

---

```python
get_meals_menu(date: datetime) -> Any
```

Retrieves the meal menu for a specific date.

Parameters:

    date (datetime): The date for which to retrieve meal options

Returns:

    JSON object (without ok and value fields)

Example:
```python
from datetime import datetime

menu = session.api.get_meals_menu(date=datetime(2026, 4, 20))
```

---

```python
set_meals_menu(
    date: datetime,
    meal_id: int,
    meal_type: str = "afternoon_snack"
) -> Any
```

Selects a meal option for a specific date and meal type.

Parameters:

    date (datetime): The date to set the meal for
    meal_id (int): The ID of the meal to select (obtained from get_meals_menu())
    meal_type (str, default: "afternoon_snack"): The type of meal (e.g., "lunch", "afternoon_snack")

> Tip: meal_id is found in the id field of the return object of Session.api.get_meals_menu

Returns:
    
    Response data confirming the meal selection

Example:
```python
from datetime import datetime

result = session.api.set_meals_menu(
    date=datetime(2026, 4, 20),
    meal_id=2901,
    meal_type="afternoon_snack"
)
```
