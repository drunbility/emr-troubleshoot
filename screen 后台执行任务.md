


screen -S session_name           # 新建一个叫session_name的session
screen -ls（或者screen -list）   # 列出当前所有的session
screen -r session_name            # 回到session_name这个session



ctrl a +d   # detach，暂时离开当前session，将当前 screen session 转到后台执行，并会返回没进 screen 时的状态，此时在 screen session 里，每个shell client内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响

输入 exit 退出session

