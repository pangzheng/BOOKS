# docker 实例和主进程分离试验
## 简介
在 docker 1.12 版本的时候引入了一个新的参数开启实例和主进程的生命周期非同步，这样可以解决 docker 主进程调整重启影响业务服务的问题。

- 参数使用 

		--live-restore
- 使用方法

		 vi /usr/lib/systemd/system/docker.service
		 ...
		 ExecStart=/usr/bin/dockerd -s overlay -g /data/docker --live-restore
		 ...

## 试验
- 测试说明

		在 mesos 环境下测试这参数对于 mesos 架构的影响，主要考虑到 mesos-executer 使用的是 docker 命令维护 task 的，所以如果 docker daemon 挂掉了，其实这块是有问题的，看是否可以接管。其次稳定性也有待观察。
- 测试环境
	- docker
		- overlay
		- 版本 17.03.1-ce 
	- mesos	
		- mesos 1.0.1
		- marathon 1.1.1 	
- 测试步骤
	- 开启 docker 的任务 
	- 使用 marathon 启动一个服务
	- 查看实例情况
	- 关闭计算节点 docker 实例
- 试验结果
	- 关闭 docker 后使用 mesos 启动的服务，因为 mesos-executer 使用 docker 命令操作 mesos 问题，导致从 mesos 系统层面看来这些服务都异常了。这会 docker 不可操作。 
	- 在重启 docker，使用 docker ps 可以看到原有的 mesos 启动的task 依然存在，这里符合 docker 这个参数设计，但是从 mesos 系统来看，任务还是 wait 状态。查看 mesos slave 日志实际上因为 mesos 使用的是 docker in docker 的模式，所以挂载的 docker sock 因为 docker 关闭没有了，但是新启动的 docker 文件没有加载，所以无法找到新的 docker 进程的 sock 文件，故无法正常。如果非 docker 使用的 mesos 应该不会有这个问题。

				W0417 15:05:48.488276     8 docker.cpp:1429] Ignoring updating unknown container: ae2fcfa9-8b2d-4c20-a93b-59d3f5c847a7
				E0417 15:05:53.506392     7 slave.cpp:3976] Container 'fd8284cb-551f-466c-8dcb-30f4e41f8043' for executor 'pro-2048.4bb0fe7c-233c-11e7-8119-0242c8354e38' of framework 30ded702-99a0-48d9-9595-fc97e8582df0-0000 failed to start: Failed to run 'docker -H unix:///var/run/docker.sock pull offlineregistry.dataman-inc.com:5000/library/blackicebird-2048:latest': exited with status 1; stderr='Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
	- 重启 mesos slave 试图恢复 mesos executer
		- 但是导致 docker 夯死,使用 docker ps 无响应，docker api 也异常。
		- 再度重启 docker daemon		
		- mesos slave 提示无法启动

					Failed to perform recovery: Collect failed: Collect failed: Failed to run 'docker -H unix:///var/run/docker.sock rm -v 2636b1be42e45342d740fa930d621a5cca7e739cbff5a071b99d43c5477d5c07': exited with status 1; stderr='Error response from daemon: Unable to remove filesystem for 2636b1be42e45342d740fa930d621a5cca7e739cbff5a071b99d43c5477d5c07: remove /data/docker/containers/2636b1be42e45342d740fa930d621a5cca7e739cbff5a071b99d43c5477d5c07/shm: device or resource busy

					To remedy this do as follows:
					Step 1: rm -f /data/mesos/meta/slaves/latest
        				This ensures agent doesn't recover old live executors.
					Step 2: Restart the agent.
		- 经过测试这个是 docker 对应一些镜像会产生这个问题，好像是什么锁和文件系统有关
			- 测试步骤
				- docker 后台启动一个 2048
				
						docker run -d offlineregistry.dataman-inc.com:5000/library/blackicebird-2048
				- service docker restart
				- docker stop 2048
					- 发现卡住
			- 查询客户端问题
				- 重复上面步骤，到 stop 步骤
				- strace docker stop 2048
				- 问题日志
					
							关键日志
							futex(0xc42002ad10, FUTEX_WAKE, 1)      = 1
							socket(PF_LOCAL, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 4
							setsockopt(4, SOL_SOCKET, SO_BROADCAST, [1], 4) = 0
							connect(4, {sa_family=AF_LOCAL, sun_path="/var/run/docker.sock"}, 23) = 0
							epoll_create1(EPOLL_CLOEXEC)            = 5
							epoll_ctl(5, EPOLL_CTL_ADD, 4, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=766271696, u64=140665240182992}}) = 0
							getsockname(4, {sa_family=AF_LOCAL, NULL}, [2]) = 0
							getpeername(4, {sa_family=AF_LOCAL, sun_path="/var/run/docker.sock"}, [23]) = 0
							futex(0xc42002a910, FUTEX_WAKE, 1)      = 1
							read(4, 0xc4203e6000, 4096)             = -1 EAGAIN (Resource temporarily unavailable)
							epoll_wait(5, {}, 128, 0)               = 0
							epoll_wait(5, {{EPOLLOUT, {u32=766271696, u64=140665240182992}}}, 128, -1) = 1
							epoll_wait(5, {{EPOLLIN|EPOLLOUT, {u32=766271696, u64=140665240182992}}}, 128, -1) = 1
							read(4, "HTTP/1.1 200 OK\r\nApi-Version: 1."..., 4096) = 200
							futex(0xc4203e8110, FUTEX_WAKE, 1)      = 1
							mmap(NULL, 262144, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fef2da31000
							futex(0xc4203e8110, FUTEX_WAKE, 1)      = 1
							read(4, 0xc4203e6000, 4096)             = -1 EAGAIN (Resource temporarily unavailable)
							sched_yield()                           = 0
							futex(0x11674e0, FUTEX_WAKE, 1)         = 0
							futex(0xc4203e8510, FUTEX_WAKE, 1)      = 1
							write(4, "POST /v1.27/containers/jolly_eli"..., 157) = 157
							futex(0xc4203e8510, FUTEX_WAKE, 1)      = 1
							futex(0x11624d0, FUTEX_WAIT, 0, NULL
				- 检查进程已经被关闭
				- 测试其他服务，如cadvisor 或者 log-agent 正常。

## 结论
- 因为 mesos slave 使用 docker 的试验方法，所以即使开放了这个模式，也会导致 mesos 删除所有 slave 上的服务。
- docker 现在还会因为这个参数导致 docker 卡主，上面分析主要可能和镜像和存储有关。
- 综上亮点，暂时还不使用
- 学习到了使用 strace 可以直接查看 docker 客户端命令操作系统过程，如 trace docker stop xxx

## 相关 github issue
- https://github.com/docker/docker/issues/31997
- https://github.com/docker/docker/issues/7905		   
