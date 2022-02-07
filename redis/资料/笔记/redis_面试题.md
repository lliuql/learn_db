# Redis 面试题

## 哨兵模式工作原理

阶段一：



sentinel



阶段二： 通知阶段

阶段三： 故障转移

1. 当有一个sentinel 发现master 挂了，会给这台服务器标记为**主观下线**
2. sentinel 在sentinel 进行发布该消息，其他sentinel收到消息后会去确认是不是挂了。
3. 当达到配置

[redis哨兵模式原理 - 猿起缘灭 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gunduzi/p/13160448.html#_label0)

[redis-Sentinel配置 - biglittleant - 博客园 (cnblogs.com)](https://www.cnblogs.com/biglittleant/p/7770960.html)

```text
# Example sentinel.conf

# *** IMPORTANT ***
# 绑定IP地址
# bind 127.0.0.1 192.168.1.1
# 保护模式（是否禁止外部链接，除绑定的ip地址外）
# protected-mode no

# port <sentinel-port>
# 此Sentinel实例运行的端口
port 26379

# 默认情况下，Redis Sentinel不作为守护程序运行。 如果需要，可以设置为 yes。
daemonize no

# 启用守护进程运行后，Redis将在/var/run/redis-sentinel.pid中写入一个pid文件
pidfile /var/run/redis-sentinel.pid

# 指定日志文件名。 如果值为空，将强制Sentinel日志标准输出。守护进程下，如果使用标准输出进行日志记录，则日志将发送到/dev/null
logfile ""

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# 上述两个配置指令在环境中非常有用，因为NAT可以通过非本地地址从外部访问Sentinel。
#
# 当提供announce-ip时，Sentinel将在通信中声明指定的IP地址，而不是像通常那样自动检测本地地址。
#
# 类似地，当提供announce-port 有效且非零时，Sentinel将宣布指定的TCP端口。
#
# 这两个选项不需要一起使用，如果只提供announce-ip，Sentinel将宣告指定的IP和“port”选项指定的服务器端口。
# 如果仅提供announce-port，Sentinel将通告自动检测到的本地IP和指定端口。
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# 每个长时间运行的进程都应该有一个明确定义的工作目录。对于Redis Sentinel来说，/tmp就是自己的工作目录。
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# 告诉Sentinel监听指定主节点，并且只有在至少<quorum>哨兵达成一致的情况下才会判断它 O_DOWN 状态。
#
#
# 副本是自动发现的，因此您无需指定副本。
# Sentinel本身将重写此配置文件，使用其他配置选项添加副本。另请注意，当副本升级为主副本时，将重写配置文件。
#
# 注意：主节点（master）名称不能包含特殊字符或空格。
# 有效字符可以是 A-z 0-9 和这三个字符 ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2

# 如果redis配置了密码，那这里必须配置认证，否则不能自动切换
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# 主节点或副本在指定时间内没有回复PING，便认为该节点为主观下线 S_DOWN 状态。
#
# 默认是30秒
sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numreplicas>
#
# 在故障转移期间，多少个副本节点进行数据同步
sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# 指定故障转移超时（以毫秒为单位）。 它以多种方式使用：
#
# - 在先前的故障转移之后重新启动故障转移所需的时间已由给定的Sentinel针对同一主服务器尝试，是故障转移超时的两倍。
#
# - 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#
# - 取消已在进行但未生成任何配置更改的故障转移所需的时间
#
# - 当进行failover时，配置所有slaves指向新的master所需的最大时间。
#   即使过了这个超时，slaves依然会被正确配置为指向master。
#
# 默认3分钟
sentinel failover-timeout mymaster 180000

# 脚本执行
#
# sentinel notification-script和sentinel reconfig-script用于配置调用的脚本，以通知系统管理员或在故障转移后重新配置客户端。
# 脚本使用以下规则执行以进行错误处理：
#
# 如果脚本以“1”退出，则稍后重试执行（最多重试次数为当前设置的10次）。
#
# 如果脚本以“2”（或更高的值）退出，则不会重试执行。
#
# 如果脚本因为收到信号而终止，则行为与退出代码1相同。
#
# 脚本的最长运行时间为60秒。 达到此限制后，脚本将以SIGKILL终止，并重试执行。

# 通知脚本
#
# sentinel notification-script <master-name> <script-path>
#
# 为警告级别生成的任何Sentinel事件调用指定的通知脚本（例如-sdown，-odown等）。
# 此脚本应通过电子邮件，SMS或任何其他消息传递系统通知系统管理员 监控的Redis系统出了问题。
#
# 使用两个参数调用脚本：第一个是事件类型，第二个是事件描述。
#
# 该脚本必须存在且可执行，以便在提供此选项时启动sentinel。
#
# 举例:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# 客户重新配置脚本
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# 当主服务器因故障转移而变更时，可以调用脚本执行特定于应用程序的任务，以通知客户端，配置已更改且主服务器地址已经变更。
#
# 以下参数将传递给脚本：
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> 目前始终是故障转移 "failover"
# <role> 是 "leader" 或 "observer"
#
# 参数 from-ip, from-port, to-ip, to-port 用于传递主服务器的旧地址和所选副本的新地址。
#
# 举例:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# 安全
# 避免脚本重置，默认值yes
# 默认情况下，SENTINEL SET将无法在运行时更改notification-script和client-reconfig-script。
# 这避免了一个简单的安全问题，客户端可以将脚本设置为任何内容并触发故障转移以便执行程序。
sentinel deny-scripts-reconfig yes

# REDIS命令重命名
#
#
# 在这种情况下，可以告诉Sentinel使用不同的命令名称而不是正常的命令名称。
# 例如，如果主“mymaster”和相关副本的“CONFIG”全部重命名为“GUESSME”，我可以使用：
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# 设置此类配置后，每次Sentinel使用CONFIG时，它将使用GUESSME。 请注意，实际上不需要尊重命令案例，因此在上面的示例中写“config guessme”是相同的。
#
# SENTINEL SET也可用于在运行时执行此配置。
#
# 为了将命令设置回其原始名称（撤消重命名），可以将命令重命名为它自身：
#
# SENTINEL rename-command mymaster CONFIG CONFIG

```

