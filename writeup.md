#Santhacklaus 2019 - digipad simulator (200 points):

[https://ctf.santhacklaus.xyz](*https://ctf.santhacklaus.xyz*)

##Challenge description

This challenge was about digipads. You'd connect to a socket with netcat and be presented with a digipad, and you are supposed to find the correct combination to unlock it. Here is the prompt you are greeted with:

xxxx

And here are a few reminders about digipads:

1. Heavy use causes the numbers to fade away.
2. You don't always need to press "enter" (or whatever button) after you enter a combination to validate it: some digipads will unlock right after you typed the last number of the correct combination.




##Approach to solving it

xxxx something new


##The code

Here is the code:

```python
#!/usr/bin/python

import socket
from time import sleep

def deBruijn(n, k):
	a = [ 0 ] * (n + 1)
	
	def gen(t, p):
		if t > n:
			for v in a[1:p + 1]:
			  yield v
		else:
			a[t] = a[t - p]
			
			for v in gen(t + 1, p):
			  yield v
			
			for j in xrange(a[t - p] + 1, k):
				a[t] = j
				for v in gen(t + 1, t):
				  yield v
	
	return gen(1, 1)
 
class Netcat:
	def __init__(self, ip, port):
		self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		self.socket.connect((ip, port))

	def read(self, length = 1024):
		return self.socket.recv(length)
 
	def write(self, data):
		self.socket.send(data)
	
	def close(self):
		self.socket.close()

while True:
	nc = Netcat('46.30.204.44', 4000)

	sleep(0.1)
	a = nc.read()
	print(a)

	b = a.split("Better")[0]

	numbers = ""

	for i in "0123456789":
		if i not in b:
			numbers += i

	if(len(numbers) == 4):
		nc.close()
		print("\033[31mThe De Bruijn sequence is too long for the digipad, it will respond with \"String is too long..\", you can test it if you like\033[0m")
		continue

	print("\033[32mThe numbers that are all used up on the digipad are: " + numbers + "\033[0m")

	print("\033[32mFull De Bruijn sequence: " + "".join([numbers[x] for x in deBruijn(4, len(numbers))]) + "\033[0m")
	nc.write("".join([numbers[x] for x in deBruijn(4, len(numbers))]) + "\n")

	sleep(0.5)
	print(nc.read())
	nc.close()
	break
```
