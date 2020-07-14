# Trying Out Yugabyte

Let's try installing a three-node Yugabyte cluster on a 2011-era laptop running Debian linux!

[The quickstart guide is here.](https://docs.yugabyte.com/latest/quick-start/create-local-cluster/linux/#create-a-3-node-cluster-with-rf-of-3).

We ran the suggested:

```sh
./bin/yb-ctl --rf 3 create
```

But it initially failed, because we had a Redis server running on the default port.  So let's turn that off...

```sh
sudo systemctl stop redis
```

We need to destroy the partially-created system before retrying:

```sh
 ./bin/yb-ctl --rf 3 destroy
 ```

Then we retried the creation of three nodes, but our disks didn't spin fast enough to satisfy the default timeout values!  So we found another failure.

```text
Creating cluster.
Waiting for cluster to be ready.
Traceback (most recent call last):
  File "./bin/yb-ctl", line 2021, in <module>
    control.run()
  File "./bin/yb-ctl", line 1998, in run
    self.args.func()
  File "./bin/yb-ctl", line 1755, in create_cmd_impl
    self.wait_for_cluster_or_raise()
  File "./bin/yb-ctl", line 1598, in wait_for_cluster_or_raise
    raise RuntimeError("Timed out waiting for a YugaByte DB cluster!")
RuntimeError: Timed out waiting for a YugaByte DB cluster!
...snip...
Error: Leader not ready to serve requests. (yb/master/scoped_leader_shared_lock.cc:93):
 Unable to list tablet servers: Leader not yet ready to serve requests: leader_ready_te
rm_ = -1; cstate.current_term = 1
```

Let's increase all the possible timeout values and hope that the install works:

```sh

# values in seconds
./bin/yb-ctl --timeout-yb-admin-sec 3000 --timeout-processes-running-sec 3000 --rf 3 create
```

This time it worked!

```text
Creating cluster.
Waiting for cluster to be ready.
---------------------------------------------------------------------------------------
-------------
| Node Count: 3 | Replication Factor: 3
            |
---------------------------------------------------------------------------------------
-------------
| JDBC                : jdbc:postgresql://127.0.0.1:5433/postgres
            |
| YSQL Shell          : bin/ysqlsh
            |
| YCQL Shell          : bin/ycqlsh
            |
| YEDIS Shell         : bin/redis-cli
            |
| Web UI              : http://127.0.0.1:7000/
            |
| Cluster Data        : /home/whoever/yugabyte-data
            |
--------------------------------------------------------------------------------------$
-------------
```

Can we connect to the redis-like interface (YEDIS)? Yes.

```sh
./bin/redis-cli
```

```text
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

Can we connect to Yugabyte as a SQL database using the provided shell script?  Yes.

```sh
bin/ysqlsh
```

Let's create a user and then try to connect with one of our favorite SQL GUIs, [SQLWorkbench/J](http://www.sql-workbench.eu/index.html).

```text
yugabyte=# create user whoever with password 'superhardtoguess';
CREATE ROLE
```

Now we can connect using SQLWorkbench/J. We need to check the box for `AUTOCOMMIT` so that it's enabled.  Notice that the port differs from the default of 5432.  Also, _helpfully_, the team at Yugabyte made sure that there's [a tutorial for connecting with SQLWorkbench/J](https://docs.yugabyte.com/latest/tools/sql-workbench/).  Thank you!

![Connect with SQLWorkbench/J](https://user-images.githubusercontent.com/38859656/87455428-6f0b3c00-c5d3-11ea-8596-f787264d7322.png)

So we're connected, but can we create a table? Yes. We waited 20 seconds for it to happen.

![Creating a basic table](https://user-images.githubusercontent.com/38859656/87456320-bba34700-c5d4-11ea-8f67-12d5e589078c.png)


