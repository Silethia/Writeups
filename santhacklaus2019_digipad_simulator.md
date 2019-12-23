# Santhacklaus 2019 - digipad simulator (200 points):

[https://ctf.santhacklaus.xyz](*https://ctf.santhacklaus.xyz*)

## The challenge

This challenge was about digipads. You'd connect to a socket with netcat and be presented with a digipad, and you are supposed to find the correct combination to unlock it. Here is the prompt you are greeted with:

```
             ,---,---,---,
             | 1 |   |   |
             '---,---,---'
             | 4 | 5 | 6 |
             '---,---,---'
             | 7 | 8 | 9 |
             '---,---,---'
             | * |   | # |
             '---'---'---'
Better be fast, you have 2 seconds.
Enter the 4 digit code :
```

And here are a few reminders about digipads:

1. Heavy use causes the numbers to fade away.
2. You don't always need to press "enter" (or whatever button) after you enter a combination to validate it: some digipads will unlock right after you typed the last number of the correct combination. It will even stay unlocked if you type in other junk numbers right afterwards. For example, if your pin code is 4269, the combination 13374269666 might unlock your digipad!

## The reasoning

If you want to do some research yourself, here are three words: "De Bruijn sequences".

Since every webpage I saw that explains De Bruijn sequences takes a very convoluted approach to it, here is a simple and imaged explanation: for a digipad, a De Bruijn sequence is a sequence of numbers that will contain every single combination that might unlock the digipad.

If you were to bruteforce a digipad, you could go the "obvious" way and test every single combination separately, like:
+ 0000
+ 0001
+ 0002
+ ...

Since the digipad does not require you to press "enter" in between each combination, you could just send it "00000001000200030004..." and wait for it to unlock. But this is not the *optimized* way of doing things. In the previous sequence, if "0000" isn't the right pincode, then there is no use in typing "0000000" with seven zeroes, because that won't unlock the digipad either. So, in the full sequence, you could remove all of the repetitions to make sure that every single combination is only ever typed once. A De Bruijn sequence is a sequence that does just that: it covers *all* of the combinations in the minimal amount of digits. 

For example, the De Bruijn sequence for a 4 digit pin code if only the numbers 0 and 1 are used is:
0000100100110101101111

Back to our challenge, all we need to do is implement a De Bruijn sequence generator, that only uses the digits that have dissappeared from the digipad, to generate the sequence of digits that covers all the combinations that one can imagine with the digits that have been rubbed out.

## The code

Here is the code. I have cleaned up the "CTF version" of it to make it more understandable.

Honorable mentions are:
+ For the python netcat (which I use in every single CTF I do): https://gist.github.com/leonjza/f35a7252babdf77c8421
+ For the deBruijn function: https://gist.github.com/rgov/891712

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
