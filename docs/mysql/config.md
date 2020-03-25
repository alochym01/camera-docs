**MySQL Master**  
- vi /etc/my.cnf  
    ```
    # For advice on how to change settings please see
    # http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

    [mysqld]
    #
    # Remove leading # and set to the amount of RAM for the most important data
    # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
    # innodb_buffer_pool_size = 128M
    #
    # Remove the leading "# " to disable binary logging
    # Binary logging captures changes between backups and is enabled by
    # default. It's default setting is log_bin=binlog
    # disable_log_bin
    #
    # Remove leading # to set options mainly useful for reporting servers.
    # The server defaults are faster for transactions and fast SELECTs.
    # Adjust sizes as needed, experiment to find the optimal values.
    # join_buffer_size = 128M
    # sort_buffer_size = 2M
    # read_rnd_buffer_size = 2M
    #
    # Remove leading # to revert to previous value for default_authentication_plugin,
    # this will increase compatibility with older clients. For background, see:
    # https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
    default-authentication-plugin=mysql_native_password
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock

    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci
    wait_timeout = 28800
    interactive_timeout = 28800

    log-error=/var/log/mysqld.log
    log_error_verbosity=2
    pid-file=/var/run/mysqld/mysqld.pid

    server-id=1
    max_connections=1000
    max_user_connections=500
    max_allowed_packet=256M
    #bin-log
    binlog_cache_size=8G
    binlog_stmt_cache_size=16M
    log-bin=/var/log/mysql/mysql-bin.log
    log-bin-index=/var/log/mysql/mysql-bin.index
    #relay-log
    relay-log=/var/log/mysql/mysql-relay-bin.log
    relay-log-index=/var/log/mysql/mysql-relay-bin.index
    expire_logs_days=10
    #database-replication
    binlog-do-db=ipc_customer
    binlog-do-db=ipc_management
    binlog-do-db=ipc_voucher
    binlog-do-db=ipc_management_flag
    #loggin
    slow_query_log=1
    slow_query_log_file=/var/log/mysql/mysql-slow.log
    #InnoDB
    innodb_buffer_pool_size=50G
    innodb_log_file_size=64M
    innodb_file_per_table=1
    innodb_flush_method=O_DIRECT
    #General Query Log
    general_log=1
    general_log_file=/var/log/mysql/general.log

    [client]
    default-character-set = utf8mb4
    ```
**MySQL Slave** 
- vi /etc/my.cnf  
    ```
    # For advice on how to change settings please see
    # http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

    [mysqld]
    #
    # Remove leading # and set to the amount of RAM for the most important data
    # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
    # innodb_buffer_pool_size = 128M
    #
    # Remove the leading "# " to disable binary logging
    # Binary logging captures changes between backups and is enabled by
    # default. It's default setting is log_bin=binlog
    # disable_log_bin
    #
    # Remove leading # to set options mainly useful for reporting servers.
    # The server defaults are faster for transactions and fast SELECTs.
    # Adjust sizes as needed, experiment to find the optimal values.
    # join_buffer_size = 128M
    # sort_buffer_size = 2M
    # read_rnd_buffer_size = 2M
    #
    # Remove leading # to revert to previous value for default_authentication_plugin,
    # this will increase compatibility with older clients. For background, see:
    # https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
    default-authentication-plugin=mysql_native_password
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock

    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci
    wait_timeout = 28800
    interactive_timeout = 28800

    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid

    server-id=2
    max_connections=1000
    max_user_connections=500
    max_allowed_packet=256M
    #bin-log
    binlog_cache_size=8G
    binlog_stmt_cache_size=16M
    log-bin=/var/log/mysql/mysql-bin.log
    log-bin-index=/var/log/mysql/mysql-bin.index
    #relay-log
    relay-log=/var/log/mysql/mysql-relay-bin.log
    relay-log-index=/var/log/mysql/mysql-relay-bin.index
    expire_logs_days=1
    #database-replication
    binlog-do-db=ipc_customer
    binlog-do-db=ipc_management
    binlog-do-db=ipc_management_flag
    binlog-do-db=ipc_voucher
    #loggin
    slow_query_log=1
    slow_query_log_file=/var/log/mysql/mysql-slow.log
    #InnoDB
    innodb_buffer_pool_size=128M
    innodb_log_file_size=64M
    innodb_file_per_table=1
    innodb_flush_method=O_DIRECT
    #General Query Log
    general_log=1
    general_log_file=/var/log/mysql/general.log
    ```
    
