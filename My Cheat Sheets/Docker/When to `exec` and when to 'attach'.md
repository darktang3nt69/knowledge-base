https://yasoob.me/posts/docker-attach-vs-exec-when-to-use-what/

Two ways to connect to the Docker container. I can either use `attach` or `exec`. It is important to know the difference.

- `attach` allows you to connect and interact with a container’s main process which has `PID 1`
- `exec` allows you to create a new process in the container

This significant difference in the way these both commands run loads to some useful and, oftentimes, problematic side-effects. You can essentially use the `attach` command as a terminal share system. If you are connected to the same Docker container from two different screens or terminal sessions, you can run commands on either one of the terminals and the output will be streamed in real-time to all other connected screens/terminal sessions. Think of this as a poor man’s pair programming setup 😅

> PSA: don’t actually use this for pair-programming

If you kill the main process the container will terminate.

Whereas, when you use `exec` a new process is started on the Docker container. This is the command you are supposed to use almost always unless you know what you are doing with the `attach` command. I just usually use the `exec` command to launch `bash` within the container and work with that. I use the `attach` command primarily when I quickly want to see the output of the main process (`PID 1`) directly and/or want to kill it.