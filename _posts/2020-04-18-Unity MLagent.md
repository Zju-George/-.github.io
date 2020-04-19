---
layout: post
title: "Unity3D ml-agents"
---
#### <center>先简单描述一下版本问题</center>
我用的 ml-agents 版本是 [1.4.1](https://github.com/Unity-Technologies/ml-agents/tree/0.14.1)，原因是 1.5 好像有点 bug，具体我在这个 [issue](https://github.com/Unity-Technologies/ml-agents/issues/3763#event-3228106437) 里描述了。

然后因为我的 cuda 版本是 10.1, 但是 tf 2.1 才支持 cuda 10.1，2.0 只支持 cuda 10.0，我懒得 downgrade cuda 了 :pensive:，所以用 tf 2.1。问题在于， ml-agents 1.4.1 要求 tf 版本小于 2.1，参考 `setup.py` 中第 [64行](https://github.com/Unity-Technologies/ml-agents/blob/89b5959128a91b5aceb46764ff2867cf49aa2cb6/ml-agents/setup.py#L64)）。

经测试 tf 2.1 可行，所以我修改了 `install_requires` 变量，即 `"tensorflow>=1.7,<=2.1"`，但是改完了还需要重新 `pip install -e .` 一下，因为 check 的时候应该是对 pip setup 后的二进制文件 `*.lib` 等来 check 的。有点像修改了 `.bashrc` 还需要 `source` 一下才生效。

<a href="#xinde">一些心得</a> (页面锚点，免得下拉麻烦)

### 环境安装

具体安装 1.4.1 的[文档](https://github.com/Unity-Technologies/ml-agents/tree/0.14.1/docs)已经写得蛮详细了，这里仅仅提一嘴：

推荐建立 virtual env，这样在任何路径下都能执行 `mlagents-learn`。但是还是建议 cmd 的 workingdir 就是 `$YOUR_MLAGENTS_REPO`，因为进入 virtual env 就用 default 的命令就行，例如

```text
D:\unityProjects\ml-agents-0.14.1> python-envs\sample-env\Scripts\activate
``` 

(***好吧主要还是因为我记不住这行命令*** :joy:)


然后训练调用训练配置 `config.yaml` 反正也在 `config/` 底下，蛮方便的。

好了，without further ado，进入正题。

{% comment %}
Might you have an include in your theme? Why not try it here!
{% include my-themes-great-include.html %}
{% endcomment %}

---

### GPU训练与CPU训练对比

**我的渣渣 GPU：**
```js
pciBusID: 0000:01:00.0 name: Quadro P2000 computeCapability: 6.1
coreClock: 1.4805GHz coreCount: 8 deviceMemorySize: 5.00GiB deviceMemoryBandwidth: 130.53GiB/s
```

**训练配置：**
```
trainer:        ppo
buffer_size:    100
batch_size:     10
beta:   0.005
epsilon:        0.2
hidden_units:   128
lambd:  0.95
learning_rate:  0.0003
learning_rate_schedule: linear
max_steps:      5.0e5
memory_size:    256
normalize:      False
num_epoch:      3
num_layers:     2
time_horizon:   64
sequence_length:        64
summary_freq:   1000
use_recurrent:  False
vis_encode_type:        simple
strength:   1.0
gamma:      0.99
summary_path:   RollerBall-1_RollerBallBrain
model_path:     ./models/RollerBall-1/RollerBallBrain
keep_checkpoints:       5
```

**训练log:**

(对不起我应该用 tensorboard 可视化的 :joy:)
```
INFO: Step: 1000. Time Elapsed: 14.591 s Mean Reward: -0.574. Std of Reward: 0.819. Training.
INFO: Step: 2000. Time Elapsed: 27.266 s Mean Reward: -0.404. Std of Reward: 0.915. Training.
INFO: Step: 3000. Time Elapsed: 40.777 s Mean Reward: -0.014. Std of Reward: 1.000. Training.
INFO: Step: 4000. Time Elapsed: 54.035 s Mean Reward: 0.111. Std of Reward: 0.994. Training.
INFO: Step: 5000. Time Elapsed: 66.390 s Mean Reward: 0.714. Std of Reward: 0.700. Training.
INFO: Step: 6000. Time Elapsed: 79.290 s Mean Reward: 0.900. Std of Reward: 0.436. Training.
INFO: Step: 7000. Time Elapsed: 91.834 s Mean Reward: 0.885. Std of Reward: 0.466. Training.
INFO: Step: 8000. Time Elapsed: 104.384 s Mean Reward: 0.904. Std of Reward: 0.428. Training.
INFO: Step: 9000. Time Elapsed: 117.229 s Mean Reward: 0.955. Std of Reward: 0.298. Training.
INFO: Step: 10000. Time Elapsed: 131.042 s Mean Reward: 0.903. Std of Reward: 0.429. Training.
INFO: Step: 11000. Time Elapsed: 143.494 s Mean Reward: 0.899. Std of Reward: 0.438. Training.
INFO: Step: 12000. Time Elapsed: 157.132 s Mean Reward: 0.933. Std of Reward: 0.359. Training.
INFO: Step: 13000. Time Elapsed: 169.695 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 14000. Time Elapsed: 182.705 s Mean Reward: 0.969. Std of Reward: 0.246. Training.
INFO: Step: 15000. Time Elapsed: 195.495 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 16000. Time Elapsed: 208.488 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 17000. Time Elapsed: 221.403 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 18000. Time Elapsed: 234.396 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 19000. Time Elapsed: 248.091 s Mean Reward: 0.983. Std of Reward: 0.186. Training.
INFO: Step: 20000. Time Elapsed: 261.451 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 21000. Time Elapsed: 274.961 s Mean Reward: 0.984. Std of Reward: 0.175. Training.
INFO: Step: 22000. Time Elapsed: 288.215 s Mean Reward: 0.970. Std of Reward: 0.242. Training.
INFO: Step: 23000. Time Elapsed: 301.342 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 24000. Time Elapsed: 314.636 s Mean Reward: 0.842. Std of Reward: 0.539. Training.
INFO: Step: 25000. Time Elapsed: 328.529 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 26000. Time Elapsed: 342.203 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 27000. Time Elapsed: 356.155 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 28000. Time Elapsed: 369.322 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 29000. Time Elapsed: 382.733 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 30000. Time Elapsed: 395.683 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
INFO: Step: 31000. Time Elapsed: 408.694 s Mean Reward: 1.000. Std of Reward: 0.000. Training.
```
因为小球找到 Target 我设置的 reward 就是 **1**，所以 `Mean Reward` 到 **1** 基本就可以了 :smile:

**结果**：

(***来看看这个被外力驱动的小球吧*** :joy:)
<center><img src="/assets/roller_test.gif" alt="Result" width="100%" height="100%" align="center" /></center>

---
<br>

**我的渣CPU：**

<img src="/assets/CPU.png" alt="CPU" width="60%" height="60%" align="center" />

**训练log:**

(加上 flag `--cpu` 更多 flag 见[文档](https://github.com/Unity-Technologies/ml-agents/blob/release-0.14.1/docs/Training-ML-Agents.md))
```console
INFO:Step: 1000. Time Elapsed: 10.179 s Mean Reward: -0.607. Std of Reward: 0.795. Training.
INFO:Step: 2000. Time Elapsed: 19.446 s Mean Reward: -0.299. Std of Reward: 0.954. Training.
INFO:Step: 3000. Time Elapsed: 28.938 s Mean Reward: -0.172. Std of Reward: 0.985. Training.
INFO:Step: 4000. Time Elapsed: 38.254 s Mean Reward: -0.052. Std of Reward: 0.999. Training.
INFO:Step: 5000. Time Elapsed: 47.259 s Mean Reward: 0.237. Std of Reward: 0.971. Training.
INFO:Step: 6000. Time Elapsed: 56.627 s Mean Reward: 0.426. Std of Reward: 0.905. Training.
INFO:Step: 7000. Time Elapsed: 66.128 s Mean Reward: 0.619. Std of Reward: 0.785. Training.
INFO:Step: 8000. Time Elapsed: 75.180 s Mean Reward: 0.532. Std of Reward: 0.847. Training.
INFO:Step: 9000. Time Elapsed: 83.878 s Mean Reward: 0.511. Std of Reward: 0.860. Training.
INFO:Step: 10000. Time Elapsed: 92.747 s Mean Reward: 0.760. Std of Reward: 0.650. Training.
INFO:Step: 11000. Time Elapsed: 102.161 s Mean Reward: 0.776. Std of Reward: 0.631. Training.
INFO:Step: 12000. Time Elapsed: 111.707 s Mean Reward: 0.687. Std of Reward: 0.727. Training.
INFO:Step: 13000. Time Elapsed: 120.681 s Mean Reward: 0.870. Std of Reward: 0.492. Training.
INFO:Step: 14000. Time Elapsed: 129.833 s Mean Reward: 0.921. Std of Reward: 0.390. Training.
INFO:Step: 15000. Time Elapsed: 138.806 s Mean Reward: 0.859. Std of Reward: 0.513. Training.
INFO:Step: 16000. Time Elapsed: 148.039 s Mean Reward: 0.667. Std of Reward: 0.745. Training.
```
果然开发者说 ppo 算法没有对 GPU 进行优化是真的。。好像 CPU 是快一点。

**结果**：

(***刻意没训练完全，看看是什么效果*** :joy:)
<center><img src="/assets/roller_test_cpu.gif" alt="Result_cpu" width="100%" height="100%" align="center" /></center>

<a name="xinde"></a>

## 一些心得（持续更新）

* `Agent` 类自带的 `Max Step`，step 可以看成是实际的一帧，和 `CollectObservations()`, `AgentAction(float[] vectorAction)` 的调用频率是不一样的。可用如下代码结合环境 Reset 时两者的步数进行简单测试：

    ```c#
    private int stepnum = 0;
    public override void CollectObservations()
    {
        stepnum++;
        Debug.Log("执行了 " + stepnum);

        // Your Collect Obeservation Code...
    }
    ```

    后者频率就跟 gym 里的 `env.step()` 一样。那么两者的比值是根据 `Decision Period` 决定的。

    <img src="/assets/decision%20period.png" alt="Decision Period" width="80%" height="80%" align="center" />

    同时官网文档里提到，如果环境不太复杂，一般设置比较大的 Decision Period——对应更小的决策频率——训练收敛会更快。

*  TODO