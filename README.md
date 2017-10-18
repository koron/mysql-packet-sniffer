# DB (MySQL) query logger

[![CircleCI](https://circleci.com/gh/koron/dbquerylog.svg?style=svg)](https://circleci.com/gh/koron/dbquerylog)

dbquerylog converts TCP dump to DB query log.

## Getting start

How to use.

```console
$ sudo tcpdump -s 0 -l -w - "tcp port 3306" | dbquerylog
2017-09-27T16:19:06Z	1506529146714406663	6810	10.0.2.2	46462	10.0.2.15	3306	108	1	2	vagrant	vagrant	SHOW DATABASES	
2017-09-27T16:19:06Z	1506529146717047737	4667	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	CREATE TABLE IF NOT EXISTS users (\n\t\tid INT PRIMARY KEY AUTO_INCREMENT,\n\t\tname VARCHAR(255) UNIQUE,\n\t\tpassword VARCHAR(255)\n\t)	
2017-09-27T16:19:06Z	1506529146718404415	11578	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"foo\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146719769968	12187	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"baz\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146720975913	30832	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"bar\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146722811855	7329	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"user001\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146723899820	7796	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"user002\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146725027794	7380	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	INSERT INTO users (name, password) VALUES (?, ?)	\"user003\", \"pass1234\"
2017-09-27T16:19:06Z	1506529146726231050	3635	10.0.2.2	46462	10.0.2.15	3306	7	0	0	vagrant	vagrant	DROP TABLE users	
```

## Recommended options for tcpdump

```console
$ sudo tcpdump -s 0 -l -w - "tcp port 3306"
```

Meanings:

*   `-s 0` no limit snaplen
*   `-l` line buffered (is `-U` better?)
*   `-w -` output to stdout
*   `"tcp port 3306"` dump port 3306 on TCP only

## Report format

Report format is based on TSV (tab separated values).
Each rows represent database queries.
Each columns are consisted by below:

*   start time (in RFC3339 format)
*   start time (in unix nanoseconds)
*   elapsed time (nanosecond)
*   client IP address
*   client port number
*   server IP address
*   server port number
*   total response size in byte
*   num of columns in response
*   num of rows which responded or updated
*   username
*   database name
*   query
*   parameters (available for prepared statement only)

Each columns are escaped by [`strconv.Quote()`][quote] then truncated by
`-column_maxlen` bytes.

## Options

*   `-select` include SELECT statemnets
*   `-debug` enable debug log
*   `-column_maxlen` max length of each columns (default 1024 bytes)
*   `-decoder` name of the decoder to use

    To parse tcpdump with `-i any`. Example:

    ```console
    $ sudo tcpdump -i any -s 0 -l -w - "tcp port 3306" | dbquerylog -decoder "Linux SLL"
    ```

[quote]:https://golang.org/pkg/strconv/#Quote
