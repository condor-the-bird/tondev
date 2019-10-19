To review logs, call the following command providing the relevant container name or id:

```shell
docker exec -it <container_name_or_id> bash
```

The log is then displayed in the terminal. The log file is stored at `cd /ton-node/log` in `output.log`.

You can also copy it directly to host by calling:

```shell
docker cp <container>:/ton-node/log/output.log <dest_path>
```

The screenshot shows log of the local node performance.

![img](https://tonlabs.zeroheight.com/uploads/0-fotA_0JZeOjDReV_9_zQ.png)

Note that the use of `sudo` depends on your local settings and preferences.  It is not necessary or mandatory.