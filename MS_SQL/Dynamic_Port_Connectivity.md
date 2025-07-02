## How MS SQL Dyamanic-Port Connectivity works ?

MS SQL may run on two ports :
- (1) **Static Port**: You manually assign SQL Server to listen on a fixed TCP port (commonly `1433`).
- (2) **Dynamic Port**: SQL Server picks a free TCP port at startup (between `49152 to 65535`), which could change every time the service restarts.

So if your SQL Server is running on a dynamic port, clients don’t automatically know which port to connect to—they must discover it somehow.

### Firewall Requirements
- **UDP 1434**: _Source_ = Application Server, _Destination_ = Database server, _Protocol_ = **UDP**, _Port_ = **1434**, _Direction_ = Unidirection
- **Dynamic TCP Port**:  _Source_ = Application Server, _Destination_ = Database server, _Protocol_ = **TCP**, _Port_ = **49152–65535**, _Direction_ = Unidirection

### How Clients Discover the Port ?
This is where **SQL Server Browser Service** comes in. SQL Server Browser listens on `UDP port 1434`.
When your application tries to connect using just the instance name (e.g., ServerB\MyInstance), the client driver:
- (a) Sends a UDP query to port 1434 on the SQL Server.
- (b) The Browser service replies with the TCP port number that the named instance is listening on.
- (c) The client then opens a TCP connection to that port.
 
_Note_ : _This dynamic discovery only happens if you don’t specify the port in your connection string._
<pre>
+--------------------+                          +--------------------+
|    Server A        |                          |    Server B        |
|   (Application)    |                          |  (MS SQL Server)   |
|                    |                          |                    |
|  +-----------+     |                          |  +-----------+     |
|  | App       |     |  1. UDP 1434 Request     |  | SQL Server|     |
|  |           |---->|------------------------->|  | Browser   |     |
|  |           |     |  (Query Instance Port)   |  | (UDP 1434)|     |
|  |           |     |                          |  |           |     |
|  |           |     |  2. UDP 1434 Response    |  |           |     |
|  |           |<----|<-------------------------|  |           |     |
|  |           |     |  (Returns Dynamic Port)  |  |           |     |
|  |           |     |                          |  +-----------+     |
|  |           |     |                          |                    |
|  |           |     |  3. TCP Dynamic Port     |  +-----------+     |
|  |           |---->|------------------------->|  | SQL Server |    |
|  |           |     |  (e.g., TCP 49152)       |  | Instance   |    |
|  |           |     |                          |  | (Dynamic   |    |
|  |           |     |                          |  |  Port)     |    |
|  +-----------+     |                          |  +-----------+     |
|                    |                          |                    |
+--------------------+                          +--------------------+
</pre>

### How to Check the Dynamic Port
Open `SQL Server Configuration Manager` > navigate to `SQL Server Network Configuration` > click `Protocols for [InstanceName]` > right click `TCP/IP` > select `Properties` > Under `IPAll`, check `TCP Dynamic Ports`.
