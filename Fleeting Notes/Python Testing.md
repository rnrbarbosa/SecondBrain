


# Application

Let's assume an application that interacts with an External API, such as Github API.
This application aims to with an username input to obtain the name of the user:

```jupyter
from requests import Session
from rich import print

def get_user_name(user, session):
  """Given a Gihub User, get his name"""
  url = f"https://api.github.com/users/{user}"
  response = session.get(url)
  return response

session = Session()
response = get_user_name("rnrbarbosa", session)
print(response)
```


# Testing Double

Testing Double is an object that has an interface similar to the original object that is an external dependency for example.

Let's an example of a testing double, where we try to double the _Session_ object which an external dependency:

```jupyter
from requests import Session
from rich import print

def get_user_name(user, session):
  """Given a Gihub User, get his name"""
  url = f"https://api.github.com/users/{user}"
  response = session.get(url)
  return response

class FakeSession:
  def get(self, url):
    return "TESTING"
    
session = FakeSession()

response = get_user_name("rnrbarbosa", session)

print(response)
```

Let's expand this example where the response is actually a JSON response:

```jupyter
from requests import Session
from rich import print

def get_user_name(user, session):
  """Given a Gihub User, get his name"""
  url = f"https://api.github.com/users/{user}"
  response = session.get(url)
  json_response = response.json()
  return json_response['name']

session = Session()
response = get_user_name("rnrbarbosa", session)
print(response)
```

Let's write a testing double to provide a JSON response:

```jupyter
class FakeSession: 
	def get(self, url): 
		return FakeResponse()

class FakeResponse:
	def json(self):
		return {"name": "User McTest"}

session = FakeSession()

response = get_user_name("rnrbarbosa", session)
print(response)
```

# Explore Mock

Let's understanding how mocks works.

Let's start an example of creating a Mock that return variables that are defined at mock creation:

```jupyter
from unittest.mock import Mock

m = Mock(a=1, b=1)
print(m.a)
print(m.b)
```

Let's make this more generic and create a Mock that no matter what it returns a value:

```jupyter
from unittest.mock import Mock

m = Mock(return_value=5)
print(m)
print(m())
```

Let's make the return the value, independently of the arguments:

```jupyter
from unittest.mock import Mock

m = Mock(return_value=1)
 
print(m(var1=5))
print(m(var1=5, var2=7))
```

For more dynamic response let's explore the side_effect parameter.

```jupyter
from unittest.mock import Mock 

m = Mock(side_effect=range(3))

print(f"Iteration 1: {m()}")
print(f"Iteration 2: {m()}")
print(f"Iteration 3: {m()}")
```

But if we pass the iteration limit, it throws an exception:
```jupyter
from unittest.mock import Mock 

m = Mock(side_effect=range(3))

print(f"Iteration 1: {m()}")
print(f"Iteration 2: {m()}")
print(f"Iteration 3: {m()}")
print(f"Iteration 4: {m()}")
```

Speaking about exceptions, we can also simulate with the Mock an exception response

```jupyter
from unittest.mock import Mock 


m = Mock(side_effect=ValueError)

print(m())
```


Let's combine different response depending on the iteration:

```jupyter
from unittest.mock import Mock 
m = Mock(side_effect=[1, "Response", ValueError])

print(f"Iteration 1: {m()}")
print(f"Iteration 2: {m()}")
print(f"Iteration 3: {m()}")
```


Let's mock now a function:

```jupyter
from unittest.mock import Mock

def fake_function(a):
	return a * 5

m = Mock(side_effect=fake_function)

m(2)
```

# Using Mock on the Application
Let's review our basic testing:

```jupyter
def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get(url)
	json_response = response.json()
	return json_response['name']
	
class FakeSession():
	def get(self, url):
		return FakeResponse()

class FakeResponse:
	def json(self):
		return {"name": "Mr. User McTest"}
  
session = FakeSession()  
get_user_name("rnrbarbosa", session)
```

and now let's use unittest.mock

```jupyter
from unittest.mock import Mock

def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get(url)
	json_response = response.json()
	return json_response['name']


fake_response = Mock()
fake_response.json = Mock(return_value={"name": "Mr. User McTest"})

fake_session = Mock()
fake_session.get = Mock(return_value=fake_response)

assert "Mr. User McTest" == get_user_name("rnrbarbosa", fake_session), "Failed"
```

# Complex Mocks

Let's mock the response value of an object method:

```jupyter
from unittest.mock import Mock

response_payload = {"name": "Mr. User McTest"}
fake_session = Mock(**{"get.return_value.json.return_value": response_payload})

fake_session.get().json()

```

Another way:
```jupyter
from unittest.mock import Mock

response_payload = {"name": "Mr. User McTest"}

fake_session = Mock()
fake_session.get.return_value.json.return_value  = response_payload

fake_session.get().json()
```


# Magic Mocks

This will fail, unless you explicitly mock the response
```jupyter
from unittest.mock import Mock, MagicMock

m = Mock()
m[0]
```

using MagicMock, it does not fail:

```jupyter
from unittest.mock import Mock, MagicMock

mm = MagicMock()
mm[0]
```
even with:

```jupyter
from unittest.mock import Mock, MagicMock

mm = MagicMock()
mm.attr.fn() + 1
```

# Validation of Mocks

Let's run some mock calls and examine what happened on the calls:

```jupyter
from unittest.mock import Mock, ANY

m = Mock()
m(1)
m(2,3)
m(4,5,6,kw=True)

# How many time the Mock was called?
print(m.call_count)

# List all args that were passed on ALL calls
print(m.call_args_list)

# List all args that were passed on LAST call
print(m.call_args)

# Check if last call was with arg: (4,5,6,kw=True)
m.assert_called_with(4,5,6,kw=True)

# Check if there was any call with arg: (2,3) 
m.assert_any_call(2,3)

# Check if any call had (4,5,ANY, kw=ANY)
m.assert_any_call(4,5,ANY,kw=ANY)
```


# Mock Spies
How to see what happen on an application without changing its behavior:

```jupyter
from requests import Session
from unittest.mock import Mock

def get_user_name(user, session):
  """Given a Gihub User, get his name"""
  url = f"https://api.github.com/users/{user}"
  response = session.get(url)
  return response

session = Session()

# Add spy on the middle :-)
spy = Mock(wraps=session)

response = get_user_name("rnrbarbosa", spy)
print(response)

# Check what args were passed to session.get()
print(f"Args passed to session.get: {spy.get.call_args}")

# Assert a value passed to session.get()
if spy.get.assert_called_with("https://api.github.com/users/rnrbarbosa") == None:
	print("Test Passed")

```

# Restricted Mocks


```jupyter
from unittest.mock import MagicMock

def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get(url)
	json_response = response.json()
	return json_response['name']


response_payload = {"name": "Mr. User McTest"}

fake_session = MagicMock()
fake_session.get.return_value.json.return_value = response_payload

if "Mr. User McTest" == get_user_name("rnrbarbosa", fake_session):
	print("TEST PASS")
else:
	print("TEST FAIL")
```

Let's suppose that session.get is changed to session.get_url:

```jupyter
from unittest.mock import MagicMock

def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get_url(url)
	json_response = response.json()
	return json_response['name']


response_payload = {"name": "Mr. User McTest"}

fake_session = MagicMock()
fake_session.get.return_value.json.return_value = response_payload

if "Mr. User McTest" == get_user_name("rnrbarbosa", fake_session):
	print("TEST PASS")
else:
	print("TEST FAIL")
```

Test failed, but we don't know where! What to do? Mock all interactions inside the function get_user_name?

## Spec
Best option is to use Spec

```jupyter
from unittest.mock import MagicMock
from requests import Session

def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get_url(url)
	json_response = response.json()
	return json_response['name']


response_payload = {"name": "Mr. User McTest"}

fake_session = MagicMock(spec=Session)
fake_session.get.return_value.json.return_value = response_payload

if "Mr. User McTest" == get_user_name("rnrbarbosa", fake_session):
	print("TEST PASS")
else:
	print("TEST FAIL")
```

What about if want to make sure that the Mock respects the original, and now new methods can be added on the Mock? We can achieve that with seal()

```jupyter
from unittest.mock import MagicMock, seal
from requests import Session

def get_user_name(user, session):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	response = session.get_url(url)
	json_response = response.json()
	return json_response['name']


response_payload = {"name": "Mr. User McTest"}

fake_session = MagicMock(spec=Session)

## Let's prevent the original api from being changed by the Mck
seal(fake_session)

## ****** CHANGE OF THE ORIGINAL API ******"
fake_session.get_url.return_value.json.return_value = response_payload

if "Mr. User McTest" == get_user_name("rnrbarbosa", fake_session):
	print("TEST PASS")
else:
	print("TEST FAIL")
```

We have recieved an alert that the Mock is trying to change the original API of Session by adding a session.get_url that DOES NOT EXIST.

So, seal() prevents further modification:

```jupyter
from unittest.mock import Mock, seal

m = Mock()
m.attr1.attr11 = 1
m.atrr2 = Mock(name="Test")

##  ****** Let's seal our Mock here
seal(m)

# Let's try to modify the Mock
m.atrr3 = 4
```


# Understanding Patch

```jupyter
import requests
print(requests.Session)
```

So, it prints that requests.Session is a class. Now let's patch:

```jupyter
from unittest.mock import patch
import requests

print("Before Patch", requests.Session)

with patch("requests.Session") as m:
	print("Inside Patch", requests.Session)
print("After Patch", requests.Session)
```

Hum, it's not class when patched, but a Mock! And it gets back to a class after the patch.

Let's add a new import "from requests import Session", and see what happens:

```jupyter
from unittest.mock import patch
import requests
from requests import Session

print("Before Patch", requests.Session)

with patch("requests.Session") as m:
	print("Inside Patch", requests.Session)
print("After Patch", requests.Session)
```



# Mocking Class Instances

In this case we instantiate an object requests.Session(), so we need to not forget to call mock.result_value which the object instantiated:

```jupyter
from unittest.mock import patch, seal
import requests

def get_user_name(user):
	"""Given a Gihub User, get his name"""
	url = f"https://api.gihub.com/users/{user}"
	session = requests.Session()
	response = session.get(url)
	json_response = response.json()
	return json_response['name']


response_payload = {"name": "Mr. User McTest"}

with patch("requests.Session") as mock:
	mock.return_value.get.return_value.json.return_value = response_payload
	seal(mock)
	print(get_user_name("rnrbarbosa"))
```


Patching Objects

```jupyter
class Duck:
	def walk(self):
		return "Walking..."
	def fly(self):
		return "Flying..."

duck1 = Duck()
duck2 = Duck()

from unittest.mock import patch

with patch.object(duck1, "fly") as mock:
	mock.return_value = "Rocket Flying..."
	print(duck1.walk())
	print(duck1.fly()) # ONLY this is patched
	print(duck2.walk())
	print(duck2.fly())
```

Let's inject another method into a single object:

```jupyter
class Duck:
	def walk(self):
		return "Walking..."
	def fly(self):
		return "Flying..."

duck1 = Duck()
duck2 = Duck()

from unittest.mock import patch

with patch.object(duck1, "dive", lambda: "Diving on Ocean...", create=True) as mock:
	print(duck1.walk())
	print(duck1.fly())
	print(duck1.dive()) # Added another method
	print(duck2.walk())
	print(duck2.fly())
```


Patching Dicts
