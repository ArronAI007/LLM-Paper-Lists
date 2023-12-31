# 训大模型的一份避坑指南

**以下为嘉宾讨论精华整理。欢迎在小宇宙搜索并关注我们的同名播客 OneMoreAI，聆听全部讨论内容。**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/yl2uL5YILohpX57dnNMYiaT1GbAfK58S25fhCZicS4A56IzwgydW1sRicKOnVtMf3LDnabektcJVZf0CfXSSXQ2wQ/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

#  

# **Part 1：**

# **当我们讨论大模型时其实是在讨论什么**

------

**Kiwi**：很好奇大家眼里的大语言模型该如何去定义呢？

**冠叔**：从产品经理的视角，现阶段它的模型类型应该属于语言模型，其次“大”的描述主要是指模型的体积和参数量。现阶段可能得**超过千亿级别的参数才能被称为大模型**，不然我们一般就叫它预训练语言模型了。

**Kiwi**：这里千亿级别的参数需要去分稀疏或者稠密吗？

**欣然**：我认为**在 NLP 上，只要是能够有一定的涌现能力，都可以叫大模型，参数量不是很重要**。至于稀疏还是稠密，现阶段一般大家都还是考虑稠密。稀疏更多是一种优化手段，现阶段还没有太多从诞生之初就是完全稀疏的语言模型。

**Kiwi**：从涌现能力的角度去切“大”这个范畴是一个很有意思的观点。涌现能力何时出现，会有一个经验的参数数值吗？

**欣然**：之前看的大概是参数超过100亿就会有涌现能力。

**Kiwi**：请预训练算法经验丰富的龙老师来介绍一下他定义的大模型是什么。

**龙老师**：正好这两天看到 Twitter 上有一个投票，问大家多少规模算大模型。我记得差不多90%的人认为需要到100 billion，也就是千亿级别。但是我的看法不太一样，我可能更多是从算法实践角度去看。**许多算法工程师尝试更大参数量的模型，第一个阻碍就是有没有模型并行**。一旦有了模型并行，难度就会陡增，对大多数算法工程师来说这是一个没有摸过的事情。如果训练框架支持模型并行的话，那实际上后面只是加参数量以及加算力规模的事情。所以大概会在3 billion 左右吧，取决于拿到什么样的机器。

# **Part 2：**

# **大模型是如何炼成的？**

------

**Part 2-1：想训大模型？这里有一张入场费账单**

**Kiwi**：假设我们今天的讨论聚焦在千亿量级的 GPT 架构的大语言模型，算力成本是多少？

**欣然**：先分享一个观点，练大模型不是简单的堆人，盯着一个指标搞就可以。**训大模型是类似于火箭发射的大规模系统工程**，像机器互联、访存优化，模型参数存储等都有许多困难。团队的工程水平会导致成本巨大波动，甚至差出一个数量级。所以可以聊算力，但是不能简单的唯算力论。GPT-1、 GPT-2 和 GPT-3，基本上每一代都是翻个百倍左右的量级：

- GPT-1 ，用 A100 需要0.1个卡/年 (1张卡算0.1年）完成一次训练，非常快。卡多一点可能几天就训出来了；
- GPT-2 就已经到了 6.81个卡/年，这也还好。咱们训练的时候都是八卡甚至更多卡，看起来也是很短时间能训出来的
- 到了 GPT-3 ，它差不多又大了100倍，所以是400-500个卡/年，大概是这么一个量级。

在这种情况之下，咱们简化问题，不计算人力成本，非常简单的用每张显卡一年多少钱来计算。假设不买显卡，租公有云，现在 8 张 A100 包年的价格大概一年 80 万。如果关系比较硬，再加上一次性量走的比较多，经常是能够打半折的。假定我们这8张卡一年就花 40 万租金，其他东西全都送我们，相当于 1 个卡/年是 5 万人民币。最后再假设我们可以完美利用这些算力，那简单计算下来：

**一次性训出 GPT-1 的成本大概是几千块钱，GPT-2 大概是 30万人民币，训练GPT-3大概是 2500 万人民币。**

**Kiwi**：那也就是说训练一个接近 175 billion 的 GPT-3 量级的模型大概需要 2000 多万人民币。这里其实是按照 GPU  100%的利用率来计算的。但事实上在大规模的工程训练中，我们知道整个 GPU 算力的有效利用率是非常低的。这里龙老师有一个经验数值吗? 假设我们用 500 到 1000 张卡去做训练，有效算力大概可以提升到什么水平？

**龙老师**：这里面其实有几个比较大的问题点。第一个大问题就是显卡不像大家笔记本里面的显卡或台式机显卡那么稳定，当卡的数量到几百或者上千级别的时候，几乎每天都会遇到有卡直接挂掉。那训练就会被迫暂停，需要去换一台机器。

**Kiwi**：我有听团队说，他们刚开始训练千亿模型的时候，GPU打太满，然后一台 A100 一天挂两次。

**龙老师**：对，所以这就是第一个比较麻烦的点，显卡的质量问题。比如说你的电力供应不是很稳定，那就经常挂，这个时候会导致你不得不接着训练，那就会产生另外一个问题，多久做一次checkpointing？一分钟一次肯定不现实，像GPT-3 这种级别的模型，可能它有一个checkpoint，除了它的parameter，可能还有一些中间状态，可能就需要 2T 或者 3T。

按 2T 去算的话，如果网络 I/O 又不是很好，比如磁盘速度不是很好的话，按 Hugging Face的数据，可能到几分钟，如果出问题的话，可能十几分钟，这又是一种浪费。所以说 GPU 永远是有被浪费的时候，这完全取决于使用的硬件的情况。

然后在算法上其实就还好，大家会用各种办法尝试做pipeline，然后把这个相差给打的很满，但其实这里面 CUDA core 大多数情况下是用不满的，更多是在等各个环节的 I/O。比如显存带宽的I/O、 IB网络的I/O等等。如果大家从Nvidia-smi去看的话，可能感觉是跑满，其实并没有。

**Part 2-2：如何训练大模型效率会更高？**

**Kiwi**：刚才听到我们在训练过程中一个非常大的瓶颈是在I/O，也就是在通讯上。底层用什么样的硬件架构对于训练大模型其实至关重要，那 Google 用 TPU 是不是非常有优势呢？

**欣然**：I/O 其实跟芯片没关系。I/O 其实就是纯粹的说，你有多大的内存通讯带宽。基本上就两套技术，DDR 和 HBM，无非是这两套技术怎么组合而已。显卡上用的比较多的是 HBM 。TPU 因为他们之前关注的还是成本，便宜比较重要，所以用DDR 比较多。HBM 的带宽速度很快，DDR 会慢很多，但是一般会放很多 DDR 把带宽给凑上去。所以其实在 I/O 这件事上，用TPU、GPU还是其他差别不算太大。

但可能我这边的认知跟龙老师还有点不太一样，如果科学的去做 pipeline ，其实卡 I/O 应该没有那么严重，我们之前做过 GPT-2 的训练，当时通过各种各样的优化，把整个 I/O 的瓶颈基本都消掉了，绝大多数情况应该都在计算。所以这其实是花多少精力做工程优化的一个问题。我们之前很多花了差不多三四个月，才把优化做好。即便这样，当时整个算力的利用率能到 40%-50% 。

**冠叔**：刚才欣然估算成本的时候有一个假设，就是一轮跑完。我们也看到 OpenAI 在论文中提到 GPT-3 总共训了 4 轮。什么原因会导致算法训练需要这么多轮次？从哪些角度可以尽量去减少训练的轮次？因为看起来训练轮次会让成本成倍的增加。

**欣然**：一轮的训练周期很长，可能上月，而且成本很高。所以实际上并不是真的一把梭，直接开始训。绝大多数其实会不断的去看训的怎么样。有很多理论去教会你怎么三岁看老，或者在一个更小规模的模型上去验证一些设定的效果。如果我没记错的话，OpenAI 训模型的时候，也都是小规模的先跑一跑，看一看模型状态是否 OK，如果不 OK 就赶紧关掉，或者回退一段时间再重新训练等等。

所以这儿其实还产生了另外一个潜在的成本。如果我上来就租几千块卡放在这儿，然后发现早期可能在不断的做小规模实验，显卡并没有充分利用起来，这一部分其实也会有相当长的一个成本。

**Kiwi**：这符合龙老师的经验吗？

**龙老师**：欣然总结的挺全的，我就补充一小点吧，大家发现很多小规模的实验都挺好的，但一到 100B 这个级别就会发现各种 loss 的不收敛。或者说训练到一半的时候，loss 突然就猛增、飞掉，然后后面再也没法收敛，在 Hugging Face 以及在 Meta 实验当中都观察到这个现象。最后大家的策略可能就是回退几步，或者扔掉这一部分数据，然后接着往前走。

**Kiwi**：但是我有在一些分析文章中看到，回退的次数过多也会导致最后的模型效果不如预期，龙老师在实际的训练过程中有遇到类似的情况吗？

**龙老师**：Hugging Face 那边微观的去看了其中一次现象，是在数据整理过程当中发现了一条样本，这条样本可能大概是上万个字符，所有字符都是一个正斜杠，还是一个反斜杠，导致它的梯度直接乱掉了，后面就再没法收敛。后来他们的策略就是找到这种脏数据给删掉。还有一个坑，FP32、FP16还有BF16这个问题，其实大家好像还没有太好的结论。FP32 大家肯定训练的挺好，但是太贵了。Hugging Face 选择 BF16 稍微好一点，META 是FP16 搞定了。所以这也是一个问题。

**Kiwi**：所以说不一定必须要用 FP16，其实 BF16 也能去训千亿的大模型，并且得到一个收敛的模型是吗？

**龙老师**：我更倾向 BF16 吧，因为首先 Google 做 T5 一直是拿 BF16，看起来会比较稳定，然后 Hugging Face 去试了一圈以后也是决定 BF16 更好收敛。

**欣然**：这个我知道一个非常有意思的事情。之前我们训 GPT-2 的时候感觉也是这样，用 FP16 确实速度快了很多。但是需要派一个人，这个人叫“崩不崩观察员”，他就每天看着这个模型，如果突然发现模型精度爆了，就赶紧停掉重启。

**Kiwi**：用人盯着模型崩不崩，没有像 Weights & Biases 的工具可以代替实现吗？

**欣然**：基本的没什么问题。重点是崩完之后，他得去看历史数据，决定怎么往前回退，回退多少。这些都很玄妙，也是实验的一部分，现在并没有成为非常有效的经验，所以还是需要一个非常专业的人在那儿盯着，也许等未来越来越多的人在训练，可能会形成一套自动化的工具，但现阶段还不行。

**冠叔**：我理解这个角色有点像老中医，号号脉之后就知道怎么做。这个角色通常是一个偏工程的人员，还是一个算法研究员，还是得两者兼具？

**欣然**：我的经验是肯定得两者兼具，当然算法的属性会多一些，因为绝大多数的问题都是算法上的问题，相对来说工程上的问题一般坏的都比较彻底，比较容易判断。

**Kiwi**：刚才在讨论算力成本的时候，都是基于 GPT-3 的实验在讨论，但事实上前段时间 Meta 发布了LLaMA-13B 模型，我们发现百亿模型其实也可以达到很好的效果。LLaMA 的文章里有提到，它训练的 token 数是远高于之前类似于 GPT-3 的训练token数的。

这里就会涉及到选择问题。在前期训练的时候，我可以选择压成本，比如说限制训练的数据量和 token 数，最终在目标效果下得到一个千亿量级的模型。另外一种选择是 fix 前期投入，就是固定投入非常高，可能把token数量和数据量都增加，然后可能有一种 overtrain 的方式，但是我可能得到了一个参数量级更小的，可以达到效果预期的模型，但换来的就是 inference 的成本可以降低很多。

从三位的经验来说，我们会如何权衡这之间的几个数值呢？

**龙老师**：首先 Meta LLaMA 这篇文章一个最主要的启示是说，在相同预算的情况下，可能能达到类似或更好的效果，而这件事情跟参数量不完全是正相关。但又因为它的实验其实缺少一个很严格的跟 GPT-3 的对比，毕竟是两家单独做的，很多细节会不一样，所以只能说大家现在观察到这么一个现象。比如只用不到100B的参数量，然后训练的量从 300B token 提到1.4T token，这样也许会达到类似的效果，但是会不会对其他各种细节，比如各种评测以及下游任务的效果有什么影响，这件事情我也在观察。这个模型前两天刚开放一批下载，放出来之后各家去做评测，去做各种各样任务才知道这到底会不会有问题。否则的话，我现在可能更倾向于保守的还是沿着 GPT-3 这个路线去走。

**Kiwi**：现阶段我们都知道A100，甚至未来 H100 的采购在中国大陆会受到很多限制，可能接下来大家不得不选择 A800 去训练。用 A100、A800 或者 H100 去训练，具体会有哪些差异呢？

**欣然**：**A100和 A800 其实算力都一样，最大的差别是卡之间的通信带宽，从 600GB/s变成 400GB/s。**当我们用上百、上千，甚至上万块显卡做这么大规模训练的时候，通信的主要成本是机器间的通讯。现在就是机器内的通信低了一点。如果我们用 A800，通过更多的工程人员去精心调教并行策略和分布式训练，其实是能够把 200G 的差别给隐藏掉的。

这件事儿对于国内追赶国外，短期来看影响有限，但长期来看可能会非常可怕。H100 的通信带宽会上升到 900GB/s。我们如果还是 400，它是 900，那这是一个非常猛的.更可怕的是 H100 的算力，就是它的 FP16 和 BF16 已经到达了 1979TFLOPs，而 A100、A800 还在 100~300 多 TFLOPs 这个级别。那一下高了四五倍，当然也看具体配置。算力差了六倍，然后通信带宽也差了两倍，那这个其实是非常可怕的。

**冠叔**：假使我们分布式训练的技术，包括工程方面的一些经验，都是基于一个落后的显卡去做的，当有一天我能用上更好的卡的时候，之前积累的分布式训练、工程这些经验，有多大比例能够被复用过来？

**欣然**：这个比例很很玄学，因为这其实是知识，可能大概有一半吧。但是如果一直用低端卡，比如我们团队之前训练 GPT-2 是拿 2080Ti 训练的，用了 512 张卡。如果当时我们能用 A100，只需要两台机器就 OK 了。这个差别是巨大的。之前我们精心调教的一些分布式的经验，怎么把模型切的碎碎的，在这些显卡上并行的去跑等等这些事情，都没用了。这些知识、经验一定是有用的，但是真的能直接 transfer 过来用的应该不太多，**什么东西在十倍这种量级的差别下都会灰飞烟灭**。

**Kiwi**：刚才也提到分布式训练、模型切分、并行计算的一些问题，龙老师在训练的时候具体有遇到过哪些卡点，然后有一些比较好的解决方法吗？

**龙老师**：这其实是一个非常痛苦的事情。**现在可能最好的并行计算方案核心还是基于 Megatron 那套框架**。但是研究员们更喜欢用 Pytorch 去反复调它的模型架构。而把 Pytorch 代码移到 Megatron 上是比较辛苦的一件事情。那就导致出现中间出现一个 gap，做分布式训练人就说你不要调结构，我们就 Megatron 支持 Bert、T5和GPT。改也小改，别大改，因为整套这个代码都不一样。研究人员可能更倾向于说我现在小的调好，将来大了也管用。但实际上现在这个假设是不成立的。涌现直接导致很多的算法实验在小规模上是做不出来的，或者说永远也看不到。这件事就会导致大家很痛苦。那我们要不然选择一个分布式架构去做模拟实验。要不然你就放弃涌现能力去做小规模实验。在去年可能这件事还说得通，但今年这件事儿已经说不通了。

**Kiwi**：除了 Megatron 之外，其实还会有像微软的 DeepSpeed，或者像最近很热的一些开源项目，比如Colossal-AI ，这些开源方案能够帮助解决分布式并行训练的问题吗？

**龙老师**：可以。Megatron-DeepSpeed 是现在比较 SOTA 的一个方案。关于Colossal-AI ，目前还没有看到哪个开源项目把模型给出来了。

**Kiwi**：作为 researcher，假设龙老师今天要跟欣然去对接，除了并行训练，还会提哪些方面的诉求？

**龙老师**：其实就刚才我说的移植的活儿，比如说 researcher 可能用 Pytorch 去验证，比如说各种模型结构、 tokenizer 上一些细微的调整。这些调整其实都要翻译到 Megatron 这套框架上。实际上是类似于研发的一个 pipeline 吧。

**Kiwi**：这些诉求可以做到很好的标准化服务吗？还是需要不停的堆人力和硬件工程师去服务 researcher？

**欣然**：就我觉得这个事儿可能得就另外看，就现在我看有很多的这创业公司，还有大公司都在说我们疯狂堆人进项目，但是其实现在大模型训练这件事情并不是一个非常清晰的分工。很难说我有几个算法的研究员，或者我有几个工程人员，大家怎么一结合，就可以一步一步做出来。**现阶段更倾向于算法人员和工程人员大家彼此知识是交融的，坐下来一起去讨论如何去实现。**

举一个例子，刚才龙老师提到 Megatron，我们之前复现 GPT-2 的时候就是自己重新做了一套类似于 Megatron 的东西。整个模型切分的时候，很多细节都要大家联合应对，所以它应该说是标准化的另外一个极端。就恨不得就是一两个人，非常少量的人坐在一起集中把它搞定。这也是为什么现在国内大家想要快速复现都这么难的一个原因，它不是标准化的。

**Kiwi**：听起来如果我们要去训练一个大模型，现阶段还是比较混沌的分工状态，这对整个项目管理和产品端的要求很高，在这件事情上冠叔会怎么去看整个大模型研发的过程管理？我们怎么去建立一些checkpoints 去控制大模型研发中的风险呢？

**冠叔**：这个问题目前国内大家可能都没有太多的经验，所以我们更多的是可以去参考 OpenAI 。从公开信息去看，至少会包含三个大的方向，分别是数据、算法，以及训练和工程这样三个方向。每个方向去看他们具体的工作，也能够再拆分出一些更细的模块。比如说数据，其实就能够分为面向预训练模型所使用的预训练 data，也包括说因为像 ChatGPT 它有 instruct tuning 这样一个非常重要的环节，那 instruct data 也是一个很重要的数据建设模块，还有就是强化学习这一部分所使用的人工去排序和打分的数据。

数据的部分都会有搜集、治理及清洗这样一个系统的过程。像算法部分，我相信即使是 OpenAI 内部也会对整个算法的选型有很多的实验，最后才能确定如何去训出一个模型。确定完算法之后，因为需要用到上万级别的 GPU 做训练。如何更高效的去做分布式，如何去提升模型的训练速度以及像欣然提到的，中间很多复杂的系统工作，都是需要去完成的。所以整体来讲，通过 OpenAI 的公开信息，**我们可以认为大模型的研发未必涉及到很多人，但是它一定是一个非常系统化的工作**。

**Kiwi**：刚才有提到，如果去做一个类 ChatGPT 的产品，整个算法可能会有三个阶段，预训练、Instruct tuning 和 RLHF。预训练模型现在有哪些可选的开源方案呢？

**龙老师**：其实现在经过这几年的发展，包括开源社区的发展，其实 GPT 模型大家可选的就是 3.0 版本。可选的起点其实挺多的。有不同尺寸，从最早的GPT-Neo（125 million） 到 最大的GPT-J（20 billion）。这个范围可以做各种各样的实验去观察模型的效果，都是比较方便的。**Pile 数据集也被大家反复验证过，是比较好的一个数据集。**

如果要想到百 billion 的级别，可能现在只有两个，OPT（纯英文） 和 Bloom（多语言）。两个模型各有千秋，对两个模型的评价，也有一些争议。最近 Meta 的 LLaMA 模型，这个模型当然现在大家还没有很多的评估，但是也是比较有潜力的一个替代方案，它支持20多种语言，不过好像都是拉丁语系的。

**Kiwi**：稍微补充一下，刚才提到的 OPT 和 LLaMA 都是 Meta 发的模型，Bloom 是开源平台 Hugging Face 大家共同训练的模型。其实我个人会非常好奇 RLHF 的环节，因为我从不同的一些 researcher 口中有听到不同的观点，有的人觉得 RL 其实是 ChatGPT 成功的非常关键的要素，但是有的 researcher 说其实是不是使用了 RL 不重要，human feedback 才是 ChatGPT 在 Allignment 做的非常好的一个关键原因，这一点上想请问各位老师有什么的看法？

**冠叔**：从算法上去讲的话，把强化学习接到一个预训练模型，再去用强化学习的反馈重新优化调整预训练模型参数，这件事情并不是 OpenAI 的原创或者独创。从产品的视角去看，我倾向认为 ChatGPT 的成功更像是算法产品化之后的成功，它把用户的反馈加入到整个产品优化的系统里面，很像搜索点击率或者推荐准确率的提升，都是因为加入了用户的行为，所以会越来越好，这是我的观点。

**龙老师**：InstructGPT 论文里做的一些实验显示 finetune 带来的提升是比较明显，所以我猜会有很多人想是不是只做finetune 就可以，或者说已经达到八成的效果，然后强化学习这部分的两成，是不是可以先不那么着急。我觉得这是主要的一个争议点，但实际上，为什么强化学习这么重要？

强化学习在 OpenAI 最近十年的技术路线上扮演了非常重要的角色。另外从数据标注的角度，你会发现人工标注会变的越来越难，因为简单的标注大家都标完了。那什么简单呢？评价标注的好坏会变得很简单。现在模型拿到 finetune 的数据，可能已经达到平均标注员的水准了，等你想让它变得更好的时候，那可能去强化它，或者说去优化评价模型会来的更简单一些。所以说，达到八成效果呢，就是人工标注，然后纯做 finetune。**想走的更远的话，那只能靠强化学习**，这是我的观点。

**冠叔**：刚才提到人工标数据会越来越难，其实是说人工标注的结果已经逐渐和模型生产出来的数据差不多了，那为什么加入了强化学习之后，它为什么能够去解决这个问题呢？或者说为什么加入强化学习之后，它是怎么样让人工标注的数据在没有非常大提升的情况下，模型的效果会有提升呢？

**龙老师**：我这里有一个猜测，大家去玩 ChatGPT 的时候观察到一个现象，这个模型会给你一些看上去很对，但实际上有事实错误的答案，我猜大家都会碰到。可能的原因是 reward model 或判别模型很容易从句子结构、语法结构以及整个输出形式上来判断它的好坏。但是涉及到知识性东西的时候，这件事会变得越来越难，所以刚才我说走的更远是因为标注成本很高，那它可以标注更多样的数据。但是想完全超过人类，我觉得这个事儿还是有一定难度的。这其实是一个比较小的改进，所以我说两成和八成这个区别。

**冠叔**：OpenAI 的模型算法效果很好，生成出来的内容质量很高，那对于一些现在模型生成效果还不够好的一些组织，他们在去做模型的时候，是否可以直接使用 OpenAI 模型生成出来的内容去做数据？

**龙老师**：目前已经出现了一种说法，就是 **OpenAI in the loop，这已经是一个公开的想法**，并不是说少数人的想法。

**Part 2-3：训练中文大语言模型，你的数据够用吗？**

**Kiwi**：假设我们要在国内做一个千亿量级的大模型，很多人会提到中文数据的质量不如英文数据高，甚至有人认为可用数据中文只有英文的 10% ，这符合大家的认知嘛？当中文数据质量不高的时候，会影响我们去做中文的大语言模型训练吗？

**龙老师**：我们可以参考 EleutherAI 做的 Pile 数据集。Pile 里面，大家最容易获取到的网页数据占比其实不是很高。剩下的比如说像 arXiv 大量论文、Github、Stackflow、Stack Exchange 等等，我们在中文上好像都很难获取。这导致我们遇到一个困难，我们可能只用了英文集合中很小的一部分。大家可能只能用一些网页数据去做训练，在中文上可能就会有一些问题，比如说它不知道一些最新的 NLP 研究的一些概念，这些概念可能只在 arXiv 上会有。

![图片](https://mmbiz.qpic.cn/mmbiz_png/yl2uL5YILohpX57dnNMYiaT1GbAfK58S24nAticX5wvkDPgicL4O0xlBVwSDqXlJ2mic2ib4DToVcCxibnulSq1gf9hQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

*图片来源：The Pile: An 800GB Dataset of Diverse Text for Language Modeling*

中文的数字化我觉得是偏落后的，因为我知道的大多数有一定工作经验的工程师，可能只会看英文，不太会去看中文的信息了。这个数字化的落后可能会在大模型时代被放大。可能大家更倾向于去看英文模型的输出结果，而不是看中文模型的输出结果，所以对一个做中文的算法研究的人员来讲，这件事情可能会变得越来越难。

**Kiwi**：我们看到翻译模型现在的能力越来越好，有没有可能现阶段用翻译软件先去批量生成一些相对低质量一些的数据，然后去弥补我们在中文数据上的一个短板？

**龙老师**：也许一定程度上是弥补吧，但是说实话，专业领域的内容其实很难翻译，所以我觉得会有一些难。

**冠叔**：GPT-3预训练的语料数据是 45T，我相信和整个英文互联网的信息数据去比，45T 是非常小的一个子集，那这里面就有两个问题。第一个问题是对于中文来讲，为什么我们不能从中文的数据里也找出一个 45T 的质量还不错的数据？第二就是为什么 45T 这么大的一个数据包里面，数据质量就会很好，它好在哪儿了？

**龙老师**：它的 45T 最后清洗完是不到 1T 的。Common Crawl中文的数据，质量是非常非常差的。如果仅仅指望 Common Crawl 这种网页级别的数据化，基本很难训练出一个很好的模型，这也是为什么后来无论是 GPT Neo 、Bloom 还是 Meta 的这几个模型，其实都会大比例的加入高质量的人工生产内容，就是我刚说的论文、社区问答或者说一些文献。这些**高质量的内容才是让模型能够达到今天效果的关键**，这是我的个人观点。

**冠叔**：所以刚才这个问题可以理解为，首先即使是在英文世界里面，能够用来去训练预训练模型的高质量数据其实也不会很多，然后在中文的里面，偏知识型的高质量数据是严重不足的。

**龙老师**：是的，就是我刚说对着 PILE 那张表去看的话，最有价值的那部分在中文都很难找到。

**Part 2-4：训出大模型，人海战术可能并不好使**

**Kiwi**：我们刚刚讨论了大模型训练需要的算力、基础设施的要求、算法的一些具体细节，以及数据准备环节会遇到的一些问题。现在回到最关键的问题，训练大语言模型需要哪些人？这些人需要具备什么样的能力呢？

**龙老师**：我会考虑先去观察 Hugging Face，BigScience 这个项目的人员配置，因为无论是 OpenAI 还是 Meta，他们的论文虽然有，但是人员配置其实不那么透明。我们很难清楚里面的人是干什么的，而 BigScience 整个项目，从头到尾每一个人的身份都是很明确的，大家也能看到这些人是怎么分工的，包括他们每一次的周会也都有录像。

所以根据 BigScience 的经验，可以总结出几种类型。**数据这块大概就是大数据工程师加少量的法务人员**。大数据工程师可能偏数据工作，因为涉及到大量数据的预处理。法务人员可能观察一下比如说数据的 license 是否合理。然后剩下不超过 10 个的 NLP 算法工程师。他们可能更关心模型架构以及训练过程中所有的超参的选型

**训练这块的人可能做一些分布式系统**，然后把训练框架给支起来，协调、运维和管理这么多机器。这块可能是有一些算法的经验在里面，但可能更多是系统管理以及软件开发的一些经验。然后可能还需要少量的前后端开发，就是我们前面讲到就是 OpenAI 做 InstuctGPT，其实他们有很多工具在里面，尤其是数据的工具，这里面其实是需要一些前后端开发工作的，然后也包括做像之前 T0 做的 PromptSource工具，所以前后端开发，我猜也会各需要一到两个人。

**欣然**：我感觉很多人说我要训一个大模型，就像传统的一些比较大的项目，我要堆一大堆人，但其实**像 ChatGPT 这一类的东西，它最大的特点其实就是极少量的 idea 要指挥的动极大的资源**，所以我会觉得比较重要的其实就是一个算法的角色，这个算法的角色指挥着所有人去做。那相应的可能像刚才提到的，有数据团队，工程团队，这些人去支撑这件事儿。相当可怕的一件事情就是堆一大堆人，然后这堆人相互争权夺利，不断的相互 challenge，然后把一大份资源切成四五份去分别用，就将会是一个最可怕的事情，所以人员的精简是非常重要的一件事情。

**Kiwi**：这是一个挺有意思的点，那从需要的人力资源和整个团队效率的角度考虑，大概多大的团队比较合适？

**冠叔**：是不是可以直接参考 OpenAI 的配置？

**龙老师**：我觉得不太能参考，OpenAI 是远远在前面的，其他所有公司都是追随者。作为追随者，其实需要人会更少，尤其在算法上。因为对我们来讲，OpenAI 那篇 paper 其实都不能说叫 paper，叫算法文档或使用文档，我们照着文档能够模仿出来，这件事情对其他所有公司来说已经是一件很难的事情了，所以我觉得人力规模应该不太需要超过十个人。

**欣然**：现在国内竞争如此激烈，数据团队和整个集群管理是自建还是外包可能是影响人员规模的一个重大因素，至于具体的几个人去训练模型，去维护整个分布式系统的可靠，这些反而用的人可能非常少，我猜四五个人就够了。

# **Part 3：One More Thing，嘉宾****的互问互答**

------

**Q：会出现训练 Transformer 的专用框架吗？**

**冠叔**：开源框架是算法生产的一个非常底层的工具，对于 Transformer 这种架构来讲，未来有没有可能出现一个专有框架，让 Transformer 训起来很爽，或者说在这样一个框架基础之上，它可能会长出非常方便的去训 ChatGPT这样带 instruct tuning、预训练模型和 reward model 这样一套系统的工具？它会不会出现，或者说需要吗？

**欣然**：我理解在不同的层面上其实已经出现了，比如前面反复提到的 Megatron 就是英伟达觉得 Transformer 是下一个时代，所以要做这么一个工具去帮助你很快的做分布式训练。有没有可能会有人把它慢慢做的像 Stable Diffusion 一样，搞成一个开箱即用的工具，然后定向是做 ChatGPT 的，我估计慢慢的肯定会出来的。

**龙老师**：我比较看好两波。一波就是欣然说的，英伟达肯定会参与其中去推动框架和它的硬件的绑定程度。另外一个需要观察的就是最近Hugging Face 和 AWS 合作。他们其实是做框架或做这种易用性框架的一个好手，在这方面其实已经储备了很多资源，比如说现在大家常用的 Tansformer 框架就是他们开发的。虽然都是一些小模型的，然后包括训练加速，推理加速以及他们在强化学习最近半年做了大量的这种课程，我猜都是在做一些储备。所以我猜未来可能小规模的训练，大家会用 Hugging Face 新出的框架。然后大规模训练用 Megatron 或者英伟达新出的框架。我是这么猜测的。

**欣然**：其实我还有一个预测。我觉得很明显，英伟达很早之前就往 Transformer 上押注了。他其实是把整个 Transformer 这些特定的结构现在直接做到了它号称比较通用性的芯片里边。

也就是说从H100开始，这个芯片里边直接就有为 Transformer 专门设计的集成电路。英伟达最近三四年往 Transformer 层面一直在努力，所以我预测，如果英伟达没有做一些降智操作的话，很有可能在未来三到五年内，英伟达会希望把整个 Transformer 的小规模的训练能做到单机八卡可以搞定的这种程度，我觉得这是一个非常有可能的事情，那在这种情况下，英伟达就非常有动力去给你做一个非常好用的工具，让人人都能做，我觉得这是一个非常现实的商业战略。

**Q：从投资人视角看来，为什么每家都要训出ChatGPT，这合乎逻辑吗？**

**欣然**：现在 ChatGPT 这么火，从投资人视角来看，为什么要这么炒 ChatGPT? 为什么如此在意每个人都要训练出 ChatGPT ？从投资人视角看来，这真的是一个合乎逻辑的投资吗？

**Kiwi**：这个问题我说说个人的观点，不代表机构。首先，我觉得大语言模型所学习的人类知识以及 ChatGPT 所提供的交互模式，创造了一种全新的人机交互界面。我们回顾科技过去 30 年的发展历史，会发现浏览器的诞生和智能手机的出现分别都创造了一种新的人机交互界面，它大大提高了人类社会对于信息获取、检索和利用的效率，都带来了一波平台性的范式转移。今天让我特别兴奋的，第一当然是大语言模型本身所表现出的学习推理能力，让我对人工智能未来的发展潜力有了更大期待。第二，我认为在这种新的人机交互界面的加持下，人类社会信息获取、信息检索、信息利用、内容生产以及知识创造，都会经历一波非常大的平台性机会。

**冠叔**：我可以从产品和市场角度做个补充。为什么大家要去追  OpenAI 或者 ChatGPT，因为它相比于 ChatGPT 上层的一些应用，或者说是一些工具厂或者开源项目/开源公司，它的确定性是更高的或者是最高的。大家都很明确一件事情，中国一定需要自己的一个大模型。不管是谁，肯定是有那么一家或者是几家会出来，但是上面比如说像去做开源工具的，或者说一些所谓中间层以及应用层的公司就不一定了。比如开源的项目，可能国内国外它是更互通的，那应用层来讲的话，你不管是去做 ToB 的 SaaS，还是去做一些 ToC 的产品，现在它都有非常高的不确定性。所以从投资人的逻辑来讲，可能就是要找一个非常确定的事情。

**Q：ChatGPT 会一统天下吗？**

**龙老师**：我有一个问题是想问所有人，因为大家背景不一样。就是在中国的这个市场下，半年或一年之后会只有一个 ChatGPT 类似产品，还是会有多个 ChatGPT 类似产品，就像云或者操作系统一样。

**Kiwi**：首先我觉得中国的大模型市场会跟美国有很大的差异，一个核心的差异点可能来自于中国现在大量业务的公有云化程度其实是低于美国的，而且中国在各种数据监管以及一些企业惯性的情况下，会导致很多在美国的一些 ToB 和 ToC 的服务没有办法在中国用公有云的形式来提供。这就会导致大量的大模型的商业化市场，其实可能会在一些 ToB 的私有云端，这就会导致头部垄断的可能性没有那么大。

但是同时又会看到，如果接下来会有一家类似于像 OpenAI 提供 ChatGPT 和 GPT-3 这种公有云的模式的话，其实可能云厂商会有一个非常强的优势，原因就在于其实目前我们看到大模型对于 inference 的整个算力要求还是非常高的，那么一些没有自有算力的小厂商，他可能在公有云的场景下面，远不如大厂有可烧的钱和成本优势去做市场占有，所以可能接下来的生态我觉得会有头部的两三家云厂商去提供一个大语言模型的  API 和其他的公有云服务，而有多家厂商能够去提供 ToB 的私有云的服务，这可能是我的一个预测。

**冠叔**：我跟 Kiwi 的观点基本上是一致的，但是我会认为这件事情可能也不光是国内的现状，包括之前 Sam Altman 也说他认为未来整个大模型行业，除了我们现在看到的就是像 OpenAI 这样去做底层基础模型服务。除了基建层，以及上面直接包这个 OpenAI 的 API 去做应用的企业之外，可能还会有一个很厚的中间层，里面可能会包含的一种角色就是非常多的垂直场景的预训练模型。

那这件事情我会认为在国内的发展趋势相对会更乐观，或者说是更符合他的这个预期，为什么呢？我们从 CV 时代的一些发展趋势其实就能够看到一个现象，很多的技术，国外它代表的是高度，中国代表的是技术应用的一个深度，这里面其实有一个非常底层的逻辑是在于体制优势，就是所谓的统筹规划能力。比如很多大企业都开始拥抱大模型了，这件事情他会很快的去做一个应用的普及。在这样一个应用普及的过程中势必会有个性化的需求。

这些个性化的需求如果没有办法在基础模型层一一得到满足，那他可能就需要有一些更定制化的算法出来。这个时候就会有中间层去做。以及因为你刚才提到了一个时间点，我觉得半年这个时间点可能会稍微有些短，因为目前我们处在的是一个叫千模大战或者是百模大战的一个阶段。这个阶段势必会带来一个结果，就是最终有少数几家会跑出来。那没有跑出来的这些企业或者是组织里他们的人，大家其实都是会训大模型的，那这些人他在市场上面其实会形成一个很强的技术外溢效应，或者说是在一些非大模型行业里面的一个渗透，这个时候其实就会有一个很有趣的市场现象，我们不妨拭目以待。

**欣然**：我在这一块现在没有什么能想象的东西，我觉得大模型很难预测它半年之后的样子，因为有可能三个月之后突然大家发现 ChatGPT 不本质，比如突然谁又发了一篇什么样的文章，所以我觉得，现阶段去预测这些东西是不太有效的，很有可能会发生说过了半年之后大家发现，其实我们需要的又不是这样的大模型。我觉得不太好说。

**Kiwi**：这确实是一个非常好的观点。我们也很期待接下来 GPT-4 是一个什么样的模型。那今天的讨论就到这里了。谢谢大家