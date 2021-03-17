# Kamailio Network Server

I come from a background of many years of Broadworks based networks and one component that I've always liked in the Broadworks Architecture is the Network Server, or NS. These servers communicate with the Application Server, and the SBCs to lookup where a number (DNIS) should be routed to. These lookups are performed lighting fast through the Oracle Times 10 database, which is 100% in memory. Once the lookup has been performed, the NS returns a 302 redirect with the list of contacts that should be used for the call. This architecture allows the NS to rapidly handle traffic as it doesn't maintain the state of any call.

I had some spare time on a rainy Saturday night during Covid-19 and thought it would be fun to try to emulate some of the Broadworks NS functionality using Kamailio.

So what do we have...

# Docker Compose

In order to rapidly create a testing environment, docker-compose is used. The docker-compose.yaml file will spin up 1 Mysql Database, 1 Kamailio (acting as the SBC), 2 Network Servers, and two PJSUA containers which act as an external B2BUA, representing carrier and an Internal B2BUA, representing a network element of the voice core.

Before you start, make a directory in the root of this folder named "db"

it's easy to spin up the instances, just run `docker-compose up`

This will start all of the containers.


# Database Setup
Once they are started connect to the database with `docker exec -it db01 bash`

followed by `mysql -u root -p` at the command line.

At the db type "use db" and then create the dnis table

```javascript
CREATE TABLE dnis(
  id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  KEY_NAME  VARCHAR(64),
  KEY_TYPE  INT,
  VALUE_TYPE  INT,
  KEY_VALUE VARCHAR(16),
  EXPIRES INT
);
```
and lets add an entry into the table
```javascript
insert into dnis (KEY_NAME, KEY_TYPE, VALUE_TYPE,KEY_VALUE,EXPIRES)
VALUES('8675309', 0, 0, "172.16.10.100", 3600);
```
Now that a DB entry is written, exit out of the database container.

# Network-Server

Now that the datbase has been populated, you will need to reload the dnis htable on ns01 and ns02 using `kamcmd htable.reload dnis`

You can verify the contents with kamcmd htable.dump dnis
```javascript
root@522c7d38f9d2:/# kamcmd htable.dump dnis
{
	entry: 12426
	size: 1
	slot: {
		{
			name: 8675309
			value: 172.16.10.100
			type: str
		}
	}
}
```
