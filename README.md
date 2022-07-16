# Reinforcement-Learning-in-Open-Environment
Reinforcement Learning in Open Environment, for course: Directed Research in AI System (I) in summer 2022.



# 开放环境下的强化学习算法——以Minecraft为例
## MineDojo环境配置

经过无数次的尝试和踩坑，终于成功。感谢袁昊琦学长、陆泳丹学姐和谢飞扬同学的帮助。本次采用博雅一号集群的notebook模式配置镜像。
### Step 1 创建指定Python版本的conda环境
创建一个含有>=3.9版本python的环境，命名为minedojo：

```
root@q7dfbb438c1e4f3aa5647fa1149e78d5-task0-0:/workspace# conda create -y -n minedojo python=3.9
```
然后刷新conda环境（如果有需要的话）：
```
root@q7dfbb438c1e4f3aa5647fa1149e78d5-task0-0:/workspace# conda init
```
此时已经可以看到目前处于base环境，说明minedojo环境创建成功。接下来转到minedojo环境中：
```
(base) root@q7dfbb438c1e4f3aa5647fa1149e78d5-task0-0:/workspace# conda activate minedojo
```
此时可以查看并验证一下python版本：
```
(minedojo) root@q7dfbb438c1e4f3aa5647fa1149e78d5-task0-0:/workspace# python --version
Python 3.9.13
```

### Step 2 在新环境中安装JDK8和vnc4server
按照minedojo官方文档（https://docs.minedojo.org/sections/getting_started/install.html ）的流程逐步操作。先是
```
apt update -y && apt install -y software-properties-common && \
    add-apt-repository ppa:openjdk-r/ppa && apt update -y && \
    apt install -y openjdk-8-jdk
```
此时可以查看并验证一下java版本：
```
(minedojo) root@q7dfbb438c1e4f3aa5647fa1149e78d5-task0-0:/workspace# java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-8u312-b07-0ubuntu1~20.04-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
```
然后转到 /etc/apt 中并修改文件sources.list：
```
mv /etc/apt/sources.list /etc/apt/sources.list.backup
vi /etc/apt/sources.list
```
这时会进入etc/apt/sources.list的页面，由于move指令已经剪切粘贴，所以此时文件内容为空。按键盘上的```Insert```键进入编辑模式，并在文件中先加入清华镜像
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```
再加入
```
deb http://archive.ubuntu.com/ubuntu/ bionic universe
```
此后可按```Esc```键退出编辑模式，并输入```:wq```保存并退出。之后更新一下apt-get：
```
apt-get update
apt-get upgrade
```
再安装vnc4server：
```
apt install xvfb xserver-xephyr vnc4server python-opengl ffmpeg
```
最后安装minedojo库：
```
pip install minedojo
```
注意不要按照官方文档中的```git clone```方式安装最新版minedojo，否则可能会产生一些bug。

### Step 3 检验是否安装成功
上传文件```validate_install.py```（默认应该在 ```/code``` 中），并运行之。注意集群notebook应当看作headless machine。
```validate_install.py```的代码内容如下：
```
import minedojo


if __name__ == "__main__":
    env = minedojo.make(
        task_id="combat_spider_plains_leather_armors_diamond_sword_shield",
        image_size=(288, 512),
        world_seed=123,
        seed=42,
    )

    print(f"[INFO] Create a task with prompt: {env.task_prompt}")

    env.reset()
    for _ in range(20):
        obs, reward, done, info = env.step(env.action_space.no_op())
    env.close()

    print("[INFO] Installation Success")
```
运行指令如下：
```
cd /code
xvfb-run python validate_install.py
```
如果出现
```
[INFO:minedojo.tasks] Loaded 1572 Programmatic tasks, 1558 Creative tasks, and 1 special task: "Playthrough". Totally 3131 tasks loaded.
/opt/conda/envs/minedojo/lib/python3.9/site-packages/gym/spaces/box.py:73: UserWarning: WARN: Box bound precision lowered by casting to float32
  logger.warn(
[INFO] Create a task with prompt: combat a spider in night plains with a diamond sword, shield, and a full suite of leather armors
[INFO:minedojo.tasks] Loaded 1572 Programmatic tasks, 1558 Creative tasks, and 1 special task: "Playthrough". Totally 3131 tasks loaded.
[INFO] Installation Success
```
则说明安装成功。
