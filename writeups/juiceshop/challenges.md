# OWASP Juice Shop v13.0.2

I suck at markdown so I apologize in advance if this document looks hideous.

## Difficulty 1

### Score Board

Go to main page. Opened Developer Tools, go to the Sources tab. Look into the main.js file.
Search for "score". Eventually you'll find a line

	path: "score-board"
	
Then, go to `http://your.juice.shop.ip:port/#/score-board`


### Confidential Document

Brute-force directories using your directory brute-force tool of choice. In my case, I used [dirsearch](https://github.com/maurosoria/dirsearch)

	python3 dirsearch.py -u "http://your.juice.shop.ip:port" -w /path/to/dictionary
	...
	Found "200 -   11KB - /ftp"

Then, go to `http://your.juice.shop.ip:port/#/ftp`

*Alternatively, you may also find the /ftp directory by going to `http://your.juice.shop.ip:port/robots.txt*`

Several files can be found here. Server only allows to download .md and .pdf files.
Open the file `acquisitions.md`


### Exposed metrics

Happened to solve this one while trying to solve "Confidential Document (1*)"

Brute-force directories using your directory brute-force tool of choice.  
In my case, I used I used [dirsearch](https://github.com/maurosoria/dirsearch)

	python3 dirsearch.py -u "http://your.juice.shop.ip:port" -w /path/to/dictionary
	...
	Found "200 - 22KB - /metrics"`

Then, go to `http://your.juice.shop.ip:port/#/metrics`


### Privacy Policy

Log into your Juice Shop account. If you don't have one, make one.  
Go to the Account menu in the top bar. Select "Privacy & Security"  
Select "Privacy Policy"

Alternatively, go to `http://your.juice.shop.ip:port/#/privacy-security/privacy-policy`

### Zero Stars

For this one I used BurpSuite since that's what's comfortable for me.
Open the left menu and go to "Customer Feedback"
Type whatever you want under the "comment" field.
Select any rating, solve the captcha.

Open BurpSuite, turn on Intercept Mode.
Submit the feedback, see a request being sent to `POST /api/Feedbacks/`

You're gonna see a JSON body. The important part lies on the "rating" field.

`{"UserId":xxxx,"captchaId":xxxx,"captcha":"xxxx","comment":"xxxx","rating":1}`

Change the value of "rating" to 0. Forward the request.

Additionally, see the server response for it

`{"message":"internal error","errors":["SQLITE_CONSTRAINT: FOREIGN KEY constraint failed"]}`

Might be useful for later.

### DOM XSS

You need to use the payload provided by the challenge's description.

	<iframe src="javascript:alert(`xss`)">
	
Paste it in the search bar on top.

### Bonus Payload

You need to paste the provided payload in the same place as the challenge "DOM XSS"
That'd be the search bar on top

	<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>

Enjoy!

## Difficulty 2

### Login Admin

This one is about injection so let's try to find the admin's email so we can attempt to log into their account.
One way to find it is by looking through product reviews. You'll quickly find "admin@juice-sh.op"

Go to the Login Page.
Input "admin@juice-sh.op" into the user field, and whatever you want into the password field.
Add "' --" at the end of admin@juice-sh.op

	admin@juice-sh.op' --
	
Attempt to log in.

This is a simple authentication bypass by SQL injection. The problem comes from unsanitized input being included into an SQL query.
It could look like somethiing like `SELECT * FROM users WHERE email = '@email' AND password = '@password'`

By putting `admin@juice-sh.op' --` into the username field, the query will then look like
`SELECT * FROM users WHERE email = 'admin@juice-sh.op' -- ' AND password = @password`

Everything after `--` will be commented, so the final query will look like
`SELECT * FROM users WHERE email = 'admin@juice-sh.op' --`
Allowing us to log in without knowing the password.