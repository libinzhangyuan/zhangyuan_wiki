### rails c 下可以出来之前的命令
```
RUN echo "IRB.conf[:LOAD_HISTORY] = true" >> ~/.irbrc && \
  echo "IRB.conf[:HISTORY_FILE] = File.expand_path('~/.irb_history')" >> ~/.irbrc && \
  echo "IRB.conf[:SAVE_HISTORY] = 1000" >> ~/.irbrc
```

### 设置shell中可以查看历史命令
```
# 实际并不一定足够。还需要readline等的安装情况。

RUN echo 'export HISTSIZE=1000' >> /root/.bashrc && \
  echo 'export HISTFILESIZE=2000' >> /root/.bashrc && \
  echo 'export HISTFILE=/root/.bash_history' >> /root/.bashrc

然后docker-compose中要启动方式要为 CMD ["/bin/bash", "-i"]

```