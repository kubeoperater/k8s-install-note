**1.prometheus dingtalk 发送报警:**
-  promtheus 配置：

``` yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: alertmanager
  namespace: kube-system
data:
  config.yml: |-
    global:
      resolve_timeout: 5m
    route:
      receiver: webhook
      group_wait: 30s
      group_interval: 1m
      repeat_interval: 1m
      routes:
      - receiver: webhook
        group_wait: 10s
    receivers:
    - name: webhook
      webhook_configs:
      - url: 'http://10.255.72.206:80/' #这里地址是dingtalk的部署地址,后期可以通过k8s docker化成服务然后配置动态发现。
        send_resolved: true
```

- app.py
	> 这里是python3的脚本

``` python
import io, sys

sys.stdout = io.TextIOWrapper(sys.stdout.detach(), encoding='utf-8')
sys.stderr = io.TextIOWrapper(sys.stderr.detach(), encoding='utf-8')

from flask import Flask, Response
from flask import request
import requests
import logging
import json
import locale
locale.setlocale(locale.LC_ALL,"en_US.UTF-8")


app = Flask(__name__)


console = logging.StreamHandler()
fmt = '%(asctime)s - %(filename)s:%(lineno)s - %(name)s - %(message)s'
formatter = logging.Formatter(fmt)
console.setFormatter(formatter)
log = logging.getLogger("flask_webhook_dingtalk")
log.addHandler(console)
log.setLevel(logging.DEBUG)


@app.route('/')


def index():
    return 'Index Page'

@app.route('/<profiledd>/send/',methods=['POST'])

def hander_session(profiledd):

    dingtalk_profile = dict(
        {
            'node' : 'https://oapi.dingtalk.com/robot/send?access_token=access_token',
            'test' : 'https://oapi.dingtalk.com/robot/send?access_token=access_token'
        }
    )

    if profiledd in dingtalk_profile:
        profile_url = dingtalk_profile[profiledd]
        post_data = request.get_data()
        post_data = json.loads(post_data.decode("utf-8"))['alerts']
        for i in post_data:
            try:
                try:
                    alert_status = i['status']
                except:
                    alert_status = "unkown!"
                try:
                    alert_name = i['labels']['alertname']
                except:
                    alert_name = "no alertname"
                try:
                    startat = i['startsAt']
                except:
                    startat = "no startsAt"
                try:
                    summary123 = i['annotations']['summary']
                except:
                    summary123 = i['annotations']['message']
                try:
                    instances123 = i['labels']['instance']
                except:
                    instances123 = i['annotations']['description']
                try:
                    describetions = i['annotations']['description']
                except:
                    describetions = i['labels']['instance']
                log.info(i)
                messa = ''' > 通知类型: **%s** \n\n 服务: %s \n\n 时间: %s \n\n 主机:%s \n\n附加信息: %s \n\n %s''' % (alert_status,alert_name,startat,instances123,describetions,summary123 )
                status = alert_data(messa, summary123, profile_url)
                log.info(status)
                return status
            except Exception as e:
                log.error(repr(e))
                return  repr(e)

def alert_data(data,title,profile_url):
    headers = {'Content-Type':'application/json'}
    send_data = '{"msgtype": "markdown","markdown": {"title": \"%s\" ,"text": \"%s\" }}' %(title,data)  # type: str
    send_data = send_data.encode('utf-8')
    reps = requests.post(url=profile_url, data=send_data, headers=headers)
    return reps.text

if __name__ == '__main__':
    app.debug = True
    app.run(host='0.0.0.0', port='8080')
```

- dockerfile

``` bash
FROM hub-dev.example.com/prometheus/ubuntu:16.04 #这里就是使用的官方的ubuntu16.04的镜像
MAINTAINER sadlar sadlar@126.com

#添加python
RUN apt-get update &&  apt-get install -y python3 python3-pip locales tzdata net-tools
RUN pip3 install flask
RUN pip3 install requests
RUN locale-gen  en_US.UTF-8
RUN echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && echo 'LANG="en_US.UTF-8"'>/etc/default/locale && echo 'LC_ALL="en_US.UTF-8"'>/etc/default/locale && dpkg-reconfigure --frontend=noninteractive locales && update-locale LANG=en_US.UTF-8
ADD app.py /usr/local/dingtalk_pyproxy.py
EXPOSE 8080
CMD ["/usr/bin/python3", "/usr/local/dingtalk_pyproxy.py"]
```
