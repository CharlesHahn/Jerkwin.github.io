---
 layout: post
 title: GROMACS教程：虚拟位点
 categories:
 - 科
 tags:
 - gmx
 math: true
---

* toc
{:toc}


<ul class="incremental">
<li>本教程由刘恒江翻译, 特此致谢.</li>
</ul>

<figure>
<img src="/GMX/GMXtut-7_CO2_CPK_2.png" alt="" />
</figure>

## 概述

<p>本教程将引导GROMACS新用户熟悉使用虚拟位点构建线性分子的过程. 教程假定用户已经熟悉GROAMCS的基本流程, 并熟悉拓扑文件的组织结构.</p>

<p>教程假定用户使用GROMACS 5.0或更高版本. 由于拓扑格式和一些术语的变化, 旧版本(4.5以前)的GROAMCS <strong>不适用于</strong> 本教程.</p>

## 第一步: 简介

<p>本教程假定读者熟悉虚拟位点的概念及相关知识, 并掌握了GROMACS手册4.7节和5.2.2节的所有内容. 若没有, 请先熟悉这些内容后再进行. 本教程提供了构建一个简单线性分子CO<sub>2</sub>的相关说明. 这是一个无法用常规方法构建的分子, 因为一些算法的原因, 具有180°键角的分子在模拟中是不稳定的.</p>

<p>总结一下虚拟位点的几个关键知识点:</p>

<ol class="incremental">
<li>虚拟位点没有质量</li>
<li>虚拟位点具有LJ和电荷相互作用</li>
<li>虚拟位点的位置由质心定义</li>
</ol>

<p>因此, 我们将从以下几点着手处理这个问题. CO<sub>2</sub>分子将由两个没有电荷与范德华参数的重原子重新构建, C原子和O原子将会被转化为虚拟位点, 其位置也由重原子重新构建.</p>

<p>以这种方式构建分子时, 需要考虑以下几点:</p>

<ol class="incremental">
<li>该分子必须具有与正常原子组成的分子相同的质量.</li>
<li>该分子必须具有与正常原子组成的分子相同的转动惯量.</li>
</ol>

<p>这些问题将在后一步建立拓扑时详细讨论.</p>

## 第二步: 构建拓扑

<p>与一般的GROMACS工作流程不同, 在这个教程中我们不需要使用<code>pdb2gmx</code>来获得拓扑文件. 我们将借助简单的文本编辑器完全手动地来构建拓扑文件. 本教程中我们将选用OPLS-AA力场, 你可以点击<a href="/GMX/GMXtut-7_topol.top">这里</a>下载拓扑文件. 在这一步中, 我们将研究拓扑文件的内容, 以解释构建方式背后的逻辑. 相关的单个CO<sub>2</sub>分子的坐标文件可以在<a href="/GMX/GMXtut-7_co2.pdb">这里</a>下载.</p>

<p>首先, 我们来考虑分子中的质量分布. 两个质心用于描述CO<sub>2</sub>分子的全部质量. 从拓扑文件中, 我们可以看出应如何计算分子总质量, 以使得它可以重新分配到两个质心.</p>

<pre><code>; Moment of inertia and total mass must be correct
; Mass is easy - virtual sites are 1/2 * mass(CO2)
;
; Total mass = (2 * 15.9994) + 12.011 = 44.0098 amu
;    each M particle has a mass of 22.0049 amu
</code></pre>

<p>转动惯量的计算更困难一点, 但仍可以从基本的方程进行计算. 包含三个原子的线性分子(本质上是一个三原子线性转子)的转动惯量为:</p>

<figure>
<img src="/GMX/GMXtut-7_triatomic_full.png" alt="" />
</figure>

<p><span class="math">\[I=2m_OL^2\]</span></p>

<p>而用于取代CO<sub>2</sub>的双原子分子的转动惯量为:</p>

<figure>
<img src="/GMX/GMXtut-7_diatomic_full.png" alt="" />
</figure>

<p><span class="math">\[I=\left({m_M^2 \over m_{Total} } \right) L^2\]</span></p>

<p>最容易计算的是CO<sub>2</sub>的转动惯量, 因为我们知道所有涉及到的物理量. 在OPLS-AA力场中氧原子的质量为15.9994 amu, C=O键长为0.125 nm. 这两个值分别对应于上面三原子线性转子转动惯量方程中的 <span class="math">\(M_O\)</span> 和 <span class="math">\(L\)</span>, 将其带入可求得转动惯量为0.49998125 amu nm<sup>2</sup>(忽略SI单位因为我们在后面将统一转换). 接下来我们就可以求解双原子分子转动惯量的方程来获得两个质心之间的距离. 前面已经求得每个质心的质量为22.0049 amu, 这样我们就能得到两质心之间的距离为0.213713 nm. 把这个值设为拓扑文件中的约束. 在这里我们使用约束而不是常规的简谐键是因为以下两点: (1) 这个距离应当保持不变; (2) 避免在力场文件中定义新的键合参数.</p>

<p>现在我们已经知道了两个质心之间的距离, 这个距离可以重现CO<sub>2</sub>分子的转动惯量, 我们接下来需要定义虚拟C位点和虚拟O位点的构建方式. 虚拟位点可以使用线性方式(使用的虚拟位点结构为类型2, 在<code>[ virtual_sites2 ]</code>指令中进行定义)进行构建, 采用两个质心之间总距离的分数形式. 在拓扑文件中, 需要给出相应于这个分数的a值, 它描述了虚拟位点到质心的距离与两质心之间距离的比值. 虚拟位点位于距第一个给定的参考原子(1-a)的地方.</p>

### 定义碳的位置

<p>我们从最简单的虚拟位点开始. 碳原子被置于分子的中心, 精确地处于两个质心之间的中点. 因此在拓扑文件中, 我们定义这个位点的a=0.5, 在拓扑文件中写为:</p>

<pre><code>2   4   5      1   0.5000      ; right in the middle
</code></pre>

### 定义氧的位置

<p>构建氧虚拟位点的位置更困难些. 这些虚拟位点不在两个质心之间, 而且它们超出了原子M1和M2之间的距离. 因此, 这里的a必须大于1, 只有这样虚拟位点才能位于M1-M2之外. 由于a值以相关质心间总长度的分数形式给出, 我们需要计算它的值, 拓扑文件中给出了简明的计算过程:</p>

<ol class="incremental">
<li>C=O键长为0.125 nm</li>
<li>M1-M2之间的约束长度为0.213173 nm, 其距离的一半为0.1065865 nm</li>
<li>因此, O虚拟位点超出M1-M2的距离为0.0184135 nm</li>
<li>这样, a值应为(0.213173+0.0184135)/0.213173=1.0851116</li>
</ol>

<p>在拓扑文件中, O虚拟位点的信息有两行, 相对于每个质心的M1(原子4)和M2(原子5):</p>

<pre><code>1   4   5      1   1.0851116   ; relative to mass center 4, extends beyond mass center 5
3   5   4      1   1.0851116   ; as in the case of site 1
</code></pre>

## 第三步: 简单模拟

<p>现在我们可以建立一个简单的体系进行模拟. 借助<code>insert-molecules</code>命令创建一个简单的CO<sub>2</sub>盒子:</p>

<p><code>gmx insert-molecules -f co2.pdb -ci co2.pdb -nmol 9 -o box.pdb</code></p>

<p>我们这样做是因为, 如果我们只模拟一个分子, 体系的自由度为零, 模拟将以0 K进行, 分子不会运动. 那样的话对展示我们创建的模型的可行性不是很有效. 当然也没有必要加入太多分子, 这只是一个简单的演示. 所以我们使用了一个很小的测试体系. 现在, 需要更新<code>topol.top</code>文件中的 <code>[ molecules ]</code>段以反映体系中共有10个CO<sub>2</sub>分子.</p>

<p>进行能量最小化, 使用的mdp文件可在<a href="/GMX/GMXtut-7_minim.mdp">此处</a>下载.</p>

<pre><code>gmx grompp -f minim.mdp -c box.pdb -p topol.top -o min.tpr
gmx mdrun -nt 1 -v -deffnm min
</code></pre>

<p>由于体系特别小(50个原子), 因此mdp文件尽量从简, 也没有使用更多的处理器.</p>

<p>现在对体系进行一个短时间的动力学模拟, 使用的mdp文件可在<a href="/GMX/GMXtut-7_md.mdp">此处</a>下载.</p>

<pre><code>gmx gormpp -f md.mdp -c min .gro -p topol.top -o md.tpr
gmx mdrun -nt 1 -v -deffnm md
</code></pre>

<p>模拟结果显示CO<sub>2</sub>分子漂浮在空间中, 这是预期结果但不是很重要. 真正重要的是我们要确认每一帧中分子的几何构型构建正确. 此外, 我们还可以借助<code>principal</code>命令来验证每个分子的转动惯量是否正确. 为了便于理解, 我们单独分析每个分子:</p>

<pre><code>gmx make_ndx -f min.gro
(at the prompt)
spliters 0
q

gmx principal -s md.tpr -f md.trr -n index.ndx
(choose any molecule for analysis when prompted)
</code></pre>

<p>输出文件<code>moi.dat</code>中包含了分子沿x, y, z轴三个方向的转动惯量. 可以看到, CO<sub>2</sub>沿x轴的转动惯量(沿C=O键方向)为零, 而沿y和z轴的转动惯量约为0.4999814, 这与建立拓扑文件时的计算值相当吻合. 因此, 我们的结论是这个CO<sub>2</sub>模型完全可以重现预期的转动惯量.</p>

## 总结

<p>现在你已经成功地使用虚拟位点为一个线性分子创建了拓扑文件, 并通过简单的模拟确认了转动惯量可以正确的重现. 这个教程可以作为创建其他线性分子或官能团拓扑文件的基础.</p>

<p>如果你对改进这个教程有些建议, 如果你发现了错误, 或者你觉得有些地方不够清楚, 请给我发邮件<code>jalemkul@vt.edu</code>, 不要客气. 请注意: 这不是邀请你因为GROMACS的问题而给我发邮件. 我并不是作为一个私人家教或个人客服在为自己打广告. 那是<a href="http://lists.gromacs.org/mailman/listinfo/gmx-users">GROMACS用户邮件列表</a>的事. 我可能会在那里帮助你, 但那只是作为对整个社区的服务, 而不只针对最终用户.</p>

<p>祝你模拟愉快!</p>
