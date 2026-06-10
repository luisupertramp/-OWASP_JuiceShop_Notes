> Disclaimer: None of this has been written by AI. Some code snippets and processes have been obtained using AI tools, but this documentation is made 100% by a human. So, hello there :)

# Scenario

* **Victim:** Windows Laptop
* **Attacker:** Raspberry pi 5

# Tools:

* [Docker Desktop](https://docs.docker.com/desktop/setup/install/windows-install/)
* [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/)


# Running Juice-Shop on Windows

Run this from the terminal:

```
 docker run -d -p 3000:3000 bkimminich/juice-shop
```

Where:
- -d, --detach: Run container in background and print container ID
- -p, --publish: Publish a container's port(s) to the host
- "bkimminich/juice-shop": is the image name pulled from `hub.docker.com`

The image will be running on 

```
http://localhost:3000/
```


# Switching to Kali

If the website is not accessible from the Kali machine, by going to `http://WindowsIPAddres:3000`/ you will need to change some Firewall settings on Windows:

- Open the Start menu, type **Windows Defender Firewall**, and select it.
- In the left-hand menu, click on **Advanced settings**. 
- Click on **Inbound Rules** in the upper-left corner, and then in the right panel, click on **New Rule...**. 
- Select **Port** and click Next. 
- Choose **TCP** and under _Specific local ports_, type: `3000`. Click Next. 
- Select **Allow the connection** and click Next. 
- Leave the boxes checked (Domain, Private, Public) and click Next. 
- Give it an easy-to-remember name, such as `Juice Shop Lab`, and click **Finish**.



# First Hacking challenges

## 1. Score-board

The first challenge consist of finding the hidden score-board.

### Solution

I found a section called something like "Help Getting started". In there it tells you that finding the score board is the first challenge and gives you a few tips. This is how I found it:

* Open DevTools
* Go to Network
* Select the `main.js` file and click on the Preview Tab
* Press `Ctrl+F` to search, I started with the word `score`
* Eventually I found the route `/score-board` 
* Finally I went to `http://localhost/#/score-board`

## 2. Error Handling

Provoke an error that is neither very gracefully nor consistently handled.

### Solution

I tried to login (`#/login`) using the next information:
- `username`: `'`
- `password`: `12345` (whatever works)

### Results

By opening the Network tab you can see the `login` call and in the `preview` tab you will find the following message:

```
{
    "error": {
        "message": "SQLITE_ERROR: unrecognized token: \"7356f158bf8ac418989bcae2330c0155\"",
        
        [...]
       
        "original": {
            "errno": 1,
            "code": "SQLITE_ERROR",
            "sql": "SELECT * FROM Users WHERE email = ''' AND password = '7356f158bf8ac418989bcae2330c0155' AND deletedAt IS NULL"
        },
        
        [...]
      
    }
}
```


### Lessons Learned
- **Why is this a problem?** Because website is not only showing an error like "Oops, something went wrong". Instead it reveals the backend structure to the frontend by showing the SQLite query `"SELECT * FROM Users WHERE email = ''' AND password = '7356f158bf8ac418989bcae2330c0155' AND deletedAt IS NULL"`
- **Real impact:** Allows the attacker to speed the recon process of the environment they are targeting. They can identify, based on the errors they are getting, the technologies the website is using (like Node.js, Express, Sequelize, SQLite, etc.) and if one of the versions has a known exploit, it will translate to a direct vulnerability.
- **How to prevent this?** Good practices can include:
	- Try-catch with generic error messages. Instead of this `if (err) res.status(500).send(err);`, use this `if (err) res.status(500).send("Somethign went wrong, please try later.");`
	- Error traces enabled only on `Dev` environments
	- Centralize logs for admins only
	- Custom error pages
