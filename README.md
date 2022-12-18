# CPSC 449 Project 4
 [Project 4](https://docs.google.com/document/d/19BqaDN9M9fMfw6WjwISGDauF_I2w20UJ4lNmW9USbn0/edit) is used to add Webhook to the Games Service, and deliver items to a message queue to the Leaderboard Service that was created in  [Project 3](https://docs.google.com/document/d/1OWltxCFRsd2s4khOdfwKLZ3vqF6dsJ087nyMn0klcQs/edit) which involved extending the base Wordle backend application from [Project 1](https://docs.google.com/document/d/14YzD8w5SpJk0DqizgrgyOsXvQ2-rrd-39RUSe2GNvz4/edit) and implementation of nginx to authenticate endpoints and load balancing from [Project 2](https://docs.google.com/document/d/1BXrmgSclvifgYWItGxxhZ72BrmiD5evXoRbA_uRP_jM/edit). This includes the following objectives:
- Configuring replication using Litefs for the database associated with Games service. (Write requests go to the primary replica, and read requests can be made from either primary, secondary or tertiary replicas)
- Developing 2 new Leaderboard services which can post results of a game and obtain the Top 10 users based on their average scores.
- Use Redis to store data for leaderboard services.

This project also builds upon concepts introduced in [Exercise 2](https://docs.google.com/document/d/1-tFBfCP2rhk5YFtXYpGD894Ghy4UY-J3o9Zs7abbS8c/edit),[Exercise 3](https://docs.google.com/document/d/14i8cpm7z1oFh5y5gmAkQ39AH3Pu8oWRr6B6TOziGYhY/edit) and [Exercise 4](https://docs.google.com/document/d/1GeF5txkEb3Jl0_YtnFKFh21xiDff1IJ54XC9Qydk3GE/edit) with regards to setting up Nginx server, building indices, using redis and associated libraries, working with Webhooks and also Message Queueing.

### Authors
Section 02
Group 23
Members:
- Harshith Harijeevan
- Heet Savla
- Sam Le
- Michael Morikawa

## Setting Up
### Development Environment 
Tuffix 2020 (Linux)

### Application Prerequisites
- Python
- Nginx
- Quart
- Sqlite3
- Foreman
- Databases
- Quart-Schema
- Curl
- HTTPie
- Redis
- LiteFS
- HTTPX

### VHost Setup
1. Make sure that nginx is running in the background
```
$ sudo service nginx status
```
2. Verify that `tuffix-vm` is in `/etc/hosts`
```
$ cat /etc/hosts
```
__Note:__ This project uses the hostname `tuffix-vm`. 
3. Go to the project's directory
```
cd project-name/
```

4. Copy the VHost file in `/share` to `/etc/nginx/sites-enabled` then restart nginx 
```
$ sudo cp share/wordle /etc/nginx/sites-enabled/wordle
$ sudo service nginx restart
```

### Initializing and Starting the Application
1. Create folder structure for read/write replication
```
./bin/create_folder_structure.sh
```
2. Start three 3 instances of game, 1 instance of user, and 1 instance of leaderboard service with foreman
```
foreman start
```
3. Run the command below to initialize user and game databases and redis in-memory data store, populate them with dummy values. Please wait as this will take some time
```
./bin/populate_data.sh
```


## REST API Features
- Register a user (includes password hashing)
- Authenticate a user (includes hashing verification)
- Start a new game
- Guess a five-letter word
- Retrieve the state of a game in progress
- List the games in progress for a user
- Check the statistics for a particular user
- Post the results of a game taking input the game decision and the number of guesses.
- Retrieve the top 10 users based on their average scores.

## Running the Application

### Registering a User
After starting up the app, create an initial user using the following command with HTTPie where `<username>` and `<password>` are custom values (will need them later when logging in):
```
http POST http://tuffix-vm/register username=<username> password=<password>
```
The whole application can only be accessed after authentication, so use the username and password that was created in this step.


### Using Quart-Schema Documentation
The API documentation generated by Quart-Schema for game microservice can be accessed with [this link](http://tuffix-vm/docs) or typing the URL `http://tuffix-vm/docs`. The API documentation for leaderboard microservice can be accessed with [this link](http://127.0.0.1:5400/docs) or typing the URL `http://127.0.0.1:5400/docs`

### Retrying failed jobs
Cron jobs allow us to schedule recurring tasks as per the requirements. In this project, after every 10 minutes if any of the job in the leaderboard service has failed which was queued in queue, then it can be sent to the leaderboard service once again for the execution.

Run the following command in the terminal
crontab -e

After executing the above command, an editor will open where we have to paste the following command

*/10 * * * * run-one rq requeue --all --queue default

Here run-one allows to run one unique instance of a command with its unique set of arguments.

### Using HTTPie
In order to run via httpie, use the following the commands:
- Creating a user
```
http POST http://tuffix-vm/register username=<username> password=<password>
```
- Authenticating a user
```
http GET http://tuffix-vm/ --auth <username>:<password>
```
- Creating a game 
```
http POST http://tuffix-vm/games --auth <username>:<password>
```
- Checking the state of a game 
```
http GET http://tuffix-vm/games/<game_id> --auth <username>:<password>
```
- Playing a certain game/making a guess
```
http POST http://tuffix-vm/games/<game_id> guess=<5-letter-word> --auth <username>:<password>
```
- Retrieving a list of in-progress games
```
http GET http://tuffix-vm/games/ --auth <username>:<password>
``` 
- Check the statistics of the user 
```
http GET http://tuffix-vm/games/statistics --auth <username>:<password>
```
- Post the results of a game for the leaderboard service. (Used internally)
```
http POST http://127.0.0.1:5400/results guess_number=<Number from 1 to 6> status=<win or loss> username=<username>
```
- Retrieve the top 10 users based on their average scores.
```
http GET http://tuffix-vm/leaderboard
```
- Register a callback url to send notification after game finishes, should be an endpoint that accepts a POST request expecting a json payload with guess_number(between 1-6), status (win,loss), and username
```
http POST http://tuffix-vm/games/registerleaderboard?url=<leaderdboard_url>
```